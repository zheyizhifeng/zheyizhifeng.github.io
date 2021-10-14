---
title: C 语言——标准库
date: 2021-10-11 17:33:57
categories: 
  - Programming
  - C 语言
tags: 
  - Programming
  - C 语言
---

# assert.h

## assert()

`assert.h`头文件定义了宏`assert()`，用于在运行时确保程序符合指定条件，如果不符合，就报错终止运行。

```c
assert(PI > 3);
```

上面代码在程序运行到这一行语句时，验证变量是否`PI`大于3。如果确实大于3，程序继续运行，否则就会终止运行，并且给出报错信息提示。
<!-- more -->

`assert()`宏接受一个表达式作为参数，如果表达式的返回值非零，`assert()`就会报错，在标准错误流`stderr`中写入一条错误信息，显示没有通过的表达式，以及包含这个表达式的文件名和行号。最后，调用`abort()`函数终止程序（`abort()`函数的原型在`stdlib.h`头文件中）。
```c
z = x * x - y * y;
assert(z >= 0);
```

上面的`assert()`语句类似于下面的代码。

```c
if (z < 0) {
  puts("z less than 0");
  abort();
}
```

如果断言失败，程序会中断执行，会显示下面的提示。

```c
Assertion failed: (z >= 0), function main, file /Users/assert.c, line 14.
```

上面报错的格式如下。

```c
Assertion failed: [expression], function [abc], file [xyz], line [nnn].
```

上面代码中，方括号的部分使用实际数据替换掉。

使用 assert() 有几个好处：它不仅能自动标识文件和出问题的行号，还有一种无需更改代码就能开启或关闭 assert() 的机制。如果已经确认程序没有问题，不需要再做断言，就在`#include <assert.h>`语句的前面，定义一个宏`NDEBUG`。

```c
#define NDEBUG
#include <assert.h>
```

然后，重新编译程序，编译器就会禁用文件中所有的 assert() 语句。如果程序又出现问题，可以移除这条`#define NDBUG`指令（或者把它注释掉），再次编译，这样就重新启用了 assert() 语句。

assert() 的缺点是，因为引入了额外的检查，增加了程序的运行时间。

## static_assert()

C11 引入了静态断言`static_assert()`，用于在编译阶段进行断言判断。

```c
static_assert(constant-expression, string-literal);
```

`static_assert()`接受两个参数，第一个参数`constant-expression`是一个常量表达式，第二个参数`string-literal`是一个提示字符串。如果第一个参数的值为false，会产生一条编译错误，第二个参数就是错误提示信息。

```c
static_assert(sizeof(int) == 4, "64-bit code generation is not supported.");
```

上面代码的意思是，如果当前计算机的`int`类型不等于4个字节，就会编译报错。

注意，`static_assert()`只在编译阶段运行，无法获得变量的值。如果对变量进行静态断言，就会导致编译错误。

```c
int positive(const int n) {
  static_assert(n > 0, "value must > 0");
  return 0;
}
```

上面代码会导致编译报错，因为编译时无法知道变量`n`的值。

`static_assert()`的好处是，尽量在编译阶段发现错误，避免运行时再报错，节省开发时间。另外，有些`assert()`断言位于函数之中，如果不执行该函数，就不会报错，而`static_assert()`不管函数是否执行，都会进行断言判断。最后，`static_assert()`不会生成可执行代码，所以不会造成任何运行时的性能损失。

# ctype.h

`ctype.h`头文件定义了一系列字符处理函数的原型。

## 字符测试函数

这些函数用来判断字符是否属于某种类型。

- `isalnum()`：是否为字母数字
- `isalpha()`：是否为字母
- `isdigit()`：是否为数字
- `isxdigit()`：是否为十六进制数字符
- `islower()`：是否为小写字母
- `isupper()`：是否为大写字母
- `isblank()`：是否为标准的空白字符（包含空格、水平制表符或换行符）
- `isspace()`：是否为空白字符（空格、换行符、换页符、回车符、垂直制表符、水平制表符等）
- `iscntrl()`：是否为控制字符，比如 Ctrl + B
- `isprint()`：是否为可打印字符
- `isgraph()`：是否为空格以外的任意可打印字符
- `ispunct()`：是否为标点符号（除了空格、字母、数字以外的可打印字符）

它们接受一个待测试的字符作为参数。注意，参数类型为`int`，而不是`char`，因为它们允许 EOF 作为参数。

如果参数字符属于指定类型，就返回一个非零整数（通常是`1`，表示为真），否则返回`0`（表示为伪）。

下面是一个例子，用户输入一个字符，程序判断是否为英文字母。

```c
#include <stdio.h>
#include <ctype.h>

int main(void) {
  char ch = getchar();

  if (isalpha(ch))
    printf("it is an alpha character.\n");
  else
    printf("it is not an alpha character.\n");

  return 0;
}
```

## 字符映射函数

这一类函数返回字符的某种对应形式，主要有两个函数。

- `tolower()`：如果参数是大写字符，返回小写字符，否则返回原始参数。
- `toupper()`：如果参数是小写字符，返回大写字符，否则返回原始参数。

```c
// 将字符转为大写
ch = toupper(ch);
```

注意，这两个函数不会改变原始字符。

# errno.h

## errno 变量

`errno.h`声明了一个 int 类型的 errno 变量，用来存储错误码（正整数）。

如果这个变量有非零值，表示已经执行的程序发生了错误。

```c
int x = -1;

errno = 0;

int y = sqrt(x);

if (errno != 0) {
  fprintf(stderr, "sqrt error; program terminated.\n");
  exit(EXIT_FAILURE);
}
```

上面示例中，计算一个负值的平方根是不允许的，会导致`errno`不等于`0`。

如果要检查某个函数是否发生错误，必须在即将调用该函数之前，将`errno`的值置为0，防止其他函数改变`errno`的值。

## 宏

变量`errno`的值通常是两个宏`EDOM`或`ERANGE`。这两个宏都定义在`errno.h`。它们表示调用数学函数时，可能发生的两种错误。

- 定义域错误（EDOM）：传递给函数的一个参数超出了函数的定义域。例如，负数传入`sqrt()`作为参数。
- 取值范围错误（ERANGE）：函数的返回值太大，无法用返回类型表示。例如，1000 传入`exp()`作为参数，因为 e^1000 太大，无法使用 double 类型表示。

使用数学函数时，可以将`errno`的值与 EDOM 和 ERANGE 比较，用来确定到底发生了哪一类错误。

# float.h

`float.h`定义了浮点数类型 float、double、long double 的一些宏，规定了这些类型的范围和精度。

(1) `FLT_ROUNDS`

宏`FLT_ROUNDS`表示当前浮点数加法的四舍五入方向。

它有以下可能的值。

- -1：不确定。
- 0：向零舍入。
- 1：向最近的整数舍入。
- 2：向正无穷方向舍入。
- 3：向负无穷方向舍入。

（2）`FLT_RADIX`

宏`FLT_RADIX`表示科学计数法的指数部分的底（base），一般总是2。

（3）浮点数类型的最大值

- `FLT_MAX`
- `DBL_MAX`
- `LDBL_MAX`

（4）浮点数类型的最小正值

- `FLT_MIN`
- `DBL_MIN`
- `LDBL_MIN`

（5）两个同类型浮点数之间可表示的最小差值（最小精度）

- `FLT_EPSILON`
- `DBL_EPSILON`
- `LDBL_EPSILON`

（6）`DECIMAL_DIG`

宏`DECIMAL_DIG`表示十进制有效位数。

（7）`FLT_EVAL_METHOD`

宏`FLT_EVAL_METHOD`表示浮点数运算时的类型转换。

它可能有以下值。

- -1：不确定。
- 0：在当前类型中运算。
- 1：float 和 double 类型的运算使用 double 类型的范围和精度求值。
- 2：所有浮点数类型的运算使用 long double 类型的范围和精度求值。

（8）浮点数尾数部分的个数

- `FLT_MANT_DIG`
- `DBL_MANT_DIG`
- `LDBL_MANT_DIG`

（9）浮点数指数部分有效数字的个数（十进制）

- `FLT_DIG`
- `DBL_DIG`
- `LDBL_DIG`

（10）科学计数法的指数部分的最小次幂（负数）

- `FLT_MIN_EXP`
- `DBL_MIN_EXP`
- `LDBL_MIN_EXP`

（11）科学计数法的指数部分的十进制最小次幂（负数）

- `FLT_MIN_10_EXP`
- `DBL_MIN_10_EXP`
- `LDBL_MIN_10_EXP`

（12）科学计数法的指数部分的最大次幂

- `FLT_MAX_EXP`
- `DBL_MAX_EXP`
- `LDBL_MAX_EXP`

科学计数法的指数部分的十进制最大次幂

- `FLT_MAX_10_EXP`
- `DBL_MAX_10_EXP`
- `LDBL_MAX_10_EXP`

# inttypes.h

C 语言还在头文件 inttypes.h 里面，为 stdint.h 定义的四类整数类型，提供了`printf()`和`scanf()`的占位符。

- 固定宽度整数类型，比如 int8_t。
- 最小宽度整数类型，比如 int_least8_t。
- 最快最小宽度整数类型，比如 int_fast8_t。
- 最大宽度整数类型，比如 intmax_t。

`printf()`的占位符采用`PRI + 原始占位符 + 类型关键字/宽度`的形式构成。举例来说，原始占位符为`%d`，则对应的占位符如下。

- PRIdn （固定宽度类型）
- PRIdLEASTn （最小宽度类型）
- PRIdFASTn （最快最小宽度类型）
- PRIdMAX （最大宽度类型）

上面占位符中的`n`，可以用8、16、32、64代入。

下面是用法示例。

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
  int_least16_t x = 3490;
  printf("The value is %" PRIdLEAST16 "!\n", x);
}
```

上面示例中，`PRIdLEAST16`对应的整数类型为 int_least16_t，原始占位符为`%d`。另外，`printf()`的第一个参数用到了多个字符串自动合并的写法。

下面是其它的原始占位符对应的占位符。

- %i：PRIin    PRIiLEASTn    PRIiFASTn    PRIiMAX
- %o：PRIon    PRIoLEASTn    PRIoFASTn    PRIoMAX
- %u：PRIun    PRIuLEASTn    PRIuFASTn    PRIuMAX
- %x：PRIxn    PRIxLEASTn    PRIxFASTn    PRIxMAX
- %X：PRIXn    PRIXLEASTn    PRIXFASTn    PRIXMAX

`scanf()`的占位符规则也与之类似。

- %d：SCNdn    SCNdLEASTn    SCNdFASTn    SCNdMAX
- %i：SCNin    SCNiLEASTn    SCNiFASTn    SCNiMAX
- %o：SCNon    SCNoLEASTn    SCNoFASTn    SCNoMAX
- %u：SCNun    SCNuLEASTn    SCNuFASTn    SCNuMAX
- %x：SCNxn    SCNxLEASTn    SCNxFASTn    SCNxMAX

# iso646.h

`iso646.h`头文件指定了一些常见运算符的替代拼写。比如，它用关键字`and`代替逻辑运算符`&&`。

```c
if (x > 6 and x < 12)
// 等同于
if (x > 6 && x < 12)
```

它定义的替代拼写如下。

- and	替代 &&
- and_eq 替代 &=
- bitand 替代 &
- bitor 替代 |
- compl 替代 ~
- not 替代 !
- not_eq 替代 !=
- or 替代 ||
- or_eq 替代 |=
- xor 替代 ^
- xor_eq 替代 ^=


# limits.h

`limits.h`提供了用来定义各种整数类型（包括字符类型）取值范围的宏。

- `CHAR_BIT`：每个字符包含的二进制位数。
- `SCHAR_MIN`：signed char 类型的最小值。
- `SCHAR_MAX`：signed char 类型的最大值。
- `UCHAR_MAX`：unsiged char 类型的最大值。
- `CHAR_MIN`：char 类型的最小值。
- `CHAR_MAX`：char 类型的最大值。
- `MB_LEN_MAX`：多字节字符最多包含的字节数。
- `SHRT_MIN`：short int 类型的最小值。
- `SHRT_MAX`：short int 类型的最大值。
- `USHRT_MAX`：unsigned short int 类型的最大值。
- `INT_MIN`：int 类型的最小值。
- `INT_MAX`：int 类型的最大值。
- `UINT_MAX`：unsigned int 类型的最大值。
- `LONG_MIN`：long int 类型的最小值。
- `LONG_MAX`：long int 类型的最大值。
- `ULONG_MAX`：unsigned long int 类型的最大值。
- `LLONG_MIN`：long long int 类型的最小值。
- `LLONG_MAX`：long long int 类型的最大值。
- `ULLONG_MAX`：unsigned long long int 类型的最大值。

下面的示例是使用预处理指令判断，int 类型是否可以用来存储大于 100000 的数。

```c
#if INT_MAX < 100000
  #error int type is too small
#endif
```

上面示例中，如果 int 类型太小，预处理器会显示一条出错消息。

可以使用`limit.h`里面的宏，为类型别名选择正确的底层类型。

```c
#if INT_MAX >= 100000
  typedef int Quantity;
