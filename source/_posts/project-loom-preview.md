---
title: 【FEATURE】Project Loom 预览
date: 2022-07-14 17:02:15
categories:
    - Feature
tags:
    - Java
    - virtual threads
    - fiber
    - asynchronization
    - concurrency
    - continuations
---
## 相关资源
> 主页：[https://openjdk.org/projects/loom/](https://openjdk.org/projects/loom/)
> 
> 提案：[http://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html](http://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html)
>
> 预览：[https://jdk.java.net/loom/](https://jdk.java.net/loom/)


## EA in JDK19
> [https://github.com/Attt/Loom-Preview](https://github.com/Attt/Loom-Preview)

```java
// 创建builder
Thread.Builder.OfVirtual ofVirtual = Thread.ofVirtual()
        // 名称，start表示种子1步进递增
        .name("virtual-thread-", 0)
        // 是否允许set threadLocal
        .allowSetThreadLocals(true)
        // 是否允许使用可继承的threadLocal
        .inheritInheritableThreadLocals(true)
        // 未捕获异常处理器
        .uncaughtExceptionHandler((t, e) -> {
            System.out.println(t);
            e.printStackTrace();
        });

// 执行
ofVirtual.start(() -> {
    // do something
    System.out.println(Thread.currentThread().getName());
    System.out.println(Thread.currentThread().isDaemon());
    System.out.println(Thread.currentThread().isVirtual());
}).join();

// 预设任务，返回Thread实例
Thread unstarted = ofVirtual
        .unstarted(() -> {
            System.out.println(Thread.currentThread().getName());
            System.out.println("Wooooow!");
        });
// 开启任务
unstarted.start();

// 基于Executors#newThreadPerTaskExecutor，使用VirtualThreadFactory作为线程工厂
// 为每个提交的任务开启一个thread，无界
ExecutorService virtualThreadPerTaskExecutor = Executors.newVirtualThreadPerTaskExecutor();
virtualThreadPerTaskExecutor.submit(() -> {
    System.out.printf("%s-newVirtualThreadPerTaskExecutor %n", Thread.currentThread().getName());
});

// 使用builder的设置值创建的perTaskExecutor
ExecutorService perTaskExecutor = Executors.newThreadPerTaskExecutor(ofVirtual.factory());
perTaskExecutor.submit(() -> {
    System.out.printf("%s-newThreadPerTaskExecutor %n", Thread.currentThread().getName());
});
```

## Benchmark
```java
Benchmark                                  Mode  Cnt     Score     Error   Units
ThreadThroughputBenchmark.platformThread  thrpt    5    10.788 ±   0.677  ops/ms
ThreadThroughputBenchmark.virtualThread   thrpt    5  4201.740 ± 475.806  ops/ms
```

## Q&A



## *函数颜色问题
[what-color-is-your-function](http://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)，假设某种语言中一种函数是<font color='red'>红色</font>的，另一种函数是<font color='#0099ff'>蓝色</font>的：

那么这种语言实际存在两种`function`关键字：`blue_func`和`red_func`
```javascript
blue_func sync(){...}

red_func async(callback){...}
```
其中，<font color='red'>红色</font>函数的结果不通过方法返回，而是传递给参数中的函数，那么就会有这样的问题：
- 在<font color='red'>红色</font>函数中可以调用<font color='#0099ff'>蓝色</font>函数，因为<font color='#0099ff'>蓝色</font>函数的返回值通过方法返回能够拿到。
- 而在<font color='#0099ff'>蓝色</font>函数中是不能轻易调用<font color='red'>红色</font>函数的，因为结果已经传递到<font color='red'>红色</font>函数的参数函数里了，在<font color='#0099ff'>蓝色</font>函数线性执行过程中不能保证能够从<font color='red'>红色</font>函数的参数函数里获取结果。

```javascript
red_func async(callback){
    sync() // 可以调用
}

blue_func sync(){
    async(callback) // 不能调用
}
```
所谓的<font color='red'>红色</font>函数就是异步函数，上面的这个“颜色问题”就是“回调地狱”("callback hell")的另一种感受方法😅。

在函数式编程中，异步存在两个比较炸裂的问题：1. 回调地狱😅；2. 一个被广泛使用的同步函数的异步化所需要的大量的修改和重写量可以让任何一个开发直接爆炸。