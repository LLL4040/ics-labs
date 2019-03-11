@[TOC](Lab 1)
# Introduction
>The purpose of this assignment is to become more familiar with bit-level representations and manipulations.
You’ll do this by solving a series of programming “puzzles.”
Many of these puzzles are quite artificial, but you’ll find yourself thinking much more about bits in working your way through them.

# Coding rules
 
  Replace the "return" statement in each function with one
  or more lines of C code that implements the function. Your code 
  must conform to the following style:
 ```c
  int Funct(arg1, arg2, ...) {
      /* brief description of how your implementation works */
      int var1 = Expr1;
      ...
      int varM = ExprM;

      varJ = ExprJ;
      ...
      varN = ExprN;
      return ExprR;
  }
```
* Each "Expr" is an expression using ==ONLY== the following:
  	* Integer constants 0 through 255 (0xFF), inclusive. You are
      not allowed to use big constants such as 0xffffffff.
  	* Function arguments and local variables (no global variables).
  	* Unary integer operations ! ~
  	* Binary integer operations & ^ | + << >>
    
* Some of the problems restrict the set of allowed operators even further.
  Each "Expr" may consist of multiple operators. You are not restricted to
  one operator per line.

* You are expressly ==forbidden== to:
  * Use any control constructs such as if, do, while, for, switch, etc.
  * Define or use any macros.
  * Define any additional functions in this file.
  * Call any functions.
  * Use any other operations, such as &&, ||, -, or ?:
  * Use any form of casting.
 
  You may assume that your machine:
  1. Uses 2's complement, 32-bit representations of integers.
  2. Performs right shifts arithmetically.
  3. Has unpredictable behavior when shifting an integer by more
     than the word size.

EXAMPLES OF ACCEPTABLE CODING STYLE:
```c
  /*
   * pow2plus1 - returns 2^x + 1, where 0 <= x <= 31
   */
  int pow2plus1(int x) {
     /* exploit ability of shifts to compute powers of 2 */
     return (1 << x) + 1;
  }

  /*
   * pow2plus4 - returns 2^x + 4, where 0 <= x <= 31
   */
  int pow2plus4(int x) {
     /* exploit ability of shifts to compute powers of 2 */
     int result = (1 << x);
     result += 4;
     return result;
  }
```

# Special requires
总结上述规则可以得出以下几点结论：
1. ==!  ˜  &  ˆ  |  +  <<  >>== 只有这些操作符允许使用，在特定题目中可能还会有所限制，而且要注意ops的数目
2. 不允许使用C语言中的保留字```if, do, while, for, switch```等
3. 右移为算术右移，整数均为32-bits
4. 需要使用```./dlc```检查是否遵守规则，仔细分析可以发现，声明变量的时候必须连续将所有变量一次性声明完毕，即不能在声明中间掺杂非声明代码，例如以下声明即为非法声明：
```c
  int x;
  int y;
  x = 1;
  y = 2;
  ... //一些非声明的代码
  int num;
```

# Puzzle 1
1. Requires
```c
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
```
2. Analysis
* 分析题意
```mermaid
flowchat
st=>start: 开始
e=>end: 结束
in=>inputoutput: 输入x
out=>inputoutput: 输出y
cond=>condition: x == 0 ?
op1=>operation: y = 1
op2=>operation: y = 0

st->in->cond
cond(yes)->op1->out
cond(no)->op2->out
out->e
```
* 可以看出==0==为特殊值，所以考虑先判断出来x是否为0，使用```x | -x```，如果x=0，则该值为0，否则符号位必为1
* 之后有两种方式得到正确输出
1. 考虑到符号位以及算术右移的特殊性，使用```(x | -x) >> 31```可获得0或-1，然后+1即为正确输出
```c
int bang(int x)
{
	return (x | (~x + 1)) >> 31 + 1;
}
```
2. 先取反，符号位变成1或0，右移之后再取最后一位 
```c
int bang(int x)
{
	return (~(x | (~x + 1)) >> 31) & 1;
}
```
==Tip: -x = ~x + 1==