#else
  typedef long int Quantity;
#endif
```

上面示例中，如果整数类型的最大值（`INT_MAX`）不小于100000，那么类型别名`Quantity`指向`int`，否则就指向`long int`。

# locale.h

## 简介

`locale.h`是程序的本地化设置，主要影响以下的行为。

- 数字格式
- 货币格式
- 字符集
- 日期和时间格式

它设置了以下几个宏。

- LC_COLLATE：影响字符串比较函数`strcoll()`和`strxfrm()`。
- LC_CTYPE：影响字符处理函数的行为。
- LC_MONETARY：影响货币格式。
- LC_NUMERIC：影响`printf()`的数字格式。
- LC_TIME：影响时间格式`strftime()`和`wcsftime()`。
- LC_ALL：将以上所有类别设置为给定的语言环境。

## setlocale()

`setlocale()`用来设置当前的地区。

```c
char* setlocal(int category, const char* locale);
```

它接受两个参数。第一个参数表示影响范围，如果值为前面五个表示类别的宏之一，则只影响该宏对应的类别，如果值为`LC_ALL`，则影响所有类别。第二个参数通常只为`"C"`（正常模式）或`""`（本地模式）。

任意程序开始时，都隐含下面的调用。

```c
setlocale(LC_ALL, "C");
```

下面的语句将格式本地化。

```c
set_locale(LC_ALL, "");
```

上面示例中，第二个参数为空字符，表示使用当前环境提供的本地化设置。

理论上，第二个参数也可以设为当前系统支持的某种格式。

```c
setlocale(LC_ALL, "en_US.UTF-8");
```

但是这样的话，程序的可移植性就变差了，因为无法保证其他系统也会支持那种格式。所以，通常都将第二个参数设为空字符串，使用操作系统的当前设置。

`setlocale()`的返回值是一个字符串指针，表示已经设置好的格式。如果调用失败，则返回空指针 NULL。

`setlocale()`可以用来查询当前地区，这时第二个参数设为 NULL 就可以了。

```c
char *loc;

loc = setlocale(LC_ALL, NULL);

// 输出 Starting locale: C
printf("Starting locale: %s\n", loc);

loc = setlocale(LC_ALL, "");

// 输出 Native locale: en_US.UTF-8    
printf("Native locale: %s\n", loc);
```

## localeconv()

`localeconv()`用来获取当前格式的详细信息。

```c
struct lconv* localeconv(void);
```

该函数返回一个 Struct 结构指针，该结构里面包含了格式信息，它的主要属性如下。

- char* mon_decimal_point：货币的十进制小数点字符，比如`.`。
- char* mon_thousands_sep：货币的千位分隔符，比如`,`。
- char* mon_grouping：货币的分组描述符。
- char* positive_sign：货币的正值符号，比如`+`或为空字符串。
- char* negative_sign：货币的负值符号，比如`-`。
- char* currency_symbol：货币符号，比如`$`。
- char frac_digits：打印货币金额时，十进制小数点后面输出几位小数，比如设为`2`。
- char p_cs_precedes：设为`1`时，货币符号`currency_symbol`出现在非负金额前面。设为`0`时，出现在后面。
- char n_cs_precedes：设为`1`时，货币符号`currency_symbol`出现在负的货币金额前面。设为`0`时，出现在后面。
- char p_sep_by_space：决定了非负的货币金额与货币符号之间的分隔字符。
- char n_sep_by_space：决定了负的货币金额与货币符号之间的分隔字符。
- char p_sign_posn：决定了非负值的正值符号的位置。
- char n_sign_posn：决定了负值的负值符号的位置。
- char* int_curr_symbol：货币的国际符号，比如`USD`。
- char int_frac_digits：使用国际符号时，`frac_digits`的值。
- char int_p_cs_precedes：使用国际符号时，`p_cs_precedes`的值。
- char int_n_cs_precedes：使用国际符号时，`n_cs_precedes`的值。
- char int_p_sep_by_space：使用国际符号时，`p_sep_by_space`的值。
- char int_n_sep_by_space：使用国际符号时，`n_sep_by_space`的值。
- char int_p_sign_posn：使用国际符号时，`p_sign_posn`的值。
- char int_n_sign_posn：使用国际符号时，`n_sign_posn`的值。

下面程序打印当前系统的属性值。

```c
#include <stdio.h>
#include <locale.h>
#include <string.h>

int main ()
{
    setlocale (LC_ALL,"zh_CN");
    struct lconv * lc;
    lc=localeconv();
    printf ("decimal_point: %s\n",lc->decimal_point);
    printf ("thousands_sep: %s\n",lc->thousands_sep);
    printf ("grouping: %s\n",lc->grouping);
    printf ("int_curr_symbol: %s\n",lc->int_curr_symbol);
    printf ("currency_symbol: %s\n",lc->currency_symbol);
    printf ("mon_decimal_point: %s\n",lc->mon_decimal_point);
    printf ("mon_thousands_sep: %s\n",lc->mon_thousands_sep);
    printf ("mon_grouping: %s\n",lc->mon_grouping);
    printf ("positive_sign: %s\n",lc->positive_sign);
    printf ("negative_sign: %s\n",lc->negative_sign);
    printf ("frac_digits: %d\n",lc->frac_digits);
    printf ("p_cs_precedes: %d\n",lc->p_cs_precedes);
    printf ("n_cs_precedes: %d\n",lc->n_cs_precedes);
    printf ("p_sep_by_space: %d\n",lc->p_sep_by_space);
    printf ("n_sep_by_space: %d\n",lc->n_sep_by_space);
    printf ("p_sign_posn: %d\n",lc->p_sign_posn);
    printf ("n_sign_posn: %d\n",lc->n_sign_posn);
    printf ("int_frac_digits: %d\n",lc->int_frac_digits);
    printf ("int_p_cs_precedes: %d\n",lc->int_p_cs_precedes);
    printf ("int_n_cs_precedes: %d\n",lc->int_n_cs_precedes);
    printf ("int_p_sep_by_space: %d\n",lc->int_p_sep_by_space);
    printf ("int_n_sep_by_space: %d\n",lc->int_n_sep_by_space);
    printf ("int_p_sign_posn: %d\n",lc->int_p_sign_posn);
    printf ("int_n_sign_posn: %d\n",lc->int_n_sign_posn);
   
    return 0;
}
```

# math.h

`math.h`头文件提供了很多数学函数。

很多数学函数的返回值是 double 类型，但是同时提供 float 类型与 long double 类型的版本，比如`pow()`函数就还有`powf()`和`powl()`版本。

```c
double      pow(double x, double y); 
float       powf(float x, float y);
long double powl(long double x, long double y);
```

为了简洁，下面就略去了函数的`f`后缀（float 类型）和`l`后缀（long double）版本。

## 类型和宏

math.h 新定义了两个类型别名。

- float_t：（当前系统）最有效执行 float 运算的类型，宽度至少与 float 一样。
- double_t`：（当前系统）最有效执行 double 运算的类型，宽度至少与 double 一样。

它们的具体类型可以通过宏`FLT_EVAL_METHOD`来了解。

| FLT_EVAL_METHOD 的值 |	float_t 对应的类型 | double_t 对应的类型 |
|-----------------|--------------|--------------|
| 0	| float	| double |
| 1	| double | double |
| 2	| long double	| long double |
| 其他 | 由实现决定 |	由实现决定 |

math.h 还定义了一些宏。

- `INFINITY`：表示正无穷，返回一个 float 类型的值。
- `NAN`：表示非数字（Not-A-Number），返回一个 float 类型的值。

## 错误类型

数学函数的报错有以下类型。

- Range errors：运算结果不能用函数返回类型表示。
- Domain errors：函数参数不适用当前函数。
- Pole errors：参数导致函数的极限值变成无限。
- Overflow errors：运算结果太大，导致溢出。
- Underflow errors：运算结果太小，导致溢出。

变量`math_errhandling`提示了当前系统如何处理数学运算错误。

| math_errhandling 的值 | 描述 |
|-----------------------|-----|
| MATH_ERRNO | 系统使用 errno 表示数学错误 |
| MATH_ERREXCEPT | 系统使用异常表示数学错误 |
| MATH_ERRNO | MATH_ERREXCEPT	| 系统同时使用两者表示数学错误 |

## 数值类型

数学函数的参数可以分成以下几类：正常值，无限值，有限值和非数字。

下面的函数用来判断一个值的类型。

- fpclassify()：返回给定浮点数的分类。
- isfinite()：如果参数不是无限或 NaN，则为真。
- isinf()：如果参数是无限的，则为真。
- isnan()：如果参数不是数字，则为真。
- isnormal()：如果参数是正常数字，则为真。

下面是一个例子。

```c
isfinite(1.23)    // 1
isinf(1/tan(0))   // 1
isnan(sqrt(-1))   // 1
isnormal(1e-310)) // 0
```

## signbit()

`signbit()`判断参数是否带有符号。如果参数为负值，则返回1，否则返回0。

```c
signbit(3490.0) // 0
signbit(-37.0)  // 1
```

## 三角函数

以下是三角函数，参数为弧度值。

- acos()：反余弦。
- asin()：反正弦。
- atan()：反正切
- atan2()：反正切。
- cos()：余弦。
- sin()：正弦。
- tan()：正切。

不要忘了，上面所有函数都有 float 版本（函数名加上 f 后缀）和 long double 版本（函数名加上 l 后缀）。

下面是一个例子。

```c
cos(PI/4) // 0.707107
```

## 双曲函数

以下是双曲函数，参数都为浮点数。

- acosh()：反双曲余弦。 
- asinh()：反双曲正弦。
- atanh()：反双曲正切。
- cosh()：双曲余弦。
- tanh()：双曲正切。
- sinh()：双曲正弦。 

## 指数函数和对数函数

以下是指数函数和对数函数，它们的返回值都是 double 类型。

- exp()：计算欧拉数 e 的乘方，即 e<sup>x</sup>。
- exp2()：计算 2 的乘方，即 2<sup>x</sup>。
- expm1()：计算 e<sup>x</sup> - 1。
- log()：计算自然对数，`exp()`的逆运算。
- log2()：计算以2为底的对数。
- log10()：计算以10为底的对数。
- logp1()：计算一个数加 1 的自然对数，即`ln(x + 1)`。
- logb()：计算以宏`FLT_RADIX`（一般为2）为底的对数，但只返回整数部分。

下面是一些例子。

```c
exp(3.0) // 20.085500
log(20.0855) // 3.000000
log10(10000) // 3.000000
```

如果结果值超出了 C 语言可以表示的最大值，函数将返回`HUGE_VAL`，它是一个在`math.h`中定义的 double 类型的值。

如果结果值太小，无法用 double 值表示，函数将返回0。以上这两种情况都属于出错。

## frexp()

`frexp()`将参数分解成浮点数和指数部分（2为底数），比如 1234.56 可以写成 0.6028125 * 2<sup>11</sup>，这个函数就能分解出 0.6028125 和 11。

```c
double frexp(double value, int* exp);
```

它接受两个参数，第一个参数是用来分解的浮点数，第二个参数是一个整数变量指针。

它返回小数部分，并将指数部分放入变量`exp`。如果参数为`0`，则返回的小数部分和指数部分都为`0`。

下面是一个例子。

```c
double frac;
int expt;

// expt 的值是 11
frac = frexp(1234.56, &expt);

// 输出 1234.56 = 0.6028125 x 2^11
printf("1234.56 = %.7f x 2^%d\n", frac, expt);
```

## ilogb()

`ilogb()`返回一个浮点数的指数部分，指数的基数是宏`FLT_RADIX`（一般是`2`）。

```c
int ilogb(double x);
```

它的参数为`x`，返回值是 log<sub>r</sub>|x|，其中`r`为宏`FLT_RADIX`。

下面是用法示例。

```c
ilogb(257) // 8
ilogb(256) // 8
ilogb(255) // 7
```

## ldexp()

`ldexp()`将一个数乘以2的乘方。它可以看成是`frexp()`的逆运算，将小数部分和指数部分合成一个`f * 2^n`形式的浮点数。

```c
double ldexp(double x, int exp);
```

它接受两个参数，第一个参数是乘数`x`，第二个参数是2的指数部分`exp`，返回“x * 2<sup>exp</sup>”。

```c
ldexp(1, 10) // 1024.000000
ldexp(3, 2) // 12.000000
ldexp(0.75, 4) // 12.000000
ldexp(0.5, -1) // 0.250000
```

## modf() 

`modf()`函数提取一个数的整数部分和小数部分。

```c
 double modf(double value, double* iptr);
 ```

它接受两个参数，第一个参数`value`表示待分解的数值，第二个参数是浮点数变量`iptr`。返回值是`value`的小数部分，整数部分放入变量`double`。

下面是一个例子。

```c
// int_part 的值是 3.0
modf(3.14159, &int_part); // 返回 0.14159
```

## scalbn() 

