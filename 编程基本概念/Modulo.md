### Modulo Calculator
---
Modulo abbreviated as mod

| Language     | Operator                        | Integer | Floating-point | Definition |
| ------------ | ------------------------------- | ------- | -------------- | ---------- |
| c/c++        | %,div                           | Yes     | No             | Truncated  |
| c/c++        | fmod(C) std::fmod(C++)          | No      | Yes            | Truncated  |
| c/c++        | remainder(C) std:remainder(C++) | No      | Yes            | Rounded    |
| Go           | %                               | Yes     | No             | Truncated  |
| Go           | math.Mod                        | No      | Yes            | Truncated  |
| Go           | big.Int.Mod                     | Yes     | No             | Euclidean  |
| Java         | %                               | Yes     | Yes            | Truncated  |
| Java         | Math.floorMod                   | Yes     | No             | Floored    |
| TypeScript   | %                               | Yes     | Yes            | Truncated  |
| lua5         | %                               | Yes     | Yes            | Floored    |
| python       | %                               | Yes     | No             | Floored    |
| python       | math.fmod                       | Yes     | Yes            | Truncated  |
| Rust         | %                               | Yes     | Yes            | Truncated  |
| Rust         | rem_euclid()                    | Yes     | Yes            | Euclidean  |
| x86 assembly | IDIV                            | Yes     | NO             | Truncated  |

主要需要注意的情况：
- Truncated  出现次数最多，最重要
- Floored   第二多
- Euclidean  第三多
- Rounded  只有一次

最常见的情况：
Given two positive numbers a and n, amodulo n (often abbreviated as a mod n) is the remainder of the Euclidean division of a by n, where a is the divideand and n is the divisor.

a mod 0 is undifined, possibly resulting in a division by zero error in some programming languages.

不常见的情况：
When exactly one of a or n is negative, the naive definition breaks down, and programming languages differ in how these values are defined.

当余数非 0 时，余数的符号仍然是有歧义的：余数非 0 时，它的符号有两种选择，一个正、一个负。 通常情况下，**在数论中总是使用正余数**。但**在编程语言中**，余数的符号取决于编程语言的类型和被除数 a 或除数 $n$ 的符号。

#### Definition
- Truncated 
- Floored 
- [Euclidean](#^Euclidean)
- Rounded

If the remainder is non-zero:two possible choices for the remainder occur, one negative and the other positive, and two possible choices for the quotient occur. **In number theory, the positive remainder is always chosen, but in computing, programming languages choose depending on the language and the signs of a or n.** 

- Many implementations use **truncated** division for which the quotient is defined by $$q=\left[{\frac{a}{n}}\right]$$
where [] is the intergral part function **(rounding toward zero)**, i.e. the truncation to zero significant digits. **Thus according to equation, the remainder has the same sign as the dividend**:$$r=a-n\left[{\frac{a}{n}}\right]$$
Quotient: 商
Divisor: 除数/因子
Remainder:余数  mod  计算的结果
Divident: 被除数
             Quotient * Divisor + Remainder = Dividend
			![[取模运算.png]]

### 性能问题
---
x % 2n == x & (2n - 1)

例子，假定 x 为正数：
x % 2 == x & 1
x % 4 == x & 3
x % 8 == x & 7

### 上面用到的数学知识
---
#### Truncated  
很多取模的实现都使用了截断除法，此时商由截断函数定义定义 $q=\left[{\frac{a}{n}}\right]$ , **余数和被除数符号一致。商向零取整**：结果等于普通除法所得的小数靠近 0 方向的第一个整数(直接截断小数部分)。$$r=a-n\left[{\frac{a}{n}}\right]$$
e.g.
- -9 % 2 = -1
- -9 % -2 = -1
- 9 % -2 = 1

#### Floored
 _floored division_, for which the quotient is defined by $q=\left\lfloor{\frac{a}{n}}\right\rfloor$ , the reminder has the same sign as the divisor $$r=a-n\left\lfloor{\frac{a}{n}}\right\rfloor$$

#### Euclidean division ^Euclidean
the quotient is defined by $$q=\operatorname{sgn}(n)\left\lfloor{\frac{a}{|n|}}\right\rfloor=\left\{{\begin{array}{l l}{\left\lfloor{\frac{a}{n}}\right\rfloor}&{{\mathrm{if}}\,n>0}\\ {\left\lceil{\frac{a}{n}}\right\rceil}&{{\mathrm{if}}\,n<0}\end{array}}\right.$$
the remainder is **non negative**:$$r=a-|n|\left|\frac{a}{|n|}\right|$$
欧几里得除法，带余除法$$a = bq + r \space and \space\space 0\leq r < |b|$$
给定两个整数a和b, 其中$b \ne 0$ ， 存在以上表达式

欧几里得除法的余数一定是正数

带余除法限定了余数的范围，使得商唯一存在

e.g.
- If a = 7 and b = 3, then q = 2 and r = 1, since 7 = 3 × 2 + 1.
- If a = 7 and b = −3, then q = −2 and r = 1, since 7 = −3 × (−2) + 1.
- If a = −7 and b = 3, then q = −3 and r = 2, since −7 = 3 × (−3) + 2.
- If a = −7 and b = −3, then q = 3 and r = 2, since −7 = −3 × 3 + 2.

可以从例子中看到， 如果除数或被除数是负数，结果和直观感受不同， 其实可以通过谁大来理解。

比如第3、4个例子， -7 比 -9大 2， 余数就是2

#### Rounded
Common Lisp and IEEE 754 use rounded division, for which the quotient is defined by $$ q=\operatorname{round}\left({\frac{a}{n}}\right)$$
算了，这个很少见，以后见了再说。
参考
[Modulo - Wikipedia](https://en.wikipedia.org/wiki/Modulo#math_1)

#### Rounding toward zero
round toward zero (or truncate, or round away from infinity): y is the integer that is closest to x such that it is between 0 and x (included); i.e. y is the integer part of x, without its fraction digits.
$$y=\operatorname{truncate}(x)=\operatorname{sgn}(x)\lfloor|x|\rfloor=-\operatorname{sgn}(x)\lceil-|x|\rceil={\left\{\begin{array}{l l}{\lfloor x \rfloor}&{x\geq0}\\ {\lceil x \rceil}&{x<0}\end{array}\right.}$$
For example, 23.7 gets rounded to 23, and −23.7 gets rounded to −23.

#### Rounding away from zero
**round away from zero** (or **round toward infinity**): y is the integer that is closest to 0 (or equivalently, to x) such that x is between 0 and y (included).
$$y=\operatorname{sgn}(x)\lceil|x|\rceil=-\operatorname{sgn}(x)\lfloor-\left|x\right|\rfloor={\left\{\begin{array}{l l}{\lceil x\rceil}&{x\geq0}\\ {\lfloor x\rfloor}&{x<0}\end{array}\right.}$$
For example, 23.2 gets rounded to 24, and −23.2 gets rounded to −24.

#### Floor and ceiling functions
In mathematics and computer science, the floor function is the function that takes as **input a real number x**, and gives as **output the greatest integer less than or equal to x**, denoted floor($x$) or $\lfloor x\rfloor$.  

Similarly, the ceiling function maps $x$ to **the least integer greater than or equal to $x$**, denoted ceil($x$) or $\lceil x \rceil$.

e.g. $$\lfloor 2.4 \rfloor= 2, \lfloor -2.4 \rfloor= -3,\lceil 2.4 \rceil = 3, and \lceil -2.4 \rceil = -2$$

