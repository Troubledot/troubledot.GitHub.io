---
title: 系统学习ES6（let和const命令）
date: 2018-12-03 11:31:02
categories: 
- 技术
- js
- es6
tags: 
- es6
---
### 1、let命令

#### let基本用法

ES6 新增了let命令，用来声明变量。它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效。

```javascript
{
  var a = 1;
  let b = 2;
}
a  // 1
b  // ReferenceError: a is not defined.
```

以上分别用var和let声明了变量a和b，输出b的时候会报错，因为let声明的变量只有在let所在的代码块内有效。

for循环其实就比较适合用let。

```javascript
for (let i = 0;i<10;i++){
  ...
}
i // ReferenceError: i is not defined.
```

这里就比较符合我们的预期：i应该只有在for循环内部起作用；
如果换成var,输出i的时候就是10；

```javascript
for (var i = 0;i<10;i++){
  ...
}
i // 10
```

这里的i是var声明的，在全局范围内有效，所以会输出最后一轮的i；

```javascript
var a = [];
for (var i = 0;i<10;i++){
  a[i] = function(){
    console.log(i)
  }
}
a[2]()   //10
```

这段代码里面a是一个数组，这个数组是由10个funcion组成的，像这样

```javascript
a // [console.log(i),console.log(i),console.log(i),console.log(i),console.log(i),console.log(i),console.log(i),console.log(i),console.log(i),console.log(i)]
```

我们会发现这里的i好像跟前面那个例子里面的i是一样的，其实函数里面console的这个i一直是一个东西，就是那个在全局声明的i，所以会输出最后一轮的i；

```javascript
var a = [];
for (let i = 0;i<10;i++){
  a[i] = function(){
    console.log(i)
  }
}
a[2]()   //2
```

如果换成let就会变成2了，怎么理解呢，let声明的变量只在当前这一轮循环中有用，也就是说，每次进入循环都会声明一次i，而不是像var那样直接取全局的变量。这样解释的话会有个疑问，如果每一轮循环的变量i都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？

> 这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算。

另外，for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

这段代码，第一次循环完成的时候，i被赋值为字符串'abc'，'abc'小于3为false，不满足循环条件，循环应该就结束了。但是事实上依然进了循环。这就很好的印证了上面的结论：

> 函数内部的变量i与循环变量i不在同一个作用域，有各自单独的作用域。

#### 不存在变量提升

所谓的变量提升，就是说对于一个变量，如果在声明之前使用它的话，会有个默认值undefined；

```javascript
console.log(a) // undefined
var a = 3 ;

```

这样容易有一些不确定因素在里面，比如看到一个undefined，它有可能是变量提升的，还有可能它是被我们赋值的undefined；
而使用let的时候呢

```javascript
console.log(a); // Uncaught ReferenceError: a is not defined
let a = 2;
```

显然这样更加符合逻辑，这个报错的意义也更加明确。
所以我感觉let不存在变量提升倒是比较规范的地方，规则越严谨，不确定因素越少。

#### 暂时性死区

只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

上面代码中，存在全局变量tmp，但是块级作用域内let又声明了一个局部变量tmp，导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错。

ES6 明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

> 总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

明白了这个道理，下面这段代码也就很好理解了

```javascript
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

tmp在声明前都是其死区，所以一直报错；

这里我们需要注意一下typeof命令。如果没有用到let的话，typeof随便用；

```javascript
typeof undeclared_variable // "undefined"
```

这里的 undeclared_variable 没有被声明过，我们也可以用tpyeof，返回undefined；

但是如果用到let呢？

```javascript
typeof a // ReferenceError
let a = 'troubledot'
```

现在我们知道**暂时性死区**这个概念了就很好理解了，typeof位于a的暂时性死区内，所以报错也就自然而然了。
> 在没有let之前，typeof运算符是百分之百安全的，永远不会报错。现在这一点不成立了。这样的设计是为了让大家养成良好的编程习惯，变量一定要在声明之后使用，否则就报错。

还有一些比较隐蔽的死区不太容易发现。

```javascript
function bar(x = y, y = 2) {
  return [x, y];
}

bar(); // 报错
```

执行函数bar的时候，给x赋值y，但是y并没有声明，也就是说我们在y的暂时性死区使用了它，所以会报错。说白了咱们认死理就好

> 先声明，再使用

```javascript
function bar(x = 2, y = x) {
  return [x, y];
}
bar(); // [2,2]
```

使用x之前已经声明过x了，所以不会报错

```javascript
// 不报错
var x = x;

