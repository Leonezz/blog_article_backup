---
title: 'CS:APP Lab1: DataLab'
date: 2021-07-28 21:06:15
update: 2021-07-28 21:06:15
tags:
    - CS:APP
    - 位运算
categories: 题解类
---

# 0x00 简介

一直在学习CS:APP这本书，本以为大名鼎鼎的Lab会出现在书本上，没想到一直看到第三章也没看见。在网上查了以下才知道是在线的。想想也十分合理，毕竟可能会经常有更新。这里就记录完成第一个Lab——DataLab的过程。

DataLab是使用受限的C语言运算子集来实现逻辑运算/补码运算/浮点运算的函数，例如，可能会要求仅使用位级运算和线性的过程来实现求绝对值的函数。这个Lab可以帮助理解C数据类型的位级表示和数据操作的位级行为。

<!--more-->

## 要求

这个Lab主要是在一个C语言文件中实现函数，在实现时对于使用的运算种类和数量都有严格的要求。同时Lab提供了一组工具用于检查代码中使用的运算是否符合要求以及用于测试代码正确与否。还提供了一个小工具可以打印整形数据和浮点数据的16进制或10禁止表示。

具体的编码要求附在文末，同时每道题可能还有自己的更加严格的要求。

## 概览

总共有不同分值的整数和浮点数函数共13个：

| 函数             | 描述                     | 分值 |
| ---------------- | ------------------------ | ---- |
| `bitXor`         | 计算异或                 | 1    |
| `tmin`           | 返回最小补码值           | 1    |
| `isTmax`         | 判断是否最大补码值       | 2    |
| `allOddBits`     | 判断是否奇数位置都为1    | 2    |
| `negate`         | 求相反数                 | 2    |
| `isAsciiDigit`   | 判断是否Ascii数字        | 3    |
| `conditional`    | 实现条件分支             | 3    |
| `isLessOrEqual`  | 实现小于等于             | 3    |
| `logicalNeg`     | 实现逻辑非(!)            | 4    |
| `howManyBits`    | 计算能表达数据的最小长度 | 4    |
| `floatScale2`    | 实现位级浮点数*2         | 4    |
| `floatFloat2Int` | 实现位级float转int       | 4    |
| `floatPower2`    | 实现位级2.0^x            | 4    |

# 0x01 bitXor

第一道题目：
```C
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
  return 2;
}
```

只使用 `~` 和 `&` 运算符实现 `^` 运算。最多可以使用14个运算符。

考虑位级异或运算的真值表：

| `x` | `y` | `x ^ y` |
| --- | --- | ------- |
| `0` | `0` | `0`     |
| `0` | `1` | `1`     |
| `1` | `0` | `1`     |
| `1` | `1` | `0`     |

可以看到令 `x ^ y = 1` 的情况分别是 `x = 0, y = 1` 和 `x = 1, y = 0`，根据卡诺图化简的方法和逻辑代数的表达方式，这个真值表可以化简为：`X'Y + XY'`，即：`x ^ y = (x & ~y) | (~x & y)`。

由于题目要求只能使用 `&` 和 `~`，消去上式中的 `|`：

```
(x & ~y) | (~x & y)
= ~~((x & ~y) | (~x & y))
= ~((~x | y) & (x | ~y))
= ~(~~(~x | y) & ~~(x | ~y))
= ~(~(x & ~y) & ~(~x & y))
```

因此直接将上面的 `return 2` 改成 `return ~(~(x & ~y) & ~(~x & y))` 即可。

这道题目还可以在卡诺图化简时选择反函数，即化简出 `~(x ^ y)` 的表达式。具体来说，是选择真值表中结果为`0`的项进行化简：`~(x ^ y) = (x & y) | (~x & ~y)`。这样的情况下，有：`(x ^ y) = ~((x & y) | (~x & ~y)) = (~x | ~y) & (x | y) = ~(x & y) & ~(~x & ~y)`。这个结果比上一个少用了1个运算符。

