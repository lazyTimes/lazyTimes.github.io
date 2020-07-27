---
title: Neo4j + VIS 数据可视化问题汇总
subtitle: Neo4j实际使用
author: lazytime
tags:
  - Neo4j
categories:
  - Neo4j
keywords: Neo4j,visjs
description: Neo4j使用
copyright: true
date: 2020-07-26 17:32:28
---

# 前言

被前端整的有点发虚，这里记录一下自己干的一些（蠢）事

> PS：这种无力感真的希望不要再有，在自己不怎么擅长的领域班门弄斧就是浪费时间

<!-- more -->

## 重要资料：

## VIS DataSet 使用：

[dataSet使用](https://visjs.github.io/vis-data/data/dataset.html)

这段时间做表标签库，内容还是比较多的，这里记录一下个人遇到的一些需求汇总问题

[什么是neo4j](https://zhuanlan.zhihu.com/p/36404872)

# Q：页面初始化如何控制节点的距离大小

A： 官方文档： [VIS的节点物理效果](https://visjs.github.io/vis-network/docs/network/physics.html)

需要调整如下参数

在`options`调整如下变量

```
 //力导向图效果
physics: {
    enabled: true,   // 是否开启vis引擎配置
        barnesHut: { // 基于四叉树的重力模型，非分布式性能最好的求值器
            gravitationalConstant: -4000, //此参数确定了合并的远程力和各个短程力之间的边界。要过分简化，较高的值会更快，但是会产生更多的错误，较低的值会很慢，但是会减少错误。
            centralGravity: 0.3, //有一个中央引力吸引器，可将整个网络拉回到中心。
            springLength: 120, // 边缘被建模为弹簧。这里的springLength是弹簧的其余长度。
            springConstant: 0.04, //这就是弹簧的强度。值越高，表示弹簧越强。
            damping: 0.09, // 接受范围：[0 .. 1]。阻尼因子是前一次物理模拟迭代中有多少速度会延续到下一次迭代中。
            avoidOverlap: 0 // 接受范围：[0 .. 1]。大于0时，将考虑节点的大小。对于两个重力模型，将从节点的包围圆的半径计算距离。值1是最大避免重叠。
		}
},
```

根据上面的值基本可以确定整个页面的物理引擎效果

# Q：要实现鼠标悬停到页面变为小手的样式

A：

官方案例：https://visjs.github.io/vis-network/examples/network/other/cursorChange.html

参考博客：[博客地址](https://www.thinbug.com/q/40532596)

步骤：

```
var networkCanvas = document.getElementById("network_id").getElementsByTagName("canvas")[0];
```

增加一个配置

最终解决方案：

在vis初始化`network` 之后,加入如下事件绑定代码

```
var cursor =  network.canvas.body.container.style.cursor; // 获取netword的cursor
network.on("hoverNode", function (params) {
    cursor = 'grab'
});
network.on("blurNode", function() {
    cursor = 'default'
});
network.on("hoverEdge", function() {
    cursor = 'grab'
});
network.on("blurEdge", function() {
    cursor = 'default'
});
network.on("dragStart", function() {
    cursor = 'grabbing'
});
network.on("dragging", function() {
    cursor = 'grabbing'
});
```

# Q：如何模仿Neo4j 用vis实现节点的展开和多层折叠

参考：[简书地址](https://www.jianshu.com/p/b22727c8fe60)

个人需求：

实现节点可以扩展和缩放，双击扩展，扩展之后再次双击为收缩

难点：如何实现多层节点收缩

```
var nodes = [
  {id: 1, subids: [2, 3],allsubids:[2,3,4,5]},
  {id: 2, pid: 1},
  {id: 3, subids: [4],pid: 1,allsubids:[4,5]}
  {id: 4, subids: [5],pid: 3}
  {id: 5, pid: 4}
];
var edges = [
  {from:1,to:2},
  {from:1,to:3},
  {from:3,to:4},
  {from:4,to:5}
]
```

## 个人实现：

目前经过查找资料发现以下代码加上

```
/*
     *获取所有子节点
     * network ：图形对象
     * _thisNode ：单击的节点（父节点）
     * _Allnodes ：用来装子节点ID的数组
     * */
    function getAllChilds(network,_thisNode,_Allnodes){
        var _nodes = network.getConnectedNodes(_thisNode);
        if(_nodes.length > 0){
            for(var i=0;i<_nodes.length;i++){
                // getAllChilds(network,_nodes[i],_Allnodes);
                _Allnodes.push(_nodes[i]);
            }
        }
        return _Allnodes
    };
```

增加两个`Array`的方法：

```
 Array.prototype.indexOf = function(val) {
        for (var i = 0; i < this.length; i++) {
            if (this[i] == val) return i;
        }
        return -1;
    };

    //code from http://caibaojian.com/js-splice-element.html
    Array.prototype.remove = function(val) {
        var index = this.indexOf(val);
        if (index > -1) {
            this.splice(index, 1);
        }
    };
```

具体判断逻辑如下：

1. 设置一个标志位，第一次为true，表示扩展该节点。扩展之后将该节点的id放到一个全局的扩展节点array里面维护
2. 如果再次点击，该标志位变成false, 执行步骤又可以分为以下几点
   - 使用vis.js的api获取当前节点的`连接`节点，`network.getConnectedNodes(_thisNode);`的`_thisNode`，为当前操作节点的唯一标识id（此方法还有很多的可选条件参数）
   - 获取`连接`节点之后，调用`nodes.remove(_allChild)`，删除关联节点，此操作会直接干掉被删除节点所有的关联关系
   - 删除完成之后，全局表的`已扩展`节点删除当前点击节点，方便下次再次扩展

# Q：双向两条箭头合并为一个单向双箭头

这个需求是比较特殊的例子，涉及到业务的内容不予描述

思路：

1. 拿到当前记录，获取当前记录的`from`和`to`
2. 使用两层循环嵌套，第一层遍历所有的节点一次，第二层遍历当前节点的后面节点，类似冒泡
3. 如果当前节点的`from` == 其他节点的`to`，并且当前节点的`to` == 其他节点的`from`

具体代码如下：

```
// 手动筛选掉双向关联的数据
    function delrepeatinfo(arrName) {
        arrName = arrName.edges;
        if (arrName == null || arrName.length == 0) {
            return null;
        }
        var repeat = [];
        for (var i = 0; i < arrName.length; i++) {
            var item = arrName[i];
            if(item.skip){
                continue;
            }
            var fromId = item.edgeFrom;
            var toId = item.edgeTo;
            for (var j = i+1; j < arrName.length; j++) {
                var next = arrName[j];
                if(next.edgeTo == fromId && next.edgeFrom == toId){
                    repeat.push(item);
                    next.skip = true;
                    break;

                }
            }

        }

        return repeat;

    }
```

# Q：如果自定义画图

需求：

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/%E6%A0%87%E7%AD%BE%E5%BA%93%E9%9C%80%E6%B1%82.png?ynotemdtimestamp=1595748679079)

A：个人无法处理，需要寻找建哥处理按钮的绘画以及按钮的事件触发操作

处理方式：

1. 使用canvas在 点击标签之后绘制对应的按钮
2. 监听坐标，当点击了坐标内的按钮触发事件，显示对应的菜单

那个多出两个自定义按钮的解决办法如下：

1. 利用canvas前端画图技术绘制按钮
2. 监听鼠标的点击坐标，查看是否在绘制的按钮坐标内，对应的触发事件
3. 触发相关操作

VIS绘制辅助线条的办法：

```
context = ctx || $('#network_id')[0].getContext("2d")
context.lineWidth = 1;
context.strokeStyle = "#CECBCE";
context.setLineDash([5]);
var offset = 800;
var padding = 50;
for (var i = 0; i < 30; i++) {
    context.beginPath();
    context.moveTo(offset * -1, i * padding - offset)
    context.lineTo(2000, i * padding - offset)
    context.stroke();

    context.beginPath();
    context.moveTo(i * padding - offset, offset * -1)
    context.lineTo(i * padding - offset, 2000)
    context.stroke();
}
context.setLineDash([]);
```

# 总结

neo4j 的资料还是太少太少，多查查官方文档才是硬道理

个人接到的需求如下

1. 标签之间都是双向箭头，需要改成一条直线，或者只有单项的。

无法自定义，因为数据的展示强制需要 头和尾两个参数，数据本身就是会存在互相关联的情况，所以无法实现

1. 标签之间的连接线点击不要有效果，就是颜色不要加深

只需要去掉inter``

1. 标签展开后能再点击折叠收回映射关系吗？
2. 所有图例鼠标放到上面都变成小手形状
3. 初始化页面这些标签过于分散，需要集中一些，根据屏幕情况集中在一屏展示

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/%E6%A0%87%E7%AD%BE%E5%BA%93%E9%9C%80%E6%B1%82.png?ynotemdtimestamp=1595748679079)