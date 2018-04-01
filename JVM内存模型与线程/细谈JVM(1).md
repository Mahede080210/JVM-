# 谈JVM可见性，原子性，有序性
> 经过两天的筛选，对于JVM内存模型刚入门的学者来说，这篇文章足以带你入门，不要担心看不懂，好了，我们看下面吧。
> 
    ----------因为热爱，所以拼搏。。2018-4-1（来自大二的一名艺术家）           


#### 前导必备
1.JVM内存模块

2.多线程操作方面的知识

3.你的大脑

#### 基本概念
1.可见性（volatile关键字）

可见性是一种复杂的属性，因为可见性中的错误总是会违背我们的直觉。通常，我们无法确保执行读操作的线程能适时地看到其
他线程写入的值，有时甚至是根本不可能的事情。为了确保多个线程之间对内存写入操作的可见性，必须使用同步机制。

可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的。也就是一个线程修改的结果。另一个线程马上就
能看到。比如：用volatile修饰的变量，就会具有可见性。volatile修饰的变量不允许线程内部缓存和重排序，即直接修改内存。
所以对其他线程是可见的。但是这里需要注意一个问题，volatile只能让被他修饰内容具有可见性，但不能保证它具有原子性。
比如volatileinta=0；之后有一个操作a++；这个变量a具有可见性，但是a++依然是一个非原子操作，也就是这个操作同样存在线程安全问题。

###### 在Java中volatile、synchronized和final实现可见性。

> 个人理解>>可见性就是使用volatile关键字修饰的变量被写入JVM主存中，不允许其被改变，不管是它的值还是它的位置。这样，就可以
实现对任何线程的可见性。

2.原子性（synchronized关键字或者Lock(),unLock()方法）

原子是世界上的最小单位，具有不可分割性。比如a=0；（a非long和double类型）这个操作是不可分割的，那么我们说这个操作时原子操作。
再比如：a++；这个操作实际是a=a+1；是可分割的，所以他不是一个原子操作。非原子操作都会存在线程安全问题，需要我们使用同步技术（
sychronized）来让它变成一个原子操作。一个操作是原子操作，那么我们称它具有原子性。java的concurrent包下提供了一些原子类，我们
可以通过阅读API来了解这些原子类的用法。比如：AtomicInteger、AtomicLong、AtomicReference等。

##### 在Java中synchronized和在lock、unlock中操作保证原子性。

3.有序性（volatile和synchroizede组合）

Java语言提供了volatile和synchronized两个关键字来保证线程之间操作的有序性，volatile是因为其本身包含“禁止指令重排序”的语义，
synchronized是由“一个变量在同一个时刻只允许一条线程对其进行lock操作”这条规则获得的，此规则决定了持有同一个对象锁的两个同步
块只能串行执行。

下面一段代码在多线程环境下，将存在问题。
```java
+ View code
/**
 * @author zhengbinMac
 */
public class NoVisibility {
 private static boolean ready;
 private static int number;
 private static class ReaderThread extends Thread {
  @Override
  public void run() {
   while(!ready) {
    Thread.yield();
   }
   System.out.println(number);
  }
 }
 public static void main(String[] args) {
  new ReaderThread().start();
  number = 42;
  ready = true;
 }
}

```

NoVisibility可能会持续循环下去，因为读线程可能永远都看不到ready的值。甚至NoVisibility可能会输出0，因为读线程可能
看到了写入ready的值，但却没有看到之后写入number的值，这种现象被称为“重排序”。只要在某个线程中无法检测到重排序
情况（即使在其他线程中可以明显地看到该线程中的重排序），那么就无法确保线程中的操作将按照程序中指定的顺序来执行。
当主线程首先写入number，然后在没有同步的情况下写入ready，那么读线程看到的顺序可能与写入的顺序完全相反。

　　在没有同步的情况下，编译器、处理器以及运行时等都可能对操作的执行顺序进行一些意想不到的调整。在缺乏足够同步的
多线程程序中，要想对内存操作的执行春旭进行判断，无法得到正确的结论。

　　这个看上去像是一个失败的设计，但却能使JVM充分地利用现代多核处理器的强大性能。例如，在缺少同步的情况下，Java内
存模型允许编译器对操作顺序进行重排序，并将数值缓存在寄存器中。此外，它还允许CPU对操作顺序进行重排序，并将数值缓存
在处理器特定的缓存中。