# 0x02 tmin

```C
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 2;
}
```

这道非常简单，返回最小的补码值，即 `0x80000000`。由于不能使用除 `0x00~0xff` 之外的常数，直接返回 `0x01 << 31` 即可。

# 0x03 isTmax

```C
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
  return 2;
}
```

这道是判断参数是否是最大补码值，即 `0x7fffffff`。不难知道对于 `0x7fffffff`，其处在溢出的边缘，再加 `1` 就会得到最小的补码值 `0x80000000`。观察最小补码值和最大补码值的位模式，得知最小补码值只有最高位为 `1`，其余位为 `0`；而最大补码值相反——只有最高位为 `0`，其余位为 `1`——因此最小补码值取反应该为最大补码值。

基于以上原理，当 `x = 0x7fffffff` 时，`!(x ^ ~(x + 1)) = 1`。这就实现了最大补码值时返回 `1`。

实际测试时发现，当 `x = 0xffffffff` 时，由于也处于溢出边缘，也具有上述的性质，对结果造成误判。因此需要过滤掉该种情况。

当 `x = 0xffffffff` 时，其位模式是所有位都为 `1`。因此对其取反可以得到特殊值 `0`。可以使用表达式 `~x` 进行过滤，这样的话总的结果就是 `!(x ^ ~(x + 1)) & ~x`。对这个结果使用 `x = 0x7fffffff` 进行验证，发现 `~x = 0x80000000`，这样的话最终结果就是 `0x80000000 & 0x1 = 0`，因此还需要对 `~x` 的值进行二值化，让其取值只落在 `0` 和 `1` 的范围内。即 `!!~x`。

最终结果为 `!(x ^ ~(x + 1)) & (!!~x)`。

# 0x04 allOddBits

```C
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
  return 2;
}
```

这道题要求判断给定的参数其位模式的奇数位是否都为 `1`。例如，数据 `0xFFFFFFFD` 的最低四位是 `1101`，第1位为 `0`，所以返回 `0`。数据 `0xAAAAAAAA` 则是所有奇数位都为 `1`，所有偶数位都为 `0`，所以返回 `1`。

因为只需要判断奇数位上的情况，偶数位则可以忽略。因此可以使用数据 `0xAAAAAAAA` 作为 mask 来对参数进行校验。如果参数 `x` 的奇数位都为 `1` 的话，表达式 `0xAAAAAAAA & x = 0xAAAAAAAA` 成立。则可以使用表达式 `!(0xAAAAAAAA ^ (0xAAAAAAAA & x))` 作为答案。因为不能直接使用这么大的常数，mask 需要从 `0xAA` 经过运算得到。以下表达式得到 `mask = 0xAAAAAAAA`:

```C
  int mask = 0xAA;
  mask |= mask << 8;
  mask |= mask << 16;
```

所以答案是：

```C
int allOddBits(int x) {
  int mask = 0xAA;
  mask |= mask << 8;
  mask |= mask << 16;
  return !((x & mask) ^ mask);
}
```

# 0x05 negate

```C
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return 2;
}
```

这道题要求使用5个以内给定种类的运算符实现补码相反数操作。

对于补码整数，其编码与无符号数编码一致；对于补码负数，其编码方式则是最高位取负权，其余位取正权。

通过学习CS:APP第二章我们知道，补码的非有两种聪明的求法：

1. 对每一位求补，再对结果加 `1`。在C语言中，对于任意整数 `x`，计算表达式 `-x` 和 `~x+1` 得到的结果完全一样。
2. 将位向量分为两部分，假设 $k$ 是位向量中最右边的 $1$ 的位置，因而 $x$ 的位级表示形如 $[x_{(w−1)}, x_{(w−2)}, …,x_{(k+1)},1,0,…,0]$，这个值的非的二进制形式为：$[~x_{(w−1)},~x_{(w−2)},…,~x_{(k+1)},1,0,…,0]$。即：对位 $k$ 左边的所有位取反。

