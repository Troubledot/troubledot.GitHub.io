---
title: 手动实现一个Symbol
date: 2019-07-18 09:03:22
tags: 自检清单 前端工程师 js
---

ES6中新引入了一种原始数据类型symbol，表示一个独一无二的值。我们可以通过了解它的特性，尝试去手动实现一个symbol。

### Symbol特性

#### 1、Symbol通过Symbol()函数生成，使用typeof操作符返回结果是"symbol"

```javascript
var s = Symbol()
consle.log(typeof s)  //symbol
```

#### 2、Symbol函数前不能使用new命令，否则会报错。这是因为生成的Symbol是一个原始类型的值，不是一个对象

#### 3、instanceof结果为false

```javascript
var s = Symbol()
console.log(s instanceof Symbol)  //false
```

#### 4、Symbol接受一个字符串作为参数来表示对其的描述，用以在打印或者转为字符串的时候便与区分

```javascript
var s1 = Symbol('s1')
var s2 = Symbol('s2')
console.log(s1))  // Symbol(s1)
console.log(s2)). // Symbol(s2)
```