`scalbn()`用来计算“x * r<sup>n</sup>”，其中`r`是宏`FLT_RADIX`。

```c
double scalbn(double x, int n);
```

它接受两个参数，第一个参数`x`是乘数部分，第二个参数`n`是指数部分，返回值是“x * r<sup>n</sup>”。

下面是一些例子。

```c
scalbn(2, 8) // 512.000000
```

这个函数有多个版本。

- scalbn()：指数 n 是 int 类型。
- scalbnf()：float 版本的 scalbn()。
- scalbnl()：long double 版本的 scalbn()。
- scalbln()：指数 n 是 long int 类型。
- scalblnf()：float 版本的 scalbln()。
- scalblnl()：long double 版本的 scalbln()。

## round()

`round()`函数以传统方式进行四舍五入，比如`1.5`舍入到`2`，`-1.5`舍入到`-2`。

```c
double round(double x);
```

它返回一个浮点数。

下面是一些例子。

```c
round(3.14)  // 3.000000
round(3.5)   // 4.000000
round(-1.5)  // -2.000000
round(-1.14) // -1.000000
```

它还有一些其他版本。

- lround()：返回值是 long int 类型。
- llround()：返回值是 long long int 类型。

## trunc() 

`trunc()`用来截去一个浮点数的小数部分，将剩下的整数部分以浮点数的形式返回。

```c
double trunc(double x);
```

下面是一些例子。

```c
trunc(3.14)  // 3.000000
trunc(3.8)   // 3.000000
trunc(-1.5)  // -1.000000
trunc(-1.14) // -1.000000
```

## ceil()

`ceil()`返回不小于其参数的最小整数（double 类型），属于“向上舍入”。

```c
double ceil(double x);
```

下面是一些例子。

```c
ceil(7.1) // 8.0
ceil(7.9) // 8.0
ceil(-7.1) // -7.0
ceil(-7.9) // -7.0
```

## floor()

`floor()`返回不大于其参数的最大整数，属于“向下舍入”。

```c
double floor(double x);
```

下面是一些例子。

```c
floor(7.1) // 7.0
floor(7.9) // 7.0
floor(-7.1) // -8.0
floor(-7.9) // -8.0
```

下面的函数可以实现“四舍五入”。

```c
double round_nearest(double x) {
  return x < 0.0 ? ceil(x - 0.5) : floor(x + 0.5);
}
```

## fmod()

`fmod()`返回第一个参数除以第二个参数的余数，就是余值运算符`%`的浮点数版本，因为`%`只能用于整数运算。

```c
double fmod(double x, double y);
```

它在幕后执行的计算是`x - trunc(x / y) * y`，返回值的符号与`x`的符号相同。

```c
fmod(5.5, 2.2)  //  1.100000
fmod(-9.2, 5.1) // -4.100000
fmod(9.2, 5.1)  //  4.100000
```

## 浮点数比较函数

以下函数用于两个浮点数的比较，返回值的类型是整数。

- isgreater()：返回`x > y`的结果。
- isgreaterequal()：返回`x >= y`的结果。
- isless()：返回`x < y`的结果。
- islessequal()：返回`x <= y`的结果。
- islessgreater()：返回`(x < y) || (x > y)`的结果。

下面是一些例子。

```c
isgreater(10.0, 3.0)   // 1
isgreaterequal(10.0, 10.0)   // 1
isless(10.0, 3.0)  // 0
islessequal(10.0, 3.0)   // 0
islessgreater(10.0, 3.0)   // 1
islessgreater(10.0, 30.0)   // 1
islessgreater(10.0, 10.0)   // 0
```

## isunordered()

`isunordered()`返回两个参数之中，是否存在 NAN。

```c
int isunordered(any_floating_type x, any_floating_type y);
```

下面是一些例子。

```c
isunordered(1.0, 2.0)    // 0
isunordered(1.0, sqrt(-1))  // 1
isunordered(NAN, 30.0)  // 1
isunordered(NAN, NAN)   // 1
```

## 其他函数

下面是 math.h 包含的其它函数。

- pow()：计算参数`x`的`y`次方。
- sqrt()：计算一个数的平方根。
- cbrt()：计算立方根。
- fabs()：计算绝对值。
- hypot()：根据直角三角形的两条直角边，计算斜边。
- fmax()：返回两个参数之中的最大值。
- fmin()：返回两个参数之中的最小值。
- remainder()：返回 IEC 60559 标准的余数，类似于`fmod()`，但是余数范围是从`-y/2`到`y/2`，而不是从`0`到`y`。
- remquo()：同时返回余数和商，余数的计算方法与`remainder()`相同。
- copysign()：返回一个大小等于第一个参数、符号等于第二个参数的值。
- nan()：返回 NAN。  
- nextafter()：获取下一个（或者上一个，具体方向取决于第二个参数`y`）当前系统可以表示的浮点值。
- nextoward()：与`nextafter()`相同，除了第二个参数是 long double 类型。
- fdim()：如果第一个参数减去第二个参数大于`0`，则返回差值，否则返回`0`。
- fma()：以快速计算的方式，返回`x * y + z`的结果。
- nearbyint()：在当前舍入方向上，舍入到最接近的整数。当前舍入方向可以使用`fesetround()`函数设定。
- rint()：在当前舍入方向上，舍入到最接近的整数，与`nearbyint()`相同。不同之处是，它会触发浮点数的`INEXACT`异常。
- lrint()：在当前舍入方向上，舍入到最接近的整数，与`rint()`相同。不同之处是，返回值是一个整数，而不是浮点数。
- erf()：计算一个值的误差函数。
- erfc()：计算一个值的互补误差函数。
- tgamma()：计算 Gamma 函数。
- lgamma()：计算 Gamma 函数绝对值的自然对数。

下面是一些例子。

```c
pow(3, 4) // 81.000000
sqrt(3.0) // 1.73205
cbrt(1729.03) // 12.002384
fabs(-3490.0) // 3490.000000
hypot(3, 4) // 5.000000
fmax(3.0, 10.0) // 10.000000
fmin(10.0, 3.0) //  3.000000
```

# signal.h

## 简介

`signal.h`提供了信号（即异常情况）的处理工具。所谓“信号”（signal），可以理解成系统与程序之间的短消息，主要用来表示运行时错误，或者发生了异常事件。

头文件`signal.h`定义了一系列宏，表示不同的信号。

- SIGABRT：异常中止（可能由于调用了 abort() 方法）。
- SIGFPE：算术运算发生了错误（可能是除以 0 或者溢出）。
- SIGILL：无效指令。
- SIGINT：中断。
- SIGSEGV：无效内存访问。
- SIGTERM：终止请求。

上面每个宏的值都是一个正整数常量。

## signal()

头文件`signal.h`还定义了一个`signal()`函数，用来指定某种信号的处理函数。

```c
signal(SIGINT, handler);
```

`signal()`接受两个参数，第一个参数是某种信号的宏，第二个参数是处理这个信号的函数指针`handler`。

信号处理函数`handler`接受一个 int 类型的参数，表示信号类型。它的原型如下。

```c
void (*func)(int);
```

`handler`函数体内部可以根据这个整数，判断到底接受到了哪种信号，因为多个信号可以共用同一个处理函数。一旦处理函数执行完成，程序会从信号发生点恢复执行。但是，如果遇到 SIGABRT 信号，处理函数执行完成，系统会让程序中止。

当系统向程序发送信号时，程序可以忽略信号，即不指定处理函数。

`signal()`的返回值是前一个处理函数的指针，常常把它保存在变量之中，当新的处理函数执行完，再恢复以前的处理函数。

```c
void (*orig_handler)(int);
orig_handler = signal(SIGINT, handler);
// SIGINT 信号发生之后
signal(SIGINT, orig_handler);
```

上面示例中，`signal()`为信号`SIGINT`指定了新的处理函数`handler`，把原来的处理函数保存在变量`orig_handler`里面。等到`handler`这个函数用过之后，再恢复原来的处理函数。

## 信号相关的宏

`signal.h`还提供了信号相关的宏。

（1）SIG_DFL

SIG_DFL 表示默认的处理函数。

```c
signal(SIGINT, SIG_IGN);
```

上面示例中，SIGINT 的处理函数是默认处理函数，由当前实现决定。

（2）SIG_IGN

SIG_IGN 表示忽略该信号。

```c
signal(SIGINT, SIG_IGN);
```

上面示例表示不对 SIGINT 信号进行处理。由于程序运行时按下 Ctrl + c 是发出 SIGINT 信号，所以使用该语句后，程序无法用 Ctrl + c 终止。

（3）SIG_ERR

SIG_ERR 是信号处理函数发生错误时，`signal()`的返回值。

```c
if (signal(SIGINT, handler) == SIG_ERR) {
  perror("signal(SIGINT, handler) failed");
  // ...
}
```

上面示例可以判断`handler`处理 SIGINT 时，是否发生错误。

## raise()

`raise()`函数用来在程序中发出信号。

```c
int raise(int sig);
```

它接受一个信号值作为参数，表示发出该信号。它的返回值是一个整数，可以用来判断信号发出是否成功，0 表示成功，非 0 表示失败。

```c
void handler(int sig) {
  printf("Handler called for signal %d\n", sig);
}

signal(SIGINT, handler);
raise(SIGINT);
```

上面示例中，`raise()`触发 SIGINT 信号，导致 handler 函数执行。

# stdarg.h

`stdarg.h`定义于函数的可变参数相关的一些方法。

- va_list 类型
- va_start()
- va_arg()：获取当前参数
- va_end()。

va_copy()：it makes a copy of your va_list variable in the exact same state.
va_copy() can be useful if you need to scan ahead through the arguments but need to also remember your current place.

接受可变函数作为参数的一些方法。

- vprintf()
- vfprintf()
- vsprintf()
- vsnprintf()

```c
#include <stdio.h>
#include <stdarg.h>

int my_printf(int serial, const char *format, ...)
{
    va_list va;

    // Do my custom work
    printf("The serial number is: %d\n", serial);

    // Then pass the rest off to vprintf()
    va_start(va, format);
    int rv = vprintf(format, va);
    va_end(va);

    return rv;
}

int main(void)
{
    int x = 10;
    float y = 3.2;

    my_printf(3490, "x is %d, y is %f\n", x, y);
}
```

# stdbool.h

`stdbool.h`头文件定义了4个宏。

- `bool`：定义为`_Bool`。
- `true`：定义为1。
- `false`：定义为0。
- `__bool_true_false_are_defined`：定义为1。

```c
bool isEven(int number) {
  if (number % 2) {
    return true;
  } else {
    return false;
  }
}
```

```c
#include <stdio.h>
#include <stdbool.h>

int main(void) {
  unsigned long num;
  unsigned long div;
  bool isPrime = true;

  num = 64457;

  for (div = 2; (div * div) <= num; div++) {
   if (num % div == 0) isPrime = false;
  }

  if (isPrime) {
    printf("%lu is prime.\n", num);
  } else {
    printf("%lu is not prime.\n", num);
  }

  return 0;
}
```

# stddef.h

`stddef.h`提供了常用类型和宏的定义，但没有声明任何函数。

这个头文件定义的类型如下。

- ptrdiff_t：指针相减运算时，返回结果的数据类型。
- size_t：`sizeof`运算符返回的类型。
- wchar_t：一种足够大、能容纳各种字符的类型。

以上三个类型都是整数类型，其中`ptrdiff_t`是有符号整数，`size_t`是无符号整数。

`stddef.h`定义了两个宏。

- NULL：空指针。
- offsetof()

## offsetof()

`offsetof()`是`stddef.h`定义的一个宏，用来返回某个属性在 Struct 结构内部的起始位置。由于系统为了字节对齐，可能会在 Struct 结构的属性之间插入空字节，这个宏对于确定某个属性的内存位置很有用。

它是一个带参数的宏，接受两个参数。第一个参数是 Struct 结构，第二个参数是该结构的一个属性，返回 Struct 起始位置到该属性之间的字节数。

```c
struct s {
  char a;
  int b[2];
  float c;
};

printf("%zu\n", offsetof(struct s, a)); // 0
printf("%zu\n", offsetof(struct s, b)); // 4
printf("%zu\n", offsetof(struct s, c)); // 12
```

对于上面这个 Struct 结构，`offsetof(struct s, a)`一定等于`0`，因为`a`属性是第一个属性，与 Struct 结构自身的地址相同。

系统为了字节对齐，在`a`属性后面分配了3个空字节，导致`b`属性存储在第4个字节，所以`offsetof(struct s, b)`和`offsetof(struct s, c)`分别是4和12。

# stdint.h

## 固定宽度的整数类型

stdint.h 定义了一些固定宽度的整数类型别名，主要有下面三类。

- 宽度完全确定的整数`intN_t`，比如`int32_t`。
- 宽度不小少于某个大小的整数`int_leastN_t`，比如`int_least8_t`。
- 宽度不小于某个大小、并且处理速度尽可能快的整数`int_fastN_t`，比如`int_fast64_t`。

上面所有类型都是有符号的，类型名前面可以加一个前缀`u`，表示无符号类型，比如`uint16_t`。

