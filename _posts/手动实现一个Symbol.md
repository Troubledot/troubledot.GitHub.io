---
title: 手动实现一个Symbol
date: 2019-07-18 09:03:22
categories: 
- 技术
- js
- es6
tags: 
- 自检清单 
- 前端工程师 
- js
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

#### 5、Symbol的参数如果是一个对象，就会调用该对象的toString方法，将其转换成字符串，然后生成Symbol值。

```js
const obj = {
  toString: function(){
    return 'abc'
  }
}
const s1 = Symbol(obj)
console.log(s1)  //'abc'
```

#### 6、Symbol的参数只是对当前值的一个描述，并不代表Symbol的值，也就是说就算参数相同，这两个Symbol也是不相等的

```js
// 没有参数的情况
Symbol() == Symbol() // false
// 有参数的情况
Symbol('123') == Symbol('123')  //false
```

#### 7、Symbol不能与其他类型的值进行运算否则会报错

```js
var mysymbol = Symbol('123')
console.log('my symbol is' + mysymbol) //TypeError: Cannot convert a Symbol value to a string
console.log(1 + mysymbol) //Cannot convert a Symbol value to a number
```

#### 8、Symbol可以显式的转换为字符串

```js
var symbol = Symbol('mysymbol')
console.log(toString(symbol))  //'Symbol('mysymbol')'
console.log(symbol.toString()) // 'Symbol(mysymbol)'
```

#### 9、Symbol可以作为对象的属性值，保证属性不会冲突

```js
var sys = Symbol()
// 第一种写法
var obj = {}
obj[sys] = 'Troubledot'
// 第二种写法
obj{
  [sys]: 'Troubledot'
}
// 第三种写法
var obj = {}
Object.defineProperty(a, symbol ,{value: 'Troubledot'});
```

#### 10、 