# Puzzle 2
1. Requires
```c
/*  
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
```
2. Analysis
* 分析题意：就是将给定的十进制数转化为二进制，再数出1的个数
* 如果忽略==Max ops: 40==的要求的话，这道题非常简单，只需要右移31次，每次将最低位取出来累加即可，代码如下：
```c
int bitCount(int x)
{
	int sum = x & 1;
	sum = sum + ((x >> 1) & 1);
	sum = sum + ((x >> 2) & 1);
	sum = sum + ((x >> 3) & 1);
	...
	sum = sum + ((x >> 31) & 1);
	return sum;
}
```
* 上述做法不但代码重复量大，而且需要94个op才能完成，显然不可取
* 考虑到输入数字位数为32 = 4 x 8，所以可以拆分为8段，每段4-bits，同时累加
```c
  int mask1 = 0x11 | (0x11 << 8);
  int mask = mask1 | (mask1 << 16);/* mask = 0001 0001 0001 0001 0001 0001 0001 0001 */
  int sum = x & mask;
  sum = sum + ((x >> 1) & mask);
  sum = sum + ((x >> 2) & mask);
  sum = sum + ((x >> 3) & mask);
```
* 这样```sum```记录了每四个中1的个数，之后再将```sum```的各段累加起来
```c
  sum = sum + (sum >> 16);/* 前后两段相加 */
  mask = 0xF | (0xF << 8);/* mask = 0000 1111 0000 1111 */
  sum = (sum & mask) + ((sum >> 4) & mask);
  sum = (sum + (sum >> 8)) & 0xFF;/* 由于最多可能有32个1，需要5bits，所以用0xFF来储存最后的值 */
```
* 这道题算是所有题目里面最难的了，总结代码如下：
```c
int bitCount(int x) 
{
    int mask1 = 0x11 | (0x11 << 8);
    int mask = mask1 | (mask1 << 16);
    int sum = x & mask;
    sum = sum + ((x >> 1) & mask);
    sum = sum + ((x >> 2) & mask);
    sum = sum + ((x >> 3) & mask);
    sum = sum + (sum >> 16);
    mask = 0xF | (0xF << 8);
    sum = (sum & mask) + ((sum >> 4) & mask);
    sum = (sum + (sum >> 8)) & 0xFF;
    return sum;
}
```

# Puzzle 3
1. Requires
```c
/*  
 * copyLSB - set all bits of result to least significant bit of x
 *   Example: copyLSB(5) = 0xFFFFFFFF, copyLSB(6) = 0x00000000
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
```
2. Analysis
* 这个题比较简单，虽然左移没有逻辑右移的特性，但是可以先左移过去得到最高位为1或0，再右移即可
* 代码如下：
```c
int copyLSB(int x) 
{
    return (x << 31) >> 31;
}
```

# Puzzle 4
1. Requires
```c
/* 
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
```
2. Analysis
* 先分析一些数据，4-bits为例：
	* 1/2 = 0 = 0001 >> 1
	* 2/2 = 1 = 0010 >> 1
	* 3/2 = 1 = 0011 >> 1
	* -1/2 = 0 ≠ 1111 >> 1 = -1
	* -2/2 = -1 = 1110 >> 1
	* -3/2 = -1 ≠ 1101 >> 1 = -2
	* -4/2 = -2 = 1100 >> 1 
	* -5/2 = -2 = (-5 + 2 - 1) >> 1
	* -6/2 = -3 = (-6 + 2 - 1) >> 1
* 观察发现负数的时候直接右移在不能整除的时候会出现问题，但是当加上2^n - 1之后再右移就没有问题了（上面是以2的一次方为例，你也可以找一些其他的例子来寻找规律）
* 所以思路如下：
```mermaid
flowchat
st=>start: 开始
e=>end: 结束
in=>inputoutput: 输入x, n
out=>inputoutput: 输出y
cond=>condition: x >= 0 ?
op1=>operation: y = x >> n
op2=>operation: y = (x + 2^n + 1) >> n

st->in->cond
cond(yes)->op1->out
cond(no)->op2->out
out->e
```
* 代码如下：
```c
int divpwr2(int x, int n) 
{
    int flag = (1 << n) + ~0;/* 计算2^n + 1的值 */
    flag = flag & (x >> 31);/* 当 x >= 0 的时候 flag & 0 = 0 */
    return (x + flag) >> n;
}
```

# Puzzle 5
1. Requires
```c
/* 
 * evenBits - return word with all even-numbered bits set to 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 2
 */
```
2. Analysis
* 该题非常简单，读题可知只需要返回二进制中奇数位全为1的数，不过由于rules限制了使用的mask的范围为```0x0 ~ 0xFF```，所以不能直接将这个数写出来，需要经过几次左移才能得到
* 代码如下：
```c
int evenBits(void) 
{
    int x = 0x55;
    x = (x << 8) + x;
    x = (x << 16) + x;
    return x;
}
```

