基于CompletableStage的一种链式调用
dependent用于接受结果（使用CompletableFuture中的result字段）

```java
static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                 Supplier<U> f) {
    if (f == null) throw new NullPointerException();
    // d用于接受结果
    CompletableFuture<U> d = new CompletableFuture<U>();
    e.execute(new AsyncSupply<U>(d, f));
    return d;
}
```

每个流程都由dependent整合，然后push到stack里