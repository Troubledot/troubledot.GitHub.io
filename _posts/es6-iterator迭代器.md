---
title: es6 iterator迭代器
date: 2019-06-28 11:10:39
categories:
- 技术
- js
- es6
tags: 
- es6
---

iterator迭代器是ES6非常重要的概念，但是很多人对它了解的不多，但是它却是另外4个ES6常用特性的实现基础（解构赋值，剩余/扩展运算符，生成器，for of循环），了解迭代器的概念有助于了解另外4个核心语法的原理，另外ES6新增的Map,Set数据结构也有使用到它。

对于可迭代的数据解构，ES6在内部部署了一个[Symbol.iterator]属性，它是一个函数，执行后会返回iterator对象（也叫迭代器对象），而生成iterator对象[Symbol.iterator]属性叫iterator接口,有这个接口的数据结构即被视为可迭代的

数组中的Symbol.iterator方法(iterator接口)默认部署在数组原型上:

![symbol](http://res.troubledot.cn/1.png)

默认部署iterator接口的数据结构有一下几个，普通对象是默认没有iterator接口的。

* Array
* Map
* Set
* String
* TypeArray（类数组）
* 函数的argument对象
* Nodelist对象

其实interator就是一个对象，又一个叫next的方法，执行以后又返回一个对象，包含两个属性一个done，一个value，done是迭代结束的标志，结束为true否则是false，value为值，迭代结束以后为undefined，我们可以尝试用es5来实现一个interator；

```javascirpt
// es5 实现interator
function crateInterator(items){
  var i = 0
  return {
    next: function() {
      done = i >= items.length
      value = done?undefined:items[i++]
      return {
        done: done,
        value: value
      }
    }
  }
}
var interator = crateInterator([1,2,3])

interator.next()   //{done: false, value: 1}
interator.next()   //{done: false, value: 2}
interator.next()   //{done: false, value: 3}
interator.next()   //{done: true, value: undefined}
```

最后整理下：

* 可迭代的数据结构都有个[Symbol.interator]方法
* [Symbol.interator]执行以后会返回一个interator对象
* interator对象有个next方法
* next方法执行后会返回一个对象，这个对象有两个属性done和value，done是迭代是否结束的标志，结束为true否则为false，value为每次迭代返回的值。