C 语言标准要求至少定义以下类型。

- int8_t      uint8_t
- int16_t     uint16_t
- int32_t     uint32_t
- int64_t     uint64_t
- int_least8_t      uint_least8_t
- int_least16_t     uint_least16_t
- int_least32_t     uint_least32_t
- int_least64_t     uint_least64_t
- int_fast8_t       uint_fast8_t
- int_fast16_t      uint_fast16_t
- int_fast32_t      uint_fast32_t
- int_fast64_t      uint_fast64_t

## 最大宽度的整数类型

以下两个类型表示当前系统可用的最大宽度整数。

- intmax_t
- uintmax_t

如果想要尽可能大的整数时，可以使用上面类型。

## 固定宽度的整数常量

以下一些带参数的宏，可以生成固定宽度的整数常量。

- INT8_C(x)     UINT8_C(x)
- INT16_C(x)    UINT16_C(x)
- INT32_C(x)    UINT32_C(x)
- INT64_C(x)    UINT64_C(x)
- INTMAX_C(x)   UINTMAX_C(x)

下面是用法示例。

```c
uint16_t x = UINT16_C(12);
intmax_t y = INTMAX_C(3490);
```

## 固定宽度的整数极限值

下面一些宏代表了固定宽度的整数最大值和最小值。

- INT8_MAX           INT8_MIN           UINT8_MAX
- INT16_MAX          INT16_MIN          UINT16_MAX
- INT32_MAX          INT32_MIN          UINT32_MAX
- INT64_MAX          INT64_MIN          UINT64_MAX
- INT_LEAST8_MAX     INT_LEAST8_MIN     UINT_LEAST8_MAX
- INT_LEAST16_MAX    INT_LEAST16_MIN    UINT_LEAST16_MAX
- INT_LEAST32_MAX    INT_LEAST32_MIN    UINT_LEAST32_MAX
- INT_LEAST64_MAX    INT_LEAST64_MIN    UINT_LEAST64_MAX
- INT_FAST8_MAX      INT_FAST8_MIN      UINT_FAST8_MAX
- INT_FAST16_MAX     INT_FAST16_MIN     UINT_FAST16_MAX
- INT_FAST32_MAX     INT_FAST32_MIN     UINT_FAST32_MAX
- INT_FAST64_MAX     INT_FAST64_MIN     UINT_FAST64_MAX
- INTMAX_MAX         INTMAX_MIN         UINTMAX_MAX

注意，所有无符号整数类型的最小值都为0，所以没有对应的宏。

## 占位符

C 语言还在头文件 inttypes.h 里面，为上面类型定义了`printf()`和`scanf()`的占位符，参见《inttypes.h》一章。

# stdio.h

`stdio.h`是 C 语言的标准 I/O 库，用于读取和写入文件，也用于控制台的输入和输出。

## 标准 I/O 函数

以下函数用于控制台的输入和输出。

- printf()：输出到控制台，详见《基本语法》一章。
- scanf()：从控制台读取输入，详见《I/O 函数》一章。
- getchar()：从控制台读取一个字符，详见《I/O 函数》一章。
- putchar()：向控制台写入一个字符，详见《I/O 函数》一章。
- gets()：从控制台读取整行输入（已废除），详见《I/O 函数》一章。
- puts()：向控制台写入一个字符串，详见《I/O 函数》一章。

## 文件操作函数

以下函数用于文件操作，详见《文件操作》一章。

- fopen()：打开文件。
- fclose()：关闭文件。
- freopen()：打开一个新文件，关联一个已经打开的文件指针。
- fprintf()：输出到文件。
- fscanf()：从文件读取数据。
- getc()：从文件读取一个字符。
- fgetc()：从文件读取一个字符。
- putc()：向文件写入一个字符。
- fputc()：向文件写入一个字符。
- fgets()：从文件读取整行。
- fputs()：向文件写入字符串。
- fread()：从文件读取二进制数据。
- fwrite()：向文件写入二进制数据。
- fseek()：将文件内部指针移到指定位置。
- ftell()：获取文件内部指针的当前位置。
- rewind()：将文件内部指针重置到文件开始处。
- fgetpos()：获取文件内部指针的当前位置。
- fsetpos()：设置文件内部指针的当前位置。
- feof()：判断文件内部指针是否指向文件结尾。
- ferror()：返回文件错误指示器的状态。
- clearerr()：重置文件错误指示器。
- remove()：删除文件。
- rename()：文件改名，以及移动文件。

## 字符串操作函数

以下函数用于操作字符串，详见《字符串操作》一章。

- sscanf()：从字符串读取数据，详见《I/O 函数》一章。
- sprintf()：输出到字符串。
- snprintf()：输出到字符串的更安全版本，指定了输出字符串的数量。

## tmpfile()

`tmpfile()`函数创建一个临时文件，该文件只在程序运行期间存在，除非手动关闭它。它的原型如下。

```c
FILE* tmpfile(void);
```

`tmpfile()`返回一个文件指针，可以用于访问该函数创建的临时文件。如果创建失败，返回一个空指针 NULL。

```c
FILE* tempptr;
tempptr = tmpfile();
```

调用`close()`方法关闭临时文件后，该文件将被自动删除。

`tmpfile()`有两个缺点。一是无法知道临时文件的文件名，二是无法让该文件成为永久文件。

## tmpnam()

`tmpname()`函数为临时文件生成一个名字，确保不会与其他文件重名。它的原型如下。

```c
char* tmpname(char* s);
```

它的参数是一个字符串变量，`tmpnam()`会把临时文件的文件名复制到这个变量里面，并返回指向该字符串变量的指针。如果生成文件名失败，`tmpnam()`返回空指针 NULL。

```c
char filename[L_tmpname];

if (tmpnam(filename) != NULL)
  // 输出诸如 /tmp/filew9PMuZ 的文件名
  printf("%s\n", filename);
else
  printf("Something wrong!\n");
```

上面示例中，`L_tmpname`是`stdio.h`定义的一个宏，指定了临时文件的文件名长度。

`tmpname()`的参数也可以是一个空指针 NULL，同样返回指向文件名字符串的指针。

```c
char* filename;
filename = tmpnam(NULL);
```

上面示例中，变量`filename`就是`tmpnam()`生成的文件名。

该函数只是生成一个文件名，稍后可以使用`fopen()`打开该文件并使用它。

## fflush()

`fflush()`用于清空缓存区。它接受一个文件指针作为参数，将缓存区内容写入该文件。

```c
fflush(fp);
```

如果不需要保存缓存区内容，则可以传入空指针 NULL。

```c
fflush(NULL);
```

如果清空成功，`fflush()`返回0，否则返回 EOF。

注意，`fflush()`一般只用来清空输出缓存区（比如写文件）。如果使用它来清空输入缓存区（比如读文件），属于未定义行为。

`fflush()`的一个用途是不等回车键，就强迫输出缓存区。大多数系统都是行缓存，这意味着只有遇到回车键（或者缓存区满了，或者文件读到结尾），缓存区的内容才会输出，`fflush()`可以不等回车键，立即输出。

```c
for (int i = 9; i >= 0; i--) {
  printf("\r%d", i);
  fflush(stdout);
  sleep(1);
}
```

上面示例是一个倒计时效果，`\r`是回车键，表示每轮循环都会回到当前行的行首，等于删除上一轮循环的输出。`fflush(stdout)`表示立即将缓存输出到显示器，这一行是必需的，否则由于上一行的输出没有回车键，不会触发缓存输出，屏幕上不会显示任何内容，只会等到程序运行结束再一次性输出。

## setvbuf()

`setvbuf()`函数用于定义某个字节流应该如何缓存。它可以接受四个参数。

```c
int setvbuf(FILE* stream, char* buffer, int mode, size_t size)
```

第一个参数`stream`是文件流。

第二个参数`buffer`是缓存区的地址。

第三个参数`mode`指定缓存的行为模式，它是下面三个宏之一，这些宏都定义在`stdio.h`。

- `_IOFBF`：满缓存。当缓存为空时，才从流读入数据；当缓存满了，才向流写入数据。一般情况下，这是默认设置。
- `_IOLBF`：行缓存。每次从流读入一行数据，或向流写入一行数据，即以行为单位读写缓存。
- `_IONBF`：无缓存。不使用缓存区，直接读写设备。

第四个参数`size`指定缓存区的大小。较大的缓存区提供更好的性能，而较小的缓存区可以节省空间。`stdio.h`提供了一个宏`BUFSIZ`，表示系统默认的缓存区大小。

它的意义在于，使得用户可以在打开一个文件之前，定义自己的文件缓冲区，而不必使用`fopen()`函数打开文件时设定的默认缓冲区。

```c
char buffer[N];

setvbuf(stream, buffer, _IOFBF, N);
```

上面示例设置文件流`stream`的缓存区从地址`buffer`开始，大小为`N`，模式为`_IOFBF`。

`setvbuf()`的第二个参数可以为空指针 NULL。这样的话，`setvbuf()`会自己创建一个缓存区。

注意，`setvbuf()`的调用必须在对文件流执行任何操作之前。

如果调用成功，`setvbuf()`的返回值为`0`，否则返回非零值。

下面的例子是将缓存区调整为行缓存。

```c
FILE *fp;
char lineBuf[1024];

fp = fopen("somefile.txt", "r");
setvbuf(fp, lineBuf, _IOLBF, 1024);
```

## setbuf()

`setbuf()`是`setvbuf()`的早期版本，可以视为后者的简化版本，也用来定义某个字节流的缓存区。

```c
void setbuf(FILE* stream, char* buffer);
```

它的第一个参数`stream`是文件流，第二个参数`buffer`是缓存区的地址。

它总是可以改写成`setvbuf()`。

```c
char buffer[BUFSIZ];

setbuf(stream, buffer);

// 等同于
setvbuf(stream, buffer, _IOFBF, BUFSIZ);
```

上面示例中，`BUFSIZ`是`stdio.h`定义的宏，表示系统默认的缓存区大小。

`setbuf()`函数没有返回值。

`setbuf()`的第二个参数如果设置为 NULL，表示不进行缓存。

```c
setbuf(stdout, NULL);

// 等同于
setvbuf(stdout, NULL, _IONBF, 0);
```

## ungetc()

`ungetc()`将从缓存里面读取的上一个字符，重新放回缓存，下一个读取缓存的操作会从这个字符串开始。有些操作需要了解下一个字符是什么，再决定应该怎么处理，这时这个函数就很有用。

它的原型如下。

```c
int ungetc(int c, FILE *stream);
```

它的第一个参数是一个字符变量，第二个参数是一个打开的文件流。它的返回值是放回缓存的那个字符，操作失败时，返回 EOF。

```c
int ch = fgetc(fp);

if (isdigit(ch)) {
  ch = fgetd(fp);
}

ungetc(ch, fp);
```

上面示例中，如果读取的字符不是数字，就将其放回缓存。

## perror()

`perror()`用于在 stderr 的错误信息之前，添加一个自定义字符串。

```c
void perror(const char *s);
```

该函数的参数就是在报错信息前添加的字符串。它没有返回值。

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <errno.h>

int main(void) {
  int x = -1;

  errno = 0;
  float y = sqrt(x);
  if (errno != 0) {
    perror("sqrt error");
    exit(EXIT_FAILURE);
  }
}
```

上面示例中，求`-1`的平方根，导致报错。头文件`errno.h`提供宏`errno`，只要上一步操作出错，这个宏就会设置成非零值。`perror()`用来在报错信息前，加上`sqrt error`的自定义字符串。

执行上面的程序，就会得到下面的报错信息。

```bash
$ gcc test.c -lm
$ ./a.out
sqrt error: Numerical argument out of domain
```

## 可变参数操作函数

（1）输出函数

下面是`printf()`的变体函数，用于按照给定格式，输出函数的可变参数列表（va_list）。

- vprintf()：按照给定格式，输出到控制台，默认是显示器。
- vfprintf()：按照给定格式，输出到文件。
- vsprintf()：按照给定格式，输出到字符串。
- vsnprintf()：按照给定格式，输出到字符串的安全版本。

它们的原型如下，基本与对应的`printf()`系列函数一致，除了最后一个参数是可变参数对象。

```c
#include <stdio.h>
#include <stdarg.h>
    
int vprintf(
  const char * restrict format,
  va_list arg
);

int vfprintf(
  FILE * restrict stream,
  const char * restrict format,
  va_list arg
);
    
int vsprintf(
  char * restrict s,
  const char * restrict format,
  va_list arg
);
    
int vsnprintf(
  char * restrict s,
  size_t n,
  const char * restrict format,
  va_list arg
);
```

它们的返回值都为输出的字符数，如果出错，返回负值。

`vsprintf()`和`vsnprintf()`的第一个参数可以为 NULL，用来查看多少个字符会被写入。

下面是一个例子。

```c
int logger(char *format, ...) {
  va_list va;
  va_start(va, format);
  int result = vprintf(format, va);
  va_end(va);

  printf("\n");

  return result;
}