// 报错
let x = x;
// ReferenceError: x is not defined
```

上面这种情况也比较有意思，其实我们要理解这个只需要明白变量的声明是 ‘let x = x’这句执行完以后才完成的。也就是说，在执行这一句的过程中遇到了x这样一个变量，此时因为赋值语句还没有执行完，所以这里也算是x的暂时性死区，当然就会报错了。

#### 不允许重复声明

> let不允许在相同作用域内，重复声明同一个变量。

```javascript
// 报错
function func() {
  let a = 10;
  var a = 1;
}

// 报错
function func() {
  let a = 10;
  let a = 1;
}
```

我们注意到，只要用到了let就不能有重复声明，不管重复的变量是不是用let声明的。

```javascript
function func(arg) {
  let arg; // 报错
}

```

不能在函数内部重复声明参数，但是下面这种不会报错

```javascript
function func(arg) {
  {
    let arg; // 不报错
  }
}
```

回忆一下最开始的时候说的，let只在let命令所在的代码块内有效。

### 2、块级作用域

通过对let的了解，看得出来let位JavaScript新增了块级作用域

```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

这个例子能够很好的帮我吗理解块级作用域，let声明的变量，只在自己的作用域内有效，代码块外层不会收到内层的影响。
ES6 允许块级作用域的任意嵌套。

```javascript
{{{{{let insane = 'Hello World'}}}}};
```

上面代码使用了一个五层的块级作用域。外层作用域无法读取内层作用域的变量。

```javascript
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 报错
}}}};
```

外层读取不到内存定义的变量

```javascript
{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}
}}}};
```

内层作用域可以定义外层作用域内的同名变量。

我们知道，立即执行函数就是为了隔离作用域，避免变量污染，所以块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

```javascript
// IIFE 写法
(function () {
  var tmp = ...;
  ...
}());

// 块级作用域写法
{
  let tmp = ...;
  ...
}
```

#### 块级作用域与函数声明

函数能不能在块级作用域之中声明？这是一个相当令人混淆的问题。

ES5 规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明。

```javascript
// 情况一
if (true) {
  function f() {}
}

// 情况二
try {
  function f() {}
} catch(e) {
  // ...
}
```

这两种情况，根据es5的规定，都是非法的。
但是，浏览器没有遵守这个规定，为了兼容以前的旧代码，还是支持在块级作用域之中声明函数，因此上面两种情况实际都能运行，不会报错。

ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。ES6 规定，块级作用域之中，函数声明语句的行为类似于let，在块级作用域之外不可引用。

```javascript
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
```

上面代码在 ES5 中运行，会得到“I am inside!”，因为在if内声明的函数f会被提升到函数头部，实际运行的代码如下。

```javascript
function f() { console.log('I am outside!'); }

(function () {
  function f() { console.log('I am inside!'); }
  if (false) {
  }
  f();
}());
```

ES6 就完全不一样了，理论上会得到“I am outside!”。因为块级作用域内声明的函数类似于let，对作用域之外没有影响。但是，如果你真的在 ES6 浏览器中运行一下上面的代码，是会报错的，这是为什么呢？

原来，如果改变了块级作用域内声明的函数的处理规则，显然会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6 在附录 B里面规定，浏览器的实现可以不遵守上面的规定，有自己的行为方式。

* 允许在块级作用域内声明函数。
* 函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
* 同时，函数声明还会提升到所在的块级作用域的头部。

注意，上面三条规则只对 ES6 的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作let处理。

根据这三条规则，在浏览器的 ES6 环境中，块级作用域内声明的函数，行为类似于var声明的变量。

```javascript
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

上面的代码在符合 ES6 的浏览器中，都会报错，因为实际运行的是下面的代码。

```javascriot
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。

```javascript
  // 函数声明语句
  {
    let a = 'secret';
    function f() {
      return a;
    }
  }

  // 函数表达式
  {
    let a = 'secret';
    let f = function () {
      return a;
    };
  }
```

另外，还有一个需要注意的地方。ES6 的块级作用域允许声明函数的规则，只在使用大括号的情况下成立，如果没有使用大括号，就会报错。

```javascript

// 不报错
'use strict';
if (true) {
  function f() {}
}

// 报错
'use strict';
if (true)
  function f() {}
  
```

### 3.const命令

#### const基本用法

const声明一个只读的常量。一旦声明，常量的值就不能改变。

```javascript
const PI = 3.1415;
PI // 3.1415

PI = 3;
// TypeError: Assignment to constant variable.
```

改变const声明的PI的值就会报错。

const声明的变量不得改变值，这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值。

```javascript
const foo;
// SyntaxError: Missing initializer in const declaration
```

