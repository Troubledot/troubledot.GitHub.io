---
title: leetCode2——整数反转
date: 2019-07-21 12:32:50
categories:
- 技术
- leetCode
tags: 
- leetCode 
- easy 
- 算法
---

题目

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

示例 1:

```js
输入: 123
输出: 321
```

 示例 2:

```js
输入: -123
输出: -321
示例 3:
```

输入: 120
输出: 21
注意:

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−231,  231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

反转首先想到的是数组的reverse方法，数字转字符串再转成数组，reverse以后再转回来，当然最后return的时候需要判断下是否溢出了。

```js
/**
 * @param {number} x
 * @return {number}
 */
var reverse = function(x) {
  let res
  let y = Math.abs(x)
  const str = String(y)
  const arr = str.split('')
  const newarr = arr.reverse()
  let val = newarr.join('')*1
  if(val < 2**31){
    res = x>0?val:-val
  } else{
    res = 0
  }
  return res
};
```

下面是提交结果：

![数组reverse提交结果](http://res.troubledot.cn/reverse.png)

通过了，但是我感觉这道题考察的显然不是数组的reverse方法，再想想别的办法。

如果不使用reverse的话，我们其实可以把字符串从后往前遍历push进一个数组再转回来。

```js
/**
 * @param {number} x
 * @return {number}
 */
var reverse = function(x) {
  const y = Math.abs(x)
  const str = String(y)
  let arr = []
  let res
  for(let i = str.length - 1; i >= 0; i--){
    arr.push(str[i])
  }
  const val = arr.join('')*1
  if(val < 2**31){
    res = x > 0 ? val : -val
  } else {
    res = 0
  }
  return res
};
```

看下提交结果:
![字符串倒序遍历提交结果](http://res.troubledot.cn/reverse1.png)
内存没咋变，但是执行速度提高了点，超过92.47%了都。
这是我现有知识能想到的两种方法了，接下来我开始看评论里的大神们怎么写的。
看了才知道，原来整数的倒序是有算法的，而这道题真正要考察的也正是那个倒序的算法。

```js
/**
 * @param {number} x
 * @return {number}
 */
var reverse = function(x) {
  let y = Math.abs(x)
  let result = 0
  let res
  while (y) {
    const digit = y%10
    y = Math.floor(y/10)
    result = result * 10 + digit
  }
  if (result < 2**31){
    res = x > 0 ? result : -result
  } else {
    res = 0
  }
  return res
};
```

果然，看下结果吧
![倒序算法](http://res.troubledot.cn/reverse2.png)

倒序算法，假设要需要倒序的数为想x：

1. 把x除以10取余，也就是拿到其个位数digit，1234的话digit为4；
2. 将x除以10取整，也就是拿到除个位数外的其他位数赋值给他自身，此时1234变成了123；
3. 结果从0开始加，*result = result * 10 + digit*也就是 0x10+4 = 4；
4. x大于0就继续执行上面的操作，digit = 123%10 = 3，x再除以10取整为12，此时的result为 4x10+3=43；x为12仍然需要继续执行，12%10 = 2，12/10取整为1，result = 43x10+2 = 432；此时x=1还需要执行，x%10商0余1,1/10取整结果为0，result = 432x10+1 = 4321；x=0了，while执行结束了。

以上 leetcode 整数反转， 共勉！
