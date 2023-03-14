### Lab1 DataLab

这个实验主要是让大家充分理解C语言中位运算、逻辑运算和算数运算以及计算机中无符号数、有符号数和浮点数是如何表示的。
通过认真阅读完CSAPP第三版中第二章的内容，完成实验难度不是太大，部分题目比较有难度~

/datalab-handout这个目录下我们只用修改`bits.c`这个文件里的代码即可。
`dlc`是帮助我们检查我们写的代码是否合法，是否按照题目要求使用了规定的运算符并且是否超出规定的运算符的数目，通过执行命令`./dlc bits.c`即可运行。

每次运行时，可以通过以下指令进行：

```
make clean
make btest
./btest
```

每道题的要求不同，在编写代码时务必仔细阅读每道题的要求！

#### 1. bitXor

这个函数要求我们只用`~`和`&`两个运算符来实现异或运算。
我们可以将异或运算写出来：

$a \oplus b = (a \land \lnot b) \lor (\lnot a \land b)$

利用德摩根律：

$a \oplus b = \neg (\neg (a \land \lnot b) \land \lnot (\lnot a \land b))$
现在就只剩下`~`和`&`运算符了，并且数目没有超过14个，该公式也可以进一步化简。

```c
//1
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
    return ~(~(~x & y) & ~(x & ~y)); 
}
```

#### 2. tmin

这个函数要求我们返回二进制形式下，补码表示的最小数。

这个数显然是`0x80000000`，因为题目要求常数不能超过`255`，所以我们最好就不写`1 << 31`了。

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
	return (0x80 << 24);
}
```

#### 3. isTmax

这个函数让我们判断一个数是不是二进制形式下，补码表示的最大数。

这个数是`0x7fffffff`。

通过观察，我们发现这个数取反后，就是`tmin`；而`tmin`有一个很好的特点就是，它和它的相反数一样。

但是，值得注意的是，`0`也和它的相反数表示形式一样，所以还需要特判一下这个数不是`0`。

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
	x = ~x;
	return !(x ^ (~x + 1)) & (!!x);
}
```

#### 4. allOddBits

这个函数让我们判断一个给定的数在二进制表示下，是不是全部奇数位都是1.

这个很容易实现，我们可以先生成一个奇数位都是1，偶数位都是0的掩码mask。

然后让`x & mask`，看看mask是否保持不变，如果不变，说明满足条件。

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
	int mask = (0xAA << 8) | 0xAA;
	mask = (mask << 16) | mask;
	return !((x & mask) ^ mask);
}
```

#### 5. negate

这个函数简单，让我们返回一个数的相反数。

直接`~x + 1`即可。

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  	return (~x + 1);
}
```

#### 6. isAsciiDigit

这个函数让我们判断一个数是否位于`0x30`到`0x39`之间。

我们可以分别进行判断，首先判断10位是否为3，我们可以利用右移运算符。

然后，判断个位是否小于等于`9`。

我们判断`a <= b`时，可以通过`a + b <= 0`来进行判断，也就是让二者相加，然后判断符号位。

```c
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
	int y = x & 0xf; // 把最低位先拿出来
  	 return !((x >> 4) ^ 0x3) & !((0x9 + (~y + 1)) >> 31);
}
```

#### 7. conditional

这个函数让我们实现一个条件表达式 `x ? y : z`。

只要`x`不为0，我们就返回`y`；否则返回`z`。

首先，设置一个变量`notZero`来判断`x`是否不为0，如果不为`0`，则该变量为`1`；否则为`0`。

然后，设置一个变量`flag`，当`notZero`为`1`时，该变量为`0xffffffff`；否则为`0x00000000`。

此时，我们就可以很巧妙地利用`(flag & y) | (~flag & z)`来进行计算了。

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  	int notZero = !!x;
	int flag = ~notZero + 1;
	return (flag & y) | (~flag & z);

}
```

#### 8. isLessOrEqual

这个函数让我们判断$x \le y$，$x = y$很容易判断，因此我们先判断$x = y$。

判断$x < y$的时候，我们仍然使用$x + y < 0$这个条件来判断。

但是要注意，可能会出现$x + y$溢出的情况，从而导致判断错误。

因此，我们首先根据符号进行判断，如果二者符号不同，那么答案显然；否则再利用上式进行判断。

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
	int flagx = (x >> 31 & 1);
	int flagy = (y >> 31 & 1);
	int notZero = !!(flagx ^ flagy);
	int flag = ~notZero + 1;
	int less = ((x + (~y + 1)) >> 31 & 1);
	return (!(x ^ y)) | ((flag & flagx) | (~flag & less));
}
```

#### 9. logicalNeg

