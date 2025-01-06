---
date:
  created: 2025-01-06
tags:
  - 6.004
authors: [Tenax]
description: >
  MIT 6.004 Spring 2019 L02: Introduction Notes
---

# MIT6.004 Note 2

_<!-- more -->_

![6.004-2-1](../assets/6.004/6.004-2-1.avif)

<u>为什么是 2^30?</u>

由于每个字（word）占 4 字节，而地址是按字节进行寻址的，若我们用 32 位的地址总数来计算内存的总字数，可以按如下方式进行：

$$
内存总字数=\frac{总字节数}{每个字的字节数}=\frac{2^{32}字节}{4字节/字}=2^{30}字
$$

寄存器文件（Register File）通常是按位（bit）寻址的，而内存是按字节（byte）寻址的

<u>Register File 中的 x0 hardwired 一般是干什么的，为什么只能 to 0，不能做任何操作？</u>

**`x0` 寄存器**是 **硬连接的**，其值永远为 **0**。它不能用于存储任何其他数据，因为硬件直接决定了它的值。

它的主要作用是提供一个常量零值，简化处理器设计，并且减少了指令集中的冗余代码。它通常用于操作中需要零值的情况，如清零寄存器、实现加法或减法的零操作等。

---

练习[L01.worksheet](../assets/sp20/L01.worksheet.pdf)

lw sw add addi 常用于初始化 li 常用于定量 lui bge blt

| 缩写 |              原文               |       中文翻译       |      备注      |
| :--: | :-----------------------------: | :------------------: | :------------: |
|  LW  |            Load Word            |        加载字        |                |
|  SW  |           Store Word            |        存储字        |                |
| ADD  |               Add               |         加法         |     地址加     |
| ADDI |          Add Immediate          |       加立即数       | 初始化值，值加 |
|  LI  |         Load Immediate          |      加载立即数      |      常量      |
| LUI  |      Load upper immediate       |    加载高位立即数    |                |
| BGE  | Branch If greater than or equal | 如果大于或等于则分支 |                |
| BLT  |       Branch if less than       |    如果小于则分支    |                |

<u>_将以下表达式编译成 RISCV 汇编。假设 a 保存在地址 0x1000，b 保存在地址 0x1004，c 保存在地址 0x1008。_</u>

<u>a = b + 3c;</u>(首先的想法是系数乘，但好像没有系数用来乘法的指令...，第二次的想法是开辟一个新内存放 3c，但好像离题了...)

```
lw x1, 0x1000
lw x2, 0x1004
lw x3, 0x1008
add x2, x2 x3
add x3, x3 x3
add x1, x2 x3
sw x1, 0x1000
```

1. **li rd, imm**：将立即数`imm`加载到寄存器`rd`。这通常被转换为`lui`和`addi`指令的组合。

---

if (a > b) c = 17;❌(不知道=是用 li...)

```
     lw x1, 0x1000
     lw x2, 0x1004
     lw x3, 0x1008
     blt x1, x2 else
     sw 17, 0x1008
else:
```

更正

```
     lw x1, 0x1000     # x1 = a
     lw x2, 0x1004     # x2 = b
     lw x3, 0x1008     # x3 = c
     blt x1, x2, else     # 如果 a < b 跳转到 end，不做任何操作
     bne x1, x2, end     # 如果 a == b 跳转到 end，不做任何操作
     li x3 17     # 如果 a > b，给 c 赋值 17
     sw x3, 0x1008     # 将 17 存储到 c 的地址
else:
end:
```

---