// 输出 x = 12 and y = 3.20
logger("x = %d and y = %.2f", x, y);
```

（2）输入函数

下面是`scanf()`的变体函数，用于按照给定格式，输入可变参数列表 (va_list)。

- vscanf()：按照给定格式，从控制台读取（默认为键盘）。
- vfscanf()：按照给定格式，从文件读取。
- vsscanf()：按照给定格式，从字符串读取。

它们的原型如下，跟对应的`scanf()`函数基本一致，除了最后一个参数是可变参数对象。

```c
#include <stdio.h>
#include <stdarg.h>
    
int vscanf(
  const char * restrict format,
  va_list arg
);
    
int vfscanf(
  FILE * restrict stream,
  const char * restrict format,
  va_list arg
);
    
int vsscanf(
  const char * restrict s,
  const char * restrict format,
  va_list arg
);
```

它们返回成功读取的项数，遇到文件结尾或错误，则返回 EOF。

下面是一个例子。

```c
int error_check_scanf(int expected_count, char *format, ...) {
  va_list va;

  va_start(va, format);
  int count = vscanf(format, va);
  va_end(va);

  assert(count == expected_count);

  return count;
}

error_check_scanf(3, "%d, %d/%f", &a, &b, &c);
```

# stdlib.h

## 类型别名和宏

stdlib.h 定义了下面的类型别名。

- size_t：sizeof 的返回类型。
- wchar_t：宽字符类型。

stdlib.h 定义了下面的宏。

- NULL：空指针。
- EXIT_SUCCESS：函数运行成功时的退出状态。
- EXIT_FAILURE：函数运行错误时的退出状态。
- RAND_MAX：rand() 函数可以返回的最大值。
- MB_CUR_MAX：当前语言环境中，多字节字符占用的最大字节数。

## abs()，labs()，llabs()

这三个函数用于计算整数的绝对值。`abs()`用于 int 类型，`labs()`用于 long int 类型，`llabs()`用于 long long int 类型。

```c
int abs(int j);
long int labs(long int j);
long long int llabs(long long int j);
```

下面是用法示例。

```c
// 输出 |-2| = 2
printf("|-2| = %d\n", abs(-2));

// 输出 |4|  = 4
printf("|4|  = %d\n", abs(4));
```

## div()，ldiv()，lldiv()

这三个函数用来计算两个参数的商和余数。`div()`用于 int 类型的相除，`ldiv()`用于 long int 类型的相除，`lldiv()`用于 long long int 类型的相除。

```c
div_t div(int numer, int denom);
ldiv_t ldiv(long int numer, long int denom);
lldiv_t lldiv(long long int numer, long long int denom);
```

这些函数把第2个参数（分母）除以第1个参数（分子），产生商和余数。这两个值通过一个数据结构返回，`div()`返回 div_t 结构，`ldiv()`返回 ldiv_t 结构，`lldiv()`返回 lldiv_t 结构。

这些结构都包含下面两个字段，

```c
int　quot;　 //　商
int　rem;　 //　余数
```

它们完整的定义如下。

```c
typedef struct {
  int quot, rem;
} div_t;
    
typedef struct {
  long int quot, rem;
} ldiv_t;
    
typedef struct {
  long long int quot, rem;
} lldiv_t;
```

下面是一个例子。

```c
div_t d = div(64, -7);

// 输出 64 / -7 = -9
printf("64 / -7 = %d\n", d.quot);

// 输出 64 % -7 = 1
printf("64 %% -7 = %d\n", d.rem);
```

## 字符串转成数值

### a 系列函数

`stdlib.h`定义了一系列函数，可以将字符串转为数组。

- atoi()：字符串转成 int 类型。
- atof()：字符串转成 double 类型。
- atol()：字符串转成 long int 类型。
- atoll()：字符串转成 long long int 类型。

它们的原型如下。

```c
int atoi(const char* nptr);
double atof(const char* nptr);
long int atol(const char* nptr);
long long int atoll(const char* nptr);
```

上面函数的参数都是一个字符串指针，字符串开头的空格会被忽略，转换到第一个无效字符处停止。函数名称里面的`a`代表 ASCII，所以`atoi()`的意思是“ASCII to int”。

它们返回转换后的数值，如果字符串无法转换，则返回`0`。

下面是用法示例。

```c
atoi("3490")   // 3490
atof("3.141593")   // 3.141593
```

如果参数是数字开头的字符串，`atoi()`会只转换数字部分，比如`atoi("42regular")`会返回整数`42`。如果首字符不是数字，比如“hello world”，则会返回`0`。

### str 系列函数（浮点数转换）

`stdlib.h`还定义了一些更强功能的浮点数转换函数。

- strtof()：字符串转成 float 类型。
- strtod()：字符串转成 double 类型。
- strtold()：字符串转成 long double 类型。

它们的原型如下。

```c
float strtof(
  const char* restrict nptr,
  char** restrict endptr
);

double strtod(
  const char* restrict nptr,
  char** restrict endptr
);
    
long double strtold(
  const char* restrict nptr,
  char** restrict endptr
);
```

它们都接受两个参数，第一个参数是需要转换的字符串，第二个参数是一个指针，指向原始字符串里面无法转换的部分。

- `nptr`：待转换的字符串（起首的空白字符会被忽略）。
- `endprt`：一个指针，指向不能转换部分的第一个字符。如果字符串可以完全转成数值，该指针指向字符串末尾的终止符`\0`。这个参数如果设为 NULL，就表示不需要处理字符串剩余部分。

它们的返回值是已经转换后的数值。如果字符串无法转换，则返回`0`。如果转换结果发生溢出，errno 会被设置为 ERANGE。如果值太大（无论是正数还是负数），函数返回`HUGE_VAL`；如果值太小，函数返回零。

```c
char *inp = "   123.4567abdc";
char *badchar;

double val = strtod(inp, &badchar);

printf("%f\n", val); // 123.456700
printf("%s\n", badchar); // abdc
```

字符串可以完全转换的情况下，第二个参数指向`\0`，因此可以用下面的写法判断是否完全转换。

```c
if (*endptr == '\0') {
  // 完全转换
} else {
  // 存在无法转换的字符
}
```

如果不关心没有转换的部分，则可以将 endptr 设置为 NULL。

这些函数还可以将字符串转换为特殊值 Infinity 和 NaN。如果字符串包含 INF 或 INFINITY（大写或小写皆可），则将转换为 Infinity；如果字符串包含 NAN，则将返回 NaN。

### str 系列函数（整数转换）

str 系列函数也有整数转换的对应函数。

- strtol()：字符串转成 long int 类型。
- strtoll()：字符串转成 long long int 类型。
- strtoul()：字符串转成 unsigned long int 类型。
- strtoull()：字符串转成 unsigned long long int 类型。

它们的原型如下。

```c
long int strtol(
  const char* restrict nptr,
  char** restrict endptr,
  int base
);
    
long long int strtoll(
  const char* restrict nptr,
  char** restrict endptr,
  int base
);
    
unsigned long int strtoul(
  const char* restrict nptr,
  char** restrict endptr,
  int base
);
    
unsigned long long int strtoull(
  const char* restrict nptr,
  char** restrict endptr, int base
);
```

它们接受三个参数。

（1）`nPtr`：待转换的字符串（起首的空白字符会被忽略）。

（2）`endPrt`：一个指针，指向不能转换部分的第一个字符。如果字符串可以完全转成数值，该指针指向字符串末尾的终止符`\0`。这个参数如果设为 NULL，就表示不需要处理字符串剩余部分。

（3）`base`：待转换整数的进制。这个值应该是`2`到`36`之间的整数，代表相应的进制，如果是特殊值`0`，表示让函数根据数值的前缀，自己确定进制，即如果数字有前缀`0`，则为八进制，如果数字有前缀`0x`或`0X`，则为十六进制。

它们的返回值是转换后的数值，如果转换不成功，返回`0`。

下面是转换十进制整数的例子。

```c
char* s = "3490";
unsigned long int x = strtoul(u, NULL, 10);

printf("%lu\n", x); // 3490
```

下面是转换十六进制整数的例子。

```c
char* end;

long value = strtol("0xff", &end, 16);
printf("%ld\n", value); // 255
printf("%s\n", end); // 无内容

value = strtol("0xffxx", &end, 16);
printf("%ld\n", value); // 255
printf("%s\n", end); // xx
```

上面示例中，`strtol()`可以指定字符串包含的是16进制整数。不能转换的部分，可以使用指针`end`进行访问。

下面是转换二进制整数的例子。

```c
char* s = "101010";
unsigned long int x = strtoul(s, NULL, 2);

printf("%lu\n", x); // 42
```

下面是让函数自行判断整数进制的例子。

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
  const char* string = "-1234567abc";
  char* remainderPtr;
  long x = strtol(string, &remainderPtr, 0);
  printf("%s\"%s\"\n%s%ld\n%s\"%s\"\n",
    "The original string is ",
    string,
    "The converted value is ",
    x,
    "The remainder of the original string is ",
    remainderPtr
  );
}
```

上面代码的输出结果如下。

```c
The original string is "-1234567abc"
The converted value is -1234567
The remainder of the original string is "abc"
```

如果被转换的值太大，`strtol()`函数在`errno`中存储`ERANGE`这个值，并返回`LONG_MIN`（原值为负数）或`LONG_MAX`（原值为正数），`strtoul()`则返回`ULONG_MAX`。

## rand()

`rand()`函数用来生成 0～RAND_MAX 之间的随机整数。`RAND_MAX`是一个定义在`stdlib.h`里面的宏，通常等于 INT_MAX。

```c
// 原型
int rand(void);

// 示例
int x = rand();
```

如果希望获得整数 N 到 M 之间的随机数（包括 N 和 M 两个端点值），可以使用下面的写法。

```c
int x = rand() % （M - N + 1) + N;
```

比如，1 到 6 之间的随机数，写法如下。

```c
int x = rand() % 6 + 1;
```

获得浮点数的随机值，可以使用下面的写法。

```c
// 0 到 0.999999 之间的随机数
printf("0 to 0.99999: %f\n", rand() / ((float)RAND_MAX + 1));

// n 到 m 之间的随机数：
// n + m * (rand() / (float)RAND_MAX)
printf("10.5 to 15.7: %f\n", 10.5 + 5.2 * rand() / (float)RAND_MAX);
```

上面示例中，由于`rand()`和`RAND_MAX`都是 int 类型，要用显示的类型转换转为浮点数。

## srand()

`rand()`是伪随机数函数，为了增加随机性，必须在调用它之前，使用`srand()`函数重置一下种子值。

`srand()`函数接受一个无符号整数（unsigned int）作为种子值，没有返回值。

```c
void srand(unsigned int seed);
```

通常使用`time(NULL)`函数返回当前距离时间纪元的秒数，作为`srand()`的参数。

```c
#include <time.h>
srand((unsigned int) time(NULL));
```

上面代码中，`time()`的原型定义在头文件`time.h`里面，返回值的类型是类型别名`time_t`，具体的类型与系统有关，所以要强制转换一下类型。`time()`的参数是一个指针，指向一个具体的 time_t 类型的时间值，这里传入空指针`NULL`作为参数，由于 NULL 一般是`0`，所以也可以写成`time(0)`。

## abort()

`abort()`用于不正常地终止一个正在执行的程序。使用这个函数的目的，主要是它会触发 SIGABRT 信号，开发者可以在程序中为这个信号设置一个处理函数。

```c
void abort(void);
```

该函数没有参数。

## exit()，quick_exit()，_Exit()

这三个函数都用来退出当前正在执行的程序。

```c
void exit(int status);
void quick_exit(int status);
void _Exit(int status);
```

它们都接受一个整数，表示程序的退出状态，`0`是正常退出，非零值表示发生错误，可以使用宏`EXIT_SUCCESS`和`EXIT_FAILURE`当作参数。它们本身没有返回值。

它们的区别是，退出时所做的清理工作不同。`exit()`是正常退出，系统会做完整的清理，比如更新所有文件流，并且删除临时文件。`quick_exit()`是快速退出，系统的清理工作稍微少一点。`_Exit()`是立即退出，不做任何清理工作。

下面是一些用法示例。

```c
exit(EXIT_SUCCESS);
quick_exit(EXIT_FAILURE);
_Exit(2);
```

## atexit()，at_quick_exit()

`atexit()`用来登记当前程序退出时（调用`exit()`或`main()`正常退出），所要执行的其他函数。

`at_quick_exit()`则是登记使用`quick_exit()`方法退出当前程序时，所要执行的其他函数。

`exit()`只能触发`atexit()`登记的函数，`quick_exit()`只能触发`at_quick_exit()`登记的函数。

```c
int atexit(void (*func)(void));
int at_quick_exit(void (*func)(void));
```

它们的参数是要执行的函数地址，即函数名。它们的返回值都是调用成功时返回`0`，调用失败时返回非零值。

下面是一个例子。

