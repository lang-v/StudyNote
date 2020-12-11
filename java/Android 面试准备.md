# Android 面试准备

### 计算机基础（通信UDP\TCP\IP 三次握手、四次挥手）

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

##### 拥塞控制

##### 重传

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

4. 左移 << 、右移>>

   1001 0010 >> 2 = 0010 0100 空位由0填充

   1001 0010 << 2 = 0100 1000 空位由0填充

##### ColorInt

表示颜色有(每一个字段对应2个比特，即表示一个完整的颜色需要8位二进制) aRGB （透明度（默认为FF）、红、绿、蓝）

```java
@ColorInt
int color = 0x331122;
int alpha = color>>6 & 0xFF; 	//获取透明度 FF
int red = color>>4 & 0xFF; 		//获取红色 33
int green = color>>2 & 0xFF;	//获取绿色 11
int blue = color & 0xFF;		//获取蓝色 22
```

##### 总结

通过灵活使用位运算可以在一个小小的int身上保存多个状态，甚至可以通过异或运算做一些更加神奇的事情。

### java基础（单例synchronized、volatile、锁）

#### 乐观锁

一种思想，在操作数据时非常乐观，认为自己在修改数据时别人不会同时修改数据，因此乐观锁不会上锁，而是在执行更新（即写入内存时）进行判断旧值是否发生了改变，如果旧值发生了改变则放弃操作。

#### 悲观锁

一种思想，在操作数据时非常悲观，认为自己在修改数据时别人会同时修改数据，因此在操作数据时把数据锁住，直到操作完成才会释放锁，上锁期间别人无法修改数据。

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

无锁 -> 偏向锁 -> 轻量级锁（自旋锁）-> 重量级锁





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



#### 写锁WriteLock



#### 公平锁、非公平锁

**公平锁**：多个线程竞争同一把锁，在锁释放时，先申请获得锁。

**非公平锁**：多个线程竞争同一把锁，在锁释放时，可能随机获得锁或者按照一定的优先级。

synchronized是非公平锁，且无法变成公平锁。

ReentrantLock默认也是非公平锁，但是可以通过构造传参变成公平锁。

#### 可中断锁、不可中断锁

顾名思义可中断锁就是可以响应中断的锁。

java中的“中断”：java没有提供直接中断线程的方法，而是提供了一种中断机制。线程A可以向线程B发出中断请求。

#### HashMap

首先要知道它线程不安全，相对应的HashTable是线程安全的，HashTable

使用了synchronzed关键字修饰关键的方法，这样的作法实现了线程安全但是带来了很大的性能损失；ConcurrentHashMap即保证了线程安全又拥有较高的性能。

在JDK8中hashmap的实现方式是**位桶+链表+红黑树**的组合方式，这样大大减少了查找时间。

