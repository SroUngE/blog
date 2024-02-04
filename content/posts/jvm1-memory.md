---
title: "JVM(一): 内存结构"           # 文章标题
description : "介绍JVM的内存结构，以及一些常用的内存诊断工具。"    # 文章描述信息
date: 2023-05-28            # 文章编写日期
tags : [                    # 文章所属标签
    "JVM"
]
featuredImage: "https://s2.loli.net/2024/01/23/GE8SiwClZc45BRU.jpg"
featuredImagePreview: "https://s2.loli.net/2024/01/23/GE8SiwClZc45BRU.jpg"
draft: false
---
<!--more-->
## 1. 程序计数器 (Program Counter Register)
1. 作用：存储下一条JVM指令的执行地址。
> 如果该方法不是native方法，则pc寄存器包含当前正在执行的Java虚拟机指令的地址。 如果线程当前正在执行的方法是native方法，则Java虚拟机的pc寄存器的值为Undefined。 

2. 线程：每条线程都需要有一个独立的程序计数器。
> 由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间的计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

3. 溢出：Java虚拟机的pc寄存器足够大，可以在特定平台上保存返回地址或本机指针。

## 2. 虚拟机栈 (Java Virtual Machine Stacks)
1. 作用：用于存储局部变量表、操作栈、动态链接、方法出口等信息方法被执行的时。
> 每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

2. 线程：Java虚拟机栈也是线程私有的，它的生命周期与线程相同。

3. 溢出：在Java虚拟机规范中，对这个区域规定了两种异常状况。
> 1.如果线程请求的栈深度大于虚拟机所设定的深度，则Java虚拟机将引发StackOverflowError。  
> 2.如果可以动态扩展Java虚拟机堆栈，尝试进行扩展时没有足够的内存来实现扩展，则Java虚拟机将引发OutOfMemoryError。  
> 3.或者如果没有足够的内存可用于为新线程创建初始Java虚拟机堆栈，则Java虚拟机将引发OutOfMemoryError。

## 3. 堆 (Heap)
1. 作用：堆是运行时数据区，从中分配了所有类实例和数组的内存。
> 1.对象的堆存储由为垃圾收集器回收。如果从内存回收的角度看，由于现在收集器基本都是采用的分代收集算法，所以Java堆中还可以细分为：新生代和老年代；再细致一点的有Eden空间、From Survivor空间、To Survivor空间等。
> 2.对象永远不会显式释放。

2. 线程：堆是在虚拟机启动时创建的，在所有线程之间共享的。

3. 溢出：如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。
> 1.堆的大小可以是固定的，也可以根据计算的需要进行扩展，如果不需要更大的堆，则可以将其收缩。堆的内存不必是连续的。  
> 2.Java虚拟机实现可以为程序员或用户提供对堆的初始大小的控制，并且，如果可以动态扩展或收缩堆，则可以控制最大和最小堆大小。

## 4. 方法区 (Method Area)
1. 作用：用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

2. 线程：方法区与堆一样，是各个线程共享的内存区域。

3. 溢出：当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。
> 方法区是在虚拟机启动时创建的。尽管方法区在逻辑上是堆的一部分，但是简单的实现可以选择不进行垃圾回收或压缩。该规范没有规定方法区的位置或管理已编译代码的策略。方法区可以是固定大小的，或者可以根据计算的需要进行扩展，如果不需要更大的方法区域，则可以缩小。方法区域的内存不必是连续的。

## 5. 本地方法栈 (Native Method Stacks)
1. 作用：为虚拟机使用到的Native方法服务。

2. 线程：在创建每个线程时为每个线程分配本机方法堆栈。

3. 溢出：与虚拟机栈一样，在Java虚拟机规范中，对这个区域规定了两种异常状况。
> 1.如果线程中的计算所需的本机方法堆栈超出允许的范围，则Java虚拟机将引发StackOverflowError。  
> 2.如果可以动态扩展本机方法堆栈并尝试进行本机方法堆栈扩展，但可以提供足够的内存，或者可以提供足够的内存来为新线程创建初始本机方法堆栈，则Java虚拟机将引发OutOfMemoryError

通过一张图来了解如何通过参数来控制各区域的内存大小。
![](https://s2.loli.net/2023/05/28/bM5RLsU6D2efxpg.png)  
-Xms设置堆的最小空间。  
-Xmx设置堆的最大空间。  
-XX:NewSize设置新生代最小空间。  
-XX:MaxNewSize设置新生代最大空间。  
-XX:PermSize设置永久代最小空间。  
-XX:MaxPermSize设置永久代最大空间。  
-Xss设置每个线程的堆栈。  

## 6. 内存诊断
### 6.1 CPU占用过高
用top定位哪个进程对cpu的占用过高(Linux)  
1. 首先运行java代码，然后输入`top`命令，可以看到进程32655占了cpu的99.3%
```bash
top
```
![](https://s2.loli.net/2023/05/28/mpIf7CbvoZNtz1y.png) 

2. 然后进一步定位是哪个线程引起的cpu占用过高，这里可以看到32665占用率最高
```bash
ps H -eo pid,tid,%cpu | grep 进程id
```
![](https://s2.loli.net/2023/05/28/9qKCFEWU6p1PLgd.png)  

3. 使用`jstake`，把32665十进制换算成16进制，就是0x7f99  
```bash
jstake 进程id
```
![](https://s2.loli.net/2023/05/28/bDOVusqU4PrATtl.png)

4. 可以看到他提示在Demo1_16这个类的第8行有问题
![](https://s2.loli.net/2023/05/28/q3Ihs6gOyBE5akb.png)  

### 6.2 程序运行很长时间没有结果
1. 还是使用jstake指令，在最后面，它会输出一段分析
```bash
jstake 进程id
```
![](https://s2.loli.net/2023/05/28/z3Bkavpr8PKoLFQ.png)

2. 查看代码可以看到ab死锁了
![](https://s2.loli.net/2023/05/28/FGVNelsczgj4P3S.png)

### 6.3 堆内存诊断
#### 6.3.1 内存诊断 jmap  
先用`jps`查看有哪些进程  
![](https://s2.loli.net/2023/06/07/2njOKDb7SrlvFqf.png)  
然后用`jmap` 查看当前时刻堆内存的使用情况
```bash
jmap -heap 进程id
```
```bash
jhsbd jmap -dump:live,format=b,file=/temp/file.hprof 18756
```
![](https://s2.loli.net/2023/06/07/iYD9KaPy5CeXvuj.png)

#### 6.3.2 内存诊断 jconsole  
终端输入jconsole，就会打开一个图形化的界面  
![](https://s2.loli.net/2023/05/28/KRlr7VaWGzEevBd.png)
![](https://s2.loli.net/2023/05/28/JHtGvmMBEwWPIRd.png)
![](https://s2.loli.net/2023/05/28/2mkI7efLHtWSuJZ.png)

#### 6.3.3 内存诊断 jvisualvm  
a. 终端输入jvisual，会打开一个图形化的界面，功能类似jconsole，可以查看堆的使用情况  
b. jvisual 有一个堆Dump的功能可以给当前的堆内存信息拍个快照  
![](https://s2.loli.net/2023/05/28/1fCNGJIVED8zsju.png)  
c. 可以选取前20个大对象，查看它的elementData排查。