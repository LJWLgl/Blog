---
title: "异步——ListenalbeFuture的使用总结"
date: 2019-04-01 16:50:33
tags: ListenalbeFuture
categories: 并发
summary:  "为了提高任务处理速度，我们经常会将一些可并行处理的步骤用异步的方式去处理，如果想要获取异步计算的结果"
---
为了提高任务处理速度，我们经常会将一些可并行处理的步骤用异步的方式去处理，如果想要获取异步计算的结果，在Java 8之前更多的用的是`Future` + `Callable`的方式来实现，<!-- more -->下面是使用Future和Callable的一个demo，其中的是`executor.submit()`方法实际返回的就是`FutureTask`的实例，另外Future的get方法会一直阻塞直至获取结果。
```java
public class FutureTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<Integer> future = executor.submit(new MyCallable(3, 10));
        // get方法会阻塞，直至获取结果
        System.out.println(future.get());
        executor.shutdown();
    }
}

class MyCallable implements Callable<Integer> {
    private int a;
    private int b;

    public MyCallable(int a, int b) {
        this.a = a;
        this.b = b;
    }
    @Override
    public Integer call() throws Exception {
        return a * b;
    }
}
```
虽然Future已经相关方法提供了异步编程的能力，但是获取结果十分不方便，只能通过阻塞或者轮询的方式获取结果，阻塞的方式显然与我们异步编程的初衷相违背，而且轮询的方式也很消耗的CPU资源，计算结果也不能及时拿到。面对这种情况，为什么不采用一种类似观察者模式的方式，当结果结算完成后实时通知到监听任务呢？著名的`guava`包就提供了拓展Future，如ListenableFuture和SettableFuture，以及辅助类Futures。JDK 8中也提供类似`ListenableFuture`的`CompletableFuture`接口，该接口包含很多api，后续的文章会逐一介绍。下面我们主要介绍一下Guava Future的使用。
### 引入Guava
最新版本的Guava可到[Maven Center](https://mvnrepository.com/artifact/com.google.guava/guava)中查找
```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>23.0</version>
</dependency>
```
### 创建ListeningExecutorService
Guava为了支持自己的Listerner模式，新建了一种`ExecutorService`，叫做`ListeningExecutorService`，我们可以使用`MoreExecutor`创建它
```java
// 创建一个由invoke线程执行的线程池
 ListeningExecutorService executorService = MoreExecutors.newDirectExecutorService();
 // 装饰自定义的线程池返回
 ListeningExecutorService executorService1 = MoreExecutors.listeningDecorator(Executors.newCachedThreadPool());
```
线程池创建完毕后，我们就可以创建`ListenableFuture`了
```java
ListenableFuture<?> listenableFuture = executorService.submit(new MyCallable(3, 10));
```
### 添加监听（addListener）
ListenableFuture接口扩展自Future接口，并添加了一个新方法 `addListener`，该方法是给异步任务添加监听
```java
  listenableFuture.addListener(() -> {
        System.out.println("listen success");
        doSomeThing();
    }, executorService);
```
### 添加回调（Futures.addCallBack）
`addListener`方法不支持获取返回值，如果需要获取返回值，可以使用`Futures.addCallBack`静态方法，该类是对JDK Future的拓展
```java
// FutureCallback接口包含onSuccess()、onFailure()两个方法
Futures.addCallback(listenableFuture, new FutureCallback<Object>() {
    @Override
    public void onSuccess(@Nullable Object result) {
        System.out.println("res: " + result);
    }

    @Override
    public void onFailure(Throwable t) {}
}, executorService);
```
### 合并多个Future（Futures.allAsList）
如果需要同时获取多个Future的值，可以使用`Futures.allAsList`，需要注意的是如果任何一个Future在执行时出现异常，都会只执行`onFailure()`方法，如果想获取到正常返回的Future，可以使用`Futures.successfulAsList`方法，该方法会将失败或取消的Future的结果用`null`来替代，不会让程序进入`onFailure()`方法
```java
ListenableFuture<Integer> future1 = executorService.submit(() -> 1 + 2);
ListenableFuture<Integer> future2 = executorService.submit(() -> Integer.parseInt("3q"));
ListenableFuture<List<Object>> futures = Futures.allAsList(future1, future2);
futures = Futures.successfulAsList(future1, future2);

Futures.addCallback(futures, new FutureCallback<List<Object>>() {
    @Override
    public void onSuccess(@Nullable List<Object> result) {
        System.out.println(result);
    }
    @Override
    public void onFailure(Throwable t) {
        t.printStackTrace();
    }
}, executorService);
```

### 返回值转换（Futures.transform）
如果需要对返回值做处理，可以使用`Futures.transform`方法，它是同步方法，另外还有一个异步方法`Futures.transformAsync`
```java
// 原Future
ListenableFuture<String> future3 = executorService.submit(() -> "hello, future");
// 同步转换
ListenableFuture<Integer> future5 = Futures.transform(future3, String::length, executorService);
// 异步转换
ListenableFuture<Integer> future6 = Futures.transformAsync(future3, input -> Futures.immediateFuture(input.length()), executorService);
```
### immediateFuture和immediateCancelledFuture
` immediateFuture`该方法会立即返回一个待返回值的`ListenableFuture`，`immediateCancelledFuture`会返回一个立即取消的`ListenableFuture`，所以它返回的Future的`isDone`方法始终是false
### JdkFutureAdapters
该方法可以将`JDK Future`转成`ListenableFuture`
```java
Future<String> stringFuture = Executors.newCachedThreadPool().submit(() -> "hello,world");
ListenableFuture<String> future7 = JdkFutureAdapters.listenInPoolThread(stringFuture);
System.out.println(future7.get());
```
### SettableFuture
`SettableFuture`可以认为是一种异步转同步工具，可以它在指定时间内获取`ListenableFuture`的计算结果
```java
SettableFuture<Integer> settableFuture = SettableFuture.create();
ListenableFuture<Integer> future11 = executorService.submit(() -> {
    int sum = 5 + 6;
    settableFuture.set(sum);
    return sum;
});
// get设置超时时间 
System.out.println(settableFuture.get(2, TimeUnit.SECONDS));
```