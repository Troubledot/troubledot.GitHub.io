---
title: 系统学习ES6（变量的解构赋值）
date: 2018-12-12 11:24:45
tags: 
- es6
---
### 1.数组的解构赋值

#### 基本用法

> ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

以前的变量赋值都是直接指定值：

```javascript
let a = 1;
let b = 2;
```

ES6里可以写成这样。

```javascript
let [a,b] = [1,2]
```

表示我们可以从数组中提取对应值，按照对应的位置对变量赋值。

本质上，这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。下面是一些使用嵌套数组进行解构的例子。

其实这种匹配模式比较好理解，我记得上学的时候有学过一个比较系数法，就是这个意思；

```javascript
let [foo,[[bar],baz]] = [1,[[2],3]];
foo // 1;
bar // 2;
baz // 3;

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

如果解构不成功，变量的值就是undefined；

```javascript
let [b] = [];
let [a,b] = [];
```

上面这两种情况下，b的值都是undefined；

还有一种情况是不完全解构，就是说等号左右两边不一定一一对应，这样也能解构成功；

```javascript
let [x,y] = [1,2,3,4,5];
x //1
y//2

let [a,[b,c],d] = [1,[2],3]
a //1
b //2
c //undefined
d //3
```

上面的情况都属于不完全解构，但是可以成功；

如果等号右边不是数组的话，会报错。

```javascript
// 报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
```

#### 默认值

> 解构赋值允许指定默认值

```javascript
let [foo = true] = [];
foo //true
let [x,y = 'b'] = ['a']  //x = 'a' y = 'b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'

```

**注意，ES6 内部使用严格相等运算符（===），判断一个位置是否有值。所以，只有当一个数组成员严格等于undefined，默认值才会生效。**

上呢判断y等于undefined，默认值生效，y = ‘b’；

```javascript
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];
x // null
```

这里x = null，null不严格等于undefined，所以默认值没有生效，x = null；

如果默认值是一个表达式，那么这个表达式是惰性求值的，也就是说如果要用到的时候才会求值；

```javascript
function f(){
  console.log('aaa')
}
let [x=f()] = [1];
```

这里x不用执行f就能取到值，所以函数不会执行，上面的代码等价于：

```javascript
function f(){
  console.log('aaa')
}
if ([1][0] === undefined){
  f()
}
else{
  x = [1][0]
}
```

默认值也可以引用解构赋值的其他变量，但是该变量必须被声明；

```javascirpt
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError: y is not defined
```

这里又复习了暂时性死区的概念。

### 2.对象的解构赋值

明白了匹配的话对象的解构赋值也就很好理解了；

```javascript
let {name,age} = {name:'troubledot',age:28}
name //'troubledot'
age // 28

let {age,name} = {name:'troubledot',age:28}
name //'troubledot'
age // 28

let {sex} = {name:'troubledot',age:28};
sex //undefined
```

不难发现，数组的解构赋值是用索引来匹配的，对象的解构赋值是用key来匹配的；

```javascript
let {name:me} = {name:'troubledot'}
me // 'troubledot'
```

在对象的解构赋值中，对象的属性是一个连通变量和值的桥梁。打个比方就是我的年龄是a，我的年龄是28，那么a = 28，而年龄就是连通a和28的桥梁，年龄是一种**模式**，真正被赋值的不是年龄，而是a；

嵌套结构的对象也可以解构；

```javascript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;
x // "Hello"
y // "World"
```

这里的p对应的是我的比方里面的年龄，被赋值的不是它而是x和y；

如果需要对p赋值，应该这样写

```JavaScript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};
let {p} = obj;

p //['Hello',{ y: 'World' }]

```

这里只对p进行了解构，如果我们还是需要x和y的话，需要对x和y再进行解构；

```JavaScript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};
let {p,p:[x,{y}]} = obj;

