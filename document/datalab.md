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

