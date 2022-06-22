---
title: Hystrix
date: 2022-06-22 22:41:52
tags:
    - Netflix Hystrix
    - Spring Cloud
    - Java
    - microservice
    - distributed services
categories:
    - Distributed services
---
> [官方文档](https://github.com/Netflix/Hystrix/wiki)
## Hystrix的主要功能
1. 线程拆分、隔离
2. 降级、熔断
3. 超时
4. 缓存
5. 监控

### 线程拆分、隔离
任务线程和请求线程隔离，每种任务（command）使用不同的线程池，相互间隔离。

### 降级
超时、异常、线程池饱和都会执行降级
```java
public class CommandHelloWorld extends HystrixCommand<String> {

  private final String name;

  public CommandHelloWorld(String name) {
    super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
    this.name = name;
  }

  @Override
  protected String run() {
      // 这里是正常业务
    throw new RuntimeException();
  }

  @Override
  protected String getFallback() {
      // 降级后的内容
      // 1. 如果这里不是固定的内容还有“降级业务”，
      // 需要在这里创建新的command，相当于再加一级。
      // 2. 如果这里“降级业务”是查询缓存一类的低I/O行为
      // 可以考虑使用“信号量”
    return "dummy message";
  }
}
```
降级策略
- fail fast 快速失败：run中直接抛出错误
- fail slient 无声失败：返回空内容，比如空map、list、null、new Object等
- static/stubbed 静态/组装默认对象： 返回静态内容或者new一个默认值的对象
- cache 缓存：查询有一定数据版本延迟的远程内容比如缓存（在上面注释里面有提）

### 熔断
`关闭`状态，如果一段时间总的请求数达到`circuitBreaker.requestVolumeThreshold`且失败的请求数大于`circuitBreaker.errorThresholdPercentage`这个百分比，断路器就会进入`打开`状态，打开状态下所有的请求都会直接执行降级逻辑，然后在`circuitBreaker.sleepWindowInMilliseconds`这个时间后会进入`半开`状态，半开状态下会尝试放一个请求，如果：
1. 该请求成功，则会进入`关闭`状态
2. 该请求失败，保持`打开`状态（将当前时间设置为打开时间，再次等待`circuitBreaker.sleepWindowInMilliseconds`进入`半开`）

根据[降级](#降级)，hystrix的熔断降级条件可以概括为`高延迟`、`低服务质量`、`超限并发`

### 任务的超时配置

```java
public CommandHelloWorld(String name) {
  super(Setter
    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
    .andCommandPropertiesDefaults(
      HystrixCommandProperties.Setter()
      // 设置1s超时
      .withExecutionTimeoutInMilliseconds(1000)
    ));
  this.name = name;
}
```
超时会调用`getFallback()`

### 缓存
（HystrixRequestContext、略）

### 监控
（hystrix-metrics-event-stream、略）

![hystrix](hystrix/hystrix.webp)