```c
void sign_off(void);
void too_bad(void);

int main(void) {

  int n;
  atexit(sign_off);　　 /* 注册 sign_off()函数 */

  puts("Enter an integer:");
  if (scanf("%d", &n) != 1) {
    puts("That's no integer!");
    atexit(too_bad);　/* 注册 too_bad()函数 */
    exit(EXIT_FAILURE);
  }

  printf("%d is %s.\n", n, (n % 2 == 0) ? "even" : "odd");
  return 0;
}

void sign_off(void) {
  puts("sign_off");
}

void too_bad(void) {
  puts("too bad");
}
```

上面示例中，用户输入失败时，会调用`sign_off()`和`too_bad()`函数；但是输入成功时只会调用`sign_off()`。因为只有输入失败时，才会进入`if`语句登记`too_bad()`。

另外，如果有多条`atexit()`语句，函数退出时最先调用的，是最后一个登记的函数。

`atexit()`登记的函数（如上例的`sign_off`和`too_bad`）应该不带任何参数且返回类型为`void`。通常，这些函数会执行一些清理任务，例如删除临时文件或重置环境变量。

`at_quick_exit()`也是同样的规则，下面是一个例子。

```c
void exit_handler_1(void) {
  printf("1\n");
}

void exit_handler_2(void) {
  printf("2\n");
}

int main(void) {
  at_quick_exit(exit_handler_1);
  at_quick_exit(exit_handler_2);
  quick_exit(0);
}
```

执行上面的示例，命令行会先输出2，再输出1。

## getenv()

`getenv()`用于获取环境变量的值。环境变量是操作系统提供的程序之外的一些环境参数。

```c
char* getenv(const char* name);
```

它的参数是一个字符串，表示环境变量名。返回值也是一个字符串，表示环境变量的值。如果指定的环境变量不存在，则返回 NULL。

下面是输出环境变量`$PATH`的值的例子。

```c
printf("PATH is %s\n", getenv("PATH"));
```

## system()

`system()`函数用于执行外部程序。它会把它的参数字符串传递给操作系统，让操作系统的命令处理器来执行。

```c
void system( char const * command );
```

这个函数的返回值因编译器而异。但是标准规定，如果 NULL 作为参数，表示询问操作系统，是否有可用的命令处理器，如果有的话，返回一个非零值，否则返回零。

下面是执行`ls`命令的例子。

```c
system("ls -l"); 
```

## 内存管理函数

stdlib.h 提供了一些内存操作函数，下面几个函数详见《内存管理》一章，其余在本节介绍。

- malloc()：分配内存区域
- calloc()：分配内存区域。
- realloc()：调节内存区域大小。
- free()：释放内存区域。

### aligned_alloc()

很多系统有内存对齐的要求，即内存块的大小必须是某个值（比如64字节）的倍数，这样有利于提高处理速度。`aligned_alloc()`就用于分配满足内存对齐要求的内存块，它的原型如下。

```c
void* aligned_alloc(size_t alignment, size_t size);
```

它接受两个参数。

- alignment：整数，表示内存对齐的单位大小，一般是2的整数次幂（2、4、8、16……）。
- size：整数，表示内存块的大小。

分配成功时，它返回一个无类型指针，指向新分配的内存块。分配失败时，返回 NULL。

```c
char* p = aligned_alloc(64, 256);
```

上面示例中，`aligned_alloc()`分配的内存块，单位大小是64字节，要分配的字节数是256字节。

## qsort()

`qsort()`用来快速排序一个数组。它对数组成员的类型没有要求，任何类型数组都可以用这个函数排序。

```c
void qsort(
  void *base,
  size_t nmemb, 
  size_t size,
  int (*compar)(const void *, const void *)
);
```

该函数接受四个参数。

- base：指向要排序的数组开始位置的指针。
- nmemb：数组成员的数量。
- size：数组每个成员占用的字节长度。
- compar：一个函数指针，指向一个比较两个成员的函数。

比较函数`compar`将指向数组两个成员的指针作为参数，并比较两个成员。如果第一个参数小于第二个参数，该函数应该返回一个负值；如果两个函数相等，返回`0`；如果第一个参数大于第二个参数，应该返回一个正数。

下面是一个用法示例。

```c
#include <stdio.h>
#include <stdlib.h>

int compar(const void* elem0, const void* elem1) {
  const int* x = elem0;
  const int* y = elem1; 
  
  return *x - *y;
}

int main(void) {
  int a[9] = {14, 2, 3, 17, 10, 8, 6, 1, 13};

  qsort(a, 9, sizeof(int), compar);

  for (int i = 0; i < 9; i++)
    printf("%d ", a[i]);
  putchar('\n');
}
```

执行上面示例，会输出排序好的数组“1 2 3 6 8 10 13 14 17”。

## bsearch()

`bsearch()`使用二分法搜索，在数组中搜索一个值。它对数组成员的类型没有要求，任何类型数组都可以用这个函数搜索值。

注意，该方法只对已经排序好的数组有效。

```c
void *bsearch(
  const void* key,
  const void* base,
  size_t nmemb,
  size_t size,
  int (*compar)(const void *, const void *)
);
```

这个函数接受5个参数。

- key：指向要查找的值的指针。
- base：指向数组开始位置的指针，数组必须已经排序。
- nmemb：数组成员的数量。
- size：数组每个成员占用的字节长度。
- compar：指向一个将待查找值与其他值进行比较的函数的指针。

比较函数`compar`将待查找的值作为第一个参数，将要比较的值作为第二个参数。如果第一个参数小于第二个参数，该函数应该返回一个负值；如果两个参数相等，返回`0`；如果第一个参数大于第二个参数，返回一个正值。

如果找到待查找的值，`bsearch()`返回指向该值的指针，如果找不到，返回 NULL。

下面是一个用法示例。

```c
#include <stdio.h>
#include <stdlib.h>

int compar(const void *key, const void *value) {
  const int* k = key;
  const int* v = value;

  return *k - *v;
}

int main(void) {
  int a[9] = {2, 6, 9, 12, 13, 18, 20, 32, 47};

  int* r;
  int key;

  key = 12; // 包括在数组中
  r = bsearch(&key, a, 9, sizeof(int), compar);
  printf("Found %d\n", *r);

  key = 30;  // 不包括在数组中
  r = bsearch(&key, a, 9, sizeof(int), compar);
  if (r == NULL)
    printf("Didn't find 30\n");

  return 0;
}
```

执行上面的示例，会输出下面的结果。

```bash
Found 12
Didn't find 30
```

## 多字节字符函数

stdlib.h 提供了下面的函数，用来操作多字节字符，详见《多字节字符》一章。

- mblen()：多字节字符的字节长度。
- mbtowc()：将多字节字符转换为宽字符。
- wctomb()：将宽字符转换为多字节字符。
- mbstowcs()：将多字节字符串转换为宽字符串。
- wcstombs()：将宽字符串转换为多字节字符串。

# string.h

`string.h`主要定义了字符串处理函数和内存操作函数。

## 字符串处理函数

以下字符串处理函数，详见《字符串》一章。

- strcpy()：复制字符串。
- strncpy()：复制字符串，有长度限制。
- strcat()：连接两个字符串。
- strncat()：连接两个字符串，有长度限制。
- strcmp()：比较两个字符串。
- strncmp()：比较两个字符串，有长度限制。
- strlen()：返回字符串的字节数。

### strchr()，strrchr()

`strchr()`和`strrchr()`都用于在字符串中查找指定字符。不同之处是，`strchr()`从字符串开头开始查找，`strrchr()`从字符串结尾开始查找，函数名里面多出来的那个`r`表示 reverse（反向）。

```c
char* strchr(char* str, int c);
char* strrchr(char *str, int c);
```

它们都接受两个参数，第一个参数是字符串指针，第二个参数是所要查找的字符。

一旦找到该字符，它们就会停止查找，并返回指向该字符的指针。如果没有找到，则返回 NULL。

下面是一个例子。

```c
char *str = "Hello, world!";
char *p;

p = strchr(str, ',');  // p 指向逗号的位置
p = strrchr(str, 'o'); // p 指向 world 里面 o 的位置
```

### strspn()，strcspn()

`strspn()`用来查找属于指定字符集的字符串长度，`strcspn()`正好相反，用来查找不属于指定字符集的字符串长度。

```c
size_t strspn(char* str, const char* accept);
size_t strcspn(char *str, const char *reject);
```

这两个函数接受两个参数，第一个参数是源字符串，第二个参数是由指定字符组成的字符串。

`strspn()`从第一个参数的开头开始查找，一旦发现第一个不属于指定字符集范围的字符，就停止查找，返回到目前为止的字符串长度。如果始终没有不在指定字符集的字符，则返回第一个参数字符串的长度。

`strcspn()`则是一旦发现第一个属于指定字符集范围的字符，就停止查找，返回到目前为止的字符串长度。如果始终没有发现指定字符集的字符，则返回第一个参数字符串的长度。

```c
char str[] = "hello world";
int n;

n = strspn(str1, "aeiou");
printf("%d\n", n);  // n == 0

n = strcspn(str1, "aeiou");
printf("%d\n", n); // n == 1
```

上面示例中，第一个`n`等于0，因为0号位置的字符`h`就不属于指定字符集`aeiou`，可以理解为开头有0个字符属于指定字符集。第二个`n`等于1，因为1号位置的字符`e`属于指定字符集`aeiou`，可以理解为开头有1个字符不属于指定字符集。

### strpbrk()

`strpbrk()`在字符串中搜索指定字符集的任一个字符。

```c
char* strpbrk(const char* s1, const char* s2);
```

它接受两个参数，第一个参数是源字符串，第二个参数是由指定字符组成的字符串。

它返回一个指向第一个匹配字符的指针，如果未找到匹配字符，则返回 NULL。

```c
char* s1 = "Hello, world!";
char* s2 = "dow!";

char* p = strpbrk(s1, s2);

printf("%s\n", p);  // "o, world!"
```

上面示例中，指定字符集是“dow!”，那么`s1`里面第一个匹配字符是“Hello”的“o”，所以指针`p`指向这个字符。输出的话，就会输出从这个字符直到字符串末尾的“o, world!”。

### strstr()

`strstr()`在一个字符串里面，查找另一个字符串。

```c
char *strstr(
  const char* str,
  const char* substr
);
```

它接受两个参数，第一个参数是源字符串，第二个参数是所要查找的子字符串。

如果匹配成功，就返回一个指针，指向源字符串里面的子字符串。如果匹配失败，就返回 NULL，表示无法找到子字符串。

```c
char* str = "The quick brown fox jumped over the lazy dogs.";
char* p = strstr(str, "lazy");

printf("%s\n", p == NULL ? "null": p); // "lazy dogs."
```

上面示例中，`strstr()`用来在源字符串`str`里面，查找子字符串`lazy`。从返回的指针到字符串结尾，就是“lazy dogs.”。

### strtok()

`strtok()`用来将一个字符串按照指定的分隔符（delimiter），分解成一系列词元（tokens）。

```c
char* strtok(char* str, const char* delim);
```

它接受两个参数，第一个参数是待拆分的字符串，第二个参数是指定的分隔符。

它返回一个指针，指向分解出来的第一个词元，并将词元结束之处的分隔符替换成字符串结尾标志`\0`。如果没有待分解的词元，它返回 NULL。

如果要遍历所有词元，就必须循环调用，参考下面的例子。

`strtok()`的第一个参数如果是 NULL，则表示从上一次`strtok()`分解结束的位置，继续往下分解。

```c
#include <stdio.h>
#include <string.h>

int main(void) {
  char string[] = "This is a sentence with 7 tokens";
  char* tokenPtr = strtok(string, " ");

  while (tokenPtr != NULL) {
    printf("%s\n", tokenPtr);
    tokenPtr = strtok(NULL, " ");
  }
}
```

上面示例将源字符串按照空格，分解词元。它的输出结果如下。

```bash
This
is
a
sentence
with
7
tokens
```

注意，`strtok()`会修改原始字符串，将所有分隔符都替换成字符串结尾符号`\0`。因此，最好生成一个原始字符串的拷贝，然后再对这个拷贝执行`strtok()`。

### strcoll()

`strcoll()`用于比较两个启用了本地化设置的字符串，用法基本与`strcmp()`相同。

```c
int strcoll(const char *s1, const char *s2);
```

请看下面的示例。

```c
setlocale(LC_ALL, "");

// 报告 é > f
printf("%d\n", strcmp("é", "f"));  

// 报告 é < f
printf("%d\n", strcoll("é", "f"));
```

上面示例比较带重音符号的`é`与`f`，`strcmp()`会返回`é`大于`f`，而`strcoll()`就会正确识别`é`排在`f`前面，所以小于`f`。注意，在比较之前，需要使用`setlocale(LC_ALL, "")`，启用本地化设置。

### strxfrm()

`strxfrm()`将一个本地化字符串转成可以使用`strcmp()`进行比较的形式，相当于`strcoll()`内部的第一部分操作。

```c
size_t strxfrm(
  char * restrict s1, 
  const char * restrict s2, 
  size_t n
);
```

它接受三个参数，将第二个参数`s2`转为可以使用`strcmp()`比较的形式，并将结果存入第一个参数`s1`。第三个参数`n`用来限定写入的字符数，防止超出`s1`的边界。

