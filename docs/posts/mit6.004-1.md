---
date:
  created: 2024-12-27
tags:
  - 6.004
authors: [Tenax]
description: >
  MIT 6.004 Spring 2019 L01: Introduction Notes
---

# MIT6.004 Note 1

_<!-- more -->_

在网上找了很久 MIT 6.004 Spring 2019 的课件，貌似只有这家，要付费。[https://www.coursehero.com/file/41528449/L01pdf/](https://www.coursehero.com/file/41528449/L01pdf/)。所以我用的是 MIT 6.004 Spring 2020 的课件，还有练习

![6.004-1-1](../assets/6.004/6.004-1-1.avif)

![image-20241224153119520](../assets/6.004/6.004-1-2.avif)

问题 1：一直没注意到 Register File 中的引用，初次理解以为 x1 ←load(Mem[x10]) 中的 x10 是 100 可能是某种进制后的变体，，x4 ← load(Mem[x1])中的 x1 我以为是是 Main Memory 中的位置 4，某种进制后变换为 x1。

问题 2：其中 x1 作为一个指针是永远都指向数组中的第一个元素吗？

答：通常我们会认为在每次读取数组元素后，`x1` 会 **递增**，指向下一个元素的地址。代码中没有显式地看到 `x1` 自增。由于数组是连续存储的，并且假设数组的每个元素大小为 4 字节（假设是 `int` 类型），`x1` 在循环时通常会增加 4（即 `x1 = x1 + 4`），这样它就会指向下一个元素的地址。（是的，在视频后面有个女孩子说出了这个问题...老师没把代码写出来）

`x1 ← load(Mem[x10])`

- 这里的 `Mem[x10]` 存储的是数组的 **起始地址**（即 `a[0]` 的地址）。
- `x1` 存储了这个地址，所以可以理解为：
  - `x1` -> `a[0]` （`x1` 存储数组第一个元素的内存地址）

`x4 ← load(Mem[x1])`

- 现在，`x1` 作为一个 **指针**（内存地址），指向数组的第一个元素 `a[0]`。
- `load(Mem[x1])` 就是通过 `x1` 所指向的地址，从内存中 **读取** 数组第一个元素的值。
- 所以，`x4` 存储的是 `a[0]` 的值。可以理解为：
  - `x1 -> a[0]` （`x1` 现在指向 `a[0]`）
  - `x4 = a[0]` （将 `a[0]` 的值加载到 `x4`）

`add x3, x3, x4`

- x3 = x3 + x4，从地址 `x1` 取出一个元素 `x4`，累加到 `x3`。

`addi x2, x2, -1`

- x2 = x2 - 1，数组剩余长度 -1。

`bnez x2, loop`

- 若 `x2 != 0`，继续。

**[Crash Course Computer Science](https://www.bilibili.com/video/BV1EW411u7th)1~ 6 集加深理解**

练习[L01.worksheet](../assets/sp20/L01.worksheet.pdf)

<u>十进制数 21 的 5 位二进制表示是多少？(十进制转换为二进制)</u>

1 2 4 8 16 32  
1 0 1 0 1

---

<u>以 8 位二进制数编码的十进制 219 的十六进制表示是什么？</u>

1 2 4 8 16 32 64 128  
1 1 0 1 1 0 1 1

0000 = 0; 0001 = 1; 0010 = 2; 0011 = 3; 0100 = 4; 0101 = 5; 0110 = 6; 0111 = 7; 1000 = 8; 1001 = 9; 1010 = A; 1011 = B; 1100 = C; 1101 = D; 1110 = E; 1111 = F

`1101` `1011` = D B

---

<u>十进制 51 以 6 位二进制数编码的十六进制表示法是什么？</u>

1 2 4 8 16 32  
1 1 0 0 1 1

由于右边的组只有 2 位，为了使其成为 4 位，需要在左边补 0  
`0011` `0011` = 33

---

<u>一个 8 位无符号二进制数的十六进制表示法是 0x9E。它的十进制表示法是什么？</u>

9 E  
9 14

$$
9E=9*16^1+E*16^0=144+14*16^0=144+14=158\\十六进制是一种 基数为16 的进制表示系统。在这种进制中，数字的位权从右到左
$$

可转换二进制`1001` `1110`  
1 2 4 8 16 32 64 128  
0 1 1 1 1 0 0 1  
128+16+8+4+2=158

---

<u>一个 8 位无符号量可表示的整数范围是多少？</u>

**最小值**：由于是无符号数，最小值是 **0**。

**最大值**：8 位二进制数可以表示 2^8=256 个不同的数，而这些数的范围是从 0 开始，到 2^8−1=255。

---

<u>自 1988 年开始官方投球统计以来，单场比赛的最高投球数为 172 球。假设这仍然是投球数的上限，我们需要多少比特才能将每场比赛的投球数记录为无符号二进制数？</u>

1 2 4 8 16 32 64 128  
0 0 1 1 0 1 0 1  
172 的二进制表示需要 8 位（`10101100`）

---

<u>计算这两个 4 位无符号二进制数的和： 0b1101 + 0b0110。用十六进制表示结果</u>

1*2^0 + 4* 8 + 2 4 = 19

$$
11001\\
0b1101=1*2^0 + 1*2^2 + 1*2^3 = 13\\
0b0110=1*2^1 + 1*2^2 = 6\\
0x13
$$

---

<u>十进制数 -21 的 6 位二进制补码表示是多少？</u>

**对于正数，补码与原码相同。对于负数，补码是将反码加 1**

**对于正数，反码与原码相同。对于负数，反码是将所有位取反。**

**原码表示法在数值前面增加了一位符号位（即最高位为符号位）：正数该位为 0，负数该位为 1（0 有两种表示：+0 和 -0）**

1 2 4 8 16 32 64 128  
1 0 1 0 1 0 0 0  
1 10101  
1 01010  
1 01011

---

<u>以 8 位二进制数编码的十进制 -51 的十六进制表示是什么？</u>

1 2 4 8 16 32 64 128  
1 1 0 0 1 1 0 0

$$
1 1 0 0 1   100\\
10110011\\
1011~0011=  B 3 (原码表示法)\\
\\
11001100(将每一位取反，得到反码)\\
11001101(将反码加1，得到补码)\\
1100~1101=CD(补码表示法)
$$

---

<u>一个 8 位二进制数的十六进制表示法是 0xD6。它的十进制表示法是什么？</u>❌

D 6  
13 6 = 1101 0110  
1 2 4 8 16 32 64 128  
0 1 1 0 1 0 1 1

$$
86 + 128 = 214 ~OR~ 13*16^1 + 6*16^0=214
$$

---

<u>使用 5 位二进制表示法，可以用一个 5 位量表示的整数范围是多少？</u>

$$
无符号二进制数（Unsigned Binary）\\
最大值：11111（二进制），对应十进制的 31。\\
最小值：00000（二进制），对应十进制的 0。\\
\\
有符号二进制数（Signed Binary）\\
最大正数：当符号位为 0 且其他位为 1，即 01111，对应十进制的 15\\
最小负数：当符号位为 1 且其他位为 0，即 10000，对应十进制的 -16。\\
$$

---

<u>两个 二进制补码之和 0xB3 + 0x47 的值能否用 8 位 二进制补码表示？如果可以，和的十六进制数是多少？如果不能，请写 “否”。</u>

B3  
11 3  
1011 0011  
0100 1100  
0100 1101 = 77  
-77

47  
0100 0111 = 71  
71

-77+71=-6  
0000 0110  
1111 1001  
1111 1010 = F A = 0xFA

---

<u>两个二进制的补码之和 0xB3 + 0xB1 的值能否用 8 位二进制的补码表示？如果可以，以十六进制表示的和是多少？如果不能，请写 “否”。</u>

B3  
11 3  
1011 0011  
0100 1100  
0100 1101 = 77  
-77

B1  
11 1  
1011 0001  
0100 1110  
0100 1111 = 64+15 =79  
-79

-77+-79=-156  
1001 1100  
0110 0011  
0110 0100 = 6 4 = 0x64❌

**负数**的范围是从 `10000000` 到 `11111111`，这些二进制数对应的十进制数是从 **-128** 到 **-1**。

**正数**的范围是从 `00000001` 到 `01111111`，这些二进制数对应的十进制数是从 **1** 到 **127**。

所以是 NO

---

<u>请使用 8 位二进制补码计算表达式 0xBB - 8 的值，并以十进制（基数 10）给出结果。</u>

BB  
1011 1011  
0100 0100  
0100 0101 = 69  
-69

8  
0000 1000

-69 - 8 = -77  
0100 1101  
1100 1101  
1011 0010 负数的反码就是在原码的基础上符号位保持不变，其他位取反。  
1011 0011

---

<u>下面是一道操作数为 5 位二进制数补码的减法题。计算结果并给出十进制数（基数 10）。</u>

$$
~~~10101\\-
\frac{00011}{}
$$

10101
01010
01011 = 11
-11

00011 = 3

-11 - 3 = -14

---

<u>给定一个无符号 n 位二进制整数 𝒗 = 𝒃𝒏-𝟏 ... 𝒃𝟏𝒃𝟎 ，证明当且仅当 𝒃𝟎 = 𝟎 且 𝒃𝟏 = 𝟎 时，𝒗 是 4 的倍数。</u>

<u>二进制编码是否也有同样的关系？</u>

<u>解码以下空尾 ASCII 字符串：</u>

```js
//0x 52 49 53 43 2D 56 20 69 73 20 63 6F 6D 69 6E 67 21 00
const str = "0x 52 49 53 43 2D 56 20 69 73 20 63 6F 6D 69 6E 67 21 00";

const Ascll = (str) => {
  let str1 = str.split(" ");
  str1.shift();
  str1.pop();
  return String.fromCharCode(...str1.map((code) => parseInt(code, 16)));
};

console.log(Ascll(str));
```

答案[L01.ws_solutions](../assets/sp20/L01.ws_solutions.pdf)
