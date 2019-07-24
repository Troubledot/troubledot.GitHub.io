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

按照Symbol的特性一条条往下写。

1、使用typeof的时候返回Symbol，无法修改typeof的结果。放弃！

2、使用new关键字的时候报错 可以实现

  ```js
  (function(){
  var root = this
  var Generatesymbol = function Symbol(description){
    if (this instanceof Generatesymbol) throw new TypeError('Symbol is not a constructor')
    return symbol
  }
  root.Generatesymbol = Generatesymbol
  })()
  ```

3、instanceof结果为false，不是由new生成的，所以这条不用专门去实现。

4、Symbol接受一个字符串作为参数来表示对其的描述，用以在打印或者转为字符串的时候便与区分，因为我们在模拟的时候返回的是一个对象，实现不了。

5、如果Symbol的参数是一个对象，调用toString方法转换成字符串在返回Symbol，如果参数是undefined直接返回undefined，否则对参数执行toString方法再返回，用一个三元表达式就好

```js
  var descString = description === undefined ? undefined : description.toString()
```
  
6、因为我们模拟的symbol是一个对象，每次都要创建一个新对象，引用不同，再加上后面会增加了一个__Name__属性，每次都不同，所以即使参数相同返回值也不同

7、Symbol不能参与运算，否则会报错。

  这里我们以‘+’运算为例，在做‘+’运算的时候，先执行了valueOf方法进行了隐式类型转换再进行运算的，所以我们可以在对Symbol执行valueOf()的时候抛出个错误就行。
  
  ```js
  var symbol = Object.create({
    valueOf:function(){
      throw new Error('Cannot convert a Symbol value')
    }
  })
  ```

  但是这样的结果就是symbol无法显式的调用valueOf方法，因为一调用就会报错，看下实际上不是这样的

  ![symbolvalueof](http://res.troubledot.cn/symbolvalueof.png)
  
  我们能看到symbol执行valueOf方法的时候并没有报错，而是返回了这个symbol值，也就是它本身。所以我们为了使其不能参与运算就改变了它的valueOf方法显然是不合理的，在能否参与运算和它显式调用valueOf方法之中显然后者要更加重要，所以我们可以舍弃它不能参与运算的特性而来实现它的valueOf方法。
  
  ```js
  var symbol = Object.create({
    valueOf: function(){
      return this
    }
  })
  ```

8、我们模拟的symbol最后返回的是一个对象，对象显式调用toString，我们在控制台里打印下发现长这样[object,object]，显然不是我们想要的结果，所以我们可以给该对象绑定一个toString方法，P拼出我们想要的值'Generatesymbol('+__Description__+')',返回即可。

```js
var symbol = Object.create({
  toString: function(){
    return 'Generatesymbol('+__Description__+')'
  }
})
```

9、symbol作为对象的属性可以保证不会出现重名属性，要作为对象的属性，需要先调用toString方法进行隐式类型转换，而在第8条中我们给它绑定的toString方法返回的是字符串'Generatesymbol('+__Description__+')'，这样的话，只要传入的参数相同，那么返回值就是相等的字符串，它作为属性名的时候就不能避免重名。也就是说这一条和第8条是冲突的，我们也需要做取舍，显然作为对象属性不重名这几乎是symbol诞生的初衷，肯定是要比调用toString返回字符串重要。我们可以舍弃第8条来实现这一条，保证唯一性，创建一个生成唯一标识的方法generateName。

```js
  //每次加入一个flag值保证每次拿到的值不同
 var generateName = function(){
    var flag = 0
    return function(descString){
      flag++
      return 'troubledot' + descString + '_' + flag
    }
  }
```

然后改造toString方法，保证在symbol在类型转换以后返回的字符串每次都不相同。当然此时已经无法满足第8条了。

```js
  Object.defineProperties(symbol,{
    '__Description__': {
      value: descString,
      writable: false,
      enumerable: false,
      configurable: false
    },
    '__Name__': {
      value: generateName(descString),
      writable: false,
      enumerable: false,
      configurable: false
    },
  })
  var symbol = Object.create({
    toString: function(){
      return this.__Name__
    }
  })
```

10、Symbol 作为属性名，该属性不会出现在 for...in、for...of 循环中，也不会被 Object.keys()、Object.getOwnPropertyNames()、JSON.stringify() 返回。但是，它也不是私有属性，有一个 Object.getOwnPropertySymbols 方法，可以获取指定对象的所有 Symbol属性名。 这条实现不了

11、使用for可以访问相同的symbol，前两天整数求和里面的哈希表，把创建的symbol都存在一个哈希表里面，for的时候去搜索，有的话直接返回，没有就存起来。

```js
var symbolMap = {}
Object.defineProperties(Generatesymbol, {
  for:{
    value: function(description){
      var descString = description == undefined ? undefined : String(description)
      return symbolMap[descString] ? symbolMap[descString] : symbolMap[descString] = Generatesymbol(descString)
    },
    writable: true,
    enumerable: false,
    configurable: true
  }
})

```

12、keyFor返回一个已经登记的symbol类型的key，遍历symbolMap，返回key即可

```js
var symbolMap = {}
Object.defineProperties(Generatesymbol,{
  keyFor:{
    value: {
      function(symbol){
        for (var key in symbolMap){
          if (symbolMap[key] === symbol)
          {
            return key
          }
        }
      }
    },
    writable: true,
    enumerable: false,
    configurable: true
  }
})
```

下面看下完整实例

```js
(function(){
  var root = this
  var generateName = (function(){
    var flag = 0
    return function(name){
      flag++
      return 'troubledot'+ name +'_'+ flag
    }
  })()
  var Generatesymbol = function Symbol(description){
    var descString = description == undefined ? undefined : String(description)
    if (this instanceof Generatesymbol) {
      throw new TypeError('Symbol is not a constructor')
    }
    var symbol = Object.create({
      toString : function(){
        return this.__Name__
      },
      valueOf: function(){
        return this
      }
    })
    Object.defineProperties(symbol, {
      __Description__:{
        value: descString,
        writable: false,
        enumerable: false,
        configurable: false
      },
      __Name__:{
        value: generateName(descString),
        writable: false,
        enumerable: false,
        configurable: false
      }
    })
    return symbol
  }
  var symbolMap = {}
    Object.defineProperties(Generatesymbol, {
      'for': {
        value: function(description){
          var descString = description == undefined ? undefined : String(description)
          return symbolMap[descString] ? symbolMap[descString] : symbolMap[descString] = Generatesymbol(description)
        },
        writable: true,
        enumerable: false,
        configurable: true
      },
      'keyFor': {
        value: function(symbol){
          for (var key in symbolMap) {
            if (symbolMap[key] === symbol){
              return key
            }
          }
        },
        writable: true,
        enumerable: false,
        configurable: true
      }
    })
  root.Generatesymbol = Generatesymbol
})()
```

手动实现Symbol，有好多基础的API需要学习，共勉！