p //['Hello',{ y: 'World' }]
x // "Hello"
y // "World"
```

```javascript
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start // Object {line: 1, column: 5}
```

上面代码分别对loc，start和line进行了解构；

看一个对象嵌套赋值的例子；

```javascript
let obj = {};
let arr = [];
({name:obj.name,age:arr[0]} = {name:'troubledot',age:28})
//obj = {name:'troubledot'}
//arr = [28]
```

对象的解构也可以指定默认值；

```javascript
var {x = 3} = {};
x // 3

var {x, y = 5} = {x: 1};
x // 1
y // 5

var {x: y = 3} = {};
y // 3

var {x: y = 3} = {x: 5};
y // 5

var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"
```

同样当对象的属性值严格等于undefined的时候，默认值生效；

```javascript
var {x = 3} = {x: undefined};
x // 3

var {x = 3} = {x: null};
x // null
```

null不严格等于undefined，所以x不等于其默认值。

解构失败，变量的值等于undefined，如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么将会报错。

```javascript
let {a} = {b:3}
a //undefined

//报错
let {name:{firstname}} = {age:28}
```

这个报错，我们倒着看的时候会比较好理解；

```JavaScript
let me = {age:28};
me.name = undefined;
me.name.firstname //报错
```

由于JavaScript 引擎会把大括号包裹的部分视为一个代码块处理，所以我们在对已经声明的对象进行解构的时候需要特别注意，下面这段代码会因为把{a}当作代码块处理而发生语法错误

```javascript
let a;
{a} = {a:20}
// SyntaxError: syntax error
```

解决这个问题我们只需要不在行首出现括号就好

```javascript
let a;
({a} = {a:20})
// SyntaxError: syntax error
```

解构赋值允许等号左边的模式之中，不放置任何变量名。因此，可以写出非常古怪的赋值表达式。

```javascript
({} = [true, false]);
({} = 'abc');
({} = []);
```

上面的表达式虽然毫无意义，但是语法是合法的，可以执行。

对象的解构赋值可以很方便地将现有对象的方法，赋值到某个变量。

我们可以这么理解

```中文编程
我：{
  姓名：张三，
  年龄：28，
  性别：男，
  宠物：{
    狗：{
      名字：’薯条‘，
      品种：’金毛‘
    }
  }，
  技能：写js
}

// 现在有一个王六
 {年龄，性别，技能}  = 我
//我们就知道王六也是个28岁的男性，也会写js 。这里相当于我们解构了年龄性别和技能

{宠物}  = 我

// 这样的话我们只解构了宠物这个属性，我们也知道王六有只宠物狗。品种和名字我们都不知道。
{
宠物：{
  狗：{
      名字：’毛毛‘，
      品种：’泰迪‘
    }
  }
} = 我；
//这样的话我们就知道了王六的狗子是一只叫毛毛的泰迪。

```

这是自创的中文编程。

明白了上面的道理我们就好理解下面的代码了；

```javascript
let { log, sin, cos } = Math;
```

这里我们给log，sin，cos赋值，分别等于Math.log，Math.sin，Math.cos，这样在使用的时候会比较方便；

由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构。

```javascript
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```

### 3.字符串的解构赋值

我们可以把字符串当作数组来处理

```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

所以类似数组的对象都有个length属性，我么也可以对其进行解构赋值

```javascript
let {length : len} = 'hello';
len//5
```

### 4.数组和布尔值的解构赋值

解构赋值的时候如果等号右边是布尔值和数值，会先转化为对象。

```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```

上面的这两个例子，toString是一个模式，对s解构赋值的时候我们，需要找123和true的toString属性，他们本身没有toString属性，但是他们的原型有，所以s能取到值；如果改成这样

```javascript
let {otherfn: s} = 123;
s // undefined

let {otherfn: s} = true;
s // undefined
```

这里的s就是undefined了，因为123和true都没有ohterfn属性，他们的原型也没有。这样的话下面这个例子就很好理解了

```javascript
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```

null和undefined不能转换成对象，所以会报错。

### 5.函数参数的解构赋值

```javascript
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```

add的参数是一个数组，但是在执行的时候解构成了变量x和y；

```JavaScript
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]
```

