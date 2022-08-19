---
title: java创建线程的7种方式
tags:
  - java
  - 多线程
categories:
  - java
  - 多线程
date: 2019-10-09
---

## java创建线程的7种方式

1. 继承Thread类并重写run\(\)方法

```java
public class MyThread extends Thread {
    @Override
  public void run() {
        System.out.println(this.getId()+this.getName()+ " is running");
  }

    public static void main(String[] arg){
        new MyThread().start();
 new MyThread().start();
 new MyThread().start();
 new MyThread().start();
  }
}
```

```text
$ "C:\Program Files\Java\jdk1.8.0_171\bin\java.exe"
12Thread-0 is running
15Thread-3 is running
13Thread-1 is running
14Thread-2 is running

Process finished with exit code 0
```

1. 实现`Runnable`接口

```java
public class MyThread implements Runnable{
    @Override
  public void run() {
        System.out.println(Thread.currentThread().getName()+" is running");
  }
    public static void main(String[] args){
        new Thread(new MyThread()).start();
 new Thread(new MyThread()).start();
 new Thread(new MyThread()).start();
 new Thread(new MyThread()).start();    }
}
```

1. 匿名内部类

```java
public class MyThread {
    public static void main(String[] args){
        new Thread(){
            public void run(){
                System.out.println(this.getName() + " is running");
  }
        }.start();   new Thread(new Runnable() {
            @Override
  public void run() {
                System.out.println(Thread.currentThread().getName() + " is running");
  }
        }).start();   new Thread(()->{
            System.out.println(Thread.currentThread().getName()+" is running");
  }).start();    }
}
```

1. 实现Callabe接口

```java
public class MyThread implements Callable<Long> {

    @Override
  public Long call() throws Exception {
        Thread.sleep(2000);
  System.out.println(Thread.currentThread().getName()+" is running");
 return Thread.currentThread().getId();
  }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask futureTask = new FutureTask(new MyThread());
 new Thread(futureTask).start();
  System.out.println("等待完成任务");
  Long result = (Long) futureTask.get();
  System.out.println("任务结果是：" + result);
  }
}
```

1. 定时器（java.util.Timer）

```java
public class MyThread {
    public static void main(String[] args){
        Timer timer = new Timer();
  timer.schedule(new TimerTask() {
            @Override
  public void run() {
                System.out.println(Thread.currentThread().getName() + " is running");
  }
        }, 0, 1000);
  }
}
```

1. 线程池

```java
public class MyThread {

    public static void main(String[] args){

        ExecutorService threadPool  = Executors.newFixedThreadPool(10);
 for (int i =0; i< 100; i++){

            threadPool.execute(()->System.out.println(Thread.currentThread().getName()+" is running"));    }
    }
}
```

1. 并行计算

```java
public class MyThread {

    public static void main(String[] args){

        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
  list.stream().forEach(System.out::print);    System.out.println();
  list.parallelStream().forEach(item->System.out.print(item));
  }
}
```

## 总结

上面介绍了那么多创建线程的方式，其实本质上就两种，一种是继承Thread类并重写其run\(\)方法，一种是实现Runnable接口的run\(\)方法。
