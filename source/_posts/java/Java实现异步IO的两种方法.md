title: Java实现异步IO的两种方法
date: 2015-08-19 20:12:23
tag: Java
update: 2015-08-19 20:12:23
comments: true
categories: Java
---

#### 1、概念

>软件模块之间的调用方式可以分为三类：
>- **同步调用**：一种阻塞式调用，调用方要等待对象执行完毕才返回。它是一种单向调用。
>- **回调**：一种双向调用模式，也就是说，被调用方在接口被调用的同时会调用对方得接口。(回调函数也是一个函数或过程，不过它是一个由调用方自己实现，供被调用方使用的特殊函数。)
>- **异步调用**：异步调用是一种类似消息或者事件的机制，不过它的调用方向刚好相反，接口的服务在收到被调用的信息或事件时，会主动调用（调用方）的接口。

	一般情况下，使用回调实现异步消息的注册，通过异步调用来实现消息的通知，回调是异步调用的基础。

<!--more-->

#### 2、实现方法
##### 回调（CallBack）
经典的使用回调的方式（if you call me ,i will call you）： 
class A实现接口InA ——背景1
class A中包含一个class B的引用b ——背景2
class B有一个参数为InA的方法test(InA a) ——背景3
A的对象a调用B的方法传入自己，test(a) ——这一步相当于you call me
然后b就可以在test方法中调用InA的方法 ——这一步相当于i call you back
###### 实现
调用者
```
/**
 * Created by hzzhengxianrui on 2015/8/29.
 */
package com.davkas.synio.action;
public class Worker {
    public void doWork(){
        Fetcher fetcher = new MyFetcher(new Data(1,0));
        fetcher.fetchData(new FetcherCallBack() {
            @Override
            public void onData(Data data) throws Exception {
                System.out.println("An Data received: " + data);
            }

            @Override
            public void onError(Throwable cause) {
                System.out.println("An error accour: "+ cause.getMessage());
            }
        });

    }
    public static void main(String[] args){
        Worker w = new Worker();
        w.doWork();
    }
}
```
回调接口
```
package com.davkas.synio.action;

import com.davkas.synio.action.Data;

/**
 * Created by hzzhengxianrui on 2015/8/29.
 */
public interface FetcherCallBack {
    void onData(Data data) throws Exception;
    void onError(Throwable cause);
}
```
被调用者的接口
```
package com.davkas.synio.action;

/**
 * Created by hzzhengxianrui on 2015/8/29.
 */
public interface Fetcher {
    void fetchData(FetcherCallBack callBack);
}

```
<font color="fff000000">被调用者的实现：Fetcher接口的实现</font>
```
package com.davkas.synio.action;
import com.davkas.synio.action.Data;
/**
 * Created by hzzhengxianrui on 2015/8/29.
 */
public class MyFetcher implements Fetcher {
    final Data data;
    public MyFetcher(Data data){
        this.data = data;
    }
    public void fetchData(FetcherCallBack callBack){
        try {
	        //此处可以执行耗时操作，在耗时操作完成后调用callback的方法返回数据或者通知调用者该事情已经完成
            callBack.onData(data);
        }catch(Exception e){
            callBack.onError(e);
        }
    }
}
```
在Java中，存在Callable接口，Callable接口一般是和ExecutorService配合来使用的，用于在线程的操作结束后完成数据的传递。ExecutorService的submit方法中有一个的参数即为Callable。

#### Futrue
<font color="fff000000">需要再看下Futrue模式的内容</font>
Future类似于期权或者订货单，一般用于需要从子线程中获取所需要的结果时使用。在多线程中一般会被认为是“虚拟代理模式”。使用ExecutorService的submit方法同样可以以Future的实例为对象。

>   A Future represents the result of an asynchronous computation. Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation. The result can only be retrieved using method get when the computation has completed, blocking if necessary until it is ready. Cancellation is performed by the cancel method. Additional methods are provided to determine if the task completed normally or was cancelled. Once a computation has completed, the computation cannot be cancelled. If you would like to use a Future for the sake of cancellability but not provide a usable result, you can declare types of the form Future<?> and return null as a result of the underlying task.

```
package com.davkas.synio.action;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * Created by hzzhengxianrui on 2015/8/29.
 */
public class FutureExample {
    public static void main(String[] args) throws Exception{
        ExecutorService executor = Executors.newCachedThreadPool();
        Runnable task1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("i am task1 ...");
            }
        };
        Callable<Integer> task2 = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return new Integer(100);
            }
        };
        Future<?> f1 = executor.submit(task1);
        Future<Integer> f2 = executor.submit(task2);
        System.out.println("task1 is completed ?" + f1.isDone());
        System.out.println("task2 is completed ?" +f2.isDone());
        while(f2.isDone()){}
            System.out.println("return value by task2: " + f2.get());
    }
}

```

#### 参考资料
---
1. [异步消息的传递－回调机制](http://www.ibm.com/developerworks/cn/linux/l-callback/)
2. [Netty In Action 第一章](www.baidu.com)
3. [Java多线程设计模式](www.baidu.com)

