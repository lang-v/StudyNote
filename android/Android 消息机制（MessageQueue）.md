# Android 消息机制（MessageQueue）

## 前言

引入问题：

**在Android的消息机制中，主线程在Looper.loop()中循环读取消息并处理。那么问题来了这个for循环是写死的，那么是不是说明主线程一直在不停的工作呢？**

显然不是的。

先看代码，可以看到每次读取消息前总是会先调用nativePollOnce方法。

```java
   Message next() {
   		...
        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            ...
            // 每次开始读取下一个Message之前都调用了这个方法。它是一个native方法，在方法中调用了epoll_wait等方法。
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
            	// 
            }
       }
}
```





## 同步屏障



## IO多路复用与 Object.wait()、Object.notify()、Object.notifyAll() 是什么关系

### Java  线程的几个状态

1. NEW 线程对象被创建，未start
2. RUNABLE ；此状态常被分为两个小状态
   1. READY；就绪态，线程可以开始运行了
   2. RUNNING；正在运行
3. BLOCKED；阻塞，调用了Thread.sleep (time) 等会阻塞线程的方法时，线程进入此状态
4. WAIT；等待状态，调用如Object.wait() 方法时进入此状态
5. TIMED_WAIT；调用Object.wait (time)  进入此状态
6. TERMINATED；线程生命周期结束

### 那么Object.wait notify 做了什么呢？



