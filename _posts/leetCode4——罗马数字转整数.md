---
title: leetCode4——罗马数字转整数
date: 2019-07-24 23:01:35
categories:
- 技术
- leetCode
tags: 
- leetCode 
- easy 
- 算法
---
题目

罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。

字符  | 数值
:-:  | :-:  |
I   |    1  |
V   |    5  |
X   |    10  |
L   |    50  |
C   |    100  |
D   |    500  |
M   |    1000  |

例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

- I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
- X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。
- C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。

给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。

前面做的那几道题多少还能有点想法，不管是类型转换硬刚，或者暴力循环。
这题自己就没啥想法了。

不知道从何下手，想了挺久，感觉好复杂。不小心看了一眼评论：这题其实很简单，前面比后面的数大就加起来，前面比后面的数小就减。

仔细一看，他说的那所谓的6种情况可不就是一种吗？左边的数小于右边的数。想法有了怎么转成代码呢。

思路：还是哈希表，哈希表的用法真的是特别广泛，先把字符数值对应的关系存进哈希表中，方便查询。

```javascript
var map = {
    'I': 1,
    'V': 5,
    'X': 10,
    'L': 50,
    'C': 100,
    'D': 500,
    'M': 1000
  }
```

循环给定的字符串，当前一位小于后一位的时候累加，否则进行自减。

```javascript
let res = 0
// 这里之所以是s.length - 1 而不是length 是因为要用到s[i+1]
for (let i = 0; i<s.length-1; i++){
  if (map[s[i] < map[s[i+1]]) {
    res += map[s[i]]
  } else {
    res -= map[s[i]]
  }
}
// 我们可以代入具体的数值进行测试，比如IV，map[s[0]] = 1 ,此时res = 0-1 = -1 ； 同样代入‘III’，我们发现累加结果为2，显然这并不是我们想要的结果，我们发现由于要考虑到s[i+1],我们循环的条件为i < length-1，这样的结果就是我们最后会少操作一位，就是s[s.length - 1]，而最右面的这一位不可能是减的，所以我们在结果中需要再加上最后这一位
res+= map[s.length - 1]
```

完整的代码如下：

```javascript
/*
 * @param {string} s
 * @return {number}
 */
var romanToInt = function(s) {
  var map = {
    'I': 1,
    'V': 5,
    'X': 10,
    'L': 50,
    'C': 100,
    'D': 500,
    'M': 1000
  }
  let res = 0
  for (let i = 0; i < s.length - 1; i++) {
    if (map[s[i]] < map[s[i+1]]) {
      res -= map[s[i]]
    } else{
      res += map[s[i]]
    }
  }
  res+=map[s[s.length-1]]
  return res
};
```

![罗马数字转整数](http://res.troubledot.cn/roman.png)

就这个思路，我们一直纠结的length那里有没有什么更好的办法呢？ 这里就发现了一个js的说不上是好还是坏的地方了，弱类型语言的各种隐式类型转换有时候真的会有一些奇技淫巧。

比如说下面的代码：

```javascript
var s = [1,2,3,4]
s[1] > s[2]  // false
s[2] > s[3]  //fasle
s[4] // undefined
s[3] > s[4]  //false
s[3] < s[4]  //false
```

也就是说s[4]虽然是undefined，但是他可以参与比较，虽然咋比都是false，但是它不会报错啊，回到这道题里面

```javascript
for (let i = 0; i < s.length; i++) { //这里就取length
  if(map[s[i]] < map[s[i+1]]){  //就算索引超出也是可以比较的结果为false进入else
    res -= map[s[i]]
  } else{
    res+= map[s[i]]
  } // 这样发现少的那一位再不会漏下了，不过也不难发先这里的if...else..的顺序不能调换，因为比较的那里永远是false
  return res
}
```

这里利用了任何数于NAN比较都返回false的特性，把漏加的最后一位成功补到了else里面，少写了最后一步，我们也可以使用三元运算符让代码更简练。


```javascript
/*
 * @param {string} s
 * @return {number}
 */
var romanToInt = function(s) {
  var map = {
    'I': 1,
    'V': 5,
    'X': 10,
    'L': 50,
    'C': 100,
    'D': 500,
    'M': 1000
  }
  let res = 0
  for (let i = 0; i < s.length; i++) {
    map[s[i]] < map[s[i+1]] ? res -= map[s[i]] : res += map[s[i]]
  }
  return res
};
```

以上，罗马数字转整数 共勉！