sum = 0;
for (i = 0; i < 10; i = i+1) sum += i; ❌(不记得用 loop，还有 li 是要用值而不是寄存器）

```
for (i = 0; i < 10; i = i+1) sum += i;

    li sum, x0
    li i, x0
    bge i, 10, end
    add i, i, 1
    add sum, sum, i
end:
```

更正(假设 sum,i 已经 sw 到寄存器中)

```
     mv sum, x0            # sum = 0, 初始化 sum 为 0
     mv i, x0             # i = 0, 初始化 i 为 0
loop:
     bge i, 10, end         # 如果 i >= 10，跳转到 end 结束循环
     addi i, i, 1            # i = i + 1，i 自增 1
     add sum, sum, i        # sum += i，将 i 加到 sum
     bne i, 10, loop        # 当 i != 10 时跳转
end:
```

---

假设 a 存储在 0x1100 地址，b 存储在 0x1200 地址，c 存储在 0x2000 地址，编译下面的表达式。假设 a、b 和 c 是数组，其元素存储在连续的内存位置。

for (i = 0; i < 10; i = i+1) c[i] = a[i] + b[i]; ❌(理解还是不够深)

```
     lw a, 0x1100
     lw b, 0x1200
     lw c, 0x2000
     mv i, x0
loop:
     bge i, 10, end         # 如果 i >= 10，跳转到 end 结束循环
     addi i, i, 1            # i = i + 1，i 自增 1
     add c, a, b
     addi c, c, 4
     addi a, a, 4
     addi b, b, 4
     j     loop          # 跳回到 loop 继续执行
 end:


```

答案

```
寄存器：
    x1：a[0] 的地址
    x2： c[0] 的地址
    x3: i - 索引
    x4： 4i--由于字的长度，我们将 i 乘以 4 以获得正确的偏移量（RISC-V 存储器按字节索引，每个字长 4 个字节）
    x5：a[i] 的地址
    x6：c[i] 的地址
    x7：1) a[i] 的值，2) a[i] + b[i] 的值
    x8： b[i] 的值
    x9: 10 - FOR 循环的条件（i < 10）

该循环的实现方法与问题 1-3 中的相同。我们必须首先获取索引 i 的地址，对于 a[i]，地址为 0x1100 + 4i；对于 b[i]，地址为 0x1200 + 4i；对于 c[i]，地址为 0x2000

    li x1, 0x1100 // x1 = address of a[0] (lui x1, 1; addi x1, x1, 0x100)
    li x2, 0x2000 // x2 = address of c[0] (lui x2, 2)
    li x3, 0 // x3 = 0 (i) (addi x3, x0, 0)
    li x9, 10
loop:
    sll x4, x3, 2 // x4 = 4 * i
    add x5, x1, x4 // x5 = address of a[i]
    add x6, x2, x4 // x6 = address of c[i]
    lw x7, 0(x5) // x7 = a[i]
    lw x8, 0x100(x5) // x8 = b[i]; b is offset from a by 0x100
    add x7, x7, x8 // x7 = a[i] + b[i]
    sw x7, 0(x6) // c[i] = a[i] + b[i]
    addi x3, x3, 1 // i = i + 1
    blt x3, x9, loop // branch back to loop if i < 10


```

---

手动将下列指令序列组装成等效的二进制编码。❌（因为看的是 19sp 的视频，但用的是 20sp 的练习题，19sp 视频上没有这个知识点...）

loop:
addi x1, x1, -1
bnez x1, loop

答案

```
addi x1, x1, -1
-1 encoded as 12 bits is 0xfff
x1 in 5 bits is 0b00001
func3 for addi = 000
op = 0010011 (since addi is a register-immediate instruction)

addi: imm[11:0],rs1,func3,rd,op = 0xfff08093 =
0b111111111111_00001_000_00001_0010011

bnez x1, loop = bne x1, x0, loop
x1 in 5 bits 0b00001 = rs1
x0 in 5 bits is 0b00000 = rs2
func3 for bne = 001
op = 1100011

我们将标签的偏移量（即 -4 (0b100)）存储到立即值中。由于偏移量的最小有效位（底位）总是 0，我们可以将立即值的第 12:1 位存储到指令中。使用第 12:1 位可以将分支的最大偏移量增加一倍，而使用第 12:1 位则不能将分支的最大偏移量增加一倍。

11:1.
imm[12:1] = distance to label in bytes / 2 = -2 = 0xffe
imm[12] = 1
imm[11] = 1
imm[10:5] = 0b111111
imm[4:1] = 0b1110

bnez: imm[12],imm[10:5],rs2,rs1,func3,imm[4:1],imm[11],op = 0xfe009ee3 =
0b1_111111_00000_00001_001_1110_1_1100011

```

**addi x1, x1, -1**

addi 属于 I-tpye

| 12 bits   | 5 bits | 3 bits | 5 bits | 7 bits |
| :-------- | ------ | ------ | ------ | ------ |
| imm[11:0] | rs1    | funct3 | rd     | opcode |

imm[11:0] 立即数为 `-1`，用 12 位补码表示，即 `111111111111`= 12 bits is 0xfff

源寄存器`rs1`是 `x1`，即 `00001`。 = 0b00001

addi`的 funct3 是`000

目标寄存器`rd` 是 `x1`，即 `00001`。= 0b00001

I 型： 立即寄存器指令：操作码 = OP-IMM = 0010011

imm[11:0],rs1,func3,rd,op = 0b111111111111 + 00001 + 000 + 00001 + 0010011 = 0b111111111111_00001_000_00001_0010011

**bnez x1,loop **

bnez x1,loop = bne x1, x0, loop

bne 属于 SB-type

| 12 bits         | 5 bits | 5 bits | 3 bits | 5 bits         | 7bits   |
| :-------------- | ------ | ------ | ------ | -------------- | ------- |
| imm[12 \| 10:5] | rs2    | rs1    | funct  | imm[4:1 \| 11] | 1100011 |

**偏移量的存储方式**

在分支指令中，**偏移量**用于计算跳转目标的地址。因为 RISC-V 是 **按字节对齐**（4 字节）来存储指令的，因此偏移量是按字节来表示的。

- 在 `B` 类型指令中，**偏移量是按 2 字节对齐的**。这意味着，偏移量的实际值（单位是字节）需要除以 2，才能填充到指令中的 `imm` 字段。
- **最小有效位**（即偏移量的最低位）总是 **0**，因为分支跳转总是以 2 字节为单位对齐。

**偏移量的表示**

pc <= (reg[rs1] != reg[rs2]) ? pc + immB : pc + 4
比较寄存器 rs1 和 rs2。如果比较结果为真，则 pc 更新为 pc + imm，否则 pc 变为 pc + 4。比较类型由操作定义。

分支指令的目标地址计算是基于当前指令的地址加上偏移量。例如，假设当前指令的地址是 `pc`，分支指令会跳转到 `pc + offset`。对于 RISC-V 中的 `B` 类型指令，偏移量是一个 **有符号的 13 位数值**，并且它是 **按 2 字节对齐的**。所以，当我们需要存储这个偏移量时：

- 偏移量的大小为 13 位，表示范围从 `-4K` 到 `4K` 字节（即 `-2048` 到 `2047` 字节）。
- **最小有效位**总是为 0，因为分支指令必须按 2 字节对齐。

**为什么将标签的偏移量存储在指令的 `imm[12:1]` 字段中？**

由于 `imm[0]`（偏移量的最低位）总是 0，分支指令通过将偏移量存储在指令中的 **`imm[12:1]` 字段**来节省存储空间。这样，可以将 `imm[12:1]` 用于表示最大范围的偏移量，减少了需要存储的字段数。

具体来说，分支指令的 `imm` 字段分布如下：

bnez: imm[12],imm[10:5],rs2,rs1,func3,imm[4:1],imm[11]

- `imm[12]`：表示偏移量的符号位（用于表示负数）。
- `imm[10:5]`：用于存储偏移量的中间 6 位。
- `rs2`：源寄存器 `x0`（`bnez` 比较的是 `x1` 是否等于零）。
- `rs1`：源寄存器 `x1`。
- `funct3`：表示比较操作符（对于 `bne` 来说是 `001`）。
- `imm[4:1]`：用于存储偏移量的低 4 位。
- `imm[11]`：表示偏移量的倒数第二位。

x1 5 位为 0b00001 = rs1  
5 位中的 x0 为 0b00000 = rs2  
bne = 001 时的 func3  
op = 1100011

我们将标签的偏移量（即 -4 (0b100)）存储到立即值中。由于偏移量的最小有效位（底位）总是 0，我们可以将立即值的第 12:1 位存储到指令中。与 11:1 相比，使用 12:1 位可将分支的最大偏移量增加一倍。

imm[12:1] = distance to label in bytes / 2 = -2 = 0xffe  
imm[10:5] = 0b111111  
rs1 = x1 = 0b00001  
rs2 = x0 = 0b00000  
imm[12] = 1  
imm[4:1] = 0b1110  
imm[11] = 1

bnez: imm[12],imm[10:5],rs2,rs1,func3,imm[4:1],imm[11],op = 0xfe009ee3 = 0b1_111111_00000_00001_001_1110_1_1100011

---

A) 假设在执行下列每条汇编指令之前，寄存器的初始化值为：x1=8，x2=10，x3=12，x4=0x1234，x5=24。请为每条指令提供指定寄存器或内存位置的值。如果您的答案是十六进制，请确保在答案前加上前缀 0x。

**SLL x6, x4, x5**

Value of x6:

**ADD x7, x3, x2**

Value of x7:

**ADDI x8, x1, 2**

Value of x8:

**SW x2, 4(x4)**

Value stored: **_ at address: _**

---

B) Assume X is at address 0x1CE8

```
    li x1, 0x1CE8
    lw x4, 0(x1)
    blt x4, x0, L1
    addi x2, x0, 17
    beq x0, x0, L2
L1: srai x2, x4, 4
L2:
X:  .word 0x87654321
```

Value left in x4?
Value left in x2?

---

将以下 Fibonacci 实现编译为 RISCV 汇编。

参考 Python 中的斐波纳契实现

```python
def fibonacci_iterative(n):
  if n == 0:
   return 0
 n -= 1
 x, y = 0, 1
 while n > 0:
 # Parallel assignment of x and y
 # The new values for x and y are computed at the same time, and then
 # the values of x and y are updated afterwards
   x, y = y, x + y
   n -= 1
 return y
```

答案[L02.ws_solutions](../assets/sp20/L02.ws_solutions.pdf)
