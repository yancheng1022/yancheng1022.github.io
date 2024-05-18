---
title: 异步非阻塞编程：CompletableFuture
date: 2023/09/17
categories:
  - coding
tags:
  - 技术文章
  - CompletableFuture
  - 并发编程
abbrlink: 20328
---

# 1、背景

我们都知道可以通过继承Thread类或者实现Runnable接口两种方式实现多线程。但是有时候我们希望得到多线程异步任务执行后的结果，也就是异步任务执行后有返回值，Thread和Runnable是不能实现的。当我们需要返回值的时候怎么办呢？ Java 1.5 推出的Callable和Future接口就解决了这个问题。但是因为Future有几个局限，由于这几个局限，在Java1.8就推出了加强版的Future类：CompletableFuture

Future用于表示异步计算的结果，只能通过阻塞或者轮询的方式获取结果，而且不支持设置回调方法，Java8之前若要设置回调一般会使用guava的ListenableFuture。 CompletableFuture对Future进行了扩展，可以通过设置回调的方式处理计算结果，同时也支持组合操作，支持进一步的编排

# 2、Future使用

假如我们现在有如下需求：

老板正在开会，开会过程中发现少一份材料，通知秘书去整理，在秘书整理过程中老板这边还在继续开会，秘书整理完以后将材料给到老板手中。

需求分析：

老板开会是主线程，不能中断。秘书就是异步任务秘书执行完任务需要将结果返回给老板这个主线程手中。咱们看看通过Future实现此需求有什么局限，然后再通过CompletableFuture实现此需求看看是否更好。

Future接口（实现类：FutureTask）定义了操作异步任务执行的一些方法：如获取异步任务执行结果、取消任务的执行结果、判断任务是否被取消、判断任务执行是否完成等。

```java
public class BossMeeting {
 
    /**
     * 主线程为老板正在开会
     */
    public static void main(String[] args) {
        System.out.println("老板开会start");
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        FutureTask<String> secretaryFuture = new FutureTask<>(() -> {
            Thread.sleep(1000);
            return "老板需要的材料";
        });
 
        //老板发现缺少材料，提交异步任务（找秘书）
        executorService.submit(secretaryFuture);
 
        /**
         * 方法1
         * 局限：导致线程堵塞
         */
        try {
            //获取秘书搜集的材料 （堵塞线程）
            String material = secretaryFuture.get();
            System.out.println("秘书搜集到的材料：" + material);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } catch (ExecutionException e) {
            throw new RuntimeException(e);
        }
 
        /**
         * 方法2
         * 通过while轮询方式会消耗cpu
         */
        while (true) {
            if (secretaryFuture.isDone()) {
                try {
                    //获取秘书搜集的材料 （堵塞线程）
                    String material = secretaryFuture.get();
                    System.out.println("秘书搜集到的材料：" + material);
                    break;
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } catch (ExecutionException e) {
                    throw new RuntimeException(e);
                }
            } else {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
        System.out.println("老板开会end");
    }
}
```

输出结果：
老板开会start
秘书搜集到的材料：老板需要的材料
老板开会end
# 3、CompletableFuture实现非阻塞式执行

CompletableFuture提供了一种观察者模式类似的机制，可以让任务执行完成后通知监听的一方。

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
```

用CompletableFuture实现上述的业务场景：

```java
 
/**
 * 老板开会Future实现
 */
public class BossMeeting {
    public static void main(String[] args) {
        System.out.println("老板开会start");
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        try {
            CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                return "秘书搜集完材料";
                //结束返回
            }, executorService).whenComplete((v, e) -> {
                //无异常说明 执行成功
                if (e == null) {
                    System.out.println("秘书搜集到的材料：" + v);
                }
                //异常处理
            }).exceptionally(e -> {
                e.printStackTrace();
                System.out.println("执行异常：" + e.getCause());
                return null;
            });
 
            System.out.println("老板继续开会");
 
            try {
                //模拟老板继续开会3秒钟
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("老板开会end");
        } finally {
            executorService.shutdown();
        }
    }
}



```

执行结果：
老板开会start  
老板继续开会  
秘书搜集到的材料：秘书搜集完材料  
老板开会end


# 4、CompletableFuture相关方法


## 4.1、CompletableFuture创建方式

官方推荐使用CompletableFuture提供的静态方法创建CompletableFuture实例，以下是提供的静态方法：

```java
// 无返回值 使用ForkJoinPool线程池
public static CompletableFuture<Void> runAsync(Runnable runnable)
// 无返回值 可以自定义线程池
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
// 有返回值 使用ForkJoinPool线程池
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
// 有返回值 可以自定义线程池
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

