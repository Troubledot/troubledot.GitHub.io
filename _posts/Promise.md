---
title: Promise
date: 2019-07-01 14:44:55
tags: es6 js
---
Promise作为es6中出现的新概念，改变了js的异步编程，现代前端的大多数异步请求都是通过Promise实现的，fetch这个web api也是基于Promise实现的，这里不得简述一下之前统治JS异步编程的回调函数，回调函数有什么缺点，Promise又是怎么改善这些缺点

### 回调函数

js一开始的作用就是操作dom元素，如果多线程改变dom很可能会导致界面紊乱，所以js一开始的设计就是单线程的，但是浏览器是多线程的，这使得JS同时具有异步的操作，即定时器，请求，事件监听等，而这个时候就需要一套事件的处理机制去决定这些事件的顺序，即Event Loop（事件循环），这里不会详细讲解事件循环，只需要知道，前端发出的请求，一般都是会进入浏览器的http请求线程，等到收到响应的时候会通过回调函数推入异步队列，等处理完主线程的任务会读取异步队列中任务，执行回调

在《你不知道的JavaScript》下卷中，这么介绍

使用回调函数处理异步请求相当于把你的回调函数置于了一个黑盒，虽然你声明了等到收到响应后执行你提供的回调函数,可是你并不知道这个第三方库会在什么时候具体会怎么执行回调函数

使用第三方的请求库你可能会这么写:

```javascript
ajax("http://localhost:3000")=>{
  console.log('你扣了1000块钱')
}
```

收到响应以后执行你提供的回调函数，打印字符串，但是如果这个第三方库有类似超时重试的功能，如果这是一个支付功能，那么你被扣的可能就不是1000块钱了。

第二个众所周知的问题就是在回调里面套回调函数，成了可怕的“回调地狱”，代码难懂难维护。

```javascript
ajax("http://localhost:3000")=>{
  ajax("http://localhost:3001")=>{
    ajax("http://localhost:3002")=>{
    }
  }
}
```

还有就是如果第三方库没有提供错误的回调，那么请求失败的一些信息可能就会被吞掉，而你却完全不知情(nodejs提供了err-first风格的回调,即异步操作的第一个回调永远是错误的回调处理,但是你还是不能保证所有的库都提供了发送错误时的执行的回调函数)

1. 多重嵌套，导致回调地狱；
2. 代码跳跃，不符合人类思维模式；
3. 信任问题，不能把回调完全交给第三方来执行，因为我们不知道他到底如何来执行；
4. 第三方库可能没有错误回调；
5. 不清楚回调是否都是异步调用的（可以同步调用ajax，在收到响应前会阻塞整个线程，会陷入假死状态，很不推荐。）

```javascript
xhr.open("GET","/try/ajax/ajax_info.txt",false); //通过设置第三个async为false可以同步调用ajax
```

### Promise

针对回调函数函数的这么多缺点，Promis应运而生，Promise是一个构造函数，通过new关键字来创建一个Promise实例，来看下Promise是如何解决回调函数的这些缺点的。

```javascript
let promise = new Promise((resolve, reject)=>{
  setTimeout(() => {
    resolve('I have been resolved')
  }, 2000);
})

promise.then(res=>{
  console.log(res) //两秒后打印字符串
})
```

**Promise并不是回调函数的衍生，而是两个概念，所以需要将之前的回调函数改为支持Promise的版本，这个过程成为"提升"，或者"promisory"，现代MVVM框架常用的第三方请求库axios就是一个典型的例子，另外nodejs中也有bluebird，Q等。**

#### 1、多重嵌套导致回调地狱

Promise在设计的时候引入了链式调用的概念，每个then方法同是也是一个Promise，所以只要你愿意可以一直then下去。

```javascript
axios.get("http://localhost:3000")
  .then(res => axios.get("http://localhost:3001"))
  .then(res => axios.get("http://localhost:3002"))
  .then(res => axios.get("http://localhost:3003"))
```

配合箭头函数，明显的比之前回调函数的多层嵌套优雅很多

#### 2、代码跳跃，不符合人类思维模式

Promise使我们在写异步的时候使用同步思维，上述代码就是自上而下执行，格式符合正常思维模式

#### 3、信任问题，不能把回调完全交给第三方来执行，因为我们不知道他到底如何来执行

