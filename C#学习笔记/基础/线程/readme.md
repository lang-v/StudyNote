# C#基础之线程

## 目录
- [无参线程]()
- [有一个参数的线程]()
- [多个参数的线程]()


***

### 无参线程

```
Thread t = new Thread(FunctionName);
Thread t = new Thread(new ThreadStart(FunctionName));
```

### 有一个参数的线程

```
Thread t = new Thread(new ParameterizedThreadStart(FunctionName));
//重点传入一个object
Object o = new Object();
t.Start(o);
//当然方法的定义只有一个参数（Object o）

```
PS:传递的参数经过强转之后，对象的引用地址会发生改变


### 多个参数的线程
C#本身是最多只支持一个参数的线程，
所以如果有多个参数的需求需要我们自己换一种方式实现。







