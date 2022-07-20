---
title: 【KNOW-HOW】异步/并发
date: 2022-07-10 21:06:58
categories:
    - Know-how
tags:
    - Java
    - asynchronization
    - concurrency
---
## Future
基本的异步模型，主要方法有`get()`、`get(time,timeUnit)`、`isDone()`等。

## CompletableFuture
支持回调，更好的支持多个异步任务的流程控制，比如先后顺序，成功或失败之后的流转。

### 假如有A、B两个任务，B需要在A之后执行：
```java
CompletableFuture<String> aQuery = CompletableFuture.supplyAsync(() -> {
    return queryA();
}).exceptionally(e -> {
    System.out.println("failed to execute aQuery");
    e.printStackTrace();
    return null;
});
//aQuery成功后继续执行bQuery
CompletableFuture<Double> bQuery = cfQuery.thenApplyAsync(aResult -> {
    if(aResult == null){
        return null;
    }
    return queryB(aResult);
}).exceptionally(e -> {
    System.out.println("failed to execute bQuery");
    e.printStackTrace();
    return null;
});
// bQuery成功后打印结果
bQuery.thenAccept(System.out::println);
```

如果不依赖CompletableFuture的话：
```java
Future<String> aFuture = asyncQueryA();
String aResult = null;
try{
    aResult = aFuture.get();
}catch(Exception e){
    System.out.println("failed to execute asyncQueryA");
    e.printStackTrace();
}

String bResult = null;
if(aResult != null){
    Future<String> bFuture = asyncQueryB(aResult);
    try{
        bResult = bFuture.get();
    }catch(Exception e){
        System.out.println("failed to execute asyncQueryB");
        e.printStackTrace();
    }
}
System.out.println(bResult);
```

### 假如有A、B两个任务，A或B任意一个任务执行完成就结束：
```java
//aQuery
CompletableFuture<String> aQuery = CompletableFuture.supplyAsync(() -> {
    return queryA();
}).exceptionally(e -> {
    System.out.println("failed to execute aQuery");
    e.printStackTrace();
    return null;
});
//bQuery
CompletableFuture<Double> bQuery = cfQuery.thenApplyAsync(() -> {
    return queryB();
}).exceptionally(e -> {
    System.out.println("failed to execute bQuery");
    e.printStackTrace();
    return null;
});

CompletableFuture<Object> aOrBQuery = CompletableFuture.anyOf(aQuery, bQuery);

// 任意一个任务成功后打印结果
aOrBQuery.thenAccept(System.out::println);
```

这种情况如果不依赖CompletableFuture的话……：
```java
Future<String> aFuture = asyncQueryA();
Future<String> bFuture = asyncQueryB();
int failedTask = 0;
String result = null;
while(true){
    if(aFuture.isDone()){
        try{
            result = aFuture.get();
            break;
        }catch(Exception e){
            System.out.println("failed to execute asyncQueryA");
            e.printStackTrace();
            failedTask++;
        }
    } else if(bFuture.isDone()){
        try{
            result = bFuture.get();
            break;
        }catch(Exception e){
            System.out.println("failed to execute asyncQueryB");
            e.printStackTrace();
            failedTask++;
        }
    }
    // 不写==2, 防止以后加任务cdef等等时忘记修改这个判断值引起死循环
    if(failedTask > 1) break;
}
```
### 实现细节和注意点
1. `thenAccept`或者`exceptionally`这种方法是在调用线程中执行的，会阻塞调用线程，`supplyAsync`或者`thenApplyAsync`这种名字里带Async的会被提交到线程池执行。
2. 如果不指定线程池的话，默认使用的是`ForkJoinPool`的`CommonPool`线程池（默认的coreSize和poolSize是8还是几来着，反正很少），可能会和其他的任务共享线程池，也许会互相抢占线程资源。
3. `CompletionStage`定义流程控制的能力，各种流程组合的实际动作都基于`CompletableFuture.Completion`实现的类

## RxJava
（待续）

## Fiber/Coroutines
[Project Loom Preview](/2022/07/14/project-loom-preview/index.html)