这个函数让我们实现`!`运算符——也就是`x = 0`时返回`1`，否则返回`0`。

因此，我们可以通过判断`x`是否为`0`来实现这个函数，不难发现，只有`0`和其相反数或之后，全部都位为`0`。

所以，我们利用这个条件判断即可。

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
  	return ((x | (~x + 1)) >> 31) + 1;
}
```

#### 10. howManyBits

这个函数让我们判断一个数在二进制补码表示下，最少需要多少位才能表示出来。

通过观察给的样例：

12：01100

-5：1011

0：0

-1：1

当`x`为正数时，假设`idx`表示为`1`的最高的位置，答案即`idx + 1`。

当`x`为负数时，我们将`x`取反，假设`idx`表示此时为`1`的最高的位置，答案即`idx + 1`。

因此，我们可以通过找出最高的1的位置来计算答案。

因为不能使用循环，所以我们利用二分查找。

首先，判断高16位是否存在1，如果存在1的话，我们就只需要在高16位上寻找最高位；如果高16位不存在1，那么我们去低16位寻找。就这样一直找下去，直到区间长度变为1停止。

具体实现可以看代码。

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
	int b16, b8, b4, b2, b1, b0;
	int sign = x >> 31;
	x = (sign & ~x) | (~sign & x);
	b16 = !!(x >> 16) << 4;
	x = x >> b16;
	b8 = !!(x >> 8) << 3;
	x = x >> b8;
	b4 = !!(x >> 4) << 2;
	x = x >> b4;
	b2 = !!(x >> 2) << 1;
	x = x >> b2;
	b1 = !!(x >> 1);
	x = x >> b1;
	b0 = x;
  	return b16 + b8 + b4 + b2 + b1 + b0 + 1;
}
```

#### 11. floatScale2

从这个函数开始，就都是涉及到浮点数表示了，如果不是很了解，建议仔细阅读一下《深入理解计算机系统》第二章后半部分的内容。

这个函数给了我们一个浮点数表示的数`f`，让我们计算出`2 * f`的浮点数表示。

首先，我们把`f`中的符号位`s`，阶数`exp`，尾数`frac`全都扣出来。

然后，判断一下是否是非规格化的值，也就是判断阶数`exp`是否为0，如果为`0`，我们直接把尾数乘2然后配上符号输出即可。

如果`exp = 255`，也就是为`NaN`或者无穷，我们直接返回`f`。

否则，就把阶数`exp`加1，如果此时`exp`为255，也就是阶码位全为1，此时溢出了，我们返回无穷。

最后，代表没有溢出，我们返回正常的值即可。

```c
//float
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
  	unsigned s = uf & (1 << 31);
	unsigned exp = (uf & 0x7f800000) >> 23;
	unsigned frac = uf & (~0xff800000);

	if(exp == 0) return frac << 1 | s;
	if(exp == 255) return uf;
	exp ++;
	if(exp == 255) return 0x7f800000 | s;
	return s | (exp << 23) | frac;
}
```

#### 12. floatFloat2Int

这个函数让我们把一个给定的浮点数`f`通过强制类型转换为`int`。

`int`的表示范围要比单精度浮点数小很多$[-2^{31}, 2^{31} - 1]$，所以我们可能会出现精度损失的情况。

首先，还是先把符号位`s`，阶码`exp`，尾数`frac`都抠出来。

我们计算一下真实的阶数，$E = exp - 127$。

如果`f`是`NaN`，或者$E > 31$，根据题意，我们应该直接返回`0x80000000u`。

如果$E < 0$，代表`f`只有小数部分，我们直接返回`0`即可。

我们算出真实的尾数$M = frac | (1 << 23)$。

如果$E > 23$我们就需要在后面补$E - 23$个0，否则，我们需要舍去$23 - E$位。

最后，搭配上符号位输出即可。

```c
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) {
   unsigned s = uf & (1 << 31);
   int exp = (uf & 0x7f800000) >> 23;
   unsigned frac = uf & (~0xff800000);

   int E = exp - 127;
   if(exp == 255 || E > 31) return 0x80000000u;
   if(E < 0) return 0;
   unsigned M = frac | (1 << 23);
   int V = (E > 23 ? M << (E - 23) : M >> (23 - E));
   if(s) V *= -1;
   return V;
}
```

#### 13. floatPower2

这个函数让我们算出$2 ^ x$的单精度浮点数表示并返回。

`x`只能位于$[-149, 128]$之间。

根据不同情况输出即可，要考虑到尾数也可以额外表示浮点数。

```c
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x) {
   if(x >= 128) return 0x7f800000;
   if(x >= -126) return (x + 127) << 23;
   if(x >= -149) return 1 << (x + 149);
   return 0;
}
```