# Puzzle 6
1. Requires
```c
/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
```
2. Analysis
* 题意可理解为```length(用二进制表示的x) <= n ? 1 : 0```，但长度很难确定而且还要考虑符号位，比如5 = 101用3-bits读就变成了负数
* 所以可以考虑对x进行变换，只保留n位，思路就变成了判断截取n位后是否与原数相等，显然相等的话就可以用n-bits表示
* 代码如下：
```c
int fitsBits(int x, int n) 
{
    int mask = 32 + ~n + 1;
    int y = (x << mask) >> mask;
    return !(x ^ y);
}
```
==Tip: if x = y then x ^ y = 0==

# Puzzle 7
1. Requires
```c
/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
```
2. Analysis
* 题意为将x(十六进制表示)分为四段，从右向左依次标记为0,1,2,3（小端法），输出第二个参数对应的段
* 其实就是用0xFF来取，第几段就左移到指定位置即可，==注意在十六进制里面移动一段实际上是二进制移动8位==
* 代码如下：
```c
int getByte(int x, int n) 
{
    n = n << 3;
    return ((x & (0xFF << n)) >> n) & 0xFF;
}
```

# Puzzle 8
1. Requires
```c
/* 
 * isGreater - if x > y  then return 1, else return 0 
 *   Example: isGreater(4,5) = 0, isGreater(5,4) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
```
2. Analysis
* ==分情况讨论==
```mermaid
flowchat
st=>start: 开始
e=>end: 结束
in=>inputoutput: 输入x, y
out=>inputoutput: 输出flag
cond1=>condition: x < 0 && y >= 0?
cond2=>condition: x >= 0 && y < 0?
op1=>operation: flag = 0
op2=>operation: flag = 1
op3=>operation: flag = !(x - y的符号位)

st->in->cond1(yes)->op1->out
cond1(no)->cond2(yes)->op2->out
cond2(no)->op3->out
out->e
```
* 讨论还是比较简单的，但是限制了不让用条件分支，所以处理起来比较麻烦
* 首先可以看出当x负y正时直接得出flag=0，所以考虑先用一个变量存储x,y符号位，并保证在x负y正的时候可以得到0，使用```int b = (!((y >> 31) & 1)) & ((x >> 31) & 1);```，则```!b```的值可以满足需要
* 如果x正y负，同样可以直接得出结论flag=1，使用```int a = (!((x >> 31) & 1)) & ((y >> 31) & 1);```，则```a```的值可以满足需要
* 否则就要作差读取符号位来判断大小了
* 代码如下：
```c
int isGreater(int x, int y) 
{
    int a = (!((x >> 31) & 1)) & ((y >> 31) & 1);
    int b = (!((y >> 31) & 1)) & ((x >> 31) & 1);
    int c = ((x + ~y + 1) >> 31) & 1;
    int d = !(x ^ y);
    return (a | !(c | d)) & (!b);
}
```

# Puzzle 9
1. Requires
```c
/* 
 * isNonNegative - return 1 if x >= 0, return 0 otherwise 
 *   Example: isNonNegative(-1) = 0.  isNonNegative(0) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 3
 */
```
2. Analysis
* 这类题一般都是通过符号位来解决，分析知，当x>=0的时候，x的符号位为0，右移31位为0；否则右移31位为-1。显然再+1即为正确输出
* 代码如下：
```c
int isNonNegative(int x) 
{
  return (x >> 31) + 1;
}
```

# Puzzle 10
1. Requires
```c
/* 
 * isNotEqual - return 0 if x == y, and 1 otherwise 
 *   Examples: isNotEqual(5,5) = 0, isNotEqual(4,5) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
```
2. Analysis
* 根据异或```^```的特性可以很容易的写出这道题，==注意正确输出格式==
* 代码如下：
```c
int isNotEqual(int x, int y) 
{
  return !!(x ^ y);
}
```