函数参数的解构也可以使用默认值。

```javascript
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

上面函数的参数是一个对象，函数执行的时候对参数进行解构，如果解构失败就取默认值。

注意，下面的写法会得到不一样的结果。

```javascript
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

这里的默认值是参数对象的默认值，不是x和y的默认值。

move({x: 3}) 的时候其实相当于move({x: 3,y:undefined});

同理 move({}); // [undefined, undefined]也就很好理解了。

move()的时候，相当于参数等于undefined，此时取其默认值。 move(); // [0, 0]

> undefined会触发函数参数的默认值

```javascript
[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```

### 6.圆括号问题

解构赋值虽然很方便，但是解析起来并不容易。对于编译器来说，一个式子到底是模式，还是表达式，没有办法从一开始就知道，必须解析到（或解析不到）等号才能知道。

由此带来的问题是，如果模式中出现圆括号怎么处理。ES6 的规则是，只要有可能导致解构的歧义，就不得使用圆括号。

但是，这条规则实际上不那么容易辨别，处理起来相当麻烦。因此，建议只要有可能，就不要在模式中放置圆括号。

#### 不能使用圆括号的情况

以下三种情况解构赋值不得使用圆括号

##### 1. 变量声明语句

```javascript
// 全部报错
let [(a)] = [1];

let {x: (c)} = {};
let ({x: c}) = {};
let {(x: c)} = {};
let {(x): c} = {};

let { o: ({ p: p }) } = { o: { p: 2 } };
```

变量声明语句，模式不能使用小括号；

##### 2.函数参数

函数参数其实也属于变量声明

```javascript
// 报错
function f([(z)]) { return z; }
// 报错
function f([z,(x)]) { return x; }
```

##### 3.赋值语句的模式

```javascript
// 全部报错
({ p: a }) = { p: 42 };
([a]) = [5];

// 报错
[({ p: a }), { x: c }] = [{}, {}];
```

不管是整个模式放在小括号中，还是将部分模式放在小括号中都会报错

#### 可以使用圆括号的情况

通过上面我们基本已经清楚可以使用圆括号的情况：

> 赋值语句的非模式部分可以使用圆括号

```javascript
[(b)] = [3]; // 正确
({ p: (d) } = {}); // 正确
[(parseInt.prop)] = [3]; // 正确
```

这三句都可以执行，我们来看下，第一行模式是数组的第一个元素跟b没关系，所以可以用圆括号，第二行模式是p，所以d用圆括号没有问题，第三行跟第一行是一样的。

### 7.用途

#### (1) 交换变量的值

```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];
```

如果没有解构赋值，我们要交换变量的值需要一个中间量，但是现在可以直接解构赋值。

#### （2）从函数返回多个值

函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。有了解构赋值，取出这些值就非常方便

```javascript
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

#### （3）函数参数的定义

解构赋值可以方便的将一组参数与变量名对应起来。

```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

#### (4）提取 JSON 数据

前后端分离的项目中，我们经常需要提取json数据的值，使用解构赋值会变得特别方便

```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

#### （5）函数参数的默认值

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

指定参数的默认值，就避免了在函数体内部再写var foo = config.foo || 'default foo';这样的语句。

#### （6）遍历 Map 结构

任何部署了 Iterator 接口的对象，都可以用for...of循环遍历。Map 结构原生支持 Iterator 接口，配合变量的解构赋值，获取键名和键值就非常方便。

```javascript
const map = new Map();
map.set('first','hello');
map.set('second','world');
for (let [key,value] of map){
  key +'is'+ value
}
// first is hello
// second is world
```

也可以只取键和值；

```javascript
const map = new Map();
map.set('first','hello');
map.set('second','world');
for (let [key] of map){
  console.log(key)
}
for (let [,value] of map){
  console.log(value)
}

```

#### （7）输入模块的指定方法

加载模块时，往往需要指定输入哪些方法。解构赋值使得输入语句非常清晰。

某些按需引用的方法也是这个原理

```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");
import { Button, Table } from 'iview';
```
