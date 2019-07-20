---
title: ztree的自定义编辑
date: 2018-08-29 11:10:04
categories:
- 组件整理
tags:
- ztree
---
组织架构树是项目中特别常见的一类组件，我一般都用[ztree](http://www.ztree.me/)，前两天又用到这个，不过需求跟平时有点区别，我感觉也算比较常见，就把他稍微整理一下。

需求大致是这样：一个普通的树结构，点击编辑。
![](http://res.troubledot.cn/edit.png)

会弹出来操作按钮，而且并不是所有用户都有增删改的权限，对该节点不具备的权限会置灰，如下图所示。

![](http://res.troubledot.cn/WX20180830-122042@2x.png)

我们知道ztree是带编辑功能的，但是长这样子

![](http://res.troubledot.cn/ztree.png)

而且ztree的官方demo的编辑按钮是在鼠标hover到节点以后才弹出来的，显然跟我们的需求不符合。我看了下代码，发现删除和编辑用的是这两个API

```javascript 
 editName(node)    //参数为要操作的节点
 removeNode (node, callbackFlag) //参数为要操作的节点 
```

鼠标的hover其实不是伪类:hover,而是调用了 addHoverDom 和 removeHoverDom这两个API，目的是拿到要操作的节点。再看我们的需求，发现我们需要所有的节点（ztree对象的getNodes()方法可以拿到所有节点），因为我们需要让每个节点的操作按钮都显示出来。
基本思路：遍历所有节点，然后操作该节点对应的dom，拼接操作按钮图标（这里需要判断是否有权限，根据权限拼接不同的图标），再给按钮绑定事件，调用对应操作的API；

 代码如下，因为还有地方要调用，所以我将其封装成了方法
 
 ```javascript
 function addnewBtn(nodeList) {
    var newCount = 1;
    var zTree = $.fn.zTree.getZTreeObj("tree");
    nodeList.forEach(function(element, index) {
        var sObj = $("#" + element.tId + "_span");
        if (zTree.setting.edit.enable) {     //判断当前是否可以编辑
            var addable = (element.addable == undefined) || (element.addable == true) ? "addable" : ""; //新增权限 权限增加了一个able的标记，根据class取不同图片，下面绑定事件也是绑在该class上 下同
            var editable = (element.editable == undefined) || (element.editable == true) ? "editable" : "";   //编辑权限
            var removeable = (element.removeable == undefined) || (element.removeable == true) ? "removeable" : "";  //删除权限
            if (element.editNameFlag || $("#addBtn_" + element.tId).length > 0 || $("#remove_" + element.tId).length > 0 || $("#edit_" + element.tId).length > 0) return;  //有按钮就不再加了
            var addStr = "<span class='button add " + addable + "' id='addBtn_" + element.tId +
                "' title='add node' onfocus='this.blur();'></span>"; //添加按钮
            var editStr = "<span class='button edit " + editable + "' id='edit_" + element.tId +"' title='rename' onfocus='this.blur();'></span>"; //编辑按钮
            var delStr = "<span class='button remove " + removeable + "' id='remove_" + element.tId + "' title='remove' onfocus='this.blur();'></span>"; //删除按钮

            function allstr() {
                var allstr;
                if (element.isParent) {
                    allstr = addStr + editStr + delStr;
                } else {
                    allstr = editStr + delStr;
                }
                return allstr;
            }   // 子节点不需要新增按钮
            sObj.after(allstr()); //拼接html
            if ((element.addable == undefined) || (element.addable == true)) {
                var btn = $("#addBtn_" + element.tId);  //取到有权限的添加按钮
            }
            if (btn) btn.bind("click", function() {  //给添加按钮绑定事件
                var newnode = zTree.addNodes(element, { id: (100 + newCount), pId: element.id, name: "new node" + (newCount++), isParent: true }, true); //新增文件夹（默认新增的是文件 需求有点怪😂）
                var tId = newnode[0].tId; //添加的节点  获取这个是因为添加的是文件夹 所以他还能继续添加
                var nStr = "<span class='button add addable' id='addBtn_" + tId + "' title='add node' onfocus='this.blur();'></span><span class='button edit editable' id='edit_" + tId + "' title='rename' onfocus='this.blur();'></span><span class='button remove removeable' id='remove_" + tId + "' title='remove' onfocus='this.blur();'></span>"; 

                var nobj = $("#" + tId + "_span").after(nStr); //给添加的节点后面加操作按钮
                var nbtn = $("#addBtn_" + tId);
                nbtn.bind('click', function() {  //给添加的节点上的添加按钮绑定事件
                    /* Act on the event */
                    zTree.addNodes(newnode[0], { id: (1000 + newCount), pId: newnode[0].id, name: "new node" + (newCount++), isParent: true }, true);
                });
                return false;
            });
        } else {
            return
        }
    });
}

 ```
这里我们发现增和删、改是有区别的。 其实比较好理解，删和改我们只需要知道操作的对象就可以了，但是添加的时候我们还需要知道添加的内容。  

这里需要注意一下，咱们添加的这几个操作按钮只对展开的节点有用，也就是说当我们点击编辑以后，展开那些一开始没有展开的节点，发现他们的子节点是没有操作按钮的，显然这是不符合我们需求的。最简单的办法，用节点展开的回调函数onExpand，展开节点的时候给他所有的子节点添加按钮，这也是我封装addnewBtn的主要原因

```javascript
function zTreeOnExpand(event, treeId, treeNode) {
    function filter(node) {
        return node;
    }
    var treeObj = $.fn.zTree.getZTreeObj("tree");
    var nodes = treeObj.getNodesByFilter(filter,false,treeNode); // 查找子节点集合
    addnewBtn(nodes);
};
```
下面就剩下编辑和删除了，我们前面只是拼了个按钮上去，并没有做功能，代码如下

```javascript
	
    var operateNode;  //定义一个变量存当前操作的节点
    function zTreeOnClick(event, treeId, treeNode) {
        operateNode = treeNode;  //获取当前操作的节点存入 operateNode
    };
    
    $(document).on('click', '.button.edit.editable', function(event) {  //事件绑定到editable 上是为了区分权限 下同
        var treeObj = $.fn.zTree.getZTreeObj("tree");
        treeObj.editName(operateNode);    // 编辑节点
    });
   
    $(document).on('click', '.button.remove.removeable', function(event) {
        var treeObj = $.fn.zTree.getZTreeObj("tree");
        treeObj.removeNode(operateNode);   // 删除节点
    });
```

这里只是部分代码，[完整的代码在这里](https://github.com/Troubledot/Common-components)，自己整理的很简单的东西，如果运气好能帮到别人，那就太棒了。

 