#### volatile原理

Java语言提供了一种稍弱的同步机制，即volatile变量，用来确保将变量的更新操作通知到其他线程。当把变量声明为volatile类
型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile变量不会
被缓存在寄存器或者对其他处理器不可见的地方，因此在读取volatile类型的变量时总会返回最新写入的值。

　　在访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此volatile变量是一种比sychronized关键字更
轻量级的同步机制。

![](http://files.jb51.net/file_images/article/201711/2017111593111246.png?2017101593126)

当对非 volatile 变量进行读写的时候，每个线程先从内存拷贝变量到CPU缓存中。如果计算机有多个CPU，每个线程可能在不同的CPU上被处理，
这意味着每个线程可以拷贝到不同的 CPU cache 中。

　　而声明变量是 volatile 的，JVM 保证了每次读变量都从内存中读，跳过 CPU cache 这一步。

当一个变量定义为 volatile 之后，将具备两种特性：

1.保证此变量对所有的线程的可见性，这里的“可见性”，如本文开头所述，当一个线程修改了这个变量的值，volatile 保证了新值能立即同
步到主内存，以及每次使用前立即从主内存刷新。但普通变量做不到这点，普通变量的值在线程间传递均需要通过主内存（详见：Java内存模型）
来完成。

2.禁止指令重排序优化。有volatile修饰的变量，赋值后多执行了一个“load addl $0x0, (%esp)”操作，这个操作相当于一个内存屏障
（指令重排序时不能把后面的指令重排序到内存屏障之前的位置），只有一个CPU访问内存时，并不需要内存屏障；（什么是指令重排序：
是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应电路单元处理）。

volatile 性能：

　　volatile 的读性能消耗与普通变量几乎相同，但是写操作稍慢，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行。

##### volatile关键字代码示例

volatile关键字的两层语义

一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

2）禁止进行指令重排序。

先看一段代码，假如线程1先执行，线程2后执行：

```java
//线程1
boolean stop = false;
while(!stop){
 doSomething();
}
  
//线程2
stop = true;
```
这段代码是很典型的一段代码，很多人在中断线程时可能都会采用这种标记办法。但是事实上，这段代码会完全运行正确么？即一定会将线
程中断么？不一定，也许在大多数时候，这个代码能够把线程中断，但是也有可能会导致无法中断线程（虽然这个可能性很小，但是只要一
旦发生这种情况就会造成死循环了）。

下面解释一下这段代码为何有可能导致无法中断线程。在前面已经解释过，每个线程在运行过程中都有自己的工作内存，那么线程1在运行的
时候，会将stop变量的值拷贝一份放在自己的工作内存当中。

那么当线程2更改了stop变量的值之后，但是还没来得及写入主存当中，线程2转去做其他事情了，那么线程1由于不知道线程2对stop变量的更
改，因此还会一直循环下去。

但是用volatile修饰之后就变得不一样了：

第一：使用volatile关键字会强制将修改的值立即写入主存；

第二：使用volatile关键字的话，当线程2进行修改时，会导致线程1的工作内存中缓存变量stop的缓存行无效（反映到硬件层的话，就是CPU的
L1或者L2缓存中对应的缓存行无效）；

第三：由于线程1的工作内存中缓存变量stop的缓存行无效，所以线程1再次读取变量stop的值时会去主存读取。

那么在线程2修改stop值时（当然这里包括2个操作，修改线程2工作内存中的值，然后将修改后的值写入内存），会使得线程1的工作内存中缓存
变量stop的缓存行无效，然后线程1读取时，发现自己的缓存行无效，它会等待缓存行对应的主存地址被更新之后，然后去对应的主存读取最新的值。

那么线程1读取到的就是最新的正确的值。

2.volatile保证原子性吗？

从上面知道volatile关键字保证了操作的可见性，但是volatile能保证对变量的操作是原子性吗？
下面看一个例子：

```java 
public class Test {
 public volatile int inc = 0;
  
 public void increase() {
  inc++;
 }
  
 public static void main(String[] args) {
  final Test test = new Test();
  for(int i=0;i<10;i++){
   new Thread(){
    public void run() {
     for(int j=0;j<1000;j++)
      test.increase();
    };
   }.start();
  }
  
  while(Thread.activeCount()>1) //保证前面的线程都执行完
   Thread.yield();
  System.out.println(test.inc);
 }
}
```
大家想一下这段程序的输出结果是多少？也许有些朋友认为是10000。但是事实上运行它会发现每次运行结果都不一致，都是一个小于10000的数字。

可能有的朋友就会有疑问，不对啊，上面是对变量inc进行自增操作，由于volatile保证了可见性，那么在每个线程中对inc自增完之后，在其他线程
中都能看到修改后的值啊，所以有10个线程分别进行了1000次操作，那么最终inc的值应该是1000*10=10000。

这里面就有一个误区了，volatile关键字能保证可见性没有错，但是上面的程序错在没能保证原子性。可见性只能保证每次读取的是最新的值，但是
volatile没办法保证对变量的操作的原子性。

在前面已经提到过，自增操作是不具备原子性的，它包括读取变量的原始值、进行加1操作、写入工作内存。那么就是说自增操作的三个子操作可能
会分割开执行，就有可能导致下面这种情况出现：

假如某个时刻变量inc的值为10，

线程1对变量进行自增操作，线程1先读取了变量inc的原始值，然后线程1被阻塞了；

然后线程2对变量进行自增操作，线程2也去读取变量inc的原始值，由于线程1只是对变量inc进行读取操作，而没有对变量进行修改操作，所以不会
导致线程2的工作内存中缓存变量inc的缓存行无效，所以线程2会直接去主存读取inc的值，发现inc的值时10，然后进行加1操作，并把11写入工作
内存，最后写入主存。

然后线程1接着进行加1操作，由于已经读取了inc的值，注意此时在线程1的工作内存中inc的值仍然为10，所以线程1对inc进行加1操作后inc的值为
11，然后将11写入工作内存，最后写入主存。

那么两个线程分别进行了一次自增操作后，inc只增加了1。

解释到这里，可能有朋友会有疑问，不对啊，前面不是保证一个变量在修改volatile变量时，会让缓存行无效吗？然后其他线程去读就会读到新的
值，对，这个没错。这个就是上面的happens-before规则中的volatile变量规则，但是要注意，线程1对变量进行读取操作之后，被阻塞了的话，
并没有对inc值进行修改。然后虽然volatile能保证线程2对变量inc的值读取是从内存中读取的，但是线程1没有进行修改，所以线程2根本就不会
看到修改的值。

根源就在这里，自增操作不是原子性操作，而且volatile也无法保证对变量的任何操作都是原子性的。

把上面的代码改成以下任何一种都可以达到效果：

采用synchronized：

```java 
public class Test {
 public int inc = 0;
  
 public synchronized void increase() {
  inc++;
 }
  
 public static void main(String[] args) {
  final Test test = new Test();
  for(int i=0;i<10;i++){
   new Thread(){
    public void run() {
     for(int j=0;j<1000;j++)
      test.increase();
    };
   }.start();
  }
  
  while(Thread.activeCount()>1) //保证前面的线程都执行完
   Thread.yield();
  System.out.println(test.inc);
 }
}
```

采用Lock
```java
public class Test {
 public int inc = 0;
 Lock lock = new ReentrantLock();
  
 public void increase() {
  lock.lock();
  try {
   inc++;
  } finally{
   lock.unlock();
  }
 }
  
 public static void main(String[] args) {
  final Test test = new Test();
  for(int i=0;i<10;i++){
   new Thread(){
    public void run() {
     for(int j=0;j<1000;j++)
      test.increase();
    };
   }.start();
  }
  
  while(Thread.activeCount()>1) //保证前面的线程都执行完
   Thread.yield();
  System.out.println(test.inc);
 }
}
```

采用Atomic子类
```java
public class Test {
 public AtomicInteger inc = new AtomicInteger();
  
 public void increase() {
  inc.getAndIncrement();
 }
  
 public static void main(String[] args) {
  final Test test = new Test();
  for(int i=0;i<10;i++){
   new Thread(){
    public void run() {
     for(int j=0;j<1000;j++)
      test.increase();
    };
   }.start();
  }
  
  while(Thread.activeCount()>1) //保证前面的线程都执行完
   Thread.yield();
  System.out.println(test.inc);
 }
}
```
#### 总结

以上就是本文关于Java中Volatile关键字详解及代码示例的全部内容，希望对大家有所帮助。

如有不足之处，欢迎留言指出。
