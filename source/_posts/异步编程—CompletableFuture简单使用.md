---
title: 异步编程—CompletableFuture简单使用
tags: 
 - Java
 - asynchronized
urlname: asynchronized-using-completablefuture
date: 2019-04-03 16:09:04
---

> 虽说到现在所接触的项目慢慢地都开始使用了`Java 8`了，但是很多时候并没有使用到`Java 8`中的新特性，还是照着老的用法写:joy:。这几天碰到的任务恰好需要用到多线程去处理，单个线程耗时太久了，本来是想用最原始的那种方式（继承`Thread`，实现`Runable`接口这种），和组内的同事交流了下，组里的同事说了下`Future`这种方式，在`Java 8` 中可以使用`CompletableFuture`这个去优雅的处理多线程的问题，因此这篇文章主要记录下`CompletableFuture`的简单使用，实践下多线程:grin:

## 介绍

![image.png](https://i.loli.net/2019/08/25/KBdxtJ4SVL2F8HI.png)

参考这篇文章: [Java8新的异步编程方式 CompletableFuture(一)](<https://www.jianshu.com/p/dff9063e1ab6>)

## 使用

1.像普通的类一样new 一个对象（实际中不会使用这种方式）：

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("begin")
        CompletableFuture<String> future = new CompletableFuture<String>();
        future.complete("do future task");
        System.out.println(future.get());
}

```

2.使用静态方法：

2.1不需要返回结果(`runAsync()`)：

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("begin["+Thread.currentThread().getName()+"]");
        CompletableFuture<Void> future = CompletableFuture.runAsync(()->{
            System.out.println("do future task ["+Thread.currentThread().getName()+"]");
        });
        System.out.println("end["+Thread.currentThread().getName()+"]");
}
```

2.2需要返回结果(`supplyAsync()`)：

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("begin["+Thread.currentThread().getName()+"]");
        CompletableFuture<String> future =CompletableFuture.supplyAsync(()->{
            System.out.println("do future task["+Thread.currentThread().getName()+"]");
            try {
                Thread.sleep(5000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "done";
        });
        System.out.println(future.get());
        System.out.println("end["+Thread.currentThread().getName()+"]");
}
```



> 需要注意：`get()`方法是阻塞的，会等到`Future`执行完才会返回

等待线程执行完再去处理其他任务（例如导入Excel入库后，新增导入历史记录）：

`thenApplyAsync()`

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("begin["+Thread.currentThread().getName()+"]");
        Executor executor =Executors.newFixedThreadPool(2);
        CompletableFuture<String> future = CompletableFuture.supplyAsync(()->{
            System.out.println("supply>>");
            return "do future task";
        }).thenApplyAsync((re)->{
            System.out.println(re+" then sysout");
            return "then";
        });
        System.out.println(future.get());
        System.out.println("end["+Thread.currentThread().getName()+"]");
}
```

目前就用到了这些，后续用到其他“功能”再补充（这篇文章拖了一个清明节假，逃...）

---

参考文章：

[Java 8 CompletableFuture 教程](<https://juejin.im/post/5adbf8226fb9a07aac240a67>)

[Java8新的异步编程方式 CompletableFuture(一)](<https://www.jianshu.com/p/dff9063e1ab6>)