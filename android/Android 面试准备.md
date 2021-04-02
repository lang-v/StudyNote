# Android 面试准备

### 计算机基础（通信UDP\TCP\IP 三次握手、四次挥手）

#### OSI七层模型

1. 应用层
2. 表示层
3. 会话层
4. 传输层 ：TCP/IP TCP报文首部20字节、IP报文首部20字节、UDP首部8个字节
5. 网络层
6. 数据链路层
7. 物理层

#### UDP

User Datagarm Protocol  用户数据报协议，位于OSI模型中的运输层

特点：

- 无连接
- 不可靠
- 快速传输（首部开销小）

##### UDP报文段格式

![UDP首部字段](https://upload-images.jianshu.io/upload_images/2053709-4b2a2f4e00243758.png?imageMogr2/auto-orient/strip|imageView2/2/w/1180/format/webp)

其中分为两个部分：UDP伪首部、UDP首部

1. UDP伪首部：

共12字节：4字节源IP地址、4字节目的IP地址、填充0(占一个字节)、一个字节协议字段（UDP为17、TCP为16）、两个字节UDP长度（与下面的首部字段中的长度一致）

*注意* ：这些字段的不会向下传送也不会向上递交，是从IP分组的头部获取的。

2. UDP**首部字段**

（仅有8个字节即64个比特，首部开销较小也是它的优势之一）分为四个字段每个字段占2字节：

1. 前两个字段分别为**16位的源端口号和16位目的端口号**（计算机端口共65535个即2的16次方-1）

2. 后两个字段分别是长度、检验和，同样都是两个字节的长度

   - 长度：单位为**字节**，最大表示长度就为2^16 = 65535字节，那么数据报单次传输内容的长度限制是不是就是它呢？

   - 检验和：UDP通过检验和提供差错检测功能。将前面三个字段（源端口号、目的端接口号、长度）相加得到的值取反码即为检验和

##### UDP差错检测

已知检验和为源端口号、目的端接口号、长度三者相加得到的值取反码，那么三者相加的值再加上检验和 等于 一个所有位数都为1的二进制数。例如： 和为1001 0110，则检验和为：0110 1001

 1001 0110+0110 1001 = 1111 1111

则可以通过的最后结果中有没有0来判断数据是否出现差错

#### TCP

Tranform Controller Protocol 传输控制协议，同样位于OSI模型中的运输层。

##### 特点

- 面向字节流
- 提供全双工通信
- 可靠的通信方式（无差错、不丢失、不重复）
- 面向连接的单播协议
- 在网络状况不佳的时候尽量降低系统由于重传带来的带宽开销
- 通信连接维护时面向通信的两个端口的，而不考虑中间网段和节点

##### TCP报文段格式

![TCP报文段格式](http://5b0988e595225.cdn.sohucs.com/images/20180910/fdb152e3d8f547b7bec66b537844dce2.jpeg)

ACK —— 确认，使得确认号有效。 RST —— 重置连接（经常看到的reset by peer）就是此字段搞的鬼。 SYN —— 用于初如化一个连接的序列号。 FIN —— 该报文段的发送方已经结束向对方发送数据。

![三次握手，四次挥手](https://pic3.zhimg.com/80/v2-e8aaab48ff996e5cd8a5b39dc450bd6a_720w.jpg)

##### 三次握手

最明显的作用就是使得双方都明确对方的收、发功能正常，同时相比直接连接无须三次握手相比，极大程度上减少了服务器被恶意攻击多次连接导致服务器大量资源处于等待阶段，从而导致资源浪费。

*注*：报文段中存在一个序号Seq number 和一个确认序号Ack number，此外还有一个ACK字段位长度为1。只有当ACK为1时Ack number才有效

1. 第一次握手 

   客户端向服务器发送SYN报文，即SYN字段值为1

2. 第二次握手

   服务器接收到SYN报文说明客户端的发功能正常，接着向客户端发送SYNACk报文，即SYN、ACK字段值为1，且Ack number = Seq number+1 表明收到了上一个报文

3. 第三次握手

   客户端接收到SYNACK报文说明服务器的收发能力都正常，向服务器发送ACK报文，Ack number = 上一个Seq number+1 表明收到了上一个报文，使得服务器知道客户端的收发能力正常，即连接成功。

##### 四次挥手

![四次挥手](https://pic3.zhimg.com/80/v2-629f51f6f535ebd7683f944707b21d1e_720w.jpg)

1. 第一次挥手

   客户端发送FIN报文，并附带当前Seq number、Ack number 和ACK用于确认收到了上一个包。

2. 第二次挥手

   服务器接收到FIN报文，会将还没发送的数据发送，然后发送ACK报文，此时客户端不会再发送消息了即发功能已关闭，只是再等待服务器发送的关闭报文。

3. 第三次挥手

   服务器发送FIN报文，Ack number  = 客户端FIN报文序号Seq number + 1,ACK = 1。

4. 第四次挥手

   客户端接收到FIN报文，向服务器发送ACK报文，Ack number = 服务器FIN报文序号Seq number + 1 ，ACK = 1。

##### 为什么建立连接是三次握手而断开连接是四次挥手？

建立连接时需要判断双方的收发能力是否正常，而要完成这个操作最少需要发送三次报文，即三次握手；分析建立连接时发送的报文可以看到，三次握手发送的报文分别为 SYN 、SYN+ACK、ACK三个报文，即第二次握手时同时发送了SYN建立连接ACK确认报文，而断开连接报文分别是：FIN、ACK、FIN、ACk，很明显是中间的ACK和FIN报文分开发送了，它们不能合并的原因是服务器接收到客户端的FIN报文，说明客户端想要关闭连接了，即客户端不会再发送消息，但是依然能接收消息，此时服务器可以选择立即关闭连接也可以选择把没发送的报文发送过去再关闭连接，所以会将FIN和ACK报文分开等到服务器将要发送的报文发送完毕，再发送FIN报文通知客户端现在关闭连接。

##### 滑动窗口机制

提高发送效率，TCP为了保证报文段的有序性使用了send-wait-send机制，需要在发送后收到确认才会继续发送下一个报文，效率较低。采用滑动窗口机制提高发送效率。

在同一时间发送多个报文段，不需要等待客户端确认。如果客户端接受时发现报文段未有序到达或者丢失，此时便返回确认报文段告知发送方自己期望收到的是哪一个序号的报文，发送方再重新发送此报文；最后接受方收到了期望的序号，会继续告知发送方下一个期望收到的报文序号，发送方通过此报文控制发送窗口的前移。

**注意**：TCP不建议控制发送窗口收缩，可能会导致错误

##### 流量控制

 通过可变窗口控制发送速率。

##### 拥塞控制 

TCP拥塞控制算法分四种

1. 慢开始&拥塞避免

   慢开始，通过控制发送窗口的大小，达到目的， RFC5681 规定初始拥塞窗口（等于发送窗口）的大小为2-4个最大报文段数量（SMSS）。拥塞窗口的大小是递增的（算法：**拥塞窗口cwnd每次的增加量 = min(N,SMSS)** ）。N为收到的确认报文段所确认收到的字节数。总结来看就是每次过一个轮次cwnd就翻一倍。

   拥塞避免，相比慢开始的翻倍增加，拥塞避免选择每经过一个往返时间RTT就让cwnd加1。线性增长使得网络不容易出现网络拥塞。

   **何时使用慢开始和拥塞避免**：ssthresh 慢开始门限，当发生超时时（即发生了网络拥塞），**ssthresh将被重置为当前cwnd/2 ,cwnd将被置为1**。

   - cwnd < ssthresh  慢开始
   - cwnd > ssthresh 停止慢开始使用拥塞避免
   - cwnd = ssthresh 两者皆可

2. 快重传&快恢复

   上述两种算法在报文丢失时会重置cwnd为1，对于网络拥塞时是非常有效的。如果报文在网络中丢失但是并没有发生网络拥塞，此时将会降低发送效率。

   使用场景：收到M1，M2 丢失M3，后续收到M4-M7的时候发送对M2的确认，共重复三次。

   ![](https://img-blog.csdn.net/20170415161738770?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTEwMDc3Mjk5OTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

   收到三次对M2的重复，发送方就会对M3进行立即重传（所以整个过程对M2进行四次重传） ，因为三次重复确认所以不会启动慢开始，而是启动快恢复，调整 ssthresh = cwnd = cwnd/2 ,然后开始执行拥塞避免算法（ssthresh <= cwnd）

##### 安全

1. DDoS 拒绝服务攻击（SYN洪水攻击）

   攻击者发送大量的TCP SYN报文段，但是不完成第三次握手，导致服务器不断地为这些半开连接（但是未被使用）从而耗尽资源。应对方法Syn cookie

##### 扩展

- Seq numbner  等于上一条报文的Ack number 

- Ack number 等于上一条报文的Seq number + 1

- ISN 即初始化序号（Initial Sequence Number），是一个随机数不可预测，每次建立新的TCP连接发起的SYn报文携带的序号就是ISN

- ISN 的计算方法 

  ```java
  int ISN = M + F(localhost, localport, remotehost, remoteport);
  ```

  其中M是一个计时器，每隔4毫秒加1，F是一个Hash算法（根据源IP、源端口、目的IP、目的端口）生成一个随机数字，这个算法可能不准确，但是打开意思就是这个。

- Syn cookie 方法：服务器在接收到SYN报文以后，不会立即生成一个半连接，而是通过使用ISN的计算方法计算得出此报文的初始ISN，这样的ISN称为“cookie”，并将携带此序列号的SYNACK报文发送给客户端，如果客户端合法那么会接着给服务器发送ACK报文确认建立连接，此时服务器将会为此客户端分配资源。、

- TCP的状态序列

  服务端

  ![服务端TCP状态序列](https://img-blog.csdn.net/20180423135904399?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RyZWFtZXI4NDExMTk1NTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

  客户端

  ![客户端TCP状态序列](https://img-blog.csdn.net/20180423135815703?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RyZWFtZXI4NDExMTk1NTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 位运算&二进制

##### 原码、反码、补码

###### 原码

就是符号位加上这个数的绝对值的二进制表示

###### 反码

正数即为原码

负数即符号位不变，其余位取反

###### 补码

正数为原码

负数为其反码加1

##### 基础运算

1. 与 & 

   1001 0010 & 1001 1101 = 1001 0000 

   规则：同为1 则为1

2. 或 | 

   1001 0010 | 1001 1101 = 1001 1111 

   规则：有一个或同为1 则为1

3. 异或  ^

   1001 0010 | 1001 1101 = 0000 1111 

   规则：相异则为1

4. 左移 << 、右移>>、无符号右移>>>

   1001 0010 >> 2 = 0010 0100 空位由0填充

   1001 0010 << 2 = 0100 1000 空位由0填充

##### ColorInt

表示颜色有(每一个字段对应8个比特，即表示一个完整的颜色需要32位二进制 刚好就是一个int值) aRGB （透明度（默认为FF）、红、绿、蓝）

```java
@ColorInt
int color = 0x331122;
int alpha = color>>24 & 0xFF; 	//获取透明度 FF
int red = color>>16 & 0xFF; 		//获取红色 33
int green = color>>8 & 0xFF;	//获取绿色 11
int blue = color & 0xFF;		//获取蓝色 22
```

##### 总结

通过灵活使用位运算可以在一个小小的int身上保存多个状态，甚至可以通过异或运算做一些更加神奇的事情。

### java基础

#### Java 泛型

1. 什么是泛型擦除？

   泛型通过泛型擦除来实现的，在编译期间擦除所有类型相关信息，即运行时不存在类型信息。

2. 什么是泛型中的限定通配符和非限定通配符？

   限定通配符有两种 

   - <? extends T> 限定类型必须为T的子类
   - <? super T> 限定类型必须为T的父类或者T

   非限定符仅一种

   - <?>  \<T> 表示接收任意类型

#### 类加载

注意变量初始化顺序

![](https://img-blog.csdn.net/20160504235346278?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##### （双亲委派模型）

**作用**：避免在内存中出现两份相同的字节码

在子级类加载器需要加载一个类的时候优先委托给父级类加载器（一级一级往上委托），最终如果顶级类加载器也无法加载此类，就交由子级类加载器处理。

***

面试时可能会问类的加载过程。

需要注意：

- 类的静态成员

***

#### 何为垃圾？

判断对象是否为垃圾，可以通过以下两种算法：

- 引用计数法

  引用一个对象是计数器加一，引用失效时计数器减一。计数器为空则为垃圾。

- 可达性分析

  从GC Roots不可达，就是垃圾。

##### 何为GC Roots

1. 在虚拟机栈（栈帧中的本地变量表）中引用的对象。
2. 方法区中的类静态属性引用的对象，譬如Java类的引用类型静态变量。
3. 方法区中常量引用的对象。
4. 在本地方法栈中JNI（Native方法）引用的对象。
5. Java虚拟机内部的引用，如基本数据类型的Class对象、常驻的异常对象、系统类加载器。
6. 同步锁（Synchronized）持有的对象
7. 反映Java虚拟机内部情况的JMXBean、JVMTI 中注册的回调、本地代码缓存等。

#### 分代垃圾回收器

将堆分为年轻代和老年代（old generation）

##### 年轻代（Young Generation）

分为一个Eden区、一个Survivor区（分两个块），即三个区比例为8：1：1

Suvivor的两个块，必须保持一块为空的。每次对年轻代进行垃圾回收是对Eden区和不为空的Survivor，将存活的对象复制到为空的Survivor区且将这些对象的对象头部的分代年龄加1（若年龄超过15(默认值)则加入老年代），若对象大小超过Survivor将会将此对象直接加入老年代。

如上所述，位于年轻代的垃圾清理算法为：**标记复制算法**

##### 老年代（Old Generation）

当老年代的空间不足时，进行垃圾回收，算法上和年轻代不同。将所有存活对象移动到一端，然后对其他的对象进行回收。

清理算法为：**标记整理算法**

> 注意：两个区是分开清理还是整体一起清理，根据不同的收集器有不同的表现

#### 四大引用

自JDK1.3 将引用类型分为以下四种

1. 强引用
2. 软引用
3. 弱引用
4. 虚引用



#### java中的几个修饰符

1. public 权限最大，可以被任意类访问。
2. private 常用来修饰属性，封装的一种体现，被修饰者只由本类能访问。
3. protected 被修饰者可以被子类、和同一个包中的类访问。
4. default 就是默认情况（即没有用上面三个修饰符去修饰的时候），同一个包中的类和自身可以访问。

![](https://img-blog.csdn.net/20170527154942485?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveG1jMjgxMTQxOTQ3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



需要注意的一点就是：Java的访问控制停留在编译层，可以通过**反射**访问任何类的任何成员。

#### Java 内存模型&Java 内存结构

![](https://pic3.zhimg.com/80/v2-041214d71e6d0fec4dfa8e30b5731546_720w.jpg)

自JDK1.8起便移除了永久代，由元空间代替。

java中的每一个线程都拥有一自己的线程栈（工作内存）。工作内存中存有主存中的变量副本，对变量的读写操作均发生在副本上，不同线程之间不能直接互相访问对方的工作内存，（通信）需要通过主存来实现。

java内存模型是一系列的规则，保证了并发时的线程可见性。Happens-before可见性原则，

具体的规则如下：

1. 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生与书写在后面的操作。【保证单线程的有序】

2. 锁定规则：一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。

3. volatile 变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作。【先写后读】

4. 传递规则：A 先于 B 且 B 先于 C 则 A 先于 C

5. 线程启动规则：Thread 对象的 start 方法先行发生于此线程的每一个动作。

6. 线程中断规则：对线程 interrupt 方法的调用先行发生于被中断线程的代码检测到中断事件的发生。【先中断，后检测】

7. 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过 Thread.join() 方法结束，Thread.isAlive() 的返回值手段检测线程已经终止执行。

8. 对象终结规则：一个对象的初始化完成先行发生于它的 finalize 方法的开始。

如果两个操作的执行顺序不能通过 happens-before 原则推导出来，就不能保证他们的执行次序，虚拟机就可以随意的对他们进行**重排序**(通过volatile避免指令重排序)。

#### Java线程和系统线程

1. 在jdk1.2之前，java的线程被称为绿色线程；该线程的调度完全在用户态实现，即这些进程同时运行在操作系统的一个进程中，而这个进程由系统进行调度。

   优点：

   - 即使系统不支持线程，也可以通过自身实现的线程调度器，在用户态实现多线程。
   - 线程的调度只在用户态，减少了系统从内核态到用户态切换的开销 //这里没太懂

   缺点：

   - 由于多线程实现在用户态，那么从系统的角度来看，只能感知到进程的存在，即这个进程只有一个线程，实则由自身实现的多线程运行在这个进程中，如果其中一个线程发生了阻塞（不释放CPU），因为用户空间没有**时钟中断机制**，这将会阻塞整个进程。

2. jdk1.2及之后的版本

现在的java线程的本质就是操作系统中的线程，在Linux上基于pthread库实现的`轻量级进程`，在windoes上是依靠的`win32 api`提供系统调用从而实现多线程。

这里的轻量级进程就是用户线程和内核线程之间的桥梁，应该可以说是提供了访问内核线程的高级接口。在Liunx上轻量级进程和内核线程是一一对应的也就是一对一模型。

#### java 线程的几种状态

六种状态：

1. 初始；新建一个线程对象
2. 运行；就绪和运行两种状态统称运行
3. 阻塞；表示线程阻塞与锁
4. 等待；进入该状态的线程需要等待其他线程的特殊动作 通知或者中断
5. 超时等待；不同于等待，可以在指定的时间返回
6. 终止；执行完毕

#### 线程池

管理线程。

优点：

- 降低资源消耗
- 提高响应速度
- 可管理性：统一管理所有线程资源，申请匿名的newThread不方便管理。
- 提供延迟、定时执行，一般都拥有一个任务队列。

#### 乐观锁

一种思想，在操作数据时非常乐观，认为自己在修改数据时别人不会同时修改数据，因此乐观锁不会上锁，而是在执行更新（即写入内存时）进行判断旧值是否发生了改变，如果旧值发生了改变则放弃操作。

#### 悲观锁

一种思想，在操作数据时非常悲观，认为自己在修改数据时别人会同时修改数据，因此在操作数据时把数据锁住，直到操作完成才会释放锁，上锁期间别人无法修改数据。

#### 死锁

##### 四个必要条件

1. 互斥条件
2. 占有等待
3. 不可强行占有
4. 循环等待条件

> 这四个条件是死锁的必要条件，只要系统发生死锁，这些条件必然成立，而只要上述条件之一不满足，就不会发生死锁。

##### 避免死锁的办法

尽力的去避免这四个条件，当然互斥条件是无法避免的。

#### synchronized

java关键字，可以用来修饰方法和代码块，其中修饰方法时锁定的是对象的this指针如果是静态方法则锁定此类的所有对象，代码块的话则看你传入的是什么了。

**作用**：一个线程访问一个对象中的synchronized步代码块时，其他试图访问该对象的线程将被阻塞，这是典型的悲观锁机制。

翻译成字节码时会在代码块头部和尾部分别加上monitorenter、monitorexit

```java
void synchronized function(){
    //锁定对象this
}

static void synchronized function2(){
    //锁定此类的所有对象
} 

void function3(){
    Object o = new Object();
    synchronized(o){
        //锁定传入的对象o
    }
}
```

**注意**：

1. 不能修饰构造方法
2. 不能修饰接口方法
3. 父类的方法使用了synchronized修饰则子类须显示添加synchronized才能同步，不加也可以不过不同步

#### synchronized锁升级过程

锁只能升级不能降级，但是偏向锁可以重置为无锁状态。

无锁 -> 偏向锁 -> 轻量级锁（自旋锁）-> 重量级锁

- 偏向锁：即对第一次获取到锁的线程“偏向”，当这个线程销毁后，会发生重置，重置为无锁状态。
- 轻量级锁：当偏向锁被其他线程尝试获取后，将会发生锁升级，也就是升级到轻量级锁。这个状态下没有获取到锁的线程，会发生自旋等待，这个过程是消耗CPU资源的。
- 重量级锁：当自旋等待的线程数量达到一定的量，这个时候的自旋等待的性价比已经不高了，所以会产生锁升级。没有获取到锁的线程会进入等待队列中，阻塞，因为synchromized是非公平锁，当锁被释放后，大家又是随机获取到锁或者按照一定的优先级。

#### volatile

java关键字，修饰的变量将具有可见性，却无法保证原子性，且屏蔽指令重排序。

- 可见性：多个线程访问同一个变量时，某个线程修改了变量，其他线程立即能看到修改的值
- 指令重排序：是编译器和处理器为了高效优化程序的手段，只能保证程序执行的结果时正确的，但是无法保证程序的操作顺序与代码顺序一致。在单线程中没有什么影响在多线程中就会出现问题。单例模式中添加volatile就是为了避免指令重排序。
- 原子性：**即一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。** 

#### CAS 

CAS 全名  Compare and Swap 即比较和交换（也有叫Compare and Set）

它是一种乐观机制，java中的各种原子类都是用它来实现的。包括ReentrantLock。同时它也是自旋锁和乐观锁的核心操作。

##### 实现原理

用旧的期望值和内存中的值进行比较，如果二者相同则用新值替换内存中的值也就是写入内存并返回true，反之返回false。

#### synchronized 和 CAS的比较

- 在高竞争、高耗时的情况下synchonized的效率更高
- 在低竞争、低耗时的情况下CAS的效率更高

#### 自旋锁

乐观锁，自旋也就是自旋等待，不断的尝试获取某个锁，直到持有该锁的线程释放锁，这个过程消耗的是CPU资源，所以在竞争激烈的情况下效率较低。

#### 可重入锁ReentrantLock

java中的ReentrantLock实现Lock接口，实现了Lock接口的类都是可重入的，从字面意思理解就是可以重新进入的锁，即允许同一个线程多次获取同一把锁。例如在递归方法中存在加锁操作，即单个线程多次获取同一把锁，这时如果不会被阻塞那么就是可重入锁。

#### 读锁ReadLock

又称共享锁。线程A获得某对象的共享锁之后，其他线程依然可以对其加共享锁，但是不能加排它锁。同时获得共享锁的线程只能读取数据，不能做修改。

#### 写锁WriteLock

又称排它锁。线程A对某对象添加排它锁之后，其他线程不能对其添加任何锁。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            

#### 公平锁、非公平锁

**公平锁**：多个线程竞争同一把锁，在锁释放时，先申请获得锁。

**非公平锁**：多个线程竞争同一把锁，在锁释放时，可能随机获得锁或者按照一定的优先级。

synchronized是非公平锁，且无法变成公平锁。

ReentrantLock默认也是非公平锁，但是可以通过构造传参变成公平锁。

#### 可中断锁、不可中断锁

顾名思义可中断锁就是可以响应中断的锁。

java中的“中断”：java没有提供直接中断线程的方法，而是提供了一种中断机制。线程A可以向线程B发出中断请求。

#### 锁粗化

将多次申请锁和释放锁的操作合并为一次，以降低短时间内大量请求同步锁，带来的性能损耗。

类似于这样的情况

```java
	for(value : array){
		synchronized(this){
			//do something...
		}
	}	
```

在循环中使用了同步代码块，这样会让程序反复的去加锁、释放。经过优化后变成这样

```java
	synchronized(this){
		for(value : array){
			//do something...
		}
	}
```

扩大了代码块的范围，提升了性能。

#### 锁消除

锁消除是指Java虚拟机在JIT编译时，通过对上下文的扫描，经过逃逸分析，去除不可能存在共享资源竞争的锁。通过这样的方式消除没有必要的锁，节省无意义的时间（锁的获取和释放所消耗的时间）

#### 逃逸分析

当一个变量（或者对象）在子程序中被分配时，一个指向变量的指针可能逃逸到其他执行线程中，或者返回调用者子程序（即此对象可能被外部访问到时）此对象的指针发生了逃逸。

也就是说该对象在程序中被访问到的地方无法被确定。

>  **注意**：Java中的逃逸分析发生在JIT即时编译阶段

##### JVM中判断对象是否发生逃逸

1. 对象被赋值给堆中的新对象和类的静态变量
2. 对象被传入了不确定的代码中去运行

##### 基于逃逸分析的优化

1. 堆分配转化为栈分配，如果对象在子程序中创建，且不会发生逃逸，那么将此对象分配到栈空间，可以降低GC频率。
2. 同步消除，如果发现某个对象只会一个线程访问，那么在这个对象上的操作不需要同步，进行锁消除提升速度。
3. 分离对象，如果对某个对象的访问方式不要求其内存结构连续，那么可以将此对象的部分或全部内容，不存储在内存而是放在CPU寄存器中。

#### HashMap

首先要知道它线程不安全，相对应的HashTable是线程安全的，HashTable

使用了synchronzed关键字修饰关键的方法，这样的作法实现了线程安全但是带来了很大的性能损失；ConcurrentHashMap即保证了线程安全又拥有较高的性能。

在JDK8中hashmap的实现方式是**位桶+链表+红黑树**的组合方式，这样大大减少了查找时间。

![](https://user-gold-cdn.xitu.io/2019/12/5/16ed3af3e16c6ed8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 扩容

HashMap的初始大小是16，并且注释写的是必须为2的次幂

- 为什么初始大小是16？
  1. 太小会导致频繁扩容
  2. 太大会导致资源浪费
  3. 避免hash碰撞，提高查询效率
     - 为什么能避免hash碰撞？
       1. 16 的二进制表示为1111，在计算坐标时是通过key值和容量按位与运算；2的次幂二进制表示每一位都是1，按位与将会获得一个可能为奇数、偶数的坐标，相比10（二进制1010，只能获得偶数，那么奇数位置将一致空缺）使得hash碰撞减少，分部更加均匀。

继续查看后续的resize源码发现，每次扩容，扩容后的大小是扩容前的2倍。

###### 触发扩容的条件

- Map中存储键值对的数量超过了阈值threshold
- 链表中的长度超过了`TREEIFY_THRESHOLD`，但表长度却小于`MIN_TREEIFY_CAPACITY`。

###### 链表转红黑树的条件

- 链表节点数达到8个，并且HashMap存储的键值对数量大于`MIN_TREEIFY_CAPACITY`（默认值为64），否则都只是做扩容。

#### 线程池

`ThreadPoolExecutor`  

**优点**：相比于随处new Thread(runable)去执行任务，使用线程池将会使得线程更加容 易控制，且提高效率，减少了性能损耗。

- 核心线程数量
- 最大线程数量
- 等待队列
- 活跃时间

### Android 基础（四大组件、Handler、Looper、View）

#### Activity

##### 启动过程

![](https://user-gold-cdn.xitu.io/2018/6/22/1642579940cd4d82?imageslim)

简单说就是将启动activity的消息发送到looper中，在

##### 生命周期

![Activity 生命周期](https://developer.android.com/guide/components/images/activity_lifecycle.png?hl=zh-cn)



	##### 启动Activity

###### 显示启动

```kotlin
val intent = Intent(CurrentContext.this,NextActivity::class.java)
//除此之外还可以通过以下两种方法达到目的，其实性质一样都是通过包名和类名进行跳转
intent.setClassName("当前包名+类名","nextActivity （包名+类名）")
intent.setComponent(Component("当前包名+类名","nextActivity （包名+类名）"))
//intent.putExtra(...)
startActivity(intent)
```

这种启动模式既可以内部使用也可以外部使用，只要知道想要跳转的activity的包名和类名就可以了。

###### 隐示启动

```kotlin
val intent = Intent("com.sl.action")//填写静态注册时填入的action
startActivity(intent)
```

```xml
   <activity android:name=".ActionActivity"; 
        <intent-filter
            action android:name="com.sl.action"
			<!--填写自定义action-->
        </intent-filter>
    </activity>
```



适用于内部跳转和外部跳转。常用于第三方应用的外部跳转，使用约定好的action。



##### Android启动模式

在介绍之前需要知道Android中在task中启动activity的默认规则：

**在不同的Task中启动相同的Activity，Activity会被创建多个实例，分别放进每一个Task。**

而对于不同的启动模式则各不一样。

###### standard

默认模式，可以拥有多个实例并且可以叠加，意味着同一个activity栈中可能存在多个相同的实例。

###### singleTop

可以拥有多个实例，但不允许叠加，当此activity处于栈顶时，启动相同的activity，会调用它的onNewIntent方法而不会新建一个实例。Android的桌面activity就是这种模式

###### singleTask

只有一个实例，在同一个应用程序中启动它时，如果task中没有它的实例，就会新建一个实例，如果有的话就会将task中位于该实例之上的所有activity destory，并且调用该实例的onNewIntent。

若是在不同的应用程序中启动它，如果最近任务中存在这个实例对应的task，则会将这个task放置到启动它的应用程序的task上并将singleTask实例置于栈顶（即清空位于该实例之上的所有activity并调用onNewIntent或者该task中没有该实例直接创建一个实例放于栈顶），如果最近任务中没有这个singleTask实例对应的应用程序的task则会新建一个task，并在这个task中启动这个activity。

singleTask允许其他的activity与它在同一个task中共存，即在这个singleTask的实例中打开一个新的activity，还是会在singleTask实例的task中。

###### singleInstance

只有一个实例，并且单独占用一个task，即不允许其他activity出现在这个task中。在多个Task启动这个Activity的时候大致与singleTask相同，但是相对规则更加严格，它会独占一个Task，且在这个实例中启动新的Activity时会新建一个对应的Task并在这之中启动新的Activity（如果这个Task在最近任务中不存在）并将该Task放置到当前Task之上。

**注意**：当用户切回桌面或者按下最近任务键，则叠加的Task会被分开。

#### Service&IntentService

##### 生命周期

![](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1jZjVjMWE5ZDJkZGRhYWNhLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

1. startService&stopService

   启动服务和停止服务，通过这种方法启动服务，**服务与调用者之间没有关联，即使调用者退出了服务也会保持正常运行**，当然调用者可以通过stopService结束服务，服务本身也可以通过stopSelf方法来结束本身。如果服务已经创建了且多次调用startService服务不会被多次创建而是会调用onStartCommand方法，即可以多次传递intent给服务。

   对应的生命周期：

   ![](https://upload-images.jianshu.io/upload_images/1441907-3dbf045663fb54a5.png?imageMogr2/auto-orient/strip|imageView2/2/w/189/format/webp)

2. bindService&unBindService

   绑定服务和解绑服务，**这种启动方式使得调用者和服务之间存在一定的关联，调用者退出后服务会跟着退出**。如果服务是初次创建那么会调用onBind方法，此方法会返回一个实现了IBinder接口的对象，多次调用bindService也不会多次调用onBind。

   对应的生命周期：

   ![](https://upload-images.jianshu.io/upload_images/1441907-08f50068b747b98d.png?imageMogr2/auto-orient/strip|imageView2/2/w/149/format/webp)

##### Service 

不是独立的进程也不是独立的线程，一般是依赖于应用程序的主线程的，所以不能在service里面执行耗时的操作，可能会引起ANR。

##### IntentService

它会创建一个工作线程来处理所有通过onStartCommand传递过来的intent，将intent放置到工作队列中，通过工作队列将intent一个一个发送给onHandleIntent。并且不需要主动调用stopSelf方法来结束自己，当intent处理完毕也就是工作队列为空时也就自动关闭服务了。

**优点**：省去了手动创建工作线程的工作，不需要手动关闭。

##### Binder

跨进程通信，作用是：连接Service进程、Client进程、Service Manager的桥梁

##### 通信方式

1. 通过实现Binder/IBinder接口，这个类的对象会在调用onBind方法时生成。

```kotlin
class MyBinder : Binder{
	fun onChanged(index:Int){
		//do something
	}
}

//****************
//MyService.kt
//****************

class MyService :Service{
    override onBind():IBinder{
        return MyBinder()
    }
}

//获取这个对象的方法
ServiceConnection connection = object:ServiceConnect(){
    override fun onServiceConnected(name:ComponentName,binder:IBinder){
        //此处binder即为onBind方法返回的对象
    }
}
//调用bindService,将connection传递进去
bindService(this,connection,MyService::class.java)
```

通过调用bindService时将ServiceConnection对象传递进去，在服务创建后就可以接收到onBind返回的Binder对象。

2. 通过内部广播的方式实现

##### 扩展

1. 如何提升服务的优先级
   1. 在清单文件中设置android:priority = “1000”，1000为最高优先级
   2. 在service的onStartCommand方法中调用startForeground方法，service会被提升为前台程序，需要在onDestory中调用stopForeground。
   3. Service+Broadcast，在onDestory中发送广播重新启动Service，但是如果是在设置在强制关闭应用则会失效，因为无法进入onDestory。
   4. 监听系统广播并判断当前Service是否还存活。
   5. Application加上Persistent属性

#### Broadcast

广播注册方式分两种：静态、动态，还可以自定义广播

```kotlin
class BatteryChangeReceiver : BroadcastReceiver() {
    private lateinit var listener: BatteryChangeListener

    override fun onReceive(p0: Context?, p1: Intent?) {
        val level = p1?.getIntExtra("level",0)
        //从intent读取数据
    }

    fun setListener(listener:BatteryChangeListener){
        this.listener = listener
    }
}
```



##### 静态注册

在Android8.0过后Google做了限制，应用不能对大部分的广播进行静态注册。

在清单文件中声明

```xml
<receiver android:name=".SystemReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BATTERY_LOW" />
                <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
                <action android:name="android.hardware.usb.action.USB_STATE"/>
        </intent-filter>
        </receiver>
```

##### 动态注册

在代码中动态注册，此时需要手动注销（即registerReceiver,unRegisterReceiver）。

```kotlin
/**
MainActivity.kt
**/
override fun onResume() {
	registerReceiver(broadcastReceiver, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
    //注册
}

override fun onPause() {
    unregisterReceiver(broadcastReceiver)
    //注销
}
```

##### 自定义广播

```xml
 		<!-- 注册自定义静态广播接收器 -->
        <receiver android:name=".StaticReceiver">
            <intent-filter>
                <action android:name="com.byread.static" />
            </intent-filter>

```

自定义广播的发送方法

```kotlin
val intent = Intent()
intent.action = "";//com.byread.static
intent.putExtra(...)
sendBoradcast(intent)
//这样StaticReceiver就可以接收到自定义广播了
```

#### ContentProvider

存储和获取数据的统一接口，可以在不同的应用程序间共享数据。

使用前需要在清单文件中声明

```xml
// 提供provider的app
<provider    
    android:sharedUserId="con.sl.share"//通过设置sharedUserId,使得外部应用可以访问,外部应用需要添加这个sharedUserId才能正常访问
          //可以添加相应权限
    android:authorities="com.sl.contentprovidertest"  
    android:name=".provider.TestProvider" />

// 访问该provider的app
<uses-permission android:name="com.sl.share"/> //添加权限才能访问
```

创建TestProvider继承ContentProvider虚类，实现其中的方法。

此时便可以通过ContentResolver中的方法进行数据操作，Android中获取**ContentResolver**使用ContextWrapper中的getContentResolver方法.附上Activity的UML类图

使用ContentResolver需要指定固定格式的Uri.

格式如下:uri = content://authority/path/id

例:

authority = com.sl.contentprovidertest  应用包名

path = book 数据库表名

id =  2 表中的列数我猜的

最后就是 content://com.sl.contentprovidertest/book/2

#### LifeCycle

LiveData

观察者模式，获取绑定lifecycler感知view生命周期。当数据发生改变时能够直接将新的数据发送给订阅者。

#### AIDL

Android Interface Description Language

用于跨进程通信

在方法中传递的参数需要继承于Parcelable，且需要将用到的类卸载aidl文件中。同时需要用in、out、inout三个关键字修饰；默认使用in修饰

- in： 参数由客户端发送到服务端，且服务端更改后无法传递会客户端
- out：参数由服务端修改后发回给客户端，服务端接收到的对象的参数是null，但是对此变量修改后客户端是可以接收到的
- inout：集合了in、out的功能

#### Binder

一个中间件，负责跨进程通信

#### 自定义View

##### onMeasure

位于FrameLayout、RelativeLayout布局下onMeasure会调用两次，首次会先使用UNSPECIFIED拿到子view想要获取的大小，第二次根据父布局的大小来对子view作出一些限制使得不会超过父布局。

由父级布局调用用于确认该view的大小，以分配合适的空间来装下这个view。

函数定义如下：

```kotlin
fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int)
```

`widthMeasureSpec` 和 `heightMeasureSpec` 两个参数可以通过`MeasureSpec` 获取其内容

- MeasureSpec.getSize(Int)  获取长度
- MeasureSpec.getMode(Int) 获取对应模式

即每个Int参数中存储了长度和模式，其中模式分为三种

1. AT_MOST 此模式下view的大小为任意但是不能超过给定的界限通过MeasureSpec.getSize(Int)获取界限；
2. EXACTLY 此模式下view的大小由父级布局决定，即大小为给定的界限；
3. UNDEFINED 此模式下没有任何限制，view可以为任意大小；

测量完毕后再该函数末尾调用`setMeasuredDimension` 保存测量的高度和宽度

```kotlin
setMeasuredDimension(wantWidth, wantHeight)
```

##### onLayout

用于确定子view在父级布局中的位置，left、right、top、bottom。自定义view的话时不需要去实现这个的，在viewgroup中需要去实现。

##### layout

父级布局在onLayout中给每个子view设置位置，通过layout方法设置，将会传入left、right、top、bottom四个值。而在view的layout实现中，通过setFrame将四个参数进行保存。

##### onDraw

view的绘制都发生在这里。



#### Handler

##### Handler 工作流程图

![](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0xODRlYTk0ZWMxYjVjZTA1LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

##### postDelay方法的实现

查阅源代码后发现，postDelay第一步将传递进去的Runable r通过getPostMsg方法写入到一个Message对象中，再通过delayMillis=SystemClock.uptimeMillis() + delayMillis，将这个时间设置为系统当前时间加上需要延迟的时间即最终这个时间变成了应该调用Runable的时间，然后将这个Message对象放入MessageQueen中的位置，这个位置需要考虑时间（msg.when）;最后主线程的Looper.loop方法中，循环读取MessageQueen中的msg，在MessageQueen.next()中也是循环等待，当now >= msg.when时，这个msg就会被取出来并返回，这时loop方法中将会开始执行保存在msg中的callback，也就是最开始传递进去的Runable。

#### 导致内存泄露

我认为有两种情况：

1. 在Activity中使用非静态内部类的Handler，当使用`handler.sendMessageDelayed()`,消息还没有执行，此时Activity被销毁，MQ中的msg持有此Handler对象，Handler对象持有外部类引用（即Activity），导致内存泄露。
2. 调用`Handler.postDelayed(Runable)、postDelayed(Runable)` 发送延时消息，此处Runable得分情况如果是一个位于Activity类中的匿名内部类，那么它是持有Actiovity引用的，此时若是Activity销毁，则会导致内存泄露。

#### view.postDelayed(Runable)发生了什么

实际的调用链为 `mAttachInfo.mHandler.postDelayed(runable,delayMills)` 实际还是使用handler将runable包装成message放入MQ，在Looper.loop中读取message，发现callback不为空，将会调用`callback.run()`



#### Android Touch事件的分发机制

触摸事件对应的是`MotionEvent`类，对应的Touch事件主要有三个：

- TOUCH_DOWN 手指按下
- TOUCH_MOVE 手指按下后开始滑动
- TOUCH_UP 手指离开屏幕 （标志着触摸事件的结束）
- TOUCH_CANCEL 手指滑动到屏幕外部

此外事件传递的三个阶段分别是：

- **分发**，具体体现在dispatchTouchEvent方法中，若是返回true代表事件被当前视图消耗掉，若是返回super.dispatchTouchEfvent则继续分发这个事件。
- **拦截**，具体体现在onInterceptTouchEvent方法中，返回true代表这个事件由自身的onTouchEvent消耗，反之则不拦截，继续传递给子view。如果return super.onIntercepttouchEvent() 则事件拦截分两种情况：
  1. ViewGroup中存在子view，且点击到了子view，则不进行拦截，相当于return false
  2. ViewGroup中不存在子view或者有子view但是没有点击到子view则进行拦截，return true
- **消费**，具体体现在onTouchEvent方法中，返回true表示view可以处理当前事件，返回false则表示不处理这个事件，那么这个事件会再次被传递到父View的onTouchEvent中进行处理，如果return super.onTouchEvent()，则分为两种情况：
  1. 如果该view是clickable或者longclickable则返回true，消费此事件。
  2. 不是clickable或者longclickable则返回false，事件会继续向上传递，即不消费此事件。

注意：在Android系统中，拥有事件传递处理能力的类有以下三种。

- Activity：拥有分发和消费两个方法。
- ViewGroup：拥有分发、拦截和消费三个方法。
- View：拥有分发、消费两个方法。

#### Android中的内存泄露

- 在Activity发生重建或者销毁时，某个地方持有该Activity的强引用，该引用没有得到及时的释放，此时Activity不会被系统gc回收掉。
  - 另外发生重建的情况可能就是，Android手机发生了屏幕方向切换
  - 销毁则可能就是正常退出

引用可能出现的地方：

1. 静态的Activity对象、view对象、context。静态对象贯穿应用程序的生命周期
2. 单例类中持有以上对象的强引用，可以改为弱引用（为什么不用软引用，因为不能由我们控制，弱引用的话发生gc便可以确认它会被回收掉而软引用可能继续存在）
3. 内部类和匿名类都会持有外部类的引用，改成静态内部类、静态匿名内部类就不会再持有引用。

##### Handler引起内存泄露

handler作为内部类存在于Activity中，内部类会持有外部类的引用，发送一条延迟消息，然后销毁activity，此时MessageQueue中的Msg村存在一个匿名类Runable，它持有handler的引用，而handler又持有activity的引用，导致内存泄露。

**解决办法** hanlder作为静态内部类，然后持有一个activity的弱引用。或者直接将hanlder放到单独文件中。

#### Android中的ANR

##### 说明

ANR是一个问题，即主线程无法正常处理用户的输入事件或者绘制操作，引起用户不满。

![](https://developer.android.com/topic/performance/images/anr-example-framed.png)



##### 触发情况

1. Activity位于前台时，主线程在5秒内没有响应用户的输入事件或者BroadcastReceiver（即接收到广播后长时间没有返回）
2. Activity位于后台时，BroadcastReceiver 长时间没有返回

**注意** 如果用户没有输入，就算主线程在执行延时操作也不会发生ANR，因为没有输入事件主线程也不需要去响应用户的输入。

***

##### 诊断

需要考虑的情况：

- 在主线程中存在耗时操作、I/O操作（时间过长）
- 主线程阻塞，存在一个等待同步调用的代码块
- 主线程对另一个进程同步一个binder调用，但是后者需要很长时间返回
- 主线程对进程进行binder调用时与另一个线程发生死锁，死锁状态下主线程阻塞。

总的来说就是主线程发生了阻塞或者等待时间过长导致的ANR

##### 辅助定位问题代码

1. 使用StrictMode

```java
//通过两个Builder设置需要检测的问题，会在控制台打印
StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder().detectAll().build());
StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder().detectLeakedSqlLiteObjects().build());
//通过penaltyDeath（检测到问题后Crash整个进程）等方法可以更加直观的定位错误代码
```

2. TraceView

废弃

3. AndroidStudio 性能分析工具 Profiler

可以直观的看到app运行时对各方面资源的消耗情况。具体方法耗时？

#### Android中的跨进程通信

1. Intent ，例：activity中打开拨打电话的界面，Uri.parse("tel:10086")
2. Content provider，提供一种在多个程序之间数据共享的方式
3. Broadcast ，广播属于单方向的发送广播，接收者只能接收广播，并不能直接与发送者沟通
4. Service，通过Aidl 通信
5. Socket，在本地建立连接，达到跨进程通信

> 前四种都是通过**Binder**来实现。

#### Android 中的注解@Hide

此注解标识的方法不可访问。

在build.gradle中配置compileSdkVersion 30，就会把位于$Android_SDK$/platform/android-30/下的android.jar 加入编译时类路径。

这个jar称为编译时jar，不会被打包到应用中去。这个jar包有两个特点：

1. 不包含具体的方法实现
2. 不包含被@hide标记的类或成员

在打包时将一个具有@hide标记的类或成员的android.jar链接到应用中去。

***

### Android开发中常用的设计模式

##### 常用设计模式

1. 单例模式（双重检查、IoDH）

2. 创建者模式：某个类需要用户传入大量的数据，这时如果有一个链式调用的Builder，将会简化很多工作。

3. 观察者模式：

   Java 封装了观察者设计模式的API

   其中Observable类又被观察者类实现、Observer接口由观察者实现。

   通过Observable添加Observer，当Observable发生改变时通过调用notifyObservers方法通知观察者们，观察者可以在update()回调中获取到更新内容。

4. 适配器模式：适配器作为一个中介角色，使得原本不匹配不能在一起工作的接口在一起正常工作。

5. 代理模式：通过引入代理对象的方法来间接访问目标对象，防止直接访问目标对象给系统带来的不必要复杂性。

##### 设计模式的六大原则

1. 单一职责原则

2. 里氏替换原则

   基类可以出现的地方，衍生类一定可以替换他。衍生类应该尽量避免去重写基类的方法，应该在基类的基础上开发新功能即可。

3. 依赖倒置原则

   依赖于抽象不依赖于具体，不直接使用具体类，而是于具体类的上层接口交互。

4. 接口隔离原则

   每个接口中不存在子类用不到却必须实现的方法，如果存在就需要将接口拆分，使用多个隔离的接口。

5. 最少知道原则

   一个类应该尽量少的依赖其他类，以降低耦合使得功能更加独立，当其依赖的类发生变动时，对他的影响才会更小。

6. 合成复用原则

### 项目中可能会问到的问题

#### Qingwen（kotlin）

1. 遇到了什么困难？

   在写阅读页面的时候，需要测量当前页面能够显示文字的数量，通过简单的计算行高就可以得到一个大概数据，但是Android的TextView的文字绘制，在不同设备上的表现不一样，测量结果总是会有差错，但是在虚拟机上总是正常。最后通过TextView 中用到的工具Layout，实现了一个具备基本功能的TextView（更换字体、字号、颜色），实现了准确测量。

2. 技术亮点是什么？

   1. 使用了MVVM设计模式：配合databinding做到view和model同步变化，只要数据正确view就正确，很方便。
   2. 利用Google的嵌套滑动组件实现了一个横向和纵向 下拉刷新上拉加载的组件继承LinearLayout，在处理嵌套滑动前判断是否满足父级view滑动的条件，满足则在onStartNestedScroll，然后再后续的onNestedPreScroll中处理滑动事件。
   
3. 登录怎么做的

   采用明文传输，加密方法 将时间戳和一些相关的字符组合在密码中，让别人不能轻易破解。

#### 加密算法 

1. 对称式加密

   生成同一个密钥，可以对数据进行加密解密。

2. 非对称式加密

   典型代表有RSA ，生成两个密钥，一个公钥为公开的所有人都可以拿到用于加密，一个私钥不公开用于解密。

现在的Https请求一般通过两种方式混合使用。

### 进程

操作系统中资源分配和调度的基本单位

### 线程

CPU调度和分派的基本单位

### Kotlin 相关

#### 协程（属于无栈协程）

Kotlin协程本质是一个线程框架，属于无栈协程。

#### 启动模式

- DEFAULT 默认模式，协程创建后立即开始调度，如果协程调度前被取消会直接进入取消响应。
- ATOMIC 协程创建后，立即开始调度，直到第一个挂起点之前不响应取消。
- LAZY 只有协程被需要时，才会开始调度。调度前被取消直接进入异常结束状态。
- UNDISPATCHED 协程创建后立即在当前函数调用栈内执行，直到遇到第一个真正的挂起点。

#### 协程的调度器

官方给出了四个调度器，可以通过Dispatchers访问。

- Default ：默认调度器，适合后台任务处理，属于CPU密集型任务调度器

- IO ： IO调度器，IO密集型任务调度器

- Main ： UI调度器，根据平台不同会被初始化为对应的UI线程调度器

- Unconfined ： “无所谓” 调度器，不要求协程在特定线程执行，**在挂起点恢复执行时会直接在恢复线程直接执行**

  *如果嵌套以它为调度器的协程，那么这些协程会在启动时被调度到协程框架内部的事件循环上，以免出现**StackOverflow***

  

##### 挂起函数（suspend）

本质是更加灵活的切线程。`withContext(Dispatchs.IO){}`

在函数执行完毕后切换回原来的线程。通过阻塞式的写法来实现非阻塞式的操作。

##### 协程状态

1. 创建，初始化状态，可有可无
2. 未完成，代码块处于运行中或者未运行
3. 已取消，代码块执行中被取消，可取消后回调invokenOnCancel
4. 已完成，代码块顺利执行完毕，可在完成后回调invokeOnCompletion

##### 热数据通道 Channel（并发安全）

`Channel<?>`  是一个接口。当没有订阅者的时候也会在通道内发送数据。

缓存容量存在四种模式。

##### （通道内缓存区容量大小 ）四种模式 

1.  `UNLIMITED` 不做限制，缓冲区无限大，**但是不能用于广播通道**
2. `RENDEZVOUS` 容量为0，意为不见不散，没有出现receive时send方法始终处于挂起状态，**同样不能用于广播通道**
3. `CONFLATED` 意为合并，但实际是后者覆盖前者，接收到的内容为最后的内容
4. `BUFFERED` 默认容量为64，此大小可以修改

##### 常用通道

模型为一对一。一个发送端一个接收端。

1. SendChannel

   - 创建：需要在内部通过receiver获取发送来的数据

   ```kotlin
   val sendChannel:SendChannel<Int> = GlobalScope.actor{
       while (true){
           println("receiver data <${receive()}>")//接收数据
       }
   }
   ```

   - 发送 send（）

   ```kotlin
   sendChannel.send(it)
   ```

   - 接收 receive()

   ```kotlin
   ActorScope.receive()
   ```

2. ReceiverChannel

   - 创建：

   ```kotlin
   val receiverChannel:ReceiveChannel<Int> = GlobalScope.produce {
       send(0)//发送数据
   }
   ```

   两者都是一样的。

3. 关闭

   1. producer和actor返回的Channel都会随着对应协程执行完毕而关闭。

   2. 调用close方法也会立即关闭。

   3. 不及时的关闭通道，虽然不会导致系统资源的泄露，但是会让接收端（`receive()`）一直处于等待状态.

> 注意：**Channel** 的数据发送时调用`sned(Element)`方法，数据发送出去直到被接收才能进行下一步发送，等待的过程send方法处于挂起状态；当然receive()方法也是一直处于挂起等待状态。

##### BroadcastChannel

一对多，一个发送端多个接收端。

> 发送的数据如果没有接收者会**直接丢弃**

```kotlin
    val broadcastChannel = BroadcastChannel<Int>(Channel.CONFLATED)//创建通道
	/*
	* 创建方法2
	* val channel = Channel<Int>()
	* channel.broadcast(value) //value 控制缓存区容量
	*/
    GlobalScope.launch {
        List(3) {
            delay(100)
            broadcastChannel.send(it)
        }
        delay(1000)
        broadcastChannel.close()
    }

    val r = broadcastChannel.openSubscription()
    List(3) {
        GlobalScope.launch {
            val r = broadcastChannel.openSubscription()
            for (i in r) {
                delay(50)
                println("$it broadcast receive $i")
            }
        }
    }.joinAll()//创建三个job去接收通道数据
    GlobalScope.launch {
        delay(100)
        println("finally receive ${r.receiveOrClosed()}")//由于使用的CONFLATED，最后接收到的是2
    }.join()

//结果如下
1 broadcast receive 0
0 broadcast receive 0
2 broadcast receive 0
0 broadcast receive 1
2 broadcast receive 1
1 broadcast receive 1
2 broadcast receive 2
0 broadcast receive 2
1 broadcast receive 2
finally receive Value(2)
```



##### 冷数据流 Flow

与热通道的特性相反，没有订阅者时不会发送数据。StateFlow是热数据流即没有订阅依然会发送数据。

可以从通道中获取Flow。

#### Rxjava & RxAndroid

响应式编程，现在已经可以通过Kotlin 协程来往完全替代

#### 协程与线程的区别

1. 一个线程可以拥有多个协程，一个进程也可以拥有多个协程
2. 线程都是同步机制，而协程是异步机制
3. 线程是分割的CPU资源，而协程是抽象于线程之上，协程是组织好的代码流程且需要线程来承载运行。

### 杂项

#### Room 框架

典型的ORM型数据库

1. 升级与降级
   - 升级：通过Migration来实现
   - 降级：
2. 实现原理

#### Retrofit

1. 用到的设计模式

   创建者模式、工厂模式、动态代理

#### Glide

**优点**：

1. 占用内存小（默认使用RGB_565）
2. 无代码侵入（不需要使用自定义View）
3. 支持Gif
4. 缓存优化：
   1. 支持内存分级缓存
   2. 为不同的ImageView尺寸缓存不同的图片
5. 与Activity生命周期绑定，不会导致内存泄露（传入Activity对象，创建一个无UI的Fragment，绑定生命周期）

##### 缓存

1. 内存缓存，正在使用的图片采用弱引用缓存，使用完成后使用Lru缓存（软引用，采用LinkedListMap实现，依据使用次数排序，当内存不足时，删除使用得最少得图片引用）
2. 磁盘缓存，默认缓存使用过的分辨率图片，使用DiskLruCache来做磁盘缓存

#### MVVM 和 MVP的优劣对比

##### MVP

优点：

1. 避免了MVC中Controller的庞大体积，View、Presenter、Model类的体积都不会太大。
2. UI和业务逻辑分离，可以单独进行开发。

缺点：

1. 对Activity的生命周期没有感知，需要手动添加相应的生命周期方法，去及时的释放资源。

##### MVVM

优点：

1. LiveData：更新UI不需要考虑生命周期问题
2. ViewModel：因为设备操作发生Activity重建时，无须从Model中再次加载数据，减少了IO操作
3. DataBinding：减少了模板代码的编写，通过view和数据的双向绑定同步更新，减少了代码量。

#### android 加载超大图片

使用BitmapRegionDecoder去显示你需要的那部分内容。

##### BitmapRegionDecoder 无损加载（当然可以通过设置Bitmap.Config.RGB_565等等）

提供一个从inputStream读取图片的接口，显示你需要的图片部分。

##### BitmapFactory 加载缩略图

对图片宽高进行压缩。通过设置BitmapOptions.inJustDecoderBounds = false；使得图片不被加载到内存中去并且我们可以拿到图片信息。

#### 图片浏览App

1. 加载缩略图做一个列表展览
   1. 通过向服务器发送规定大小质量的要求拿到缩略
   2. 加载完整图片，自己手动压缩
2. 查看原图（大图）
   1. 采用上面所说的方法 加载超大图片
3. 提升图片的下载速度
   1. 给服务器传递图片大小质量，拿到我想要的图片规格
   2. 多线程下载

#### 发起网络请求的整个过程

1. 在本地缓存中检索url对应的服务器IP
2. 本地缓存找不到，发送UDP数据包向本地域名服务器请求域名解析，本地域名解析服务器找不到时会向上级发送请求，直到根服务器如果还是没有找到那么就返回错误信息，找到就返回给客户端服务器的IP
3. 通过此IP建立TCP连接，发送请求头等等数据
4. 服务器处理数据，给予反馈
5. 客户端拿到数据，关闭连接。

#### Get 和 Post的区别

Http 请求的底层全部都是TCP/IP协议 ，到Http3.0 开始支持UDP

可以肉眼看到的区别：

1. Get请求将参数写在链接里面，Post时将参数放在body里面不会再链接中体现出来。

请求过程的区别：

1. Get 过程中，建立TCP连接，发送一次数据包（Http Header，Params），服务器返回200，结束。
2. Post过程中，建立TCP连接，发送一次数据包（http header）,服务器返回100，再次发送数据包（Params），服务器返回200，结束。

**注意**：有的浏览器post请求也只发一次数据（firefox）。

### 版本管理工具Git

1. merge 和 rebase有什么区别？

   merge通过新建一个节点把两个分支的内容合并成一个分支；rebase把其他分支的提交直接并到合并分支上，看起来就是一条直线。

### 经典蓝牙和低功耗蓝牙的区别

1. 经典蓝牙的连接周期大概是几百毫秒，低功耗蓝牙的连接周期仅有3毫秒左右。
2. 低功耗蓝牙使用非常短的数据包，用于实时性较高且数据速率较低的场景，经典蓝牙的数据包长度较长，用于传输数据量大的数据。
3. 低功耗蓝牙的发送和接收会以最快的速度完成，且空闲期间只会接收不会继续发送射线，等待下一次连接，这样的做法减少了功耗，而经典蓝牙采用长连接。



### Http和Https的区别

Https 有一个ssl加密技术



### 分支预测&指令流水线

#### 分支预测

如果分支走向情况为大概率偏向一方，可以采用if做判断，这样能触发分支预测，或者使用if+switch；

只有当分支足够多时switch的效率才会高于if。switch在字节码中生成一tableswitch，根据索引跳转。

分为三类：

1. 静态预测

   编译阶段就已经确定分支预测方向

2. 动态预测

   运行阶段记录上次分支走向，或者说多次记录后，对下次分支走向作出预测，预执行该分支。

3. 随机预测

   顾名思义，就是随机预执行分支。

#### 指令流水线

一条指令的执行一般分为三个阶段：取指、译码、执行

常见的六级流水线：

1. 获取指令（FI）
2. 指令解码（DI）
3. 计算操作数地址（CO）
4. 取操作数（FO）
5. 执行指令（EI）
6. 写操作数（WO）

即支持同时对六条指令进行加工，提高效率。

![](https://pic4.zhimg.com/80/v2-dae81ab10a52f8226d78fbbc3daedef3_720w.jpg)

### 面试算法题

1. 判断链表是否有环？

   采用快慢指针的思想，指针1每次走一步，指针2每次走两步，两个指针相遇就说明链表有环。时间复杂度为：O（1）

2. topk

   - 采用快速排序，找到最大的k位数
   - 采用小顶推（PriorityQueue），使queue的大小为K，遇到大于顶堆的值就推出顶推，添加此值。

3. 判断链表元素是否为回文

   - 使用快慢指针将链表分为两部分，将后半部分链表逆转，逐一与前段比较，出现差异就说明不是回文。

   