它返回转换后的字符串长度，不包括结尾的终止符。

如果第一个参数是 NULL，第三个参数是0，则不进行实际的转换，只返回转换后所需的字符串长度。

下面的示例是用这个函数自己实现一个`strcoll()`。

```c
int my_strcoll(char* s1, char* s2) {
  int len1 = strxfrm(NULL, s1, 0) + 1;
  int len2 = strxfrm(NULL, s2, 0) + 1;

  char *d1 = malloc(len1);
  char *d2 = malloc(len2);

  strxfrm(d1, s1, len1);
  strxfrm(d2, s2, len2);

  int result = strcmp(d1, d2);

  free(d2);
  free(d1);

  return result;
}
```

上面示例中，先为两个进行比较的本地化字符串，分配转换后的存储空间，使用`strxfrm()`将它们转为可比较的形式，再用`strcmp()`进行比较。

### strerror()

`strerror()`函数返回特定错误的说明字符串。

```c
char *strerror(int errornum);
```

它的参数是错误的编号，由`errno.h`定义。返回值是一个指向说明字符串的指针。

```c
// 输出 No such file or directory
printf("%s\n", strerror(2));
```

上面示例输出2号错误的说明字符“No such file or directory“。

下面的例子是自定义报错信息。

```c
#include <stdio.h>
#include <string.h>
#include <errno.h>

int main(void) {
  FILE* fp = fopen("NONEXISTENT_FILE.TXT", "r");

  if (fp == NULL) {
    char* errmsg = strerror(errno);
    printf("Error %d opening file: %s\n", errno, errmsg);
  }
}
```

上面示例中，通过`strerror(errno)`拿到当前的默认报错信息，其中`errno`是`errno.h`定义的宏，表示当前的报错编号。然后，再输出一条自定义的报错信息。

## 内存操作函数

以下内存操作函数，详见《内存管理》一章。

- memcpy()：内存复制函数。
- memmove()：内存复制函数（允许重叠）。
- memcmp()：比较两个内存区域。

### memchr()

`memchr()`用于在内存区域中查找指定字符。

```c
void* memchr(const void* s, int c, size_t n);
```

它接受三个参数，第一个参数是内存区域的指针，第二个参数是所要查找的字符，第三个参数是内存区域的字节长度。

一旦找到，它就会停止查找，并返回指向该位置的指针。如果直到检查完指定的字节数，依然没有发现指定字符，则返回 NULL。

下面是一个例子。

```c
char *str = "Hello, world!";
char *p;

p = memchr(str, '!', 13); // p 指向感叹号的位置
```

### memset()

`memset()`将一段内存全部格式化为指定值。

```c
void* memset(void* s, int c, size_t n);
```

它的第一个参数是一个指针，指向内存区域的开始位置，第二个参数是待写入的字符值，第三个参数是一个整数，表示需要格式化的字节数。它返回第一个参数（指针）。

```c
memset(p, ' ', N);
```

上面示例中，p 是一个指针，指向一个长度为 N 个字节的内存区域。`memset()`将该块内存区域的每个字节，都改写为空格字符。

下面是另一个例子。

```c
char string1[15] = "BBBBBBBBBBBBBB";

// 输出 bbbbbbbBBBBBBB
printf("%s\n", (char*) memset(string1, 'b', 7));
```

`memset()`的一个重要用途，就是将数组成员全部初始化为0。

```c
memset(arr, 0, sizeof(arr));
```

下面是将 Struct 结构都初始化为0的例子。

```c
struct banana {
  float ripeness;
  char *peel_color;
  int grams;
};

struct banana b;

memset(&b, 0, sizeof b);

b.ripeness == 0.0;     // True
b.peel_color == NULL;  // True
b.grams == 0;          // True
```

上面示例，将 Struct banana 的实例 b 的所有属性都初始化为0。

## 其他函数

```c
void* memset(void* a, int c, size_t n);

size_t strlen(const char* s);
```

# time.h

## time_t

time_t 是一个表示时间的类型别名，可以视为国际标准时 UTC。它可能是浮点数，也可能是整数，Unix 系统一般是整数。

许多系统上，time_t 表示自时间纪元（time epoch）以来的秒数。Unix 的时间纪元是国际标准时 UTC 的1970年1月1日的零分零秒。time_t 如果为负数，则表示时间纪元之前的时间。

time_t 一般是32位或64位整数类型的别名，具体类型取决于当前系统。如果是32位带符号整数，time_t 可以表示的时间到 2038年1月19日03:14:07 UTC 为止；如果是32位无符号整数，则表示到2106年。如果是64位带符号整数，可以表示`-2930`亿年到`+2930`亿年的时间范围。

## struct tm

struct tm 是一个数据结构，用来保存时间的各个组成部分，比如小时、分钟、秒、日、月、年等。下面是它的结构。

```c
struct tm {
  int tm_sec;    // 秒数 [0, 60]
  int tm_min;    // 分钟 [0, 59]
  int tm_hour;   // 小时 [0, 23]
  int tm_mday;   // 月份的天数 [1, 31]
  int tm_mon;    // 月份 [0, 11]，一月用 0 表示
  int tm_year;   // 距离 1900 的年数
  int tm_wday;   // 星期几 [0, 6]，星期天用 0 表示
  int tm_yday;   // 距离1月1日的天数 [0, 365]
  int tm_isdst;  // 是否采用夏令时，1 表示采用，0 表示未采用
};
```

## time()

`time()`函数返回从时间纪元到现在经过的秒数。

```c
time_t time(time_t* returned_value);
```

`time()`接受一个 time_t 指针作为参数，返回值会写入指针地址。参数可以是空指针 NULL。

`time()`的返回值是 time_t 类型的当前时间。 如果计算机无法提供当前的秒数，或者返回值太大，无法用`time_t`类型表示，`time()`函数就返回`-1`。

```c
time_t now;

// 写法一    
now = time(NULL);

// 写法二    
time(&now);
```

上面示例展示了将当前时间存入变量`now`的两种写法。

如果要知道某个操作耗费的精确时间，需要调用两次`time()`，再将两次的返回值相减。

```c
time_t begin = time(NULL);

// ... 执行某些操作

time_t end = time(NULL);

printf("%d\n", end - begin);
```

注意，上面的方法只能精确到秒。

## ctime()

`ctime()`用来将 time_t 类型的值直接输出为人类可读的格式。

```c
char* ctime( time_t const * time_value );
```

`ctime()`的参数是一个 time_t 指针，返回一个字符串指针。该字符串的格式类似“Sun Jul 4 04:02:48 1976\n\0”，尾部包含换行符和字符串终止标志。

下面是一个例子。

```c
time_t now; 

now = time(NULL);

// 输出 Sun Feb 28 18:47:25 2021
printf("%s", ctime(&now));
```

注意，`ctime()`会在字符串尾部自动添加换行符。

## localtime()，gmtime()

`localtime()`函数用来将 time_t 类型的时间，转换为当前时区的 struct tm 结构。

`gmtime()`函数用来将 time_t 类型的时间，转换为 UTC 时间的 struct tm 结构。

它们的区别就是返回值，前者是本地时间，后者是 UTC 时间。

```c
struct tm* localtime(const time_t* timer);
struct tm* gmtime(const time_t* timer);
```

下面是一个例子。

```c
time_t now = time(NULL);

// 输出 Local: Sun Feb 28 20:15:27 2021
printf("Local: %s", asctime(localtime(&now)));

// 输出 UTC  : Mon Mar  1 04:15:27 2021
printf("UTC  : %s", asctime(gmtime(&now)));
```

## asctime()

`asctime()`函数用来将 struct tm 结构，直接输出为人类可读的格式。该函数会自动在输出的尾部添加换行符。

用法示例参考上一小节。

## mktime()

`mktime()`函数用于把一个 struct tm 结构转换为 time_t 值。

```c
time_t　mktime(struct tm* tm_ptr);
```

`mktime()`的参数是一个 struct tm 指针。

`mktime()`会自动设置 struct tm 结构里面的`tm_wday`属性和`tm_yday`属性，开发者自己不必填写这两个属性。所以，这个函数常用来获得指定时间是星期几（`tm_wday`）。

struct tm 结构的`tm_isdst`属性也可以设为`-1`，让`mktime()`决定是否应该采用夏令时。

下面是一个例子。

```c
struct tm some_time = {
  .tm_year=82,   // 距离 1900 的年数
  .tm_mon=3,     // 月份 [0, 11]
  .tm_mday=12,   // 天数 [1, 31]
  .tm_hour=12,   // 小时 [0, 23]
  .tm_min=00,    // 分钟 [0, 59]
  .tm_sec=04,    // 秒数 [0, 60]
  .tm_isdst=-1,  // 夏令时
};
    
time_t some_time_epoch;
some_time_epoch = mktime(&some_time);
    
// 输出 Mon Apr 12 12:00:04 1982
printf("%s", ctime(&some_time_epoch));

// 输出 Is DST: 0
printf("Is DST: %d\n", some_time.tm_isdst);
```

## difftime()

`difftime()`用来计算两个时间之间的差异。Unix 系统上，直接相减两个 time_t 值，就可以得到相差的秒数，但是为了程序的可移植性，最好还是使用这个函数。

```c
double difftime( time_t time1, time_t time2 );
```

`difftime()`函数接受两个 time_t 类型的时间作为参数，计算 time1 - time2 的差，并把结果转换为秒。

注意它的返回值是 double 类型。

```c
#include <stdio.h>
#include <time.h>
    
int main(void) {
  struct tm time_a = {
    .tm_year=82, 
    .tm_mon=3,   
    .tm_mday=12, 
    .tm_hour=4,  
    .tm_min=00,  
    .tm_sec=04,  
    .tm_isdst=-1,
  };
    
  struct tm time_b = {
    .tm_year=120,
    .tm_mon=10,  
    .tm_mday=15, 
    .tm_hour=16, 
    .tm_min=27,  
    .tm_sec=00,  
    .tm_isdst=-1,
  };
    
  time_t cal_a = mktime(&time_a);
  time_t cal_b = mktime(&time_b);
    
  double diff = difftime(cal_b, cal_a);
    
  double years = diff / 60 / 60 / 24 / 365.2425;
  
  // 输出 1217996816.000000 seconds (38.596783 years) between events
  printf("%f seconds (%f years) between events\n", diff, years);
}
```

上面示例中，折算年份时，为了尽量准确，使用了一年的准确长度 365.2425 天，这样可以抵消闰年的影响。

## strftime()

`strftime()`函数用来将 struct tm 结构转换为一个指定格式的字符串，并复制到指定地址。

```c
size_t strftime(
  char* str, 
  size_t maxsize, 
  const char* format, 
  const struct tm* timeptr
)
```

`strftime()`接受四个参数。

- 第一个参数：目标字符串的指针。
- 第二个参数：目标字符串可以接受的最大长度。
- 第三个参数：格式字符串。
- 第四个参数：struct tm 结构。

如果执行成功（转换并复制），`strftime()`函数返回复制的字符串长度；如果执行失败，返回`-1`。

下面是一个例子。

```c
#include <stdio.h>
#include <time.h>

int main(void) {
  char s[128];
  time_t now = time(NULL);

  // %c: 本地时间
  strftime(s, sizeof s, "%c", localtime(&now));
  puts(s);   // Sun Feb 28 22:29:00 2021

  // %A: 完整的星期日期的名称
  // %B: 完整的月份名称
  // %d: 月份的天数
  strftime(s, sizeof s, "%A, %B %d", localtime(&now));
  puts(s);   // Sunday, February 28

  // %I: 小时（12小时制）
  // %M: 分钟
  // %S: 秒数
  // %p: AM 或 PM
  strftime(s, sizeof s, "It's %I:%M:%S %p", localtime(&now));
  puts(s);   // It's 10:29:00 PM

  // %F: ISO 8601 yyyy-mm-dd 格式
  // %T: ISO 8601 hh:mm:ss 格式
  // %z: ISO 8601 时区
  strftime(s, sizeof s, "ISO 8601: %FT%T%z", localtime(&now));
  puts(s);   // ISO 8601: 2021-02-28T22:29:00-0800
}
```

下面是常用的格式占位符。

- %%：输出 % 字符。
- %a：星期几的简写形式，以当地时间计算。
- %A：星期几的完整形式，以当地时间计算。
- %b：月份的简写形式，以当地时间计算。
- %B：月份的完整形式，以当地时间计算。
- %c：日期和时间，使用“%x %X”。
- %d：月份的天数（01-31）。
- %H：小时，采用24小时制（00-23）。
- %I：小时，采用12小时制（00-12）。
- %J：一年的第几天（001-366）。
- %m：月数（01-12）。
- %M：分钟（00～59）。
- %P：AM 或 PM。
- %R：相当于"%H:%M"。
- %S：秒（00-61）。
- %U：一年的第几星期（00-53），以星期日为第1天。
- %w：一星期的第几天，星期日为第0天。
- %W：一年的第几星期(00-53)，以星期一为第1天。
- %x：完整的年月日的日期，以当地时间计算。
- %X：完整的时分秒的时间，以当地时间计算。
- %y：两位数年份（00-99）。
- %Y：四位数年份（例如 1984）。
- %Z：时区的简写。