# Puzzle 11
1. Requires
```c
/*
 * isPower2 - returns 1 if x is a power of 2, and 0 otherwise
 *   Examples: isPower2(5) = 0, isPower2(8) = 1, isPower2(0) = 0
 *   Note that no negative number is a power of 2.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 60
 *   Rating: 4
 */
```
2. Analysis
* 题目为判断是否为2的幂，可理解为```(x >= 0 && bitCount(x) == 1) ? 1 : 0```
* 这道题也是个难题，不过由于我们已经解决了bitCount(x)，所以这道题就简化为了判断x是否非负
* 代码如下：
```c
int isPower2(int x) 
{
    int mask1 = 0x11 | (0x11 << 8);
    int mask = (mask1 | (mask1 << 16));
    int sum = (x & mask);
    sum = sum + ((x >> 1) & mask);
    sum = sum + ((x >> 2) & mask);
    sum = sum + ((x >> 3) & mask);
    sum = sum + (sum >> 16);
    mask = 0xF | (0xF << 8);
    sum = (sum & mask) + ((sum >> 4) & mask);
    sum = (sum + (sum >> 8)) & 0xFF;
    return !((x >> 31) & 1) & !(sum + ~0);
}
```

# Puzzle 12
1. Requires
```c
/* 
 * leastBitPos - return a mask that marks the position of the
 *               least significant 1 bit. If x == 0, return 0
 *   Example: leastBitPos(96) = 0x20
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 4 
 */
```
2. Analysis
* 题意为返回一个mask，只有与x从右数第一个1的位置对应的位置为1，其余位置均为0，没有思路的时候不妨举几个例子观察一下，以4-bits为例：
	* 1 = 0001，-1 = 1111
	* 2 = 0010，-2 = 1110
	* 3 = 0011，-3 = 1101
	* 4 = 0100，-4 = 1100
	* 5 = 0101，-5 = 1011
* 观察发现，正负数除了从右数第一个1的位置上的数相同，其余位置的数都相反，所以可以用```&```来获取需要的位
* 代码如下：
```c
int leastBitPos(int x) 
{ 
    return x & (~x + 1);
}
```

# Puzzle 13
1. Requires
```c
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 1 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3 
 */
```
2. Analysis
* 题意是实现逻辑右移，其实就是前面用0补齐，从算术右移的角度来实现很简单
* 代码如下：
```c
int logicalShift(int x, int n) 
{
  int mask = ~((1 << 31) >> (n + ~0));
  return (x >> n) & mask;
}
```

# Puzzle 14
1. Requires
```c
/*
 * satAdd - adds two numbers but when positive overflow occurs, returns
 *          maximum possible value, and when negative overflow occurs,
 *          it returns minimum positive value.
 *   Examples: satAdd(0x40000000,0x40000000) = 0x7fffffff
 *             satAdd(0x80000000,0xffffffff) = 0x80000000
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 30
 *   Rating: 4
 */
```
2. Analysis
* 题目是做加法，难在处理溢出上，分情况讨论可以知道异号相加肯定不会溢出，所以重点分析同号的情况
```mermaid
flowchat
st=>start: 开始
e=>end: 结束
in=>inputoutput: 输入x, y
out=>inputoutput: 输出sum
cond=>condition: x*y <= 0?
op1=>operation: flag = 0
op2=>operation: flag = 1
op3=>operation: num = 对应临界值
op4=>operation: sum = flag*(num - x - y) + x + y

st->in->cond
cond(yes)->op1->op4
cond(no)->op2->op3->op4
op4->out->e
```
* 代码如下：
```c
int satAdd(int x, int y) 
{
    int mask = 1 << 31;
    int sum = x + y;
    int flag = ((x ^ sum) >> 31) & ((y ^ sum) >> 31);
    int num = (sum >> 31) ^ mask;
    return (flag & (num + ~x + ~y + 2)) + x + y;
}
```

# Puzzle 15
1. Requires
```c
/* 
 * tc2sm - Convert from two's complement to sign-magnitude 
 *   where the MSB is the sign bit
 *   You can assume that x > TMin
 *   Example: tc2sm(-5) = 0x80000005.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 4
 */
```
2. Analysis
* 题意相当于将补码转换成原码，但是要保留符号位
* 根据补码规则可以知道，非负数不用变，负数需要取反加一（不考虑符号位）
* 所以我们需要
	* 保留符号位
	* 对负数取反并加一
* 代码如下：
```c
int tc2sm(int x) 
{
  /* For nonnegative number, m1,m2,m3 are all 0. */
  /* But for negative number, m1 will change x's 1 to 0 and 0 to 1, m2 saves the sign bit and m3 let x + 1. */
    int m1 = x >> 31;
    int m2 = x & (1 << 31);
    int m3 = m1 & 1;
    return ((x ^ m1) | m2) + m3;
}
```
==Tip: 涉及到正负数的时候，符号位以及算术右移都是常用的处理技巧==
