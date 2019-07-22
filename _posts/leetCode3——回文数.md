---
title: leetCode3——回文数
date: 2019-07-22 22:13:12
categories:
- 技术
- leetCode
tags: 
- leetCode 
- easy 
- 算法
---

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

示例 1:

```js
输入: 121
输出: true
```

示例 2:

```js
输入: -121
输出: false
```

解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
示例 3:

```js
输入: 10
输出: false
```

解释: 从右向左读, 为 01 。因此它不是一个回文数。

我自己的思路，数字转成字符串，长度为len，当该数为偶数位数时，从左往右看前len/2位和从右往左看后len/2位相同，如1221前两位位‘12’，后两位也是‘12’；如果位数为奇数时，前(len+1)/2位和后(len+1)/2位相同，如12321，前三位位‘123’，后三位也是‘123’。

代码如下：

```js
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
    const str = String(x)
    const len = str.length
    const times = len % 2 == 0 ? len/2 : (len+1)/2
    let head = ''
    let tail = ''
    let res
    console.log(len, times)
    for (let index = 0; index < times; index++) {
      head += str[index] 
    }
    for (let index = len-1; index >= len - times; index--) {
      tail += str[index]
    }
    console.log(head, tail)
    res = head === tail
    return res

};

```

结果如下：

![可怜的5%](http://res.troubledot.cn/palindrome.png)

如果不转字符串还有一种办法，昨天刚做过的整数反转，负数肯定不是回文数，正数反转一下看是否和原来的数字相同就知道是否是回文数了。

```js
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
  let y = x;  //复制一个x
  let z = 0;  //反转需要累加的基数
  let res
  if (x < 0) {
    res = false
  } else {
    while(y){
      const digit = y % 10
      y = Math.floor(y/10)
      z = z * 10 + digit
    }
    res = z === x
  }
  return res
}
```

结果比转字符串好多了

![反转比较](http://res.troubledot.cn/palindrome1.png)

最后看看下别人咋写的吧。。

同样是转字符串有的人这么写，不是多厉害的算法，这个写法让人眼前一亮。

```js
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
  return x == x.toString().split('').reverse().join('')
}
```

这个没看明白，应该很厉害

```js
var isPalindrome = function(x) {
  let w = x,
      y = 0;

    while(w > 0) {
      let z = w % 10;
      y *= 10;
      y += z;
      w -= z;
      w /= 10;
    }
    return x === y;
};
```

这个配合讲解能看明白，反转一半就能知道该数是不是回文数了，巧妙的地方在于while的条件，如何知道已经反转了一半了，其实所谓的*10和/10说白了就是多一位和少一位，在while中，x每次少一位，要反转的数每次在多一位，当要反转的数大于x的时候，就是要反转的数位数要大于原始数了，此时说明反转的数已经超过一半了，所以while条件就是原始数大于反转数，这样就保证了反转不会超过一半。。太巧妙了这里。

```js
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
  let res
  if (x < 0 ||(x % 10 === 0 && x !== 0)){
    res = false
  } else {
    let y = 0
    while( x > y){
      y = y * 10 + x % 10
      x = Math.floor(x / 10)
    }
    res = x == y || x == y / 10 
  }
  return res
}


```

我感觉或许有些东西需要背下来的，包括前面的反转，和今天这个神奇的算法。

以上，判断回文数，共勉！
