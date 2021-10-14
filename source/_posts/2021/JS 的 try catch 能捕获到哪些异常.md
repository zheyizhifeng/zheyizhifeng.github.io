---
title: JS 的 try catch 能捕获到哪些异常
date: 2021-10-13 15:49:51
categories: 
  - Programming
  - JavaScript
  - 异常
tags: 
  - Programming
  - JavaScript
  - 异常
---

`try catch` 执行主要分为三段：try catch 之前，之中，之后。

## try catch 之前

代码报错的时候，线程执行未进入 try catch，那么无法捕捉异常。

比如语法异常（syntaxError），因为语法异常是在语法检查阶段就报错了，线程执行尚未进入 try catch 代码块，自然就无法捕获到异常。

例子 1：

```js
try{
    a.
}catch(e){
    console.log("error",e);
}
// Output
// Uncaught SyntaxError: Unexpected token '}'
```

## try catch 之中

代码报错的时候，线程执行处于 try catch 之中，则能捕捉到异常。

看如下例子：

- 方法和执行都在 try 里面，能捕捉到异常。

```js
try{
    function d(){a.b;}
   d();
}catch(e){
    console.log("error",e);
}
// Output
// error ReferenceError: a is not defined
```

- 方法定义在外部，执行方法在 try 里面，能捕捉到异常

```js
function d(){a.b;}
try{
   d();
}catch(e){
     console.log("error",e);
}
// Output
// error ReferenceError: a is not defined
```

上述报错的时机，都是代码执行进入了 try catch ，执行 d() 方法的时候，线程执行处在 try 里面，所以能捕捉到。

## try catch 之后

代码报错的时候，线程已经执行完 try catch，这种不能捕捉到异常。

例子：

```js
try{
    setTimeout(()=>{
         console.log(a.b);  
    }, 100)
}catch(e){
    console.log('error',e);
}
console.log(111);
// Output
// 111
// Uncaught ReferenceError: a is not defined
```

setTimeout 里面报错，实际上是 100ms 之后执行的代码报错，此时代码块 try catch 已经执行完成，111 都已经被执行了，故无法捕捉异常。

例子：

```js
try{
   function d(){a.b;}
}catch(e){
     console.log("error",e);
}
console.log(111);
d();
// Output
// 111
// Uncaught ReferenceError: a is not defined
```

方法定义在 try catch 代码块里面，但是执行方法在 try catch 外，在执行 d 方法的时候报错，此时 try catch 已经执行完成，111 都已经被执行了，故而无法捕捉异常。

## 总结

**能被 try catch 捕捉到的异常，必须是在报错的时候，线程执行已经进入 try catch 代码块，且处在 try catch 里面，这个时候才能被捕捉到。**

如果是在之前，或者之后，都无法捕捉异常。

## Promise VS 异常

相对于外部 try catch，Promise 没有异常！

例子 6：

```js
try{
    new Promise(function (resolve, reject) {
        a.b;
    }).then(v=>{
        console.log(v);
    });
}catch(e){
    console.log('error',e);
}
// output
Uncaught (in promise) ReferenceError: a is not defined
```

看如上报错，线程在执行 a.b 的时候，事实上属于同步执行，try catch 并未执行完成，按理应该能捕捉到异常，这里为啥无法捕捉呢？

事实上，Promise 的异常都是由 reject 和 Promise.prototype.catch 来捕获，不管是同步还是异步。

核心原因是因为 Promise 在执行回调中都用 try catch 包裹起来了，其中所有的异常都被内部捕获到了，并未往上抛异常。

如下来自 Promises/A+ 的实现 then/promise 源码：

```js
function getThen(obj) {
  try {
    return obj.then;
  } catch (ex) {
    LAST_ERROR = ex;
    return IS_ERROR;
  }
}

function tryCallOne(fn, a) {
  try {
    return fn(a);
  } catch (ex) {
    LAST_ERROR = ex;
    return IS_ERROR;
  }
}
function tryCallTwo(fn, a, b) {
  try {
    fn(a, b);
  } catch (ex) {
    LAST_ERROR = ex;
    return IS_ERROR;
  }
}
```

可以看到，这里执行 then (Promise.prototype.then 回调), tryCallTwo (doResolve 回调), tryCallOne (handleResolved 回调) 方法都被 try catch了。

异常都被包裹起来了。所以异常都不会被外层的 try catch 捕捉，因此在外层的 try catch 看来，Promise 根本没有异常，事实上也确实没有“异常”，比如：

```js
try{
    new Promise(function (resolve, reject) {
        a.b;
    }).then(v=>{
        console.log(v);
    });
    console.log(111);
}catch(e){
    console.log('error',e);
}
console.log(222);
// output
111
222
Uncaught (in promise) ReferenceError: a is not defined
```

显然，a.b 报错之后的，111 和 222 都能正常运行，promise 的异常都已经被内部 catch 了，在外层的 try catch 看来就是没有异常，线程继续执行。

try catch 无法捕捉 Promise 的异常，是因为 Promise 的异常没有往上抛。

再看一例：

```js
function a(){
    return new Promise((resolve, reject) =>{
        setTimeout(() => {
            reject(1);
        })
    })
}
try{
    await a();
}catch(e){
    console.log('error',e);
}
console.log(111);
//output
error 1
111
```

这个例子的异常被 catch 捕获到了，那么这里的 Promise 为啥能捕获到异常呢？

我们还是看开始的“一句话总结”

报错的时候( setTimeout 里面的 reject )，线程执行已经进入 try catch 代码块，但是并未执行完成，这样的话当然可以捕获到异常。await 将代码执行停留在 try catch 代码块里面的缘故。

**敲黑板了： 不要用 try catch 包裹 Promise , Promise 很强大，不用担心异常会往上抛！我们只需要给 Promise 增加 Promise#catch 就 OK 了**