这题中应用第一种求法，得到答案：`~x + 1`。

# 0x06 isAsciiDigit

```C
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
  return 2;
}
```

这道题判断所给参数是否是 ASCII 数字，即判断给定参数是否在 `0x30 ~ 0x39` 范围内。

观察 `0x30 ~ 0x39` 这十个数字的位模式，发现其高28位都相同，低4位中涵盖了从 `0` 到 `9` 的范围。下面对照4位二进制数的真值表观察其特征：

| 位模式 | 整数值 | 位模式 | 整数值 |
| ------ | ------ | ------ | ------ |
| `0000` | `0`    | `0101` | `5`    |
| `0001` | `1`    | `0110` | `6`    |
| `0010` | `2`    | `0111` | `7`    |
| `0011` | `3`    | `1000` | `8`    |
| `0100` | `4`    | `1001` | `9`    |

可以看到范围内的值除了 `8` 和 `9` 之外最高位都为 `0`，且最高位为 `0` 时，低3位所有的取值情况都在范围内。对应到32位整数就是除了 `0x38` 和 `0x39` 之外，高29位都相同，且低3位可以为任意值。

我们将 `0x38` 和 `0x39` 这两个数字作为特例，除这两个值外，判断 `x` 高29位是否是合法值，如果是，那么该数字一定是 ASCII Code。如果不是，再判断是否是两个特例值中的一个。

判断高29位的表达式为：`!(((x >> 3) << 3) ^ 0x30)`，判断 `x` 是否是特殊值的表达式是：`!((x ^ 0x38) & (x ^ 0x39))`，最终结果为：`!(((x >> 3) << 3) ^ 0x30) | !((x ^ 0x38) & (x ^ 0x39))`

# 0x07 conditional

```C
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  return 2;
}
```

使用给定的几种不超过16个运算实现三元运算符，即：`x ? y : z`，若 `x == 0`，返回 `z`，否则返回 `y`。

在不能使用条件分支的情况下，根据 `x` 的取值进行返回，首先比较容易想到使用 `x` 作为 mask 对 `y` 和 `z` 进行过滤。比如当 `x == 0` 时，可以用表达式 `x & y` 过滤掉 `y`；当 `x != 0` 时，根据相反的原则，首先将 `x` 做成所有位都为 `1` 的样子：`~!!x + 1`，然后表达式 `x | z` 可以过滤掉 `z`。

根据上面所说的情况，可以知道使用 `x` 的值作为 mask 的思路大概是可行的。不过要首先将 `x` 转换为全 `0` 或全 `1` 的形式——这里使用额外的变量表示——`int isXZero = ~!!x + 1`。对于两种过滤表达式，考察其取值：

由于：

$$\mathrm{isXZero} = \begin{cases}
  0 & when & x = 0 \\
  -1 & when & x \neq 0
\end{cases} $$

有：

$$\mathrm{isXZero\  \&\  y} = \begin{cases}
  0 & when & x = 0 \\
  y & when & x \neq 0
\end{cases}$$

和：

$$\mathrm{isXZero\  |\  z} = \begin{cases}
  -1 & when & x \neq 0 \\
  z & when & x = 0
\end{cases}$$

可以看到两种滤网滤出的数值根据 `x` 的取值分别取有效值和较为规则的无效值，且取有效值的情况彼此之间错开。这样的情况很容易想到将其相加：

$$\mathrm{(isXZero\ \&\ y) + (isXZero\ |\ z)} = \begin{cases}
  z & when & x = 0 \\
  y - 1 & when & x \neq 0
\end{cases}$$

这样答案就较为明显了，只需在 `x != 0` 时将上述的和式加上 `1` 就可以。可以认为 `x == 0` 时上式加上 `0`，这样加上的实际上是 `x` 的某种变形，即 `~!!x`。所以答案为：`(isXZero | z) + (isXZero & y) + !!x`