supply开头：这种方法，可以返回异步线程执行之后的结果。  
run开头：这种不会返回结果，就只是执行线程任务。  
如果你想异步运行一些后台任务并且不想从任务中返回任何东西，那么你可以使用run开头的


## 4.2、CompletableFuture获取返回值

通过get、join、getNow获取返回值，区别如下：

join：返回结果或者抛出一个unchecked异常(CompletionException)，不需要显示捕获异常。
get：返回结果或者一个具体的异常(ExecutionException, InterruptedException)，此方法继承至Future是堵塞的。
getNow：如果当前任务执行完成，返回执行结果，否则返回valueIfAbsent（默认值）。

```java
 
    /**
     * 通过get获取方法
     */
    public void test1() {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "get方法需要显示捕获异常");
        try {
            System.out.println(future.get());
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } catch (ExecutionException e) {
            throw new RuntimeException(e);
        }
    }
 
    /**
     * join 不需要显示捕获异常
     */
    public void test2() {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "join方法不需要显示捕获异常");
        System.out.println(future.join());
    }
 
    /**
     * getNow方法可以设置默认值
     * 在有效的时间内，未返回结果，则直接返回默认值
     */
    public void test3() {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "getNow获取返回值");
        System.out.println(future.getNow("默认值"));
    }
```

## 4.3、其它方法

- thenApply()：拿到上一个异步执行的结果继续后续操作

```java
// 模拟 1 + 1 + 1
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 1).thenApply(v -> v + 1).thenApply(v -> v + 1);
System.out.println("执行结果：" + future.getNow(-1)); //3
```

- thenAccept() 和 thenRun()方法:如果你不想从你的回调函数中返回任何东西，只想在 Future 完成后运行一些代码，那么你可以使用thenAccept()andthenRun()方法。这些方法是消费者Consumer< ? super T> action，通常用作回调链中的最后一个回调。

```java
// 模拟 1 + 1 + 1
CompletableFuture.supplyAsync(() -> 1).thenApply(v -> v + 1).thenApply(v -> v + 1)
        thenAccept(r -> System.out.println("1+1+1=" + r));
```

- complete()：当前阶段异步任务执行完成.complete()其实也是个消费操作，但是与thenRun()不同的是，里面可以可抛出的异常
  
```java
// 区别就是不是异步处理
public CompletableFuture<T>  whenComplete(BiConsumer<? super T,? super Throwable> action)
// 使用异步处理
public CompletableFuture<T>  whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
// 区别在于可以指定线程池
public CompletableFuture<T>  whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
// 接收一个可抛出的异常，且必须有返回值
public CompletableFuture<T>  exceptionally(Function<Throwable,? extends T> fn)
```

- handle()：相比thenApply()抛出异常后还可以继续执行

```java
public <U> CompletableFuture<U> handle(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor)
```


# 5、综合应用

需求：要查找10个订单信息以及关联的商品、图片信息

订单上有商品ID，通过商品ID可以查询到商品详细信息，图片信息存储在商品详细信息中。

那就需要查询完订单再查询商品最后查询图片信息，这3个异步任务需要串行执行。

```java
public class Test {
 
    public static void main(String[] args) {
        //10个订单号
        List<String> orderCodeList = Arrays.asList(new String[]{"order_01", "order_02", "order_03", "order_04",
                "order_05", "order_06", "order_07", "order_08", "order_09", "order_10"});
 
        //定义线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(15);
 
        try {
            List<String> collect = orderCodeList.stream().map(o ->
                    CompletableFuture.supplyAsync(() -> String.format("订单：%s，关联商品ID为：%s", o, ThreadLocalRandom.current().nextInt()), threadPool)
                            .thenApplyAsync((v) -> String.format(v + ",关联图片ID为：%s", ThreadLocalRandom.current().nextInt()), threadPool)
                            .thenApplyAsync((v) -> String.format(v + ",关联图信息获取成功"), threadPool)
                            .exceptionally(e -> {
                                e.printStackTrace();
                                return null;
                            })
                            .join()
            ).collect(Collectors.toList());
 
            //打印结果
            System.out.println(collect);
        } finally {
            //释放资源
            threadPool.shutdown();
        }
 
    }
}
```