只声明不赋值就会报错。
其他的地方，const和let一样，不能重复声明，存在暂时性死区，只能在声明以后再使用。

#### 本质

const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。

```javascript
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```

这里的foo是一个对象，它存储的其实是一个地址，这个地址指向一个对象。对象本身可以改变的，所以我们给他添加一个属性的时候没有报错，但是如果改变了指向的话，就会报错了。

对象是这样，数组也是。

```javascript
const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
```

当然，有时候或许我们还真的需要这种功能，就是真正的让对象都不能变化，我们可以使用Object.freeze方法。

```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

这里的foo指向一个冻结的对象Object，所以添加新属性不起作用，严格模式时还会报错。

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。

```javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```

#### ES6声明变量的6种方法

ES5声明变量只有var和function两种命令；ES6中除了这两种，还有四种：我们前面提到的let、const，还有import和class命令；

### 4.顶层对象的属性

顶层对象，在浏览器环境指的是window对象，在node指的是global对象。ES5之中，顶层对象的属性与全局变量是等价的

```javascript
window.a = 1;
a. // 1
a = 2;
window.a //2
```

也就是所有我们定义的全局变量，都可以看作是顶层对象的属性；也就是说上面的顶层对象的属性赋值和全局变量的赋值其实是同一件事情。

顶层对象的属性与全局变量挂钩，被认为是 JavaScript 语言最大的设计败笔之一。这样的设计带来了几个很大的问题，首先是没法在编译时就报出变量未声明的错误，只有运行时才能知道（因为全局变量可能是顶层对象的属性创造的，而属性的创造是动态的）；其次，程序员很容易不知不觉地就创建了全局变量（比如打字出错）；最后，顶层对象的属性是到处可以读写的，这非常不利于模块化编程。另一方面，window对象有实体含义，指的是浏览器的窗口对象，顶层对象是一个有实体含义的对象，也是不合适的。

ES6 为了改变这一点：

* 一方面规定，为了保持兼容性，var命令和function命令声明的全局变量，依旧是顶层对象的属性；
* 另一方面规定，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。

```javascript
var a = 1;
// 如果在 Node 的 REPL 环境，可以写成 global.a
// 或者采用通用方法，写成 this.a
window.a // 1

let b = 1;
window.b // undefined
```

这里var声明的全局变量a就是顶层对象的属性，而用let声明的b则不是。

### 5.global对象

ES5 的顶层对象，本身也是一个问题，因为它在各种实现里面是不统一的。

* 浏览器里面，顶层对象是window，但 Node 和 Web Worker 没有window。
* 浏览器和 Web Worker 里面，self也指向顶层对象，但是 Node 没有self。
* Node 里面，顶层对象是global，但其他环境都不支持。

同一段代码为了能够在各种环境，都能取到顶层对象，现在一般是使用this变量，但是有局限性。

* 全局环境中，this会返回顶层对象。但是，Node 模块和 ES6 模块中，this返回的是当前模块。
* 函数里面的this，如果函数不是作为对象的方法运行，而是单纯作为函数运行，this会指向顶层对象。但是，严格模式下，这时this会返回undefined。
* 不管是严格模式，还是普通模式，new Function('return this')()，总是会返回全局对象。但是，如果浏览器用了 CSP（Content Security Policy，内容安全策略），那么eval、new Function这些方法都可能无法使用。

也就是说，我们的js代码有可能因为换了执行环境就出错了
综上所述，很难找到一种方法，可以在所有情况下，都取到顶层对象。下面是两种勉强可以使用的方法。

```javascript
  // 方法一
  (typeof window !== 'undefined'
      ? window
      : (typeof process === 'object' &&
        typeof require === 'function' &&
        typeof global === 'object')
        ? global
        : this);

  // 方法二
  var getGlobal = function () {
    if (typeof self !== 'undefined') { return self; }
    if (typeof window !== 'undefined') { return window; }
    if (typeof global !== 'undefined') { return global; }
    throw new Error('unable to locate global object');
  };
```

有点猥琐。。

现在有一个提案，在语言标准的层面，引入global作为顶层对象。也就是说，在所有环境下，global都是存在的，都可以从它拿到顶层对象。

垫片库system.global模拟了这个提案，可以在所有环境拿到global。

```javascript
// CommonJS 的写法
require('system.global/shim')();

// ES6 模块的写法
import shim from 'system.global/shim'; shim();
```

上面代码可以保证各种环境里面，global对象都是存在的。

```javascript
// CommonJS 的写法
var global = require('system.global')();

// ES6 模块的写法
import getGlobal from 'system.global';
const global = getGlobal();
```

上面代码将顶层对象放入变量global。
