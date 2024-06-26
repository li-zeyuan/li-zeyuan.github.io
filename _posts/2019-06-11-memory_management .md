---
title: 内存管理
date: 2019-06-11 00:00:00 +0800
categories: [root, go]
tags: [go]
author: ahern
---

### 内存管理-内存分配

- 分配器

  - 线性分配器

    - 定义：当用户程序需要申请内存时，从指针所在的位置开始分配内存，并向后移动指针。

      ![线性分配器](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210130115653.png){:height="10%" width="50%"}

    - 局限性：分配速度快；指针前面的释放的内存块不能重用，需要配合回收算法进行内存拷贝、合并才能复用。

  - 空闲链表分配器

    - 定义：内存被划分成不同大小的快，空闲的块以链表的形式连接起来，当用户程序需要申请内存时，从链表中找到合适的内存块分配，重新修改链表。

      ![空闲链表分配器](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210130120221.png){:height="10%" width="50%"}

    - 分配方式

      - 首次适应：从头到尾遍历空闲链表，找到第一个合适的内存块。

      - 循环首次适应：从上次遍历结果位置开始，找到第一个合适的内存块。

      - 最优适应：从头到尾遍历空闲链表，找到最合适的内存块。

      - 隔离适应：将大的链表分割成小的链表，小链表内存块一致，分配时，先找到小链表，再找空闲的内存块，提高分配效率。

        ​	![隔离适应方式](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210130133531.png){:height="10%" width="50%"}

- go内存管理

  ​		![go内存管理](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210130133635.png){:height="10%" width="50%"}

  - 主要指堆、栈内存的分配和回收
  - 借鉴了TCMalloc内存分配思想：缓存、分级分配
  - 增加了逃逸分析、gc

- go内存分配组成部分

  ![分级分配](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210130133811.png){:height="10%" width="50%"}

  - page：内存被划分成大小不等的页
  - span（跨度）：内存管理的基本单位，一组连续的page组成一个span
  - mcache：类似TCMalloc的线程缓存，go的每个P挂载一个mcache，可以无锁访问
  - mcentral：类似TCMalloc的中心缓存，线程共享，需要加锁访问
  - mheap：与TCMalloc中的PageHeap类似，也需要加锁访问

- go小对象分配

  - 1、计算对象所需要的内存大小
  - 2、跟进转化表，找出所属的span（跨度）
  - 3、从span中分配对象空间，按照**隔离适应**的方式
  - 4、优先从mcache中的span分配，若不够，从mcentral中申请span
  - 5、若mcentral中也不够，向mheap申请
  - 6、mheap向os申请

- go大对象分配

  - 直接向mheap申请

- 参考

  - https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/
  - https://segmentfault.com/a/1190000020338427
  - https://blog.csdn.net/aaronjzhang/article/details/8696212

### 内存管理-垃圾回收

- 回收机制：并行

  - 三色标记			
    ![三色标记法](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210130133912.png){:height="10%" width="50%"}

    - 1、初始状态，所有的对象都是白色
    - 2、从根对象开始扫描，将引用的对象标记成灰色
    - 3、分析灰色对象是否引用了其他对象，若无，标记自己成白色；若有，将自己标记成黑色，将引用对象标记成灰色
    - 重复2、3步骤，直到所有的对象都变成黑或白。白色对象即是要被回收的垃圾

  - 写屏障

    - gc和用户程序并行，这个过程会修改内存引用，就需要一个组件记录下来
    - 每一轮的GC都会初始化一个屏障，记录对象的状态。以便和第二轮的对象状态对比，引用状态变化的标记为灰色防止丢失，状态未变化的对象继续处理。

  - 辅助GC

    - 如果GC的速度小于用户程序分配对象的速度，就会把用户程序暂停（STW）

- GC触发条件
  - 超过内存的阈值
  - 到达特定的时间
  - 手动GC
- 参考
  - https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/
  - https://juejin.cn/post/6844903917650722829

### 逃逸分析

- 定义：go的内存分配由编译器完成，通过逃逸分析，决定内存分配是在栈上还是在堆上。若变量的生命周期是完全可知，则分配到栈上，否则分配到堆（逃逸）。
 - 编译器尽可能地内存分配到栈，几种内存分配到堆得情况（逃逸）
   - 变量类型不确定
   - 函数内暴露给外部的指针
   - 变量所占内存较大
   - 变量的大小不确定

- 逃逸分析的作用：写出更好的程序，使内存尽可能地分配到栈，减小gc压力，减少内存分配开销。
- 参考
  - 逃逸分析：https://mp.weixin.qq.com/s/xhBVv6JEPY8R3kCJlbirYw
  - 堆：https://www.jianshu.com/p/6b526aa481b1

### 内存分析

- 参考：https://www.oschina.net/translate/debugging-performance-issues-in-go-programs
