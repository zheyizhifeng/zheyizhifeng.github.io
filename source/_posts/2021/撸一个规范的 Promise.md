---
title: 撸一个一个规范的 Promise
date: 2021-10-13 17:44:27
categories: 
  - Programming
  - JavaScript
tags: 
  - Programming
  - JavaScript
  - ES 6
  - Promise
---

转载自掘金，原文链接 [点这里查看](https://juejin.cn/post/6844904147884441608?utm_source=gold_browser_extension%3Futm_source%3Dgold_browser_extension)

---
## 前言

关于 `Promise` 原理解析的优秀文章已经屡见不鲜，但是笔者总 **看了就会，一写就废**，为了理一下 `Promise` 的编写思路，从零开始手写一波代码，同时也方便自己日后回顾。

## Promise 的作用

`Promise` 是 `JavaScript` 异步编程的一种流行解决方案，它的出现是为了解决 **回调地狱** 的问题，让使用者可以通过链式的写法去编写写异步代码，具体的用法笔者就不介绍了，大家可以参考阮一峰老师的 [ES6 Promise教程](https://link.juejin.cn/?target=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fpromise)。
<!-- more -->
 
## 课前知识

### 观察者模式

什么是观察者模式：

> 观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个目标对象，当这个目标对象的状态发生变化时，会通知所有观察者对象，使它们能够自动更新。

`Promise` 是基于 **观察者的设计模式** 实现的，`then` 函数要执行的函数会被塞入观察者数组中，当 `Promise` 状态变化的时候，就去执行观察组数组中的所有函数。

### 事件循环机制

实现 `Promise` 涉及到了 `JavaScript` 中的事件循环机制 `EventLoop`、以及宏任务和微任务的概念。

事件循环机制的流程图如下：


![img](https://camo.githubusercontent.com/e25d50e7473d8a8d90bd58b8e3b09c1ec301fd9b9ea25488c9c08e988ba5747a/68747470733a2f2f7374617469632e7675652d6a732e636f6d2f36316566626332302d376362382d313165622d383566362d3666616337376330633962332e706e67)


大家可以看一下这段代码：

```js
console.log(1);

setTimeout(() => {
  console.log(2);
},0);

let a = new Promise((resolve) => {
  console.log(3);
  resolve();
}).then(() => {
  console.log(4);
}).then(() => {
  console.log(5);
});

console.log(6);
```

如果不能一下子说出输出结果，建议大家可以先查阅一下 **事件循环** 的相关资料。
### Promises/A+ 规范

[Promises/A+](https://link.juejin.cn/?target=https%3A%2F%2Fpromisesaplus.com%2F) 是一个社区规范，如果你想写出一个规范的 `Promise`，我们就需要遵循这个标准。之后我们也会根据规范来完善我们自己编写的 `Promise`。

 

## Promise 核心知识点

在动手写 `Promise` 之前，我们先过一下几个重要的知识点。

### executor

```js
// 创建 Promise 对象 x1
// 并在 executor 函数中执行业务逻辑
function executor(resolve, reject){
  // 业务逻辑处理成功结果
  const value = ...;
  resolve(value);
  // 失败结果
  // const reason = ...;
  // reject(reason);
}

let x1 = new Promise(executor);
```

首先 `Promise` 是一个类，它接收一个执行函数 `executor`，它接收两个参数：`resolve` 和 `reject`，这两个参数是 `Promise` 内部定义的两个函数，用来改变状态并执行对应回调函数。

> 因为 `Promise` 本身是不知道执行结果失败或者成功，它只是给异步操作提供了一个容器，实际上的控制权在使用者的手上，使用者可以调用上面两个参数告诉 `Promise` 结果是否成功，同时将业务逻辑处理结果（`value/reason`）作为参数传给 `resolve` 和 `reject` 两个函数，执行回调。

### 三个状态

`Promise` 有三个状态：

- `pending`：等待中
- `resolved`：已成功
- `rejected`：已失败

在 `Promise` 的状态改变只有两种可能：从 `pending` 变为 `resolved` 或者从 `pending` 变为 `rejected`，如下图（引自 Promise 迷你书）：


![引自 Promise 迷你书](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAATAAAACmCAMAAABqbSMrAAABklBMVEX///92w//Aa2ZNrP//T2UAAACRkZHAamX/OVRGqv+VlZWVyf84pf//SWCjo6N6x///jJhotv//aHlasP//XXBxwf+9ZWAqfMZsufZdi8S5XFZvuf/Dc25huv+Gyv/4+PiwyeTFeHNKjMja8P/m5uZlZWWwsLDp6enGxsan1/+4WFIxitTs9/97y/92dnbR0dHu19bWLUTR3u4iOUtFRUUqKirCwsIQEBA5OTmqNS7nxsSzTkhNKymuYVzZQVWX0f+72PFJltlorOEpRFpfnc5CbpCXut6nxOIbGxtVVVWDg4PaqqfQk4/KhYH25+ZmOTaXVFDftbP0xcuGtuRIgL+Vsdfg8v9OgKi03f8OFx82WnZWj7wYKDUWJDD/srypLiZ+RkMpFxb/dYb/oqw+IiHaTV7jhI7vtrw6eb1+pNESaLVWpecAXbLI1elYi7JxseQrUW+Sqb/MZoyxdaSHjszeXn2pLyf/2t8jExOeRD/MWl1nQT5FLSyIPTmtbmoqEA9xNTJZKSZBGhjqmqPid4SgAABi9WO4AAASJUlEQVR4nO2di0Pa2LbGQ+w2U2cU+8pQmmlkBLSAEkHFArG8fQABZSpVsIMOaud4Wjv33ql2zu20vXr+77v2zgsVe6QFIpqvUxN3ApKf31p77b2TKUWZMmXKlClTpkyZMmXKlClTpkyZMtWD4rF8DVqTFWjU69dGf8yv11pp4ZJnPn94Wf3yn/Tq1e++jl5VBxWLey934vN//Bjf/6Jsl9bjwM+/r3X2ujqmWDxxqfOe//PgwU8PmshyRo3Hfnrw03lB+2Pq9T+fd/jCOqVYvHSZ0wK/kLTjGx2NxWIviJ6AnmE9BlnPoGqkdgYbvMUoRY087FGLxeKZy5y28HMAb0abmqolPZDfsGctdklgz18pwL4JVgOwwM+xTl1SWxVLQCT4SiMUHyuVYjwGtkBlICr5gA8fSOC2JsqkSLe21kZgI9265m/SgRcMFfixRC1443FvxoeB8V7oKBegzZfxxr2lph1+JkVSTjuAyb+RXgE24gU3xbyZtUQ8NlryBjRgAe8BFYgfBDLehWYeayMw+RfSK8B83gQOwxFfCbiMeDM8APN54xhYCUCO4F6zmcUWZGC+GweMiscpfmE/QOEhSyZeagCW4TPeTKDU3GE3F1gpHuNLBz4+dhCHJHbgg16SAIMw9R1ACouXmtZHNxdYzFta289A5O0vxErehA8XrhgY5C5fIp4ZCfiaJv2bC4zfj5fiAYi+0lpABpag4t5YYB9CshRfWAssZDqcw2QH9wwwPNrGHeV+PEFC0geD71g8vo9byTZ+ATBSuPJfCalRvQaMio2s4Tp1JDOysJ/hqYUMbhqJlWJQzGYyC7FGXto3GBie3Woy7Fa/s5IvMKi0Wh8/nnis6pmqJ4rW8JxZLwHTxQcwOqVTlHPX6R7Sd+f7FzKyEQAWg8F246xDKpVIVcoVmwysUk5YHiTK5YTV9ubNwZs31gZkpdIZai98PQnsP4r/oe+ejGwEBt9PTs9CpJLBZHU9VAVO4DNbcr1iTbxdD6UtieRmpVq16RGZ2HibOB2Wj0evArC1F0+efd9e3e7ru02QEWCnE1MZoWC6kkrJZGzJaspSXk+Xg2kgmaoG9ROtQDBlPfXaKwDM9+L7R3fu9d1ur/qI7lheLDw8CwzstFmxkaQlAwvabBvJhG0DlZNvU9WNU8CCVw5Y7AHA6uuY7tyJnwe2EUxgDHKeB2AQi4ApkUolN1LVNDROkMMWWyL5NnHFgD2+1zlYsm6PnndYEmOwltMWayVhSQatifX0hK1cVoBZU8m0zWo92AgG1zeuFjDfg07j6mvmMPANhOREEFmsKGhJJi1v0NtEYmOzrDhsHZJcwlJB1SDasFmuErCflGC8ffuHNqrBXZa180nfmgqiEAqlHwTXLRNoExxWQQgl37ytviHArGmUOEBvbclkohJKXylgFpnX7b7HsXau9qnQbltGKep1E2DVYPpNOWWpbgIw4rAySuMEX0mmMbAgsqVQ8mB9YyK1eaWAvVD89X3710ahDuuzkKFM7DywynoZkr1VBpa0BIOWFEpPEGAbFQAWAsNVEwloq6C05QoBk/P9vRcdeGvfDxZlsqeJwzAw0ASEZAKAQVmhAUtCWWFF6xXbhEUGVr5CwJ7JvGIdeXPNtM1CUg60iQ2UrqKgNb2ZwsCgU0xtbCaCVQjJchrGAcFkuYquUkjKBnvS4Z+iAdPHkvtvN/bxYrjt7WYwmLRWqpUEBKKlnEykqwcbVUuiur6ZrFgqm+vB4BUqK0YJsEfNF8faJwxsFC93N4y+9/eVUTj8gd19uQVGijYb/g8KDxuUrlbLARSzVqs6GH9m8OD7SVcMRgXkle9TM4hW66ld66kW0qpuGtofjKpvONLpz9xcFpLBRjv9Y1Rg37zOZjgwUuTf63REXiNgjwiwjv8YFdg331xhOLA7vQZsTX3DkY5/6KYygbUoE1iL6jlgyuAh8OOl7ktrv7oFTL5lk4q1C9jrf1z27u02q0vA+NRDcqVtA5Z5aNCN510CRo28+h0Xe6N4RudbZCHvFkg9NOrZhm4B4xde/fLw4cOfvbp+/CqRRx9e/RLo/Edurm4BwzcXpDLNtfBljZzXa+MeBOkaMEp5jui0uvOD26luAuu63NNOanJu0eW/8IzxobEW3/NaA1tCSzMIhdCi+6Iz+tFAiy6/1sAmEULLYDTkcl5wxgBa5t1fcOB5XXdgLtg4py+02AAa4l3I1YLLrjUwF1rE1hqbRENNj/MQkrOQ6VoJy2sNDHIY3vADGBg/5sRcxpyUvEPxfvf4MhqHA2DCMd4pp3/e6fwivmsNbBr1440MbHZ6bsiJw3R2eqYfw3Ej3CGMU8vTfn9ocmDRNYtJzc6gyYsyHtY1B0ZC0emCrX8pNI2T/xxgcoUgazmXFl1LCIC5UP9QCKGlJQhPaJ1ZWuz/gseuNbA5NIk3/sXpcWoILfsxuGnccU6G3NQsmuTH5wgw9yyUHv4xHMButOycRAMXV2fXHNiMH6zUD2h4nK2WASC08dQ4tCwDPfCen19CfncIDVD89Jx/DKN0h0IXFxrXGtgiQtP97kkEBgN3LQ/NIbfsujHkGluacUOcgrFkYDzFuxbdzjk4d3Zu+oYCQ6jfNYNmcDafnYM0NT3EUxgaBkbQjE/PKA4LQd8wGRpyoiU/OOzCQvfaA/M7x2fH8dXPzs0Mzfr9fgDmx8CW/NMAzB2a8UMO87vRHAaGBvy4Xxj6Uj95rYHhzKQIstXA7JDLpQFzLqP+WahsFWAzMjDsO4jeyRua9F1y4UrkXkTwZwj3iQQY758OzcwsQimxjNx+hAAY1Bd8Py4vbqzDnG49efP+oaFx3Ge6wT28240L/aFx96Sfco6PUbNQglF+KGydQ67l5ZmbmsNOP/qkTFfy2j584cf4M6dDi/vmAmtZY9h/ULzepByGQ+2rtbw47YJKxKW3/GF7dErkfrpHRl9kO+Uf6P96LUPGh3ptuaHpj77zT091/AbELmrM/Q28CLPJ5dMN++eea7nXs//3sib6ZmDnlDjH679Wjb7KdsrfZl5/nDPYf3933+iLbKeg4Bpoo/7Yv3da//Pn4K1bL42+yp6RtD0IvG4NG/05ekfSfeB1a9voj9FDwsAG7xr9KXpIxGEmsMvLdFiLMh3WokyHtSjTYS3KdFiLMoG1KDMkW5TpsBZlOqxFmQ5rUabDWpTpsBZlAmtRZki2KBNYizJDskX1GDDjH4LrWkjyvKApfEYrurK6co3K63LYw1IXPvAF6pbDJPuv734Fvfv1qcfjefru3bunHjtW5CmI1fae4mYW70agkWU9eA+2LE3bI1gsx7GRyIphLuuSwwT2qR0AMAzN5CVeyNE0kyM2kbIcTefJKdAI4sJyI5PFjXw4D4cledcBhxk4zAv5SLbjH/kCdcdhfP4pWIiViWRXOLzHrvC8BLsM7IaBESMfZnPCioM0CpgM2cvymBzepRlOEnLcSo42Kiq74zABxxwtA7F7yJXTjCOsQAAKYYeyR3N5ByPv5oUsyypnCjmFJ8M6aJbNrUTCnf7MF6g7DluJZPPK9Xo8MjmAwtGMvGv32OU9mnPkOaWRtSuMMDGFJ80SnzIc57newMK0FMYgwF6K0XRcxHMyF87hUOylkDkjzaeMnTESWOdDMkwLfA6bRnUS0wQC2MuhHGBZNYIZ/WzGrr2cpD1j1CWHMQIlcHZWzUOsFmwaQ4Z2cJzeqNtLD1GtEfoNQ4F1x2Fy30jswzJaNGroOI5rEo1cgxH1Rm5FMAzY3e4A47JZxSk0XLmKS3MahKOa0ljtOKPjaoxGDnegWcOAbWNg9ztd1IS5nIM94ykcmGpG4xryVEOjCgy6ClY/k3w1LOkPE2CdvstVoIUsweXReTTB1XAcKgcVF6vj0hFCWWuMXmJgHb8FUWAEycHQHg+rd49auGl7jdHo0KNRf5Ea1TTjMQyYRIDdPezsTwFg1Aobacjb2qXT7DkyDKPhovWaltEyGs06coYBk7P+reHOjv6xw3I6Lp2HhgsHnmYkvW+MaBD1l9g9WT5sHLDVwc4Rk7aV5ChwubxWskJ5SssVhF5fMB6PlqccGi9PRI9GLc1FImwWBqeGAaO2ZWLbHYhK6bv7MjKBDudYJdowueLWUbRQ1NI+bddwMToue0QdRkE01moqQzueITISmCQDG7y1vdfu6kL6TvlNQEiSsSRD5/BGjOJnoFBdVE1jV8dAnEN3kkfFhfvGAioShqSRyRoJTA3KW4OD37Vb8vtuH0JZQeUYmskp8zlF9MkuFmRiEI2qpzgHp4Xo+zrdIKaARBKN8ndgUwOBKaVFBzV49y8YSzryWdU+AIzBEIrQN2IIDF0/KkJyU3tJiMYaKij2YvCrCiFRDVGGpWEsaSQwavV+x5ENS9RKnlHH0R8QTkk1VISqQRQxlsJW0UGLcoyK7z0MHMSniMRyYlGMbuFGEqueiLFJH+tw+P6twY4SGxayGi67vRbCNArog4cR66gAnIr/4hjoCmoiwxYQioqQ5qIiXQzVa0rK+yjDBPN5WGOTvqzV4e37OI21T0BJNe728F+5xjlTsXBUo8VC6DNAqIeiqM5x/3tUFOtb0VDB/uFjtIBq0RCq10QELTUw2+fPITnfgb1wBGcNrMM0SXu//TbcRg0O3r8rP8n0GyR9NZ7IzGANesjC1hF2lnhUYOrobzq6JRZRTYx+LEJmw41RJDIQluJRtFioi2Id/Aaw5TRmdNLvjAg0ggsXrnY1GnEKr22hj0coiguFWihaC22xdLQugpE+HUXBfWJxq4iB0VFUgNz/4WOBxsBg7CRjZ5h8nr5+wChchxFcGBiuR/VJwNrRp2LhaKuGe8o6lLAMBlaAACyAl+oiC2UqBraF6p8/iUWI3+JWVPRoU/854SqEZAck/aYMHwR6Jde40vYJMpNYq6Ma9y9UKBZrRTr68f0neReCE8wm4pqjHqp9+FAs1gvwXfS9UlQw+dyK4b1kpyUw4ZxqL7yoViQ1QzEa+luEQdJRqMh8/vi+eLQVPaoXa6EC4ME1B/MBfYyGosXaVnRLHRZA+srlHcK1B0bnlRlXefZUBE/BKPvvGlQOhUINyrHPW8ouHItGoV2siVB+yC2fPsOODCwP6YuBXvIaJv1GCbRdrTrlOUIGRtkOZZwt4sVdCERtHETqV32ymswrym0OeZ0XbHY9k74mQV4xgmGjXV+zVWZsYNgIjVvqSPz0ehJDIrjhNepKCmfgHU9dkEBn84y+lgHRmOeUa4fK/f8KxRpkrQbpSyVs4/qHdm8Bx113YIyQbbz0vGoVNuKxM7WjIwR5X8OlR2PDtDXNaO6zc+GwYXfvdEcCHc7qCxx5PRrlaX6xSDoB3V76IpK2hKkf9gAwoWeAre7u8VBgra7uHUuUsHp4fAyN0u7JHqW1NpFAqxFIopHWorHBP8xZXIzHrq2Fa7wIQya30isheTg/NX8iUavzO/Pzu9Iu3hxT/A60AjiltcnLYGikZp+ctsStTgfSOq2G1cqGaWv9LgFGXgDH6axHgP05f7w3tcuvTs3vHc8f7k7NS/M71O7UifRyfo+SW5v19wK+TQ7jyKkLRixEo+opTl860tCwnnNprPEuA4+jN4DxUyeU9OfO6urUyaEkAaldXuL5qR2eknZ2AdiJIDW9kJWn4RzLsPmcvuytdphsg7v0hTi7R22LeNTXNNyOwka43qjDAAlE5cnh6hSJvN0pnLr2oJGSTk4kpbWZ8tmwgwk71GtncpLAER6c2siwjpy2q0WjersKg2lzeprz9EhIHk/NT03t4OA7xuuYu1OrpHEXA9s5VFqbKfurEOapFeWK8/hW6CxGkeWhQyDiYHdFppMXVpS1tjy5wwDLAS/JqXNq4Ltcb/wTPyfzJyfHhzi9484RUAmUCmznRFBam4nP/1vged7BwpiIzkr4cY6wg83jNiqPb3vKC/jRB7zL4MOkjc1JvLRCbtInLwnjG6Twt5F3+d4wGGHDSy8PD1VgePpmdWoH0vo8dAUXA6Ok/NN/Q+XqwdW+3ePxRCIwTPKQRxU88l/8BIMiskNOkrfKXsSuKtetC/5WSVA2nEztrAIw/Cs+JiFJ7cwfH57gQJ1vXoUR8eF8Vn9C5vSjM8IpSafVw//MFmjv5ORkd5USSPkKVSyZIFz9a2f+r5eU2mrqlKQmTKS9vQ7fL2XKlClTpkyZMmXKlClTpkyZMtVT+n8heCnypVXB4gAAAABJRU5ErkJggg==)



而且需要注意的是一旦状态改变，状态不会再变了，接下来就一直是这个结果。也就是说当我们在 `executor` 函数中调用了 `resolve` 之后，之后调用 `reject` 就没有效果了，反之亦然。

```js
// 并在 executor 函数中执行业务逻辑
function executor(resolve, reject){
  resolve(100);
  // 之后调用 resolve，reject 都是无效的，
  // 因为状态已经变为 resolved，不会再改变了
  reject(100);
}

let x1 = new Promise(executor);
```

### then

每一个 `promise` 都一个 `then` 方法，这个是当 `promise` 返回结果之后，需要执行的回调函数，他有两个可选参数：

- `onFulfilled`：成功的回调；
- `onRejected`：失败的回调；

如下图（引自 Promise 迷你书）：



![引自 Promise 迷你书](data:image/png;base64,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)


```js
// ...
let x1 = new Promise(executor);

// x1 延迟绑定回调函数 onResolve
function onResolved(value){
  console.log(value);
}

// x1 延迟绑定回调函数 onRejected
function onRejected(reason){
  console.log(reason);
}

x1.then(onResolved, onRejected);
```

 
## 手写 Promise 大致流程

在这里我们简单过一下手写一个 `Promise` 的大致流程：

### executor 与三个状态

- `new Promise` 时，需要传递一个 `executor` 执行器函数，在构造函数中，**执行器函数立刻执行**
- `executor` 执行函数接受两个参数，分别是 `resolve` 和 `reject`
- `Promise` 只能从 `pending` 到 `rejected`, 或者从 `pending` 到 `fulfilled`
- `Promise` 的状态一旦确认，状态就凝固了，不在改变

### then 方法

- 所有的 `Promise` 都有 `then` 方法，`then` 接收两个参数，分别是 `Promise` 成功的回调 `onFulfilled`，和失败的回调 `onRejected`
- 如果调用 `then` 时，`Promise` 已经成功，则执行 `onFulfilled`，并将 `Promise` 的值作为参数传递进去；如果 `Promise` 已经失败，那么执行 `onRejected`，并将 `Promise` 失败的原因作为参数传递进去；如果 `Promise` 的状态是 `pending`，需要将 `onFulfilled` 和 `onRejected` 函数存放起来，等待状态确定后，再依次将对应的函数执行(观察者模式)
- `then` 的参数 `onFulfilled` 和 `onRejected` 可以不传，`Promise` **可以进行值穿透**。

### 链式调用并处理 then 返回值

- `Promise` 可以 `then` 多次，`Promise` 的 `then` 方法返回一个新的 `Promise`。
- 如果 `then` 返回的是一个正常值，那么就会把这个结果（`value`）作为参数，传递给下一个 `then` 的成功的回调（`onFulfilled`）
- 如果 `then` 中抛出了异常，那么就会把这个异常（`reason`）作为参数，传递给下一个 `then` 的失败的回调(`onRejected`)
- 如果 `then` 返回的是一个 `promise` 或者其他 `thenable` 对象，那么需要等这个 `promise` 执行完撑，`promise` 如果成功，就走下一个 `then` 的成功回调；如果失败，就走下一个 `then` 的失败回调。

## 第一版（从一个简单例子开始）

> 我们先写一个简单版，这版暂不支持状态、链式调用，并且只支持调用一个 `then` 方法。

### 来个 🌰

```js
let p1 = new MyPromise((resolve, reject) => {
    setTimeout(() => {
      resolved('成功了');
    }, 1000);
})

p1.then((data) => {
    console.log(data);
}, (err) => {
    console.log(err);
})
```

例子很简单，就是 `1s` 之后返回 `成功了`，并在 `then` 中输出。

### 实现

我们定义一个 `MyPromise` 类，接着我们在其中编写代码，具体代码如下：

```js
class MyPromise {
  // ts 接口定义 ...
  constructor (executor: executor) {
    // 用于保存 resolve 的值
    this.value = null;
    // 用于保存 reject 的值
    this.reason = null;
    // 用于保存 then 的成功回调
    this.onFulfilled = null;
    // 用于保存 then 的失败回调
    this.onRejected = null;

    // executor 的 resolve 参数
    // 用于改变状态 并执行 then 中的成功回调
    let resolve = value => {
      this.value = value;
      this.onFulfilled && this.onFulfilled(this.value);
    }

    // executor 的 reject 参数
    // 用于改变状态 并执行 then 中的失败回调
    let reject = reason => {
      this.reason = reason;
      this.onRejected && this.onRejected(this.reason);
    }

    // 执行 executor 函数
    // 将我们上面定义的两个函数作为参数 传入
    // 有可能在 执行 executor 函数的时候会出错，所以需要 try catch 一下 
    try {
      executor(resolve, reject);
    } catch(err) {
      reject(err);
    }
  }

  // 定义 then 函数
  // 并且将 then 中的参数复制给 this.onFulfilled 和 this.onRejected
  private then(onFulfilled, onRejected) {
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
  }
}
```

好了，我们的第一版就完成了，是不是很简单。

> 不过这里需要注意的是，`resolve` 函数的执行时机需要在 `then` 方法将回调函数注册了之后，在 `resolve` 之后在去往赋值回调函数，其实已经完了，没有任何意义。
>
> 上面的例子没有问题，是因为 `resolve(成功了)` 是包在 `setTimeout` 中的，他会在下一个宏任务执行，这时回调函数已经注册了。
>
> 大家可以试试把 `resolve(成功了)` 从 `setTimeout` 中拿出来，这个时候就会出现问题了。

### 存在问题

这一版实现很简单，还存在几个问题：

- 未引入状态的概念

未引入状态的概念，现在状态可以随意变，不符合 `Promise` 状态只能从等待态变化的规则。

- 不支持链式调用

正常情况下我们可以对 `Promise` 进行链式调用：

```js
let p1 = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolved('成功了');
  }, 1000);
})

p1.then(onResolved1, onRejected1).then(onResolved2, onRejected2)
```

- 只支持一个回调函数，如果存在多个回调函数的话，后面的会覆盖前面的

在这个例子中，`onResolved2` 会覆盖 `onResolved1`，`onRejected2` 会覆盖 `onRejected1`。

```js
let p1 = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolved('成功了');
  }, 1000);
})

// 注册多个回调函数
p1.then(onResolved1, onRejected1);
p1.then(onResolved2, onRejected2);
```

## 第二版（实现链式调用）

> 这一版我们把状态的概念引入，同时实现链式调用的功能。

### 加上状态

上面我们说到 `Promise` 有三个状态：`pending`、`resovled`、`rejected`，只能从 `pending` 转为 `resovled` 或者 `rejected`，而且当状态改变之后，状态就不能再改变了。

- 我们定义一个属性 `status`：用于记录当前 `Promise` 的状态
- 为了防止写错，我们把状态定义成常量 `PENDING`、`RESOLVED`、`REJECTED`。
- 同时我们将保存 `then` 的成功回调定义为一个数组：`this.resolvedQueues` 与 `this.rejectedQueues`，我们可以把 `then` 中的回调函数都塞入对应的数组中，这样就能解决我们上面提到的第三个问题。

```js
class MyPromise {
  private static PENDING = 'pending';
  private static RESOLVED = 'resolved';
  private static REJECTED = 'rejected';

  constructor (executor: executor) {
    this.status = MyPromise.PENDING;
    // ...

    // 用于保存 then 的成功回调数组
    this.resolvedQueues = [];
    // 用于保存 then 的失败回调数组
    this.rejectedQueues = [];

    let resolve = value => {
      // 当状态是 pending 是，将 promise 的状态改为成功态
      // 同时遍历执行 成功回调数组中的函数，将 value 传入
      if (this.status == MyPromise.PENDING) {
        this.value = value;
        this.status = MyPromise.RESOLVED;
        this.resolvedQueues.forEach(cb => cb(this.value))
      }
    }

    let reject = reason => {
      // 当状态是 pending 是，将 promise 的状态改为失败态
      // 同时遍历执行 失败回调数组中的函数，将 reason 传入
      if (this.status == MyPromise.PENDING) {
        this.reason = reason;
        this.status = MyPromise.REJECTED;
        this.rejectedQueues.forEach(cb => cb(this.reason))
      }
    }

    try {
      executor(resolve, reject);
    } catch(err) {
      reject(err);
    }
  }
}
```

### 完善 then 函数

接着我们来完善 `then` 中的方法，之前我们是直接将 `then` 的两个参数 `onFulfilled` 和 `onRejected`，直接赋值给了 `Promise` 的用于保存成功、失败函数回调的实例属性。

现在我们需要将这两个属性塞入到两个数组中去：`resolvedQueues` 和 `rejectedQueues`。

```js
class MyPromise {
  // ...

  private then(onFulfilled, onRejected) {
    // 首先判断两个参数是否为函数类型，因为这两个参数是可选参数
    // 当参数不是函数类型时，需要创建一个函数赋值给对应的参数
    // 这也就实现了 透传
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason}

    // 当状态是等待态的时候，需要将两个参数塞入到对应的回调数组中
    // 当状态改变之后，在执行回调函数中的函数
    if (this.status === MyPromise.PENDING) {
      this.resolvedQueues.push(onFulfilled)
      this.rejectedQueues.push(onRejected)
    }

    // 状态是成功态，直接就调用 onFulfilled 函数
    if (this.status === MyPromise.RESOLVED) {
      onFulfilled(this.value)
    }

    // 状态是成功态，直接就调用 onRejected 函数
    if (this.status === MyPromise.REJECTED) {
      onRejected(this.reason)
    }
  }
}
```

#### then 函数的一些说明

- 什么情况下 `this.status` 会是 `pending` 状态，什么情况下会是 `resolved` 状态

这个其实也和事件循环机制有关，如下代码：

```js
// this.status 为 pending 状态
new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve(1)
  }, 0)
}).then(value => {
  console.log(value)
})

// this.status 为 resolved 状态
new MyPromise((resolve, reject) => {
  resolve(1)
}).then(value => {
  console.log(value)
})
```

- 什么是 **透传**

如下面代码，当 `then` 中没有传任何参数的时候，`Promise` 会使用内部默认的定义的方法，将结果传递给下一个 `then`。

```js
let p1 = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolved('成功了');
  }, 1000);
})

p1.then().then((res) => {
  console.log(res);
})
```

因为我们现在还没支持链式调用，这段代码运行会出问题。

#### 支持链式调用

支持链式调用，其实很简单，我们只需要给 `then` 函数最后返回 `this` 就行，这样就支持了链式调用：

```js
class MyPromise {
  // ...
  private then(onFulfilled, onRejected) {
    // ...
    return this;
  }
}
```

每次调用 `then` 之后，我们都返回当前的这个 `Promise` 对象，因为 `Promise` 对象上是存在 `then` 方法的，这个时候我们就简单的实现了 `Promise` 的简单调用。

这个时候运行上面 **透传** 的测试代码了。

但是上面的代码还是存在相应的问题的，看下面代码：

```js
const p1 = new MyPromise((resolved, rejected) => {
  resolved('resolved');  
});

p1.then((res) => {
  console.log(res);
  return 'then1';
})
.then((res) => {
  console.log(res);
  return 'then2';
})
.then((res) => {
  console.log(res);
  return 'then3';
})

// 预测输出：resolved -> then1 -> then2
// 实际输出：resolved -> resolved -> resolved
```

输出与我们的预期有偏差，因为我们 `then` 中返回的 `this` 代表了 `p1`，在 `new MyPromise` 之后，其实状态已经从 `pending` 态变为了 `resolved` 态，之后不会再变了，所以在 `MyPromise` 中的 `this.value` 值就一直是 `resolved`。

这个时候我们就得看看关于 `then` 返回值的相关知识点了。

#### then 返回值

实际上 `then` 都会返回了一个新的 `Promise` 对象。

先看下面这段代码：

```js
// 新创建一个 promise
const aPromise = new Promise(function (resolve) {
  resolve(100);
});

// then 返回的 promise
var thenPromise = aPromise.then(function (value) {
  console.log(value);
});

console.log(aPromise !== thenPromise); // => true
```

从上面的代码中我们可以得出 `then` 方法返回的 `Promise` 已经不再是最初的 `Promise` 了，如下图（引自 Promise 迷你书）：



![引自 Promise 迷你书](https://lh3.googleusercontent.com/proxy/lr0YKuQsKF2WSNXRArldIbv4tf6Gvleo-lnhF-IsNpzkHlefyjdou7u4cvcobhjmk2Up6BM75C5BK9H0yIOhDmSedgL9-iUBoHyuIcJzPLry5lEBwUtFlKE)


> `promise` 的链式调用跟 `jQuery` 的链式调用是有区别的，`jQuery` 链式调用返回的对象还是最初那个 `jQuery` 对象；`Promise` 更类似于数组中一些方法，如 `slice`，每次进行操作之后，都会返回一个新的值。

#### 改造代码

```js
class MyPromise {
  // ...

  private then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason}

    // then 方法返回一个新的 promise
    const promise2 = new MyPromise((resolve, reject) => {
      // 成功状态，直接 resolve
      if (this.status === MyPromise.RESOLVED) {
        // 将 onFulfilled 函数的返回值，resolve 出去
        let x = onFulfilled(this.value);
        resolve(x);
      }

      // 失败状态，直接 reject
      if (this.status === MyPromise.REJECTED) {
        // 将 onRejected 函数的返回值，reject 出去
        let x = onRejected(this.reason)
        reject && reject(x);
      }

      // 等待状态，将 onFulfilled，onRejected 塞入数组中，等待回调执行
      if (this.status === MyPromise.PENDING) {
        this.resolvedQueues.push((value) => {
          let x = onFulfilled(value);
          resolve(x);
        })
        this.rejectedQueues.push((reason) => {
          let x = onRejected(reason);
          reject && reject(x);
        })
      }
    });
    return promise2;
  }
}

// 输出结果 resolved -> then1 -> then2
```

### 存在问题

到这里我们就完成了简单的链式调用，但是只能支持同步的链式调用，如果我们需要在 `then` 方法中再去进行其他异步操作的话，上面的代码就 GG 了。

如下代码：

```js
const p1 = new MyPromise((resolved, rejected) => {
  resolved('我 resolved 了');  
});

p1.then((res) => {
  console.log(res);
  return new MyPromise((resolved, rejected) => {
    setTimeout(() => {
      resolved('then1');
    }, 1000)
  });
})
.then((res) => {
  console.log(res);
  return new MyPromise((resolved, rejected) => {
    setTimeout(() => {
      resolved('then2');
    }, 1000)
  });
})
.then((res) => {
  console.log(res);
  return 'then3';
})
```

上面的代码会直接将 `Promise` 对象直接当作参数传给下一个 `then` 函数，而我们其实是想要将这个 `Promise` 的处理结果传递下去。



## 第三版（异步链式调用）

> 这一版我们来实现 `promise` 的异步链式调用。

### 思路

先看一下 `then` 中 `onFulfilled` 和 `onRejected` 返回的值：

```js
// 成功的函数返回
let x = onFulfilled(this.value);

// 失败的函数返回
let x = onRejected(this.reason);
```

从上面的的问题中可以看出，`x` 可以是一个 **普通值**，也可以是一个 **`Promise`** 对象，普通值的传递我们在 **第二版** 已经解决了，现在需要解决的是当 `x` 返回一个 `Promise` 对象的时候该怎么处理。

其实也很简单，当 `x` 是一个 `Promise` 对象的时候，我们需要进行等待，直到返回的 `Promise` 状态变化的时候，再去执行之后的 `then` 函数，代码如下：

```js
class MyPromise {
  // ...

  private then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason}

    // then 方法返回一个新的 promise
    const promise2 = new MyPromise((resolve, reject) => {
      // 成功状态，直接 resolve
      if (this.status === MyPromise.RESOLVED) {
        // 将 onFulfilled 函数的返回值，resolve 出去
        let x = onFulfilled(this.value);
        resolvePromise(promise2, x, resolve, reject);
      }

      // 失败状态，直接 reject
      if (this.status === MyPromise.REJECTED) {
        // 将 onRejected 函数的返回值，reject 出去
        let x = onRejected(this.reason)
        resolvePromise(promise2, x, resolve, reject);
      }

      // 等待状态，将 onFulfilled，onRejected 塞入数组中，等待回调执行
      if (this.status === MyPromise.PENDING) {
        this.resolvedQueues.push(() => {
          let x = onFulfilled(this.value);
          resolvePromise(promise2, x, resolve, reject);
        })
        this.rejectedQueues.push(() => {
          let x = onRejected(this.reason);
          resolvePromise(promise2, x, resolve, reject);
        })
      }
    });
    return promise2;
  }
}
```

我们新写一个函数 `resolvePromise`，这个函数是用来处理异步链式调用的核心方法，他会去判断 `x` 返回值是不是 `Promise` 对象，如果是的话，就直到 `Promise` 返回成功之后在再改变状态，如果是普通值的话，就直接将这个值 `resovle` 出去：

```js
const resolvePromise = (promise2, x, resolve, reject) => {
  if (x instanceof MyPromise) {
    const then = x.then;
    if (x.status == MyPromise.PENDING) {
      then.call(x, y => {
        resolvePromise(promise2, y, resolve, reject);
      }, err => {
        reject(err);
      })
    } else {
      x.then(resolve, reject);
    }
  } else {
    resolve(x);
  }
}
```

### 代码说明

#### resolvePromise

`resolvePromise` 接受四个参数：

- `promise2` 是 `then` 中返回的 `promise`；
- `x` 是 `then` 的两个参数 `onFulfilled` 或者 `onRejected` 的返回值，类型不确定，有可能是普通值，有可能是 `thenable` 对象；
- `resolve` 和 `reject` 是 `promise2` 的。

#### then 返回值类型

当 `x` 是 `Promise` 的时，并且他的状态是 `Pending` 状态，如果 `x` 执行成功，那么就去递归调用 `resolvePromise` 这个函数，将 `x` 执行结果作为 `resolvePromise` 第二个参数传入；

如果执行失败，则直接调用 `promise2` 的 `reject` 方法。

 
到这里我们基本上一个完整的 `promise`，接下来我们需要根据 [Promises/A+](https://link.juejin.cn/?target=https%3A%2F%2Fpromisesaplus.com%2F) 来规范一下我们的 `Promise`。


## 规范 Promise

前几版的代码笔者基本上是按照规范来的，这里主要讲几个没有符合规范的点。

### 规范 then（规范 2.2）

> `then` 中 `onFulfilled` 和 `onRejected` 需要异步执行，即放到异步任务中去执行（规范 2.2.4）

#### 实现

我们需要将 `then` 中的函数通过 `setTimeout` 包裹起来，放到一个宏任务中去，这里涉及了 `js` 的 `EventLoop`，大家可以去看看相应的文章，如下：

```js
class MyPromise {
  // ...

  private then(onFulfilled, onRejected) {
    // ...
    // then 方法返回一个新的 promise
    const promise2 = new MyPromise((resolve, reject) => {
      // 成功状态，直接 resolve
      if (this.status === MyPromise.RESOLVED) {
        // 将 onFulfilled 函数的返回值，resolve 出去
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch(err) {
            reject(err);
          }
        })
      }

      // 失败状态，直接 reject
      if (this.status === MyPromise.REJECTED) {
        // 将 onRejected 函数的返回值，reject 出去
        setTimeout(() => {
          try {
            let x = onRejected(this.reason)
            resolvePromise(promise2, x, resolve, reject);
          } catch(err) {
            reject(err);
          }
        })
      }

      // 等待状态，将 onFulfilled，onRejected 塞入数组中，等待回调执行
      if (this.status === MyPromise.PENDING) {
        this.resolvedQueues.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              resolvePromise(promise2, x, resolve, reject);
            } catch(err) {
              reject(err);
            }
          })
        })
        this.rejectedQueues.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason)
              resolvePromise(promise2, x, resolve, reject);
            } catch(err) {
              reject(err);
            }
          })
        })
      }
    });
    return promise2;
  }
}
```

#### 使用微任务包裹

但这样还是有一个问题，我们知道其实 `Promise.then` 是属于微任务的，现在当使用 `setTimeout` 包裹之后，就相当于会变成一个宏任务，可以看下面这一个例子：

```js
var p1 = new MyPromise((resolved, rejected) => {
  resolved('resolved');
})

setTimeout(() => {
  console.log('---setTimeout---');
}, 0);

p1.then(res => {
  console.log('---then---');
})

// 正常 Promise：then -> setTimeout
// 我们的 Promise：setTimeout -> then
```

输出顺序不一样，原因是因为现在的 `Promise` 是通过 `setTimeout` 宏任务包裹的。

我们可以改进一下，使用微任务来包裹 `onFulfilled` 、`onRejected`，常用的微任务有 `process.nextTick`、`MutationObserver`、`postMessage` 等，我们这个使用 `postMessage` 改写一下：

```js
// ...
if (this.status === MyPromise.RESOLVED) {
  // 将 onFulfilled 函数的返回值，resolve 出去
  // 注册一个 message 事件
  window.addEventListener('message', event => {
    const { type, data } =  event.data;

    if (type === '__promise') {
      try {
        let x = onFulfilled(that.value);
        resolvePromise(promise2, x, resolve, reject);
      } catch(err) {
        reject(err);
      }
    }
  });
  // 立马执行
  window.postMessage({
    type: '__promise',
  }, "http://localhost:3001");
}

// ...
```

实现方法很简单，我们监听`window` 的 `message` 事件，并在之后立马触发一个 `postMessage` 事件，这个时候其实 `then` 中的回调函数已经在微任务队列中了，我们重新运行一下例子，可以看到输出的顺序变为了 `then -> setTimeout`。

> 当然 `Promise` 内部实现肯定没有这么简单，笔者在这里只是提供一种思路，大家有兴趣可以去研究一波。

### 规范 resolvePromise 函数（规范 2.3）

#### 重复引用

> 重复引用，当 `x` 和 `promise2` 是一样的，那就需要报一个错误，重复应用。（规范 2.3.1）
>
> **因为自己等待自己完成是永远都不会有结果的。**

```js
const p1 = new MyPromise((resolved, rejected) => {
  resolved('我 resolved 了');  
});

const p2 = p1.then((res) => {
  return p2;
});
```



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/4/171dee30c47e8637~tplv-t2oaga2asx-watermark.awebp)



#### x 的类型

大致分为一下这么几条：

- 2.3.2：当 `x` 是一个 `Promise`，那么就等待 `x` 改变状态之后，才算完成或者失败（这个也属于 `2.3.3`，因为 `Promise` 其实也是一个 `thenable` 对象）
- 2.3.3：当 `x` 是一个对象 或者 函数的时候，即 `thenable` 对象，那就那 `x.then` 作为 `then`
- 2.3.4：当 `x` 不是一个对象，或者函数的时候，直接将 `x` 作为参数 `resolve` 返回。

我们主要看一下 `2.3.3` 就行，因为 `Prmise` 也属于 `thenable` 对象，那什么是 `thenable` 对象呢？

> 简单来说就是具有 **then**方法的对象/函数，所有的 `Promise` 对象都是 `thenable` 对象，但并非所有的 `thenable` 对象并非是 `Promise` 对象。如下：
>
> ```js
> let thenable = {
>  then: function(resolve, reject) {
>    resolve(100);
>  }
> }
> ```

根据 `x` 的类型进行处理：

- 如果 `x` 不是 `thenable` 对象，直接调用 `Promise2` 的 `resolve`，将 `x` 作为成功的结果；
- 当 `x` 是 `thenable` 对象，会调用 `x` 的 `then` 方法，成功后再去调用 `resolvePromise` 函数，并将执行结果 `y` 作为新的 `x` 传入 `resolvePromise`，直到这个 `x` 值不再是一个 `thenable` 对象为止；如果失败则直接调用 `promise2` 的 `reject`。

```js
if (x != null && (typeof x === 'object' || typeof x === 'function')) {
  if (typeof then === 'function') {
    then.call(x, (y) => {
      resolvePromise(promise2, y, resolve, reject);
    }, (err) => {
      reject(err);
    })
  }
} else {
  resolve(x);
}
```

#### 只调用一次

> 规范（`Promise/A+ 2.3.3.3.3`）规定如果同时调用 `resolvePromise` 和 `rejectPromise`，或者对同一参数进行了多次调用，则第一个调用优先，而所有其他调用均被忽略，确保只执行一次改变状态。

我们在外面定义了一个 `called` 占位符，为了获得 `then` 函数有没有执行过相应的改变状态的函数，执行过了之后，就不再去执行了，主要就是为了满足规范。

##### x 为 Promise 对象

如果 `x` 是 `Promise` 对象的话，其实当执行了`resolve` 函数 之后，就不会再执行 `reject` 函数了，是直接在当前这个 `Promise` 对象就结束掉了。

##### x 为 thenable 对象

当 `x` 是普通的 `thenable` 函数的时候，他就有可能同时执行 `resolve` 和 `reject` 函数，即可以同时执行 `promise2` 的 `resolve` 函数 和 `reject` 函数，但是其实 `promise2` 在状态改变了之后，也不会再改变相应的值了。其实也没有什么问题，如下代码：

```js
// thenable 对像
{
 then: function(resolve, reject) {
   setTimeout(() => {
     resolve('我是thenable对像的 resolve')；
     reject('我是thenable对像的 reject')
    })
 }
}
```

#### 完整的 resolvePromise

完整的 `resolvePromise` 函数如下：

```js
const resolvePromise = (promise2, x, resolve, reject) => {
  if(x === promise2){
    return reject(new TypeError('Chaining cycle detected for promise'));
  }
  let called;
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      let then = x.then;
      if (typeof then === 'function') {
        then.call(x, y => {
          if(called)return;
          called = true;
          resolvePromise(promise2, y, resolve, reject);
        }, err => {
          if(called)return;
          called = true;
          reject(err);
        })
      } else {
        resolve(x);
      }
    } catch (e) {
      if(called)return;
      called = true;
      reject(e); 
    }
  } else {
    resolve(x);
  }
}
```

最后我们可以通过测试脚本跑一下我们的 `MyPromise` 是否符合规范。

### 测试

有专门的测试脚本（`promises-aplus-tests`）可以帮助我们测试所编写的代码是否符合 `Promise/A+` 的规范。

> 但是貌似只能测试 `js` 文件，所以笔者就将 `ts` 文件转化为了 `js` 文件，进行测试

在代码里面加上：

```js
// 执行测试用例需要用到的代码
MyPromise.deferred = function() {
  let defer = {};
  defer.promise = new MyPromise((resolve, reject) => {
      defer.resolve = resolve;
      defer.reject = reject;
  });
  return defer;
}
```

需要提前安装一下测试插件：

```sh
# 安装测试脚本
npm i -g promises-aplus-tests

# 开始测试
promises-aplus-tests MyPromise.js
```

结果如下：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/4/171dee469816c2f6~tplv-t2oaga2asx-watermark.awebp)



完美通过，接下去我们就可以看看 `Promise` 更多方法的实现了。

 

## 更多方法

实现上面的 `Promise` 之后，其实编写其实例和静态方法，相对来说就简单了很多。

### 实例方法

#### Promise.prototype.catch

##### 实现

其实这个方法就是 `then` 方法的语法糖，只需要给 `then` 传递 `onRejected` 参数就 `ok` 了。

```js
private catch(onRejected) {
  return this.then(null, onRejected);
}
```

##### 例子：

```js
const p1 = new MyPromise((resolved, rejected) => {
  resolved('resolved');
})

p1.then((res) => {
  return new MyPromise((resolved, rejected) => {
    setTimeout(() => {
      rejected('错误了');
    }, 1000)
  });
})
.then((res) => {
  return new MyPromise((resolved, rejected) => {
    setTimeout(() => {
      resolved('then2');
    }, 1000)
  });
})
.then((res) => {
  return 'then3';
}).catch(error => {
  console.log('----error', error);
})

// 1s 之后输出：----error 错误了
```

#### Promise.prototype.finally

##### 实现

`finally()` 方法用于指定不管 `Promise` 对象最后状态如何，都会执行的操作。

```js
private finally (fn) {
  return this.then(fn, fn);
}
```

##### 例子

```js
const p1 = new MyPromise((resolved, rejected) => {
  resolved('resolved');
})

p1.then((res) => {
  return new MyPromise((resolved, rejected) => {
    setTimeout(() => {
      rejected('错误了');
    }, 1000)
  });
})
.then((res) => {
  return new MyPromise((resolved, rejected) => {
    setTimeout(() => {
      resolved('then2');
    }, 1000)
  });
})
.then((res) => {
  return 'then3';
}).catch(error => {
  console.log('---error', error);
  return `catch-${error}`
}).finally(res => {
  console.log('---finally---', res);
})

// 输出结果：---error 错误了" -> ""---finally--- catch-错误了
```

 

### 静态方法

#### Promise.resolve

##### 实现

有时需要将现有对象转为 `Promise` 对象，`Promise.resolve()`方法就起到这个作用。

```js
static resolve = (val) => {
  return new MyPromise((resolve,reject) => {
    resolve(val);
  });
}
```

##### 例子

```js
MyPromise.resolve({name: 'darrell', sex: 'boy' }).then((res) => {
  console.log(res);
}).catch((error) => {
  console.log(error);
});

// 输出结果：{name: "darrell", sex: "boy"}
```

#### Promise.reject

##### 实现

`Promise.reject(reason)` 方法也会返回一个新的 `Promise` 实例，该实例的状态为 `rejected`。

```js
static reject = (val) => {
  return new MyPromise((resolve,reject) => {
    reject(val)
  });
}
```

##### 例子

```js
MyPromise.reject("出错了").then((res) => {
  console.log(res);
}).catch((error) => {
  console.log(error);
});

// 输出结果：出错了
```

#### Promise.all

`Promise.all()`方法用于将多个 `Promise` 实例，包装成一个新的 `Promise` 实例，

```js
const p = Promise.all([p1, p2, p3]);
```

- 只有 `p1`、`p2`、`p3` 的状态都变成 `fulfilled`，`p` 的状态才会变成 `fulfilled`；
- 只要 `p1`、`p2`、`p3` 之中有一个被 `rejected`，`p` 的状态就变成 `rejected`，此时第一个被 `reject` 的实例的返回值，会传递给`p`的回调函数。

##### 实现

```js
static all = (promises: MyPromise[]) => {
  return new MyPromise((resolve, reject) => {
    let result: MyPromise[] = [];
    let count = 0;

    for (let i = 0; i < promises.length; i++) {
      promises[i].then(data => {
        result[i] = data;
        if (++count == promises.length) {
          resolve(result);
        }
      }, error => {
        reject(error);
      });
    }
  });
}
```

##### 例子

```js
let Promise1 = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('Promise1');
  }, 2000);
});

let Promise2 = new MyPromise((resolve, reject) => {
  resolve('Promise2');
});

let Promise3 = new MyPromise((resolve, reject) => {
  resolve('Promise3');
})

let Promise4 = new MyPromise((resolve, reject) => {
  reject('Promise4');
})

let p = MyPromise.all([Promise1, Promise2, Promise3, Promise4]);

p.then((res) => {
  // 三个都成功则成功  
  console.log('---成功了', res);
}).catch((error) => {
  // 只要有失败，则失败 
  console.log('---失败了', err);
});

// 直接输出：---失败了 Promise4
```

#### Promise.race

`Promise.race()`方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。

```js
const p = Promise.race([p1, p2, p3]);
```

只要 `p1`、`p2`、`p3` 之中有一个实例率先改变状态，`p` 的状态就跟着改变。那个率先改变的 `Promise` 实例的返回值，就传递给 `p` 的回调函数。

##### 实现

```js
static race = (promises) => {
  return new Promise((resolve,reject)=>{
    for(let i = 0; i < promises.length; i++){
      promises[i].then(resolve,reject)
    };
  })
}
```

##### 例子

例子和 `all` 一样，调用如下：

```js
// ...

let p = MyPromise.race([Promise1, Promise2, Promise3, Promise4])

p.then((res) => { 
  console.log('---成功了', res);
}).catch((error) => {
  console.log('---失败了', err);
});

// 直接输出：---成功了 Promise2
```

#### Promise.allSettled

此方法接受一组 `Promise` 实例作为参数，包装成一个新的 `Promise` 实例。

```js
const p = Promise.race([p1, p2, p3]);
```

只有等到所有这些参数实例都返回结果，不管是 `fulfilled` 还是 `rejected`，而且该方法的状态只可能变成 `fulfilled`。

> 此方法与 `Promise.all` 的区别是 `all` 无法确定所有请求都结束，因为在 `all` 中，如果有一个被 `Promise` 被 `rejected`，`p` 的状态就立马变成 `rejected`，有可能有些异步请求还没走完。

##### 实现

```js
static allSettled = (promises: MyPromise[]) => {
  return new MyPromise((resolve) => {
    let result: MyPromise[] = [];
    let count = 0;
    for (let i = 0; i < promises.length; i++) {
      promises[i].finally(res => {
        result[i] = res;
        if (++count == promises.length) {
          resolve(result);
        }
      })
    }
  });
}
```

##### 例子

例子和 `all` 一样，调用如下：

```js
let p = MyPromise.allSettled([Promise1, Promise2, Promise3, Promise4])

p.then((res) => {
  // 三个都成功则成功  
  console.log('---成功了', res);
}, err => {
  // 只要有失败，则失败 
  console.log('---失败了', err);
})

// 2s 后输出：---成功了 (4) ["Promise1", "Promise2", "Promise3", "Promise4"]
```

 
