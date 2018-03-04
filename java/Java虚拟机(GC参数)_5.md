# GC参数

## 1 串行收集器

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/jvm/gc1.png)

- -XX:+UseSerialGC

  - 新生代、老年代使用串行回收
  - 新生代复制算法
  - 老年代标记-压缩

- 最古老，稳定，效率高
- 可能会产生较长时间停顿

## 2 并行收集器

### 2.1 ParNew

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/jvm/gc2.png)

- -XX:+UseParNewGC

  - 新生代并行，复制算法
  - 老年代串行

- 多线程，需要多核支持
- -XX:ParallelGCThreads 限制线程数量

### 2.2 Parallel收集器

- -XX:+UseParallelGC 

  - 使用Parallel收集器+ 老年代串行
  
- -XX:+UseParallelOldGC

  - 使用Parallel收集器+ 老年代并行

- 更加关注吞吐量

### 2.3 参数

- -XX:MaxGCPauseMills

  - 最大停顿时间，单位毫秒
  - GC尽力保证回收时间不超过设定值
  
- -XX:GCTimeRatio

  - 0-100的取值范围
  - 垃圾收集时间占总时间的比
  - 默认99，即最大允许1%时间做GC
  
- 这两个参数是矛盾的。因为停顿时间和吞吐量不可能同时调优

## 3 CMS

- -XX:+UseConcMarkSweepGC
- Concurrent Mark Sweep 并发标记清除
- 标记-清除算法
- 并发阶段会降低吞吐量
- 老年代收集器（新生代使用ParNew）

### 3.1 过程

- 初始标记

  - 根可以直接关联到的对象
  - 速度快
  
- 并发标记（和用户线程一起）

  - 主要标记过程，标记全部对象
  
- 重新标记

  - 由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正

- 并发清除（和用户线程一起）

  - 基于标记结果，直接清理对象

### 3.2 特点

- 尽可能降低停顿
- 会影响系统整体吞吐量和性能
- 清理不彻底（因为在清理阶段，用户线程还在运行，会产生新的垃圾，无法清理）
- 因为和用户线程一起运行，不能在空间快满时再清理

  - -XX:CMSInitiatingOccupancyFraction设置触发GC的阈值
如果不幸内存预留空间不够，就会引起concurrent mode failure

### 3.4 参数

- -XX:+UseCMSCompactAtFullCollection Full GC后，进行一次整理<br>整理过程是独占的，会引起停顿时间变长
- -XX:+CMSFullGCsBeforeCompaction <br>设置进行几次Full GC后，进行一次碎片整理
- -XX:ParallelCMSThreads<br>设定CMS的线程数量

## 4 G1


