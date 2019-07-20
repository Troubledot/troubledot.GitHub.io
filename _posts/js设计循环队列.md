---
title: js设计循环队列
date: 2019-07-17 09:21:36
categories:
- 技术
- js
- 基础
tags: 
- js 
- 算法 
---
循环队列是一种线性数据结构，其操作基于FIFO（先进先出）原则并且队尾被连接在队首形成一个循环。它也被称为“环形缓冲器”。循环队列的一个好处是我们可以利用这个队列之前用过的空间。在一个普通队列里面，一旦队列满了，我们就不能再插入元素，即使再队列前面还有空间。但是在循环队列里面，我们就能继续利用前面的这些空间去存储新的值。

设计一个队列，实现应该支持一下操作：

* **MyCircularQueue(k)**：构造器，设置队列长度为k。
* **Front**：从队首获取元素，如果队列为空则返回-1。
* **Rear**：从队尾获取元素，如果队列为空则返回-1。
* **enQueue(value)**：向循环队列插入一个元素。如果成功则返回真。
* **deQueue()**：从循环队列中删除一个元素，如果成功则返回真。
* **isEmpty()**：判断队列是否为空。
* **isFull()**：判断队列是否满了。

> 示例

```javascript
MycircularQueue  circularQueue = new MrcircularQueue(3)    //设置队列长度为3
circularQueue.enQueue(1) //向队列中插入1 返回true
circularQueue.enQueue(2)  //向队列中插入2 返回true
circularQueue.enQueue(3)  //向队列中插入3 返回true
circularQueue.enQueue(4)  //返回false因为队列已经满了
circularQueue.Rear()  //返回3
circularQueue.isFull()  //返回true
circularQueue.deQueue()  //返回true从队首删掉一个元素
circularQueue.enQueue(4)  //像队列中新增一个元素，因为之前删掉了一个元素，所以队列中有位置可以插入，返回true
circularQueue.Rear()  //返回4
```

下面是js代码实现：

```JavaScript
var MycircularQueue = function(k){
  this.size = k
  this.head = -1
  this.tail = -1
  this.data = []
}

MycircularQueue.prototype.enQueue(value){
  if (this.isFull()){
    return false
  }
  if (this.isEmpty()){
    this.head = 0
  }
  this.tail = (this.tail+1)%this.size
  this.data[this.tail] = value
  return true
}

MycircularQueue.prototype.deQueue(){
  if (!this.isEmpty()){
    if(this.head===this.tail){
      this.head = -1
      this.tail = -1
    } else {
      this.head = (this.head+1)%this.size
    }
  }
  return false
}

MycircularQueue.prototype.Front(){
  return this.isEmpty()?-1:this.data[this.head]
}

MycircularQueue.prototype.Rear(){
  return this.isEmpty()?-1:this.data[this.tail]
}

MycircularQueue.prototype.isFull(){
  return (this.tail+1)%this.size === this.head
}

MycircularQueue.prototype.isEmpty(){
  return this.head === -1 && this.tail === -1
}

MycircularQueue.prototype.isEmpty(){
  return this.head === this.tail === -1
}
```

这其中有几个地方有过疑惑，一个是Front和Rear那里
在取Front和Rear的时候，我判断了队列是否为空，空则返回-1，否则返回对应值，如下。

```javascript
MycircularQueue.prototype.Front(){
  return this.isEmpty()?-1:this.data[this.head]
}
```

我看了有些人是这么写的，Front的时候判断head值是否等于-1，Rear的时候判断tail是否为-1，也就是判断是否有队首或者队尾，有则返回，无则返回-1。诚然，在这道题下，两种方法其实并无区别，因为队列是根本不可能存在有头无尾或者有尾无头的情况的。但是从这个区别中也能反应出一种思维习惯，就是编程中比较常见的思想：模块和独立。队首我们只关心head，队尾只关心tail，这样能保证所有的方法高度独立，是一个值得去刻意培养的习惯。代码如下：

```javascript
MycircularQueue.prototype.Front(){
  return this.head === -1?-1:this.data[this.head]
}
```

Rear同理。

还有一处区别就是：我把普通的&&写成了连等的形式。

```javascript
MycircularQueue.prototype.isEmpty(){
  return this.head === this.tail === -1
}
```

还有一个地方就是isFull的时候，因为自己初识算法，思维比较固化。

```javascript
MycircularQueue.prototype.isFull(){
  return this.tail+1 = this.size
}
```

现在回头来看错误就比较明显了，循环算法head不可能永远在第一位的。

以上就是如何用js实现一个循环队列。 共勉
