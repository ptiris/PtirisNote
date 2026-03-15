# FlexGen

## Codebase Map

```
.
├── apps                        
│   ├── completion.py               -- 可直接运行的 demo
│   ├── data_wrangle                -- 和对外外部结构化任务的接口
│   ├── helm_fast_test.py           -- 对接外部 benchmark
│   ├── helm_passed_30b.sh
│   ├── helm_run.py
│   ├── __init__.py
│   └── README.md
├── compression.py                  -- 权重/Cache 压缩相关
├── dist_flex_opt.py                -- 分布式版本的 flex_opt.py
├── dist_utils.py                   -- 分布式训练相关工具
├── flex_opt.py                     -- FlexGen 的入口
├── opt_config.py                   -- 模型配置相关
├── profile_bandwidth.py            -- 带宽测试工具 
├── profile_matmul.py               -- 矩阵乘法性能测试工具
├── pytorch_backend.py              -- PyTorch 设备抽象和算子实现
├── timer.py
└── utils.py
```

---

## Key Modules

### Flex Opt

在 `flex_opt.py` 中，首先定义了`Policy`类，从命令行参数中解析一些基础参数：  

- 各个Tensor 在三层存储的放置比例 (`w_gpu_percentage` ... )
- sep_layer：transformer block 是拆成 Attention+MLP 两层，还是合并一层对象
- cpu_cache_compute：attention 的 cache 相关计算是否放 CPU
- compress_weight / compress_cache：权重/KV cache 是否做分组量化压缩
- attn_sparsity：attention 稀疏控制

然后 `OPT` 类根据配置构建模型(由 embedding、N 层 transformer、output head 组成)。最关键在于 `OptLM.generate`。

1. 构建 Task （维护 prompt 的长度，采样温度和 stop_token等
2. 初始化输出数组 output_ids 与停止标记 stopped。
3. 清空/重建各类缓冲区：
    + cache_home/cache_read_buf/cache_write_buf
    + weight_read_buf
    + attention_mask
    + hidden（三维：token-step × layer × gpu-batch）

4. 为每层每个 gpu-batch 初始化 KV cache。
5. 根据 debug_mode 和 overlap 选择不同 generation loop：
    + generation_loop_normal：无重叠，逻辑直观
    + generation_loop_overlap_single_batch
    + generation_loop_overlap_multi_batch

!!! note 
    在每一个 Loop 中正对应论文中伪代码的流程：

    - load_weight
    - load_cache
    - load_hidden
    - compute_layer
    - store_hidden
    - store_cache

    其中 overlap 是通过 torch.cuda.Stream 实现的，OPTLM 维护`load_weight_stream`，`load_cache_stream`，`store_cache_stream` 三个 Stream 来实现重叠。

!!! warning

    需要额外注意的是 `Attention` 中关于 `load_cache`的设计，这里有三条路径：

    - path 0：直接拷贝到目标计算设备（GPU 或 CPU）
    - path 1：拷到 CPU 临时 workspace 再算
    - path 2：cache 同时分布在 GPU+其他设备，GPU 和 CPU 各算一部分(在 pytorch_backend.TorchDevice._mixed_device_attention中实现)
 

### Pytorch Backend

`pytorch_backend.py` 主要实现了在 PyTorch 上的封装：设备类型抽象：DeviceType；张量统一抽象：TorchTensor

对于`tonsor`，字段还是 shape/dtype/device，但 data 会因设备类型变化：

- 普通设备（CPU/GPU）：data 是 torch.Tensor。
- 磁盘设备：data 是 npy file 在打开时通过 memmap 读入。
- Mixed 设备：data 是 (tensors, segment_points)，即多个分段 tensor + 切分点。
- 压缩设备：data 是压缩结构（三元组）。

在 `device` 中，关于 allocate 部分：

- 对于 CPU ，默认考虑 pin_memory ，GPU 则不考虑。然后 numpy dtype 映射成 torch dtype，调用 torch.empty(..., device=self.dev) 分配，再封装成 TorchTensor 返回。

- 在某些路径下要把分散 cache 拷到 CPU 连续空间再算 attention 的情况下，通过 `iinit_attention_compute_workspace` 分配一个 CPU 连续空间并拷贝，避免避免每个 token 反复申请释放大块内存。

然后 `opt_output_embed` 和 `opt_input_embed` 则是实际上的embedding 层的实现。

关于 attention 的 prefilling 是由 mha 实现的，而 decode 是由 mha_gen 实现的。在这一部分根据 TensorDevice 的类型进行算子的选择。

较为关键的是 `copy` 的部分。逻辑层的李TorchTensor.copy 负责“我要复制一个张量到哪个设备、目标 shape 是什么;`general_copy`负责“根据源/目标设备组合，走哪条数据搬运路径”

- dst 是 MIXED：递归把源切片后复制到 mixed 的各段子 tensor。
- src 是 MIXED：递归从每个段复制到目标 tensor 的对应区间。
- 任一端是 COMPRESSED：转给 general_copy_compressed(...)。
- src 或 dst 是 DISK：不立刻拷贝，投递到磁盘 copy 线程队列（异步）。
- CUDA -> CPU 且目标 CPU tensor 非 pinned：也丢给后台线程，利用 pinned relay。
- CPU -> CUDA 且源非 pinned：先 pin_memory() 再 dst.copy_(..., non_blocking=True)。
- 其他普通情况：直接 dst.copy_(src, non_blocking=True)。

### Cost Model

关于论文中提及的所有的策略的搜索，则是在 `expieriments/cost_model.py` 中实现的。这里通过 pulp 进行线性规划求解。输入大体如下：

- 模型结构：层数 l、隐藏维 h1、FFN 维 h2、头数 nh（由 get_opt_config 注入）
- 任务规模：prompt_len=s、gen_len=n
- 设备资源：gpu_mem、cpu_mem、nvme_mem
- 硬件常数：带宽、FLOPS（CostModelConfig 里）
- 可选固定条件：gbs/num-gb/percent 等

输入一个较佳的策略和预估的性能指标（latency、throughput）。

除了 pulp 完成的内部搜索，还有几个关键变量在外部控制：

- 对固定 gbs，二分找可行 num_gpu_batches 上界。