Promise本身是一个状态机，有以下三种状态：

* pending（等待）
* fulfilled（成功）
* rejected（拒绝）

当请求发送没有得到响应的时候是pending状态，得到响应后会resolve（决议）当前这个Promise实例，将它变成fulfilled还是rejected。
当请求发生错误后会执行reject(拒绝)将这个Promise实例变为rejected状态

一个Promise实例的状态只能从pending变成fulfilled或者从pending变成rejected，也就是说一个Promise实例自pending状态改变以后就不能在改变了，也就是说不存在从rejected到fulfilled或者从fulfilled到rejected的情况。

而Promise实例必须主动调用then方法才能从Promise实例中取到值（前提是Promise不是pending状态），这个主动的做法正是解决这一问题的关键，也就是说第三方库做的就是改变Promise的状态，至于响应的值怎么处理就是开发者来做的事情了，而es6的Promise中我们可以主动的去掉用then方法来改变状态，这就实现控制反转，将原来第三方库的控制权移到了自己手上。

```javascript
axios.get("http://localhost:3000") //收到响应后这个promise的状态会变成resolved，但是取到值需要用户主动去调用then方法
axios.get("http://localhost:3000")
      .then(res =>{
        console.log(res) //主动调用then方法取到值再对值进行处理
      })
```

#### 4、第三方库可能没有提供错误回调 catch

Promise的then方法接受两个函数作为参数，一个是响应resolved的回调，一个是响应rejected的回调，其中，第二个函数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

使用Promise在异步请求发送错误的时候，即使没有捕获错误，也不会阻塞主线程的代码（准确的来说，异步的错误都不会阻塞主线程的代码）

```javascript
const promise = new Promise(function(resolve, reject) {
  resolve('ok');
  throw new Error('test');
});
promise
  .then(function(value) { console.log(value) })
  .catch(function(error) { console.log(error) });
// ok

```

上面代码中，Promise 在resolve语句后面，再抛出错误，不会被捕获，等于没有抛出。因为 Promise 的状态一旦改变，就永久保持该状态，不会再变了。

Promise的错误具有冒泡性质，会一直向后传递，知道被捕获，也就是说，错误总会被下一个catch捕获。

```javascript
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```

上面代码中，一共有三个 Promise 对象：一个由getJSON产生，两个由then产生。它们之中任何一个抛出的错误，都会被最后一个catch捕获。

一般来说，不要在then方法里面定义 Reject 状态的回调函数（即then的第二个参数），总是使用catch方法。

如果再then方法中抛出错误，也会被后面的catch捕获到，而使用then的第二个参数就不可以。

```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};

someAsyncThing().then(function() {
  console.log('everything is great');
});

setTimeout(() => { console.log(123) }, 2000);
// Uncaught (in promise) ReferenceError: x is not defined
// 123

```

上面的代码，resolve中x没有定义会报错，所以then方法的回调没有执行，但是两秒会还是会打印出123来，证明promise内部的错误不会影响到外面，会吞掉错误。

#### 5、不清楚回调是否是异步的

Promise在设计的时候保证所有响应的处理回调都是异步调用的，不会阻塞代码的执行，Promise将then方法的回调放入一个叫微任务的队列中（MicroTask），确保这些回调任务在同步任务执行完以后再执行，这部分同样也是事件循环的知识点，有兴趣的朋友可以深入研究一下

对于第三个问题中,为什么说执行了resolve函数后"大部分情况"会进入fulfilled状态呢?考虑以下情况

```javascript
let promise = new Promise(resolve => {
  resolve(Promise.reject('报错了'))
})
setTimeout(() => {
  console.log(promise)
}, 1000);
```

这里用一个定时器再下一轮事件循环中打印promise吗，不然promise的状态一直是pending。

![promise](http://res.troubledot.cn/promise.png)

我们可以看到打印的结果返回的仍然是一个拒绝状态的promise，原因是如果在一个promise的resolve函数中又传入了一个Promise,会展开传入的这个promise，也就是说上面的代码等同于这样：

```javascript
let promise = Promise.reject('报错了')
setTimeout(() => {
  console.log(promise)
}, 1000);
```

日常开发中建议全面使用Promise，配合后面的async和await可以使用同步的形式来写异步代码，并且能够更优雅的实现异步代码顺序执行以及在发生异步的错误时提供更精准的错误信息。
