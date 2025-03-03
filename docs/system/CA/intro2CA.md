# Introduction to Computer Architecture

计算机体系结构注重的是硬件与软件的交界面：

!!! Quote "What are covered in this class?"
    - How the hardware influence the software or program?
    - What is the working and design principle of computer?
    - The tradeoffs of different designs and ideas.

??? note "分数组成"
    - 期末考试 40%
    - 作业     6%
    - 展示     6%
    - 项目    48%


!!! info "**Contents**"

    1. Fundamentals of computer design
    2. Pipelining
    3. Memory Hierachy
    4. ILP
    5. DLP and TLP

我们在实践中往往遵循这样的规律：从上至下的我们可以划分为不同层级的问题，其中 Runtime System(OS,VM,etc.)、ISA、Micro Architecture、Logic 四个部分是最为挑战性的,也是我们体系结构期望去探究的。

<center>

<img src="../assets/CAlec101.png" alt="图片描述" width=50% height=50%>

</center>

What is computer architecture? 更具体的而言，我们探究

- Difference with *Instruction Set* 指令集
- Difference with *Computer Organization* 计算机的组成
- Difference with *Computer Implementation* 计算机的实现