![](https://user-gold-cdn.xitu.io/2019/12/5/16ed3af3e16c6ed8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 扩容

HashMap的初始大小是16，并且注释写的是必须为2的次幂

继续查看后续的resize源码发现，每次扩容，扩容后的大小是扩容前的2倍。

###### 触发扩容的条件

- 哈希表为空或者长度为0
- Map中存储键值对的数量超过了阈值threshold
- 链表中的长度超过了`TREEIFY_THRESHOLD`，但表长度却小于`MIN_TREEIFY_CAPACITY`。

###### 链表转红黑树的条件

- 链表节点数达到8个，并且HashMap存储的键值对数量大于64，否则都只是做扩容。

### Android 基础（四大组件、Handler、Looper、View）

#### Activity

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

1. startService&stopService

   启动服务和停止服务，通过这种方法启动服务，服务与调用者之间没有关联，即使调用者退出了服务也会保持正常运行，当然调用者可以通过stopService结束服务，服务本身也可以通过stopSelf方法来结束本身。如果服务已经创建了且多次调用startService服务不会被多次创建而是会调用opnStartCommand方法，即可以多次传递intent给服务。

   对应的生命周期：

   ![](https://upload-images.jianshu.io/upload_images/1441907-3dbf045663fb54a5.png?imageMogr2/auto-orient/strip|imageView2/2/w/189/format/webp)

2. bindService&unBindService

   绑定服务和解绑服务，这种启动方式使得调用者和服务之间存在一定的关联，调用者退出后服务会跟着退出。如果服务是初次创建那么会调用onBind方法，此方法会返回一个实现了IBinder接口的对象，多次调用bindService也不会多次调用onBind。

   对应的生命周期：

   ![](https://upload-images.jianshu.io/upload_images/1441907-08f50068b747b98d.png?imageMogr2/auto-orient/strip|imageView2/2/w/149/format/webp)

##### Service 

不是独立的进程也不是独立的线程，一般是依赖于应用程序的主线程的，所以不能在service里面执行耗时的操作，可能会引起ANR。

##### IntentService

它会创建一个工作线程来处理所有通过onStartCommand传递过来的intent，将intent放置到工作队列中，通过工作队列将intent一个一个发送给onHandleIntent。并且不需要主动调用stopSelf方法来结束自己，当intent处理完毕也就是工作队列为空时也就自动关闭服务了。

**优点**：省去了手动创建工作线程的工作，不需要手动关闭。

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



#### Handler

##### Handler 工作流程图

![](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0xODRlYTk0ZWMxYjVjZTA1LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

##### postDelay方法的实现

查阅源代码后发现，postDelay第一步将传递进去的Runable r通过getPostMsg方法写入到一个Message对象中，再通过delayMillis=SystemClock.uptimeMillis() + delayMillis，将这个时间设置为系统当前时间加上需要延迟的时间即最终这个时间变成了应该调用Runable的时间，然后将这个Message对象放入MessageQueen中的位置，这个位置需要考虑时间（msg.when）;最后主线程的Looper.loop方法中，循环读取MessageQueen中的msg，在MessageQueen.next()中也是循环等待，当now >= msg.when时，这个msg就会被取出来并返回，这时loop方法中将会开始执行保存在msg中的callback，也就是最开始传递进去的Runable。



#### Android Touch事件的分发机制

触摸事件对应的是`MotionEvent`类，对应的Touch事件主要有三个：

- TOUCH_DOWN 手指按下
- TOUCH_MOVE 手指按下后开始滑动
- TOUCH_UP 手指离开屏幕 （标志着触摸事件的结束）

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



***

### Android开发中常用的设计模式

1. 单例模式（双重检查、IoDH）

2. 创建者模式：某个类需要用户传入大量的数据，这时如果有一个链式调用的Builder，将会简化很多工作。

3. 观察者模式：

   Java 封装了观察者设计模式的API

   其中Observable类又被观察者类实现、Observer接口由观察者实现。

   通过Observable添加Observer，当Observable发生改变时通过调用notifyObservers方法通知观察者们，观察者可以在update()回调中获取到更新内容。

4. 适配器模式：适配器作为一个中介角色，使得原本不匹配不能在一起工作的接口在一起正常工作。

5. 代理模式：通过引入代理对象的方法来简介访问目标对象，防止直接访问目标对象给系统带来的不必要复杂性。

   

### 项目中可能会问到的问题

#### Qingwen（kotlin）

1. 遇到了什么困难？

   在写阅读页面的时候，需要测量当前页面能够显示文字的数量，通过简单的计算行高就可以得到一个大概数据，但是Android的TextView的文字绘制，在不同设备上的表现不一样，测量结果总是会有差错，但是在虚拟机上总是正常。最后通过TextView 中用到的工具Layout，实现了一个具备基本功能的TextView（更换字体、字号、颜色），实现了准确测量。

2. 技术亮点是什么？

   1. 使用了MVVM设计模式：配合databinding做到view和model同步变化，只要数据正确view就正确，很方便。
   2. 利用Google的嵌套滑动组件实现了一个横向和纵向 下拉刷新上拉加载的组件继承LinearLayout，在处理嵌套滑动前判断是否满足父级view滑动的条件，满足则在onStartNestedScroll，然后再后续的onNestedPreScroll中处理滑动事件。



