---
title: C语言——标准库——stdbool.h
date: 2021-10-11 12:45:15
categories: 
  - Programming
  - C 语言
tags: 
  - Programming
  - C 语言
---

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
<!-- more -->

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
