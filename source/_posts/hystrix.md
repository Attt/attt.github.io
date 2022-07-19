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

### 配置

#### Pool

| Property | Description | Comment | Default Value |
| --- | --- | --- | --- |
| corePoolSize | Core thread-pool size that gets passed to ThreadPoolExecutor.setCorePoolSize(int) | | 10|
| maximumPoolSize | Maximum thread-pool size configured for threadpool. May conflict with other config, so if you need the actual value that gets passed to ThreadPoolExecutor.setMaximumPoolSize(int), use actualMaximumSize(). Given all of the thread pool configuration, what is the actual maximumSize applied to the thread pool via ThreadPoolExecutor.setMaximumPoolSize(int) Cases: 1) allowMaximumSizeToDivergeFromCoreSize == false: maximumSize is set to coreSize 2) allowMaximumSizeToDivergeFromCoreSize == true, maximumSize >= coreSize: thread pool has different core/max sizes, so return the configured max 3) allowMaximumSizeToDivergeFromCoreSize == true, maximumSize < coreSize: threadpool incorrectly configured, use coreSize for max size | 仅当allowMaximumSizeToDivergeFromCoreSize是true且maximumPoolSize大于coreSize时这里的值为实际传到线程池的maximumPoolSize，否则传到线程池的maximumPoolSize实际大小为coreSize | 10|
| keepAliveTime | Keep-alive time in minutes that gets passed to ThreadPoolExecutor.setKeepAliveTime(long, TimeUnit) | | 1|
| maxQueueSize | Max queue size that gets passed to BlockingQueue in HystrixConcurrencyStrategy.getBlockingQueue(int) This should only affect the instantiation of a threadpool - it is not eliglible to change a queue size on the fly. For that, use queueSizeRejectionThreshold(). -1 turns it off and makes us use SynchronousQueue | 控制线程池的等待队列大小，不过为了能够在运行时修改这个值，hystrix往隔离线程池提交任务的时候实际参考的是queueSizeRejectionThreshold，这个值默认-1使用SynchronousQueue | -1|
| queueSizeRejectionThreshold | Queue size rejection threshold is an artificial "max" size at which rejections will occur even if maxQueueSize has not been reached. This is done because the maxQueueSize of a BlockingQueue can not be dynamically changed and we want to support dynamically changing the queue size that affects rejections. This is used by HystrixCommand when queuing a thread for execution. | 实际控制等待队列大小的值，不过如果这个值小于maxQueueSize，那实际生效的还是maxQueueSize | 5|
| allowMaximumSizeToDivergeFromCoreSize | should the maximumSize config value get read and used in configuring the threadPool. turning this on should be a conscious decision by the user, so we default it to false. | | false|
| threadPoolRollingNumberStatisticalWindowInMilliseconds | Duration of statistical rolling window in milliseconds. This is passed into HystrixRollingNumber inside each HystrixThreadPoolMetrics instance. | 单个窗口的时间 | 10000|
| threadPoolRollingNumberStatisticalWindowBuckets | Number of buckets the rolling statistical window is broken into. This is passed into HystrixRollingNumber inside each HystrixThreadPoolMetrics instance. | 单个窗口的桶的数量 | 10|


#### Command