## timespec_get()

`timespec_get()`用来将当前时间转成距离时间纪元的纳秒数（十亿分之一秒）。

```c
int timespec_get ( struct timespec* ts, int base ) ;
```

`timespec_get()`接受两个参数。

第一个参数是 struct timespec 结构指针，用来保存转换后的时间信息。struct timespec 的结构如下。

```c
struct timespec {
  time_t tv_sec;   // 秒数
  long   tv_nsec;  // 纳秒
};
```

第二个参数是一个整数，表示时间计算的起点。标准只给出了宏 TIME_UTC 这一个可能的值，表示返回距离时间纪元的秒数。

下面是一个例子。

```c
struct timespec ts;
    
timespec_get(&ts, TIME_UTC);
    
// 1614581530 s, 806325800 ns
printf("%ld s, %ld ns\n", ts.tv_sec, ts.tv_nsec);
    
double float_time = ts.tv_sec + ts.tv_nsec/1000000000.0;

// 1614581530.806326 seconds since epoch
printf("%f seconds since epoch\n", float_time);
```

## clock()

`clock()`函数返回从程序开始执行到当前的 CPU 时钟周期。一个时钟周期等于 CPU 频率的倒数，比如 CPU 的频率如果是 1G Hz，表示1秒内时钟信号可以变化 10^9 次，那么每个时钟周期就是 10^-9 秒。

```c
clock_t　clock(void);
```

`clock()`函数返回一个数字，表示从程序开始到现在的 CPU 时钟周期的次数。这个值的类型是 clock_t，一般是 long int 类型。 

为了把这个值转换为秒，应该把它除以常量`CLOCKS_PER_SEC`（每秒的时钟周期），这个常量也由`time.h`定义。

```c
printf("CPU time: %f\n", clock() / (double)CLOCKS_PER_SEC);
```

上面示例可以输出程序从开始到运行到这一行所花费的秒数。

如果计算机无法提供 CPU 时间，或者返回值太大，无法用`clock_t`类型表示，`clock()`函数就返回`-1`。

为了知道某个操作所耗费的精确时间，需要调用两次`clock()`，然后将两次的返回值相减。

```c
clock_t start = clock();

// ... 执行某些操作

clock_t end = clock();

long double seconds = (float)(end - start) / CLOCKS_PER_SEC;
```

## 参考链接

- [How to Measure Execution Time of a Program](https://serhack.me/articles/measure-execution-time-program/)

# wchar.h

宽字符使用两个或四个字节表示一个字符，导致 C 语言常规的字符处理函数都会失效。wchar.h 定义了许多宽字符专用的处理函数。

## 类型别名和宏

wchar.h 定义了一个类型别名 wint_t，表示宽字符对应整数值。

wchar.h 还定义了一个宏 WEOF，表示文件结束字符 EOF 的宽字符版。

## btowc()，wctob()

`btowc()`将单字节字符转换为宽字符，`wctob()`将宽字符转换为单字节字符。

```c
wint_t btowc(int c);
int wctob(wint_t c);
```

`btowc()`返回一个宽字符。如果参数是 EOF，或转换失败，则返回 WEOF。

`wctob()`返回一个单字节字符。如果参数是 WEOF，或者参数宽字符无法对应单个的单字节字符，则返回 EOF。

下面是用法示例。

```c
wint_t wc = btowc('B'); 

// 输出宽字符 B
wprintf(L"Wide character: %lc\n", wc);

unsigned char c = wctob(wc);

// 输出单字节字符 B
wprintf(L"Single-byte character: %c\n", c);
```

## fwide()

`fwide()`用来设置一个字节流是宽字符流，还是多字节字符流。

如果使用宽字符专用函数处理字节流，就会默认设置字节流为宽字符流，否则就需要使用`fwide()`显式设置。

```c
int fwide(FILE* stream, int mode);
```

它接受两个参数，第一个参数是文件指针，第二个参数是字节流模式，有三种选择。

- 0：字节流模式保持原样。
- -1（或其他负值）：设为多字节字符流。
- 1（或其他正值）：设为宽字符流。

`fwide()`的返回值也分成三种情况：如果是宽字符流，返回一个正值；如果是多字节字符流，返回一个负值；如果是普通字符流，返回`0`。

一旦设置了字节流模式，就无法再更改。

```c
#include <stdio.h>
#include <wchar.h>

int main(void) {
  wprintf(L"Hello world!\n");
  int mode = fwide(stdout, 0);
  wprintf(L"Stream is %ls-oriented\n", mode < 0 ? L"byte" : L"wide");
}
```

上面示例中，`wprintf()`将字节流隐式设为宽字符模式，所以`fwide(stdout, 0)`的返回值大于零。

## 宽字符专用函数

下面这些函数基本都是 stdio.h 里面的字符处理函数的宽字符版本，必须使用这些函数来操作宽字符。

- fgetwc()	从宽字符流中获取宽字符，对应 fgetc()。
- fgetws()	从宽字符流中读取宽字符串，对应 fgets()。
- fputwc()	将宽字符写入宽字符流，对应 fputc()。
- fputws()	将宽字符串写入宽字符流，对应 fputs()。
- fwprintf()	格式化宽输出到宽字符流，对应 fprintf()。
- fwscanf()	来自宽字符流的格式化宽字符输入，对应 fscanf()。
- getwchar()	从 stdin 获取一个宽字符，对应 getchar()。
- getwc()	从 stdin 获取一个宽字符，对应 getc()。
- putwchar()	写一个宽字符到 stdout，对应 putchar()。
- putwc()	写一个宽字符到 stdout，对应 putc()。
- swprintf()	格式化宽输出到宽字符串，对应 sprintf()。
- swscanf()	来自宽字符串的格式化宽输入，对应 sscanf()。
- ungetwc()	将宽字符推回输入流，对应 ungetc()。
- vfwprintf()	可变参数的格式化宽字符输出到宽字符流，对应 vfprintf()。
- vfwscanf()	来自宽字符流的可变参数格式化宽字符输入，对应 vfscanf()。
- vswprintf()	可变参数的格式化宽字符输出到宽字符串，对应 vswprintf()。
- vswscanf()	来自宽字符串的可变参数格式化宽字符输入，对应 vsscanf()。
- vwprintf()	可变参数格式化宽字符输出，对应 vprintf()。
- vwscanf()	可变参数的格式化宽字符输入，对应 vscanf()。
- wcscat()	危险地连接宽字符串，对应 strcat()。
- wcschr()	在宽字符串中查找宽字符，对应 strchr()。
- wcscmp()	比较宽字符串，对应 strcmp()。
- wcscoll()	比较两个考虑语言环境的宽字符串，对应 strcoll()。
- wcscpy()	危险地复制宽字符串，对应 strcpy()。
- wcscspn()	不是从宽字符串前面开始计算字符，对应 strcspn()。
- wcsftime()	格式化的日期和时间输出，对应 strftime()。
- wcslen()	返回宽字符串的长度，对应 strlen()。
- wcsncat()	更安全地连接宽字符串，对应 strncat()。
- wcsncmp()	比较宽字符串，长度有限，对应 strncmp()。
- wcsncpy()	更安全地复制宽字符串，对应 strncpy()。
- wcspbrk()	在宽字符串中搜索一组宽字符中的一个，对应 strpbrk()。
- wcsrchr()	从末尾开始在宽字符串中查找宽字符，对应 strrchr()。
- wcsspn()	从宽字符串前面的集合中计算字符，对应 strspn()。
- wcsstr()	在另一个宽字符串中找到一个宽字符串，对应 strstr()。
- wcstod()	将宽字符串转换为 double，对应 strtod()。
- wcstof()	将宽字符串转换为 float，对应 strtof()。
- wcstok()	标记一个宽字符串，对应 strtok()。
- wcstold()	将宽字符串转换为 long double，对应 strtold()。
- wcstoll()	将宽字符串转换为 long long，对应 strtoll()。
- wcstol()	将宽字符串转换为 long，对应 strtol()。
- wcstoull()	将宽字符串转换为 unsigned long long，对应 strtoull()。
- wcstoul()	将宽字符串转换为 unsigned long，对应 strtoul()。
- wcsxfrm()	转换宽字符串以根据语言环境进行比较，对应 strxfrm()。
- wmemcmp()	比较内存中的宽字符，对应 memcmp()。
- wmemcpy()	复制宽字符内存，对应 memcpy()。
- wmemmove()	复制宽字符内存，可能重叠，对应 memmove()。
- wprintf()	格式化宽输出，对应 printf()。
- wscanf()	格式化宽输入，对应 scanf()。

## 多字节字符专用函数

wchar.h 也定义了一些多字节字符的专用函数。

- mbsinit() 判断 mbstate_t 是否处于初始转换状态。
- mbrlen()	给定转换状态时，计算多字节字符串的字节数，对应 mblen()。
- mbrtowc()	给定转换状态时，将多字节字符转换为宽字符，对应 mbtowc()。
- wctombr()	给定转换状态时，将宽字符转换为多字节字符，对应 wctomb()。
- mbsrtowcs()	给定转换状态时，将多字节字符串转换为宽字符串，对应 mbstowcs()。
- wcsrtombs() 给定转换状态时，将宽字符串转换为多字节字符串，对应 wcstombs()。

# wctype.h

wctype.h 提供 ctype.h 里面函数的宽字符版本。

## 宽字符类型判断函数

下面函数判断宽字符的类型。

- iswalnum()	测试宽字符是否为字母数字
- iswalpha()	测试宽字符是否为字母
- iswblank()	测试这是否是一个宽空白字符
- iswcntrl()	测试这是否是一个宽控制字符。
- iswdigit()	测试这个宽字符是否是数字
- iswgraph()	测试宽字符是否是可打印的非空格字符
- iswlower()	测试宽字符是否为小写
- iswprint()	测试宽字符是否可打印
- iswpunct()	测试宽字符是否为标点符号
- iswspace()	测试宽字符是否为空格
- iswupper()	测试宽字符是否为大写
- iswxdigit()	测试宽字符是否为十六进制数字

## wctype()，iswctype()

`iswctype()`是上一节各种宽字符类型判断函数的通用版本，必须与`wctype()`配合使用。

```c
int iswctype(wint_t wc, wctype_t desc);
```

`iswctype()`接受两个参数，第一个参数是一个需要判断类型的宽字符，第二个参数是宽字符类型描述，来自`wctype()`的返回值。

如果宽字符属于指定类型，`iswctype()`返回一个非零值，否则返回零。

`wctype()`用来获取某个种类宽字符的类型描述。

```c
wctype_t wctype(const char* property);
```

`wctype()`的参数是一个给定的字符串，可用的值如下：alnum、alpha、blank、cntrl、digit、graph、lower、print、punct、space、upper、xdigit。

`wctype()`的返回值的类型为 wctype_t，通常是一个整数。如果参数是一个无效值，则返回`0`。

```c
if (iswctype(c, wctype("digit")))
// 相当于
if (iswdigit(c))
```

上面示例用来判断宽字符`c`是否为数值，相当于`iswdigit()`。

`iswctype()`的完整类型判断如下。

```c
iswctype(c, wctype("alnum"))	// 相当于 iswalnum(c)
iswctype(c, wctype("alpha"))	// 相当于 iswalpha(c)
iswctype(c, wctype("blank"))	// 相当于 iswblank(c)
iswctype(c, wctype("cntrl"))	// 相当于 iswcntrl(c)
iswctype(c, wctype("digit"))	// 相当于 iswdigit(c)
iswctype(c, wctype("graph"))	// 相当于 iswgraph(c)
iswctype(c, wctype("lower"))	// 相当于 iswlower(c)
iswctype(c, wctype("print"))	// 相当于 iswprint(c)
iswctype(c, wctype("punct"))	// 相当于 iswpunct(c)
iswctype(c, wctype("space"))	// 相当于 iswspace(c)
iswctype(c, wctype("upper"))	// 相当于 iswupper(c)
iswctype(c, wctype("xdigit"))	// 相当于 iswxdigit(c)
```

## 大小写转换函数

wctype.h 提供以下宽字符大小写转换函数。

- towlower()	将大写宽字符转换为小写
- towupper()	将小写宽字符转换为大写
- towctrans()	宽字符大小写转换的通用函数
- wctrans()	大小写转换的辅助函数，配合 towctrans() 使用

先看`towlower()`和`towupper()`的用法示例。

```c
towlower(L'B') // b
towupper(L'e') // E
```

`towctrans()`和`wctrans()`的原型如下。

```c
wint_t towctrans(wint_t wc, wctrans_t desc);
wctrans_t wctrans(const char* property);
```

下面是它们的用法示例。

```c
towctrans(c, wctrans("toupper"))	// 相当于 towupper(c)
towctrans(c, wctrans("tolower"))	// 相当于 towlower(c)
```
