---
title: "JVM(二): 垃圾回收"           # 文章标题
description : "介绍JVM垃圾回收的算法，垃圾收集器，以及调优。"    # 文章描述信息
date: 2023-06-07            # 文章编写日期
tags : [                    # 文章所属标签
    "JVM"
]
featuredImage: "https://s2.loli.net/2024/01/23/GE8SiwClZc45BRU.jpg"
featuredImagePreview: "https://s2.loli.net/2024/01/23/GE8SiwClZc45BRU.jpg"
draft: false
---
<!--more-->
## 1. 垃圾标记算法
### 1.1 引用计数法
在 Java 中，要操作对象则必须用引用进行。设置引用计数器，对象被引用时计数器加1，引用失效时计数器减1，如果计数器为0，则被标记为垃圾。引用计数法存在循环引用问题。
### 1.2 可达性分析
为了解决引用计数法的循环引用问题，Java 使用了可达性分析的方法。通过一系列的“GC roots”对象作为起点搜索，根据引用关系向下搜索。如果在“GC roots”和一个对象之间没有可达路径，则称该对象是不可达的。要注意的是，不可达对象不等价于可回收对象，不可达对象变为可回收对象至少要经过两次标记过程。两次标记后仍然是可回收对象，则将面临回收。
可以作为GC Root的对象包括下面几种：
- 虚拟机栈帧中引用的对象
- 本地方法栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象

## 2. 垃圾回收算法
### 2.1 标记清除
- 标记清除：标记要回收的对象，标记完成后统一回收被标记对象
- 优点：速度快
- 缺点：有内存碎片  
![](https://s2.loli.net/2023/06/07/TXKkH1GS75poNeZ.png)

### 2.2 标记整理
- 标记整理：标记要回收的对象后，将存活对象移动到一端，然后将边界外的内存回收
- 优点：没有碎片
- 缺点：速度慢  
![](https://s2.loli.net/2023/06/07/uazGpkYRLgdW2Po.png)

### 2.3 标记复制
- 标记复制：将没有标记的对象复制到To区，然后清除From区的所有对象
- 优点：没有内存碎片，速度快
- 缺点：占用两份内存  
![](https://s2.loli.net/2023/06/07/jGpMPLZ4t2aFfsH.png)

### 2.4 分代GC
![](https://s2.loli.net/2023/06/07/HundWiPD28T91Xe.png)
- gc 会引发 stop the world，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复行。
- 将堆分为新生代和老年代，新生代用复制算法，老年代用标记清除或者标记整理算法。
- 新生代有划分为Eden、From Survivor和To Survivor三个部分，对应的内存空间的大小比例为8:1:1。
- 对象首先分配在Eden区，当对象刚分配到Eden区域时，对象的年龄为0。
- 当Eden区第一次空间不足时，触发 minor gc，Eden区中存活的对象复制到To区中，存活的对象年龄加 1，清除所有Eden区不可达对象，并交换from 区和 to区。
- 当Eden区第二次空间不足时，触发 minor gc，Eden区和 from 区中存活的对象复制到To区中，存活的对象年龄加 1，清除所有Eden区和 from区的不可达对象，交换 from 区和 to区 。
- 当对象寿命超过阈值时，会晋升至老年代，最大寿命是15（4bit） 。
- 经过minor gc之后又有一部分存活的对象会进入老年代，当老年代空间不足时，会触发老年代上的垃圾回收major gc。

### 2.5 实例展示
![](https://s2.loli.net/2023/06/07/KCbTAczyer8RadU.png)
![](https://s2.loli.net/2023/06/07/AUMFC9bBXrevhZ8.png)
![](https://s2.loli.net/2023/06/07/HCrfViGvxLFBWyR.png)  
第一次放7MB，新生代空间不足时，触发 minor gc，伊甸园和 from 存活的对象使用 copy复制到 to 中
![](https://s2.loli.net/2023/06/07/W4yUbJ3ZqGIxd6Y.png)  
第二次放512KB，新生代空间不足时，触发 minor gc，伊甸园和 from 存活的对象使用 copy 复制到 to 中，存活的对象年龄加 1并且交换 from to.
当对象寿命超过阈值时，会晋升至老年代，最大寿命是15（4bit）。或内存紧张时会把大对象直接晋升到老年代.
![](https://s2.loli.net/2023/06/07/Z4a15ASiL2WjOBK.png)  
如果是大对象，在新生代内存不够，老年代内存足够的时候，会直接晋升老年代，不触发GC 
![](https://s2.loli.net/2023/06/07/WZc3LrOYtUfFz8T.png)  
触发Full GC，新生代垃圾回收发现内存不够，触发老年代垃圾回收，内存也不够，则会抛出OutOfMemoryError:Java heap space

### 2.6 常见面试问题
为什么GC的时候要stop the world？/GC的时候不stop the world 会发生什么？
因为GC的时候有采用复制算法，即会对对象的地址进行复制，如果这个时候不暂停其他户的线程，可能会引发混乱。