| Property | Description | Comment | Default Value |
| --- | --- | --- | --- |
| circuitBreakerEnabled | Whether to use a HystrixCircuitBreaker or not. If false no circuit-breaker logic will be used and all requests permitted. This is similar in effect to circuitBreakerForceClosed() except that continues tracking metrics and knowing whether it should be open/closed, this property results in not even instantiating a circuit-breaker. | 是否启用熔断，跟circuitBreakerForceClosed的区别就是如果这个关掉的话也不会有统计数据出现 | true |
| circuitBreakerErrorThresholdPercentage |  Error percentage threshold (as whole number such as 50) at which point the circuit breaker will trip open and reject requests. It will stay tripped for the duration defined in circuitBreakerSleepWindowInMilliseconds(); The error percentage this is compared against comes from HystrixCommandMetrics.getHealthCounts(). | 触发断路的请求数量百分比 | 50 |
| circuitBreakerForceClosed | If true the HystrixCircuitBreaker.allowRequest() will always return true to allow requests regardless of the error percentage from HystrixCommandMetrics.getHealthCounts(). The circuitBreakerForceOpen() property takes precedence so if it set to true this property does nothing. | | false |
| circuitBreakerForceOpen | If true the HystrixCircuitBreaker.allowRequest() will always return false, causing the circuit to be open (tripped) and reject all requests. This property takes precedence over circuitBreakerForceClosed(); | | false |
| circuitBreakerRequestVolumeThreshold | Minimum number of requests in the metricsRollingStatisticalWindowInMilliseconds() that must exist before the HystrixCircuitBreaker will trip. If below this number the circuit will not trip regardless of error percentage. | 触发断路逻辑的最小阈值，在一个统计窗口内至少有一定数量的请求发生才会走断路判断逻辑 | 20 |
| circuitBreakerSleepWindowInMilliseconds | The time in milliseconds after a HystrixCircuitBreaker trips open that it should wait before trying requests again. | “半开”的尝试间隔，熔断触发之后在这个时间之后进入“半开”状态，观察请求是否恢复正常 | 5000 |
| executionIsolationSemaphoreMaxConcurrentRequests | Number of concurrent requests permitted to HystrixCommand.run(). Requests beyond the concurrent limit will be rejected. Applicable only when executionIsolationStrategy() == SEMAPHORE. | 最大的并发请求提交数，只在“信号量”模式下生效 | 10 |
| executionIsolationStrategy | What isolation strategy HystrixCommand.run() will be executed with. If HystrixCommandProperties.ExecutionIsolationStrategy.THREAD then it will be executed on a separate thread and concurrent requests limited by the number of threads in the thread-pool. If HystrixCommandProperties.ExecutionIsolationStrategy.SEMAPHORE then it will be executed on the calling thread and concurrent requests limited by the semaphore count. | 隔离策略 | ExecutionIsolationStrategy.THREAD |
| executionIsolationThreadInterruptOnTimeout | Whether the execution thread should attempt an interrupt (using Future.cancel) when a thread times out. Applicable only when executionIsolationStrategy() == THREAD. | 超时的时候是否尝试打断执行线程 | true |
| executionIsolationThreadInterruptOnFutureCancel | Whether the execution thread should be interrupted if the execution observable is unsubscribed or the future is cancelled via Future.cancel(true)). Applicable only when executionIsolationStrategy() == THREAD. | 取消订阅或者调用future.cancel(true)时是否打断工作线程（结构化并发？） | false |
| executionIsolationThreadPoolKeyOverride | Allow a dynamic override of the HystrixThreadPoolKey that will dynamically change which HystrixThreadPool a HystrixCommand executes on. Typically this should return NULL which will cause it to use the HystrixThreadPoolKey injected into a HystrixCommand or derived from the HystrixCommandGroupKey. When set the injected or derived values will be ignored and a new HystrixThreadPool created (if necessary) and the HystrixCommand will begin using the newly defined pool. | | null |
| executionTimeoutInMilliseconds | Time in milliseconds at which point the command will timeout and halt execution. If executionIsolationThreadInterruptOnTimeout == true and the command is thread-isolated, the executing thread will be interrupted. If the command is semaphore-isolated and a HystrixObservableCommand, that command will get unsubscribed. | | 1000 |
| executionTimeoutEnabled | Whether the timeout mechanism is enabled for this command | | true |
| fallbackIsolationSemaphoreMaxConcurrentRequests | Number of concurrent requests permitted to HystrixCommand.getFallback(). Requests beyond the concurrent limit will fail-fast and not attempt retrieving a fallback. | 最大的并发执行fallback的请求数量，超过了就不执行fallback且抛出异常算”执行失败“ | 10 |
| fallbackEnabled | Whether HystrixCommand.getFallback() should be attempted when failure occurs. | | true |
| metricsHealthSnapshotIntervalInMilliseconds | Time in milliseconds to wait between allowing health snapshots to be taken that calculate success and error percentages and affect HystrixCircuitBreaker.isOpen() status. On high-volume circuits the continual calculation of error percentage can become CPU intensive thus this controls how often it is calculated. | 健康快照的执行间隔，健康快照是用来计算成功/失败的百分比的 | 500 |
| metricsRollingPercentileBucketSize | Maximum number of values stored in each bucket of the rolling percentile. This is passed into HystrixRollingPercentile inside HystrixCommandMetrics. | | 100 |
| metricsRollingPercentileEnabled | Whether percentile metrics should be captured using HystrixRollingPercentile inside HystrixCommandMetrics. | | true |
| metricsRollingPercentileWindowInMilliseconds | Duration of percentile rolling window in milliseconds. This is passed into HystrixRollingPercentile inside HystrixCommandMetrics. | | 60000 |
| metricsRollingPercentileWindowBuckets | Number of buckets the rolling percentile window is broken into. This is passed into HystrixRollingPercentile inside HystrixCommandMetrics. | | 6 |
| metricsRollingStatisticalWindowInMilliseconds | Duration of statistical rolling window in milliseconds. This is passed into HystrixRollingNumber inside HystrixCommandMetrics. | | 10000 |
| metricsRollingStatisticalWindowBuckets | Number of buckets the rolling statistical window is broken into. This is passed into HystrixRollingNumber inside HystrixCommandMetrics. | | 10 |
| requestCacheEnabled | Whether HystrixCommand.getCacheKey() should be used with HystrixRequestCache to provide de-duplication functionality via request-scoped caching. | | true |
| requestLogEnabled | Whether HystrixCommand execution and events should be logged to HystrixRequestLog. | | true |