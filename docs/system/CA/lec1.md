# Fundamentals of Computer Design

## Performance

影响性能的几个原因：

- Architecture
- Hardware implementation
- Compiler for Architecture
- Operating System

对于不同的需求，我们需要定义不同的衡量性能的标准

- 对于单一的用户而言，我们需要更快地反应时间 Response Time
    + Latency ： the time between the start and completion of an event
- 对于大量数据而言，我们需要更大的吞吐量 Throughput 
    + Total work per unit time

### Execution Time

我们可以使用执行时间来衡量性能：$ Performance = \frac{1}{Execution Time} $

- Elapsed time：总的消耗时间，包括各个方面
- CPU time: 在给定任务上消耗的时间
    + User Cpu Time : 用户使用的 Cpu 时间
    + System Cpu Time ： 系统消耗的 Cpu 时间

!!! quote
    The main goal of architecture improvement is to improve the performance of the system.

## Quantitative approaches

Measuring Data Size 衡量数据量大小

- bit: A sigle binary digit
- nibble: Four bits
- byte: Eight bits
- word: four bytes
- kibibyte: $2^{10}$ bytes

Formula for CPU performance

$$CPU Execution Time = CPU Clock Cycles \times Clock Peroid$$

$$CPU Execution Time = CPU Clock Cycles \times \frac{1}{Clock Rate}$$

$$CPU Time = InstructionCount \times CPI \times CLock Peroid$$

$$CPU Time =\frac {Instruction Count \times CPI }{Clock Rate}$$

Amdahl's Law

阿姆达尔定律，一个计算机科学界的经验法则。它代表了处理器并行运算之后效率提升的能力。

$$T_{improved} = \frac{T_{affected}}{improvement factor} + T_{unaffected}$$


## Instruction Set 

### Instruction Set Design Issues

- 操作数在何处保存？
    + registers,memory,stack,accumulator
- 需要多少的显式的操作数？ （ISA的分类）
    + 0,1,2,...
- 操作数的位置该如何确定？ （如何寻址）
    + regitster,immediate,indirect...
- 操作数的类型和大小？    （数据的表示）
    + byte,int,float,double,string,vector
- 支持哪些功能？          (指令的类型)
    + add,sub,mul,move,compare...

### ISA Classes
- Stacck Architecture
- Accumulator Architecture
- Gerneral Purpose Regiter Architecture
    + Register-Memorty Architecture (任何指令都可以访问内存)
    + Load-store Architecture (只有Load和Store可以访问)


