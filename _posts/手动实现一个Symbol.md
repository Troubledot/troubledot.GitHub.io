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

#### 2、Symbol函数前不能使用new命令，否则会报错

这是因为生成的Symbol是一个原始类型的值，不是一个对象

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

#### 5、Symbol的参数如果是一个对象，就会调用该对象的toString方法，将其转换成字符串，然后生成Symbol值

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

#### 10、Symbol作为属性名，该属性不会出现在for...in，for...of中，也不会被Object.keys()、Object.getOwnPropertyNamens、JSON.stringify()返回。但是也不是私有属性，有一个Object.getOwnpropertySymbols方法，可以获取指定对象的所有Symbol属性名

```js
var obj = {}
var a = Symbol('a')
var b = Symbol('b')
obj[a] = 'Hello'
obj[b] = 'World'

var objectSymbol = Object.getOwnPropertySymbols(obj)
console.log(objectSymbol) //[Symbol('a'), Symbol('b')]

```

### 11、如果我们希望使用同一个Symbol值，可以用for，它接受一个字符串作为参数，然后搜索有没有以这个字符串作为key值得Symbol，有的话就返回，否则就新建并返回以该字符串为名称的Symbol值

```js
var a = Symbol.for('foo')
var b = Symbol.for('foo')
a == b // true
```

### 12、Symbol的keyfor方法返回已登记的Symbol类型的key值

```js
var s1 = Symbol.for('foo')
console.log(s1.keyfor())  //'foo'
var s2 = Symbol('foo')
console.log(s2.keyfor())  //undefined
```

了解了Symbol的特性，实现一个Symbol就是去实现这些特性。
先看下再规范中调用Symbol的时候都做了哪些工作：

1. If NewTarget is not undefined, throw a TypeError exception.
2. If description is undefined, var descString be undefined.
3. Else, var descString be ToString(description).
4. ReturnIfAbrupt(descString).
5. ReturnIfAbrupt(descString).

翻译下 ：

1. 如果使用 new ，就报错
2. 如果 description 是 undefined，让 descString 为 undefined
3. 否则 让 descString 为 ToString(description)
4. 如果报错，就返回
5. 返回一个新的唯一的 Symbol 值，它的内部属性 [[Description]] 值为 descString

按照Symbol的特性一条条往下写。  、·   

```js
(function(){
  var root = this
  var Mysymbol = function Symbol(description){
    // 1.使用typeof的时候返回Symbol，无法修改typeof的结果。放弃
    // 2.使用new关键字的时候报错 可以实现
    if (Mysymbol instanceof Mysymbol) throw new TypeError('Symbol is not a constructor')
    // 3.instanceof结果为false，不是由new生成的，所以这条不用专门去实现
    // 4、Symbol接受一个字符串作为参数来表示对其的描述，用以在打印或者转为字符串的时候便与区分，因为我们在模拟的时候返回的是一个对象，实现不了。
    //5、如果Symbol的参数是一个对象，调用toString方法转换成字符串在返回Symbol
    var descString = description === undefined ? undefined : description.toString()
    var symbol = Object.create(null)
    Object.defineProperties(symbol,{
      '__Description__': {
        
      }
    })
    
  }
})()
```
