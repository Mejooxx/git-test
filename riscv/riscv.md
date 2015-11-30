### 2.5 控制流指令
+ RV32I 提供两种控制流指令：无条件跳转指令和条件分支指令。RV32I 的控制流指令并没有结构上可见的延迟槽。

#### 无条件跳转
+ UJ 格式的 JAL 指令运用到 J 格式的立即数编码方式，先左移为 2 字节的倍数之后进行符号位扩展。符号位扩展之后的立即数加上 pc 值作为跳转目标地址，因此可以控制指令跳转到当前位置前后 1 MiB 的范围内。JAL 指令将（pc + 4）存放在目的寄存器 rd 中。标准的软件中系统调用约定用寄存器 x1 作为返回地址寄存器。
+ 当 JAL 的 rd = x0 的时候，即为简单的 jump 指令（汇编程序中的微指令 J）。

![jal_format][1]

+ 用 I 格式编码的间接跳转指令 JALR（jump and link register），由寄存器 rs1 中的操作数加上 12 比特位的 I 格式的有符号立即数，然后把最小有效位设置为 0 来产生分支目标地址。把 jump 指令的下一地址（pc + 4）写入寄存器 rd。在不需要保存这个地址的时候，我们可以把目的寄存器设置为 x0。

![jalr_format][2]

###### 在这一段里，我们看到源寄存器的基址和 12 比特位的偏移量直接相加，为什么不把偏移量左移 2 位来扩大跳转范围呢？为什么只把最低有效位置 0，riscv 的指令存储不应该是按字对齐的么？

------------------------------------------------------

###### 所有的无条件跳转指令用 pc 相对寻址来为位置无关指令提供支持，JALR 指令可以是使一个 2 条指令的指令序列实现在 32 比特位绝对地址空间范围内的任意位置的跳转。一个 LUI 指令可以先把目标地址的高 20 位放到 rs1 中，然后由 JALR 指令将它与低 20 位相加。同样的，AUIPC 后接 JALR 指令也可以实现 32 比特位 pc 相对寻址范围的任意地址跳转。

###### 需要注意的是,JALR 指令不同于条件分支指令，它不以 2 字节的倍数的形式来处理立即数。这样避免了在硬件上有更多的立即数格式，同时也重用了全局 load 指令相同的重定位格式。实际上，大多数的 JALR 指令在使用的时候，要么是一个 0 立即数，要么跟 LUI 指令或者 AUIPC 指令成对出现，这样，稍微地减少跳转范围也不会对它造成很大的影响。

###### JALR指令忽略了所计算出分支目标地址的最低比特位，这样既稍微地简化了硬件，又允许了函数指针的低比特位可以存放一些附加的信息。尽管这样在错误检测中会稍微有一些损失，而实际上跳到错误的指令地址会很快地引起异常。

###### 返回地址预测栈是高性能取址单元所共有的特征。我们知道 rd 和 rs1 可以用来引导取址预测逻辑的实现，它们表征 JALR 指令应该增加（rd = x1），取出（rd = x0,rs1 = x1），或者是不去接触返回地址栈。相同的，一个 JAL 指令的 rd = x1 时也可以增加一个返回地址到返回地址栈中。

###### 在 JALR 指令中，如果把基址 rs1 设置为 x0 的时候，它就可以实现一个地址空间在 ±2KiB 范围内的单个子程序调用。这样就可以用来实现调用一个小运行时间的库。

-----------------------------------------------

#### 条件分支

+ 所有的分支指令都用 SB 格式进行编码。12 位的 B 格式的立即数以 2 的整数倍形式编码成有符号偏移量，加上当前的 pc 值作为分支目标地址。条件分支的范围是 ±4 KiB。

![conditional_branches][3]

Branch instructions compare two registers. BEQ and BNE take the branch if registers rs1 and rs2
are equal or unequal respectively. BLT and BLTU take the branch if rs1 is less than rs2, using
signed and unsigned comparison respectively. BGE and BGEU take the branch if rs1 is greater
than or equal to rs2, using signed and unsigned comparison respectively. Note, BGT, BGTU,
BLE, and BLEU can be synthesized by reversing the operands to BLT, BLTU, BGE, and BGEU,
respectively.

+ 分支指令比较两个寄存器，BEQ 指令和 BNE 指令分别在寄存器 rs1 和 rs2 相当和不想等的时候进行分支，BLT 指令和 BLTU 指令分别对寄存器 rs1 和 rs2 进行有符号和无符号的比较，在 rs1 小于 rs2的时候进行分支。需要注意的是，BGT 指令，BGTU 指令，BLE 指令和 BLEU 指令分别可以通过颠倒 BLT指令，BLTU 指令，BGE 指令和 BGEU 指令的操作数来实现。

[1]: /riscv/image/jal_format.png
[2]: /riscv/image/jalr_format.png
[3]: /riscv/image/conditional_branch.png