# JUC

## 介绍

JUC就是java.util.concurrent工具包的简称，这是一个处理线程和并发的工具包

![concurrent包实现整体示意图.png](https://user-gold-cdn.xitu.io/2018/5/3/163260cff7cb847c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 多线程三种方式

### 继承Thread类覆写内部run方法

完成覆写之后可以把该类对象直接使用对象.start();的方式启动线程

### 实现Runnable接口，并实现内部的run方法

实现完run方法之后该对象需要挂载到Thread对象中去才能和那个Thread对象一同运行，比如Thread thread=new Thread(实现Runnable类对象),然后通过thread.start();即可启动成功

### 实现Callable接口

```
public class TestCallable {
    public static void main(String[] args){
        CallableDemo callableDemo = new CallableDemo();
        //执行callable方式，需要FutureTask实现类的支持，用来接收运算结果
        FutureTask<Integer> result = new FutureTask<>(callableDemo);
        new Thread(result).start();
        //接收线程运算结果
        try {
            Integer sum = result.get();//当上面的线程执行完后，才会打印结果。跟闭锁一样。所有futureTask也可以用于闭锁
            System.out.println(sum);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class CallableDemo implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
       int sum = 0;
       for (int i = 0;i<=100;i++){
           sum += i;
       }
       return sum;
    }
}
```

实现Callable<T>内部的call方法，call方法是带有返回值的，实现call接口的后需要使用FutureTask<T> result=new FutureTask<>(实现Callable的对象)，再将FutureTask对象交给Thread启动，**用FutureTask对象.get()方法可以获得Callable对象call方法的返回值，需要等待上面的对应的futuretask创建出的线程执行完成后才会执行get方法**,futuretask本身继承了RunnableFuture它可以直接放到Thread中去，**但是它同一个futuretask对象创建的Thread线程它只会执行一次线程,以后无论用它创建多少个线程并启动都只会执行第一次的线程**



## 主线程

当JAVA启动时主线程立即运行他是程序开始时就执行的，他时产生其他子线程的线程通常我们所说的线程都是其子线程，通常它必须最后完成执行因为它要执行一系列的关闭操作，主线程也可以控制，必须调用方法currentThread()获得他的一个引用currentThread()时Thread类的公有静态成员

### 多线程常用方法

Thread.currentThread().getName()可以返回代码段正在被哪个线程调用

sleep()休眠

join()将当前的主线程放入休眠池中但是不影响同时在运行的线程，例如

```
t1.start();
t1.join();//此时主线程停止直到t1线程全部结束
t2.start();
```

此时是要等t1执行完才会把主线程放出来再创建t2线程再执行，join必须放在start之后，如果t1.join()放在t2.start之后那么t1和t2还是并行运行

## volatile关键字与内存可见性

```
public class TestVolatile {
    public static void main(String[] args){ //这个线程是用来读取flag的值的
        ThreadDemo threadDemo = new ThreadDemo();
        Thread thread = new Thread(threadDemo);
        thread.start();
        while (true){
            if (threadDemo.isFlag()){
                System.out.println("主线程读取到的flag = " + threadDemo.isFlag());
                break;
            }
        }
    }
}

@Data
class ThreadDemo implements Runnable{ //这个线程是用来修改flag的值的
    public  boolean flag = false;
    @Override
    public void run() {
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        flag = true;
        System.out.println("ThreadDemo线程修改后的flag = " + isFlag());
    }
}
```

虽然控制台打印了ThreadDemo里面的话但是没有打印主程序中如果读到flag为true时打印的话，这表示在主线程中读到的flag还是false,这就是内存可见性的问题

### synchronized原理及应用

关键字synchronized可以保证在同一时间只有一个线程可以去执行某个方法或某个代码块直到执行完这个方法或者代码块为止别的线程都不能进来只有等待已经进去的线程解放才行，当然如果线程作用于不同的对象那么每个线程获得的是不同的锁所以互相并不影响，同时synchronized可以保证一个线程的变化可见(可见性),即可以代替volatile,还能保证共享变量的内存可见性



jdk1.6之前为重量级锁每次访问会调用操作系统函数让CPU调用内核

jdk1.7之后他从重量级锁重新划分为了无锁，偏向锁，轻量锁，重量锁，即每次**尽量从JVM的角度去解决**而不是每次都会去调用操作系统的函数让CPU负载过高





synchrnoized加锁是对于一个对象来说的例如我两个线程thread1，thread2启动都是用同一个对象test1创建的那么thread1和thread2他们两个线程共用同一把锁，如果一个线程进入了一个带锁的代码块，直到代码块运行结束之前带有相同锁的别的线程不能进入同样带有synchrnoized的其他方法但是可以在thread1进入带锁方法的同时thread2进入不带锁的其他方法



如果thread1,thread2是由两个不同的对象test1,test2分别创建的那么thread1,thread2所对应的锁是不同的两把，他们之间互不影响

#### 原理解析

JDK1.5之前synchronized是一个重量级锁相比于juc里面的lock同步锁来说它显得太过于笨重，我们慢慢摒弃它

但是再**JDK1.6之后对synchronized进行了各种优化**之后synchronized并不会显得那么笨重了，此时synchronized的**效率**和lock的效率基本没有差别

**死锁：子类同步方法调用了父类同步方法如果没有可重入的特性，则会发生死锁**

在java中synchronized可以使用在代码块和方法中，**其中锁方法是实例方法和静态方法分别锁的是该类的实例对象和该类对象**，**锁代码块也可以分为三种，锁同步代码块对应实例对象比如synchronized(this)锁住的是该类的实例对象，锁同步代码块对应类对象比如synchronized(TestDemo.class),还可以锁任意实例对象Object**

![Synchronized的使用场景](https://user-gold-cdn.xitu.io/2018/4/30/16315cc79aaac173?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**需要注意的是如果锁的是类对象那么尽管new多个实例对象但是如果他们依然属于同一个类那么依然会被锁住(阻塞)，即线程之间保证同步关系**

#### 对象锁(monitor)机制

在添加了Synchronized关键字之后执行同步代码块后要先执行monitorenter指令，**使用Synchronized进行同步其关键就是必须要对对象的监视器monitor进行获取，当线程获取monitor后才能继续往下执行，否则只能等待，而这个获取的过程是互斥的**，**在同一锁程中线程不需要再次获取同一把锁，这就是锁的重入性**，Synchronized先天具有重入性

每个锁拥有一个计数器，当线程获取该对象锁后计数器就会加一，释放锁后就会将计数器减一

**任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取到该对象的监制戚才能进入同步块和同步方法，如果没有获去到监视器的线程将会被阻塞在同步块和同步方法的入口处**，**同一时刻只能有一个线程能够获得对象的监视器**

线程被阻塞了并不是就一直停留在同步方法的入口处而是会进入同步队列，这也是为什么解锁之后并不一定会运行第一个被阻塞的线程

![对象，对象监视器，同步队列和线程状态的关系](https://user-gold-cdn.xitu.io/2018/4/30/16315cd5fa7cf91c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



happens-before关系:

如果A happens-before B则A的执行结果对B是可见的(A的执行结果更新到主存中去了，B执行时会从主存中获取到最新的值)，并且A的执行顺序先于B ，线程A先对共享变量的改变在B中是能够看得到的

#### synchronized优化

synchronized最大的特征就是在同一时刻只有一个线程能够获得对象的监视器(monitor)，从而进入到同步代码块或者同步方法之中，即**互斥性**，这样**每次只能通过一个线程的方法效率是非常低下的**，既然这种每次只能通过一个线程的额形式不能改变的话那么我们能不能让每次通过的速度变快一点，**锁优化需要了解CAS算法和四种锁状态(无锁，偏向锁，轻量锁，重量锁)**



#### CAS操作(compareAndSet(0,1)即判断该锁的状态是否为0，如果为0就更改为1，如果不为0就返回false,这是一个原子性操作)

使用锁时线程获取锁是一种**悲观锁**，**即假设每一次执行临界区代码都会产生冲突，所以当前线程获取到锁的时候同样也会阻塞其他线程获取该锁**。而C**AS操作(又称为无锁操作)是一种乐观锁策略**，**它假设所有线程访问共享资源的时候不会出现冲突，既然不会出现冲突就不会阻塞其他线程的操作因此线程就不会出现阻塞停顿的状态**，CAS又叫做比较交换来鉴别线程上是否出现冲突，**出现冲突就重试当前操作直到没有冲突为止**

CAS操作是指有三个值V(内存值)  A(预估值)  B(新值)，如果V==A 就把B更新给V，如果V!=A那么直接返回V即可，**当多个线程使用CAS操作一个变量时只有一个线程会成功其余会失败**

原来这个值是变为`3`了，我这个线程想修改这个值的时候我一定期望你现在是`3`，是`3`我才改，如果在我修改的过程你变`4`了，说明就有另外一个线程修改过该值，那我`cas`就再重新试一下，再试的时候，我希望你的这个值是`4`，在修改的时候期望值是`4`，没有其它线程修改该值，那好我给你改成`5`，这样就是`cas`操作



**synchronized未优化前最主要的问题是在存在线程竞争的情况下会出现线程阻塞和唤醒带来的巨大消耗，但CAS并不是直接将其放到同步队列中而是失败后会进行一定的尝试而非直接进行耗时的挂起唤醒操作**

#### CAS的应用场景

在JUC包中利用CAS实现的类有很多基本支撑起整个concurrency包的实现，在Lock实现中会有CAS改变state变量，在原子变量atomic包中的类也几乎都是用CAS实现的

#### CAS的问题

**ABA问题**：因为CAS会检查旧值有没有发生变化，这里存在这样一个有意思的问题，**比如一个旧值A变成了B然后再变成了A，刚好在做CAS时检查发现旧值依然为A**，解决方案可以使用增加版本号或者**使用atomic包下的AtomicStampedReference来解决ABA问题**

**自旋时间过长**：C**AS是非阻塞同步也就是说不会将线程挂起而是会一直自旋进行下一次尝试，如果在这里的自旋时间过长对性能是很大的消耗**

**只能保证一个共享变量的原子操作**：当对一个共享变量执行操作时CAS能保证其原子性，但是如果**对多个共享变量进行操作CAS不能保证其原子性**，有一个**解决方案是利用对象整合多个变量然后对这个对象做CAS操作就可以保证其原子性，atomic中提供了AtomicReference来保证引用对象之间的原子性**

#### JAVA对象头

在同步的时候获取对象的monitor即获取对象锁就是对象的一个标志，那么这个标志就是存放在Java对象的对象头

在JDK1.6中**锁一共有4种状态，级别从低到高依次是无锁状态，偏向锁状态，轻量锁状态和重量级锁状态**，这几个状态会随着竞争情况逐渐升级，**锁可以升级但是不能降级**，不能降级的目的是为了提高获得锁和释放锁的效率



#### 偏向锁

**大多数情况下,锁不仅不存在多线程竞争，而且总是由同一线程多次获得**，为了让线程获得锁的代价更低而引入了偏向锁，如果发生了竞争就升级为轻量锁或者重量锁

**偏向锁的获取:**

当一个线程访问同步块并获取锁时，会在**对象头和栈帧中的锁记录里存储锁偏向的线程ID**，以后该线程**在进入和退出同步块时不需要进行CAS操作来加锁和解锁只是简单的测试一下对象头的 MarkWord里是否存储着当前线程的偏向锁ID**。**如果测试成功表示线程已经获得了锁，如果测试失败则需要再测试一下MarkWork中偏向锁的标识是否设置成1(表示当前是偏向锁)，如果没有设置成1则使用CAS竞争锁，如果是1(是偏向锁)则尝试使用CAS将对象头的偏向锁指向当前线程**

**偏向锁的撤销：**

偏向锁使用了一种**等到竞争出现才释放锁**的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁

![偏向锁撤销流程](https://user-gold-cdn.xitu.io/2018/4/30/16315d0b13b37da4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**偏向锁总结:**

偏向锁并不存在互斥性，而是**为了减少同一个线程拿锁时的消耗只会在指向线程ID的时候才会使用一次CAS，所以效率非常的高**，**如果有别的线程来竞争这个锁时这个线程就会撤销偏向锁**，尝试把Mark Word里的线程ID指向竞争的锁,然后判断这个线程是否还存活，**如果还活着就会把该线程放入到轻量级锁当中(通过竞争之后锁升级了)**



#### 轻量级锁(CAS)

**加锁：**

线程在执行同步块之前，JVM会先在当前线程的栈帧中创建用于存储锁记录的空间，并**将对象头中的Maik Word复制到锁记录中**，**然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针，如果成功当前线程获得锁，如果失败表示其他线程也在竞争锁，当前线程便尝试使用自旋来获取锁**

**解锁**：

轻量级解锁时，会使用原子的CAS操作将Mark Work替换回对象头如果成功则表示没有发生竞争，如果失败表示当前锁存在竞争锁就会膨胀成重量级锁

**总结轻量级锁：**

**如果说偏向锁是只允许一个线程获取锁如果出现竞争就升级为轻量级锁的话，那么轻量级锁就是允许多个线程获得锁但是只允许他们按顺序拿锁，不允许出现竞争，一旦出现竞争有一个线程拿锁失败就会升级为重量级锁**，**一旦升级称为重量级锁那么之前CAS自旋的线程都会被阻塞住，然后后面就按照重量级锁的步骤来(当持有锁的线程释放锁之后会唤醒这些被阻塞的线程，唤醒的线程又开始新的一轮竞争)**

#### 各种锁的比较

![各种锁的对比](https://user-gold-cdn.xitu.io/2018/4/30/16315cb91da523d9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 内存可见性

![](http://maplefi.gitee.io/picture/bed/image-20200612135637607.png)



解决方案可以加上synchronized来让同一时间只取到一个线程的值但是会造成线程阻塞问题导致效率特别低

### volatile关键字

用法：

volatile关键字：当多个线程操作共享数据时可以保证内存中的数据可见，用这个关键字修饰共享数据就会及时的把线程缓存中的数据刷新到主存中去也可以理解为直接操作主存中的数据，所以在不使用锁的情况下可以使用volatile修饰

```
public volatile boolean flag=false;
```



#### volatile和synchronized的区别

volatile不具备互斥性(当一个线程持有锁时别的线程进不来)，即每个线程都可对其进行操作并更新到主线程中相当于对一个全局变量的更新



volatile不具备原子性

## 原子性

```
public class TestIcon {
    public static void main(String[] args){
        AtomicDemo atomicDemo = new AtomicDemo();
        for (int x = 0;x < 10; x++){
            new Thread(atomicDemo).start();
        }
    }
}

class AtomicDemo implements Runnable{
    private int i = 0;
    public int getI(){
        return i++;
    }
    @Override
    public void run() {
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(getI());
    }
}
```

这里线程每次都睡眠一段时间然后执行那个线程的getI()

结果为：

![image-20200612152932586]( http://maplefi.gitee.io/picture/bed/image-20200612152932586.png)

可以发现出现了重复数据明显产生了多线程安全问题或者说原子性问题，**所谓的原子性问题就是操作不可再细分**而i++操作分为读改写三步

```
int temp = i;
i = i+1;
i = temp;
```

所以i++明显不是原子操作。上面10个线程进行i++时内存图解如下：

![image-20200612153147580]( http://maplefi.gitee.io/picture/bed/image-20200612153147580.png)

再当前线程的run还没有执行完的时候另一个线程启动了该线程的操作延后

**但是这里如果加了volatile也不能解决问题，加上volatile之后知识相当于所有线程都是再主存中操作数据而已，但是不具备互斥性比如两个线程同时读取主存中的0，然后又同时自增同时写入主存结果还是会出现重复数据**



### 原子变量

JDK1.5之后Java提供了原子变量在java.util.concurrent.atomic包下，原子变量具备如下特性

有volatile保障内存可见性

用CAS算法保证原子性

### CAS算法

CAS算法是计算机硬件对并发操作共享数据的支持，CAS包含三个操作数

内存值V

预估值A

更新值B

当且仅当V==A时才会把B的值赋给V，如果不相等就不操作

线程安全类即current类中的底层实现保证线程安全并非使用的同步锁synchronized而是使用的CAS

### 改进上述程序利用原子变量

```
    private AtomicInteger i = new AtomicInteger();
    public int getI(){
        return i.getAndIncrement();
    }
```

只需要改进这两个就行了，利用原子变量的线程安全的包来使得操作为原子性并且保障其内存可见性

## 锁分段机制

再java.util.concurrent包中提供了多种并发容器类来改进同步类的性能其中最主要的就是ConcurrentHashMap

原来的HashMap是线程不安全的，HashTable加了锁是线程安全的因此效率极低，HashTable几所就是将整个hash表锁起来，当有多个线程访问时同一时间只有能有一个线程访问所以JDK1.5之后提供了ConcurrentHashMap

![image-20200612202534003](  http://maplefi.gitee.io/picture/bed/image-20200612202534003.png)

ConcurrentHashMap默认分成了16个segment每个Segment都对应一个Hash表并且都有独立的锁这样可以每个线程访问一个Segment,但是在**Java1.8之后还是用CAS代替的这个原有的底层**



java,util.concurrent包还提供了设计用于多线程上下文中的Collection实现：

ConcurrentHashMap,CocurrentSkipListMap,ConcurrentSkipListSet,CopyOnWriteArrayList和CopyOnWriteArraySet



当期望许多线程访问一个给定的collection时,ConcurrentHashMap通常优于同步的HashMap,CCocurrentSkipListMap通常优于同步的TreeMap,当期望的读数和遍历远远大于列表的更新数时，CopyOnWriteArrayList优于同步的ArrayList，多线程并发读写集合一定得使用CopyOnWriteArrayList而不能使用ArrayList否则会出错

## 闭锁

java.util.concurrent包中提供了多种并发容器来改进同步容器的性能，ContDownLatch是一个同步辅助类 ，**在完成某些运算时，只有其他所有线程的运算全部完成当前运算才继续执行，这就叫做闭锁，即阻塞主线程继续往下走，直到CountDownLatch为0为止主程序都不会继续往下走，await()上面的部分不影响**

```
public class TestCountDownLatch {
    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(10);//有多少个线程这个参数就是几
        LatchDemo ld = new LatchDemo(latch);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10; i++) {
            new Thread(ld).start();
        }
        try {
            latch.await();//这10个线程执行完之前先等待
        } catch (InterruptedException e) {
        }
        long end = System.currentTimeMillis();
        System.out.println("耗费时间为：" + (end - start));
    }
}

class LatchDemo implements Runnable {
    private CountDownLatch latch;
    public LatchDemo(CountDownLatch latch) {
        this.latch = latch;
    }
    @Override
    public void run() {
        synchronized (this) {
            try {
                for (int i = 0; i < 50000; i++) {
                    if (i % 2 == 0) {//50000以内的偶数
                        System.out.println(i);
                    }
                }
            } finally {
                latch.countDown();//每执行完一个就递减一个
            }
        }
    }
}
```

主要就是通过使用latch.countDown()和latch.await()实现闭锁，latch.countDown()会把CountDownLatch本身的计数减一，**如果将一个CountDownLatch赋值给另一个CountDownLatch那么它们两个变量值共用一个主存即一个改变之后另一个也会跟着改变**，**因为它是current包下的底层为CAS是一个原子变量**，latch.await()直到在此之前**对应的countdownlatch为0为止**都阻塞主程序往下走，在此之前的线程都可以正常运行，CountDownLatch本身只是一个计数器阻塞往下的线程，它本身没有互斥性的性能

## 信号量(Semaphore)

计数信号量用来控制同时访问某个特定资源的操作适量或者同时执行某个指定操作的数量，信号量还可以用来实现某种资源池，或者对容器施加边界



Semaphore管理着一组许可，许可的初始数量可以通过构造函数设定，操作时首先要获取到许可才能进行操作，操作完成后需要释放许可，如果没有获取到许可，则被阻塞直到有许可被释放。如果初始化了一个许可为1的Semaphore那么就相当于一个不可重入的互斥锁(Mutex)



**实例场景:**

假设生活中一个常见的场景:每天早上，大家都热衷于带薪上厕所，但是公司一共就只有10个坑位，那么只能同时10个人用着，后面来的人都得等着(阻塞)，如果走了2个人那么又可以进去两个人，这就是Semaphore的应用场景争夺有限的资源



代码实战：

```
package concurrency;

import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;


class Employee implements Runnable {
    private String id;
    private Semaphore semaphore;
    private static Random rand= new Random(47);

    public Employee(String id, Semaphore semaphore) {
        this.id = id;
        this.semaphore = semaphore;
    }

    public void run() {
            try {
                semaphore.acquire();
                System.out.println(this.id + "is using the toilet");
                TimeUnit.MILLISECONDS.sleep(rand.nextInt(2000));
                semaphore.release();
                System.out.println(this.id + "is leaving");
            } catch (InterruptedException e) {
            }
    }
}

public class ToiletRace {
    private static final int THREAD_COUNT = 30;

    private static ExecutorService threadPool = Executors
            .newFixedThreadPool(THREAD_COUNT);

    private static Semaphore s = new Semaphore(10);

    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            threadPool.execute(new Employee(String.valueOf(i), s));
        }

        threadPool.shutdown();
    }
}


```



## Lock同步锁



### 介绍

在JDK1.5之前解决多线程安全问题有两种方式:使用synchronized锁代码块或者锁方法，但是在JDK1.5之后出现了更加灵活的方式Lock锁(显式锁/同步锁)



Lock需要通过lock()方法上锁，通过unlock()方法释放锁，为了保证锁能释放unlock()一般放在finally中去执行，一旦锁定了代码块之后直到释放前这个线程会一直走下去直到lock释放(也就是阻塞线程)，如果中途遇到睡眠也不会去执行另一个线程而是一直等到这个线程的工作全部执行完，同样的也是只能限制同一对象开启的多个线程



和synchronized的区别，1.8之后性能差距不大，synchronized是JAVA关键字而ReentrantLock是线程AIP提供的互斥锁，为了避免死锁的状态最好在finally里面写上unlock()

### 初步使用

创建对象

```
private Lock lock=new ReentrantLock();

在需要上锁的地方加上lock.lock();
然后使用try{}将需要上锁的代码块框起来
最后最好使用lock.unlock()来将锁释放
```

**ReentrantLock使用的公平锁或者非公平锁取决于创建对象时的方法，如果什么参数都不加那么默认为非公平锁**

### AQS

AQS即AbstractQueueSynchronizer得缩写，是并发编程中实现同步器得一个框架

AQS基于一个FIFO双向队列实现，被设计给那些依赖一个代表状态得原子int值得同步器使用，**在AQS中有一个state得int值代表同步状态，该值通过CAS进行原子修改**

**AQS中存在一个FIFO队列(同步队列)，当共享资源被某个线程占有，其他请求该资源的线程将会阻塞，从而进入同步队列，队列节点元素有4种类型，**每种类型标识线程被阻塞得原因**，这四种类型分别是

**CANCELLED=1**:表示该线程是因为超时或者终端得原因而被放到队列中,节点从同步队列中轮到它时将其从同步队列移除

**CONDITION=-2**:表示线程是因为某个条件不满足而被放到**等待队列**中

**SINGAL=-1**:后继节点的线程处于等待状态，如果当前节点释放，同步状态会通知后继节点使得后继节点的线程能够运行

**PROPAGATE=-3**:表示下一次共享式同步状态获取将会无条件传播下去

由于一个共享资源同一时间只能由一条线程持有也可以被多个线程持有(不会阻塞)，因此AQS中存在**两种模式**

**1.独占模式**

**独占模式标识共享状态值每次只能由一条线程持有其他线程如果需要获取则需要阻塞**，如JUC中得ReetrantLock

```
void acquire(int arg)：独占式获取同步状态，如果获取失败则插入同步队列进行等待；
void acquireInterruptibly(int arg)：与acquire方法相同，但在同步队列中进行等待的时候可以检测中断；
boolean tryAcquireNanos(int arg, long nanosTimeout)：在acquireInterruptibly基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false;
boolean release(int arg)：释放同步状态，该方法会唤醒在同步队列中的下一个节点
```

**2.共享模式**

**共享模式标识共享状态值state每次可以由多个线程池持有(线程不会被阻塞)**如JUC中得CountDownLatch

```
void acquireShared(int arg)：共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态；
void acquireSharedInterruptibly(int arg)：在acquireShared方法基础上增加了能响应中断的功能；
boolean tryAcquireSharedNanos(int arg, long nanosTimeout)：在acquireSharedInterruptibly基础上增加了超时等待的功能；
boolean releaseShared(int arg)：共享式释放同步状态
```



### AQS中得核心数据结构方法

**既然AQS基于一个FIFO队列，那么我们先来看下队列得元素节点Node得数据结构，源码如下:**

```
static final class Node {
    /**共享模式*/
    static final Node SHARED = new Node();
    /**独占模式*/
    static final Node EXCLUSIVE = null;

    /**标记线程由于中断或超时，需要被取消，即踢出队列*/
    static final int CANCELLED =  1;
    /**线程需要被唤醒，即前一个结点释放之后就会通知这个结点的线程执行*/
    static final int SIGNAL = -1;
    /**线程正在等待一个条件，条件队列只能是独占模式，条件执行之后会将其移动到同步队列中去尝试获取锁*/
    static final int CONDITION = -2;
    /**
     * 传播，在共享模式中该类型结点表示该结点的线程处于可运行状态，只能是共享模式
     */
    static final int PROPAGATE = -3;
    
    // waitStatus只取上面CANCELLED、SIGNAL、CONDITION、PROPAGATE四种取值之一，当前节点状态，属于等待队列
    volatile int waitStatus;

    // 表示前驱节点
    volatile Node prev;

    // 表示后继节点
    volatile Node next;

    // 使该节点进入队列得线程，即对应线程
    volatile Thread thread;

    // 等待队列中的下一个节点
    Node nextWaiter;

    /**
     * 是否当前结点是处于共享模式
     */
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    /**
     * 返回前一个节点，如果没有前一个节点，则抛出空指针异常
     */
    final Node predecessor() throws NullPointerException {
        // 获取前一个节点的指针
        Node p = prev;
        // 如果前一个节点不存在
        if (p == null)
            throw new NullPointerException();
        else
        // 否则返回
            return p;
    }

    // 初始化头节点使用
    Node() {}

    /**
     *  当有线程需要入队时，那么就创建一个新节点，然后关联该线程对象，由addWaiter()方法调用
     */
    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    /**
     * 一个线程需要等待一个条件阻塞了，那么就创建一个新节点，关联线程对象
     */
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}

```

**总结Node节点数据结构设计**

队列中得元素，**肯定是为了保存由于某种原因导致无法获取共享资源state而被入队得线程**，**因此Node中使用了waitStatus标识节点入队得原因**，**使用Thread对象表示使该节点入队列得线程**，使用prev和next因为FIFO是双向队列所以必须提供上一个和下一个节点用于对队列得操作

**prev和next对应AQS中的双向队列，用于阻塞队列**

**而nextWaiter是用于单链表的，只有条件队列才使用**

**AQS中阻塞队列和条件队列所使用的数据结构不同**

**案例：**

```
public class LockDemo {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(() -> {
                lock.lock();
                try {
                    Thread.sleep(10000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            });
            thread.start();
        }
    }
}
```

这里开启了5个线程，Thread-0进去拿到锁之后睡眠，剩下的4个线程无法获取锁进入同步队列，当最后一个线程进入同步队列后的AQS状态

![LockDemo debug下 .png](https://user-gold-cdn.xitu.io/2018/5/3/163261637bcef7e2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

由于lock是独占锁，所以第一个得状态为SINGLE,后面得等待前面得状态更新所以是0

### 独占锁得获取(acquire方法)

**lock()方法是获取独占锁，获取失败就将当前线程加入同步队列中，成功就执行线程，而lock()方法实际上会调用AQS得acquire()方法**

```
public final void acquire(int arg) {
		//先看同步状态是否获取成功，如果成功则方法结束返回
		//若失败则先调用addWaiter()方法再调用acquireQueued()方法
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```

**获取锁是否成功在于tryAcquire(arg)方法**:

```




```



**如果成功获取当前同步状态就方法结束，如果失败就会先调用addWaiter()然后再调用acquireQueued()方法**,Node.EXCLUSIVE即创建一个null得Node对象

**获取同步状态失败，入队操作**

当线程获取独占式锁失败后就会将当前线程加入同步队列，那么就应该眼界一下addWaiter()和acquireQueued()源码了

**addWaiter()源码如下:**

```
private Node addWaiter(Node mode) {
		// 1. 将当前线程构建成Node类型
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        // 2. 当前尾节点是否为null？
		Node pred = tail;
        if (pred != null) {
			// 2.2 将当前节点尾插入的方式插入同步队列中
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
		// 2.1. 当前同步队列尾节点为null，说明当前线程是第一个加入同步队列进行等待的线程
        enq(node);
        return node;
}
```

**即先把作为参数得mode重新构造出一个以当前线程为属性得新Node对象，然后将其加到队列得末尾处，如果当前末尾为null那么就说明该对象是第一个加入同步队列得调用enq()方法进行插入，如果当前末尾不为null则采用compareAndSetTail()方法进行插入，由于compareAndSetTail()是一个CAS操作锁如果它失败之后应该会进行自旋尝试，所以这里得enq()承担两个任务，一个是将当前对象加入同步队列中，另一个是CAS尾插失败后负责自旋进行尝试**

这里我们再看一下enq得源码

```
private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
			if (t == null) { // Must initialize
				//1. 构造头结点
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
				// 2. 尾插入，CAS操作失败自旋尝试
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
}
```

**总结enq()方法:**

**1.在当前线程是第一个加入同步队列时，调用compareAndSetHead(new Node())初始化头节点**

**2.自旋不断尝试CAS尾插入节点直到成功为止**

现在已经清楚获取独占锁失败得线程包装成Node进入同步队列得操作了，但是同步队列中得节点会做什么来保证自己能够有机会获得独占式锁了?**acquireQueued()方法他的作用就是排队获取锁得过程**

```
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
				// 1. 获得当前节点的先驱节点
                final Node p = node.predecessor();
				// 2. 当前节点能否获取独占式锁					
				// 2.1 如果当前节点的先驱节点是头结点并且成功获取同步状态，即可以获得独占式锁
                if (p == head && tryAcquire(arg)) {
					//队列头指针用指向当前节点
                    setHead(node);
					//释放前驱节点
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
				// 2.2 获取锁失败，线程进入等待状态等待获取独占式锁
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
}
```

**首先获取该节点得先驱节点，如果先驱节点是头结点得并且该节点能成功获得同不在状态得时候，当前节点所指向得线程能够获取锁**，反之获取锁进入等待状态

**如果获取锁成功，出队操作**

即将头结点引用指向当前节点并释放原来得头结点

**如果获取锁失败，等待操作，会调用shouldParkAfterFaailedAcquire()方法和parkAndCheckInterrupt()方法**，**park方法是由Unsafe类所提供的一个让该线程永远睡眠的方法(阻塞)，除非使用unpark方法唤醒对应的线程(即刻唤醒)才会被唤醒**

```
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;//获取前一个节点的额状态
    if (ws == Node.SIGNAL)//如果前一个节点为-1即SIGNAL状态表示还没轮到该节点
        /*
         * This node has already set status asking a release
         * to signal it, so it can safely park.
         */
        return true;
    if (ws > 0) {//超时状态CANCELLED,如果为这个状态应该将其从队列提出
        /*
         * Predecessor was cancelled. Skip over predecessors and
         * indicate retry.
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {//如果不为上述两种状态那么就使用CAS将前置节点状态由INITIAL设置成SIGNAL表示阻塞当前线程
        /*
         * waitStatus must be 0 or PROPAGATE.  Indicate that we
         * need a signal, but don't park yet.  Caller will need to
         * retry to make sure it cannot acquire before parking.
         */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

shouldParkAfterFailedAcquire()方法**主要逻辑是使用compareAndSetWaitStatus(pred,ws,Node.SIGNAL)使用CAS当前节点节点状态由INITIAL设置成SIGNAL表示当前线程阻塞**，如果compareAndSetWaitStatus设置失败则说明shouldParkAfterFailedAcquire()方法返回false然后acquireQueued()方法中for(;;)死循环中会继续重试，直至将其前一个节点得状态置为SIGNAL为止

shouldParkAfterFailedAcquire()方法返回true之后会执行**parkAndCheckInterruput()方法**用来阻塞当前 线程

acquireQueued()在自选过程中主要完成了两件事

**1.如果当前节点的前驱节点是头结点，并且能够获得同步状态的话当前线程能够获得锁该方法执行结束退出**

**2.获取锁失败的话，先将当前节点状态设置成SIGNAL然后调用LookSupport.park方法使得当前线程阻塞**

acquire的工作流程图为:



![独占式锁获取（acquire()方法）流程图.png](https://user-gold-cdn.xitu.io/2018/5/3/163261637c891cc2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**有关获取同步状态的tryacquire()方法在acquire中执行了一次，失败之后将节点尾插到同步队列(如果插入失败或者是第一个到同步队列中的线程的话会执行enq方法来保证将该节点插入尾节点之后)，之后在acquireQueued()里面的for循环中又执行tryacquire()方法，还是失败之后尝试将当前节点置为SIGNAL并且阻塞当前线程，直到成功将当前节点置为SIGNAL之前会反复执行tryacquire()方法来尝试获取锁。**

### 独占锁的释放(release())

先看一下release()方法的源码

```
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
}
```

将当前独占锁释放成功后会去**检查同步队列的头结点指向的节点是否为空并且会判断他的共享同步状态不为INITIAL=0后会调用unparkSuccessor(Node node)方法**

```
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */

	//头节点的后继节点
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
		//后继节点不为null时唤醒该线程
        LockSupport.unpark(s.thread);
}
```

**每一次锁释放后就会唤醒队列中该节点的额后继节点所引用的线程，从而进一步验证获得锁的过程是先进先出的**



总结独占锁（SIGNAL==-1）

**1.线程获取锁失败，线程被封装成Node进入同步队列(核心方法在于addWaiter()和enq(),enq()方法完成对同步队列的头结点初始化工作以及CAS操作(加入到队尾)失败的重试)**

**2.线程获取锁是一个自旋的过程，当且仅当当前的节点的前驱节点是头结点并且成功获取同步状态时节点出队即该节点引用的线程获取锁，否则当不满足条件的时候就会使用LookSupport.park()方法使得线程阻塞**

**3.释放锁的时候会唤醒头节点的后继节点**

**总体来说:在获取同步状态时AQS维护一个同步队列，获取同步状态失败的线程会进入同步队列进行自旋，移除队列的条件是前驱节点是头结点并且成功获得了同步状态，在释放同步状态时，，同步器会调用unparkSuccessor()方法唤醒后继节点**

### 共享锁的获取

共享锁的获取为acquireShared:

```
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

在该方法中会首先调用tryAcquireShared方法，**当返回值为大于等于0的时候方法结束说明成功获取锁**，**否则表明获取同步状态失败即获取锁失败会执行doAcquireShared()方法**，共享式锁获取失败后不会将该线程加入同步队列而是直接从同步队列中拿节点出来尝试获取锁

```
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
					// 当该节点的前驱节点是头结点且成功获取同步状态
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

```

**逻辑几乎和独占式锁的获取失败一模一样，只是退出的条件独占式锁是前驱节点是头结点并且tryacquire()成功，共享式锁的退出条件是前驱节点是头结点并且tryacquireShared()返回值大于等于0就能成功获取同步状态**，

### 共享锁的释放

```
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

释放成功后会调用doReleaseShared()方法

```
private void doReleaseShared() {
    /*
     * Ensure that a release propagates, even if there are other
     * in-progress acquires/releases.  This proceeds in the usual
     * way of trying to unparkSuccessor of head if it needs
     * signal. But if it does not, status is set to PROPAGATE to
     * ensure that upon release, propagation continues.
     * Additionally, we must loop in case a new node is added
     * while we are doing this. Also, unlike other uses of
     * unparkSuccessor, we need to know if CAS to reset status
     * fails, if so rechecking.
     */
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

**共享式锁的释放和独占式锁释放过程有点点不同，在共享式锁释放过程中，对于能够支持多个线程同时访问的并发组件必须保证多个线程能够安全的释放同步状态，这里采用CAS保证当CAS失败就continue,在下一次循环中进行重试**

### 可中断式获取锁(acquireInterruptibly方法)

我们直到lock相较于synchronized有一个更方便的特性，比如能响应中断以及超时等待等特性，可响应中断式锁可调用方法lock.lockInterruptibly(),而其底层会调用AQS的acquireInterruptibly()方法

```
public final void acquireInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
		//线程获取锁失败
        doAcquireInterruptibly(arg);
}
```

在获取同步状态失败后就会调用doAcquireInterruptibly()方法

```
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
	//将节点插入到同步队列中
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            //获取锁出队
			if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
				//线程中断抛异常
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```



### AQS设计妙处

**自旋锁**

当我们执行一个有确定结果的 操作，同时又需要并发正确执行，通常可以采用自旋锁实现，在AQS中**自旋锁采用死循环+CAS实现**，死循环和CAS可保证并发安全，同一时间只有一个节点安全入队，入队失败的线程则循环重试

### 在AQS基础上定义并发同步器

在AQS基础上定义并发同步器一般来说都要先定义一个内部类Sync

```
class Mutex implements Lock, java.io.Serializable {
    // 自定义同步器
    private static class Sync extends AbstractQueuedSynchronizer {
        // 判断是否锁定状态
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // 尝试获取资源，立即返回。成功则返回true，否则false。
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // 这里限定只能为1个量
            if (compareAndSetState(0, 1)) {//state为0才设置为1，不可重入！
                setExclusiveOwnerThread(Thread.currentThread());//设置为当前线程独占资源
                return true;
            }
            return false;
        }

        // 尝试释放资源，立即返回。成功则为true，否则false。
        protected boolean tryRelease(int releases) {
            assert releases == 1; // 限定为1个量
            if (getState() == 0)//既然来释放，那肯定就是已占有状态了。只是为了保险，多层判断！
                throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);//释放资源，放弃占有状态
            return true;
        }
    }

    // 真正同步类的实现都依赖继承于AQS的自定义同步器！
    private final Sync sync = new Sync();

    //lock<-->acquire。两者语义一样：获取资源，即便等待，直到成功才返回。
    public void lock() {
        sync.acquire(1);
    }

    //tryLock<-->tryAcquire。两者语义一样：尝试获取资源，要求立即返回。成功则为true，失败则为false。
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    //unlock<-->release。两者语文一样：释放资源。
    public void unlock() {
        sync.release(1);
    }

    //锁是否占有状态
    public boolean isLocked() {
        return sync.isHeldExclusively();
    }
}
```

**然后在自定义同步类中加上一个属性private final Sync sync来利用AQS来实现自定义同步类**



### AQS总结

**AQS是一个并发同步框架，它内部有一个同步队列如果是独占式锁线程获取锁失败后就会进入到同步队列中(双向队列)并加入队尾，如果此时同步队列为空会先创建一个头结点然后一直尝试直到将该线程放入同步队列中，成功放到同步队列尾部之后会将队首的节点拿出来尝试获取锁，失败就阻塞，同步队列中有四种状态表示进入同步队列的原因，如果是共享式锁获取锁失败之后线程不会进入同步队列中而是跳过这一步骤将同步队列中的首个节点拿出来尝试获取锁失败就阻塞，AQS除了同步队列以外还有外部的用volatile修饰的共享同步状态值state,锁的获取成功与否与其息息相关，独占锁的释放成功之后会将队首阻塞的线程唤醒并将其在同步队列中的状态置为0，共享式锁的释放和独占式锁相同只是它会使用CAS+死循环的方式保证多个线程能安全的释放同步状态然后将队首的状态值置为0并唤醒**

## 等待唤醒机制(代价非常的高)

### wait和notify以及notifyAll

wait  notify  notifyAll这三个最好都在synchronized或者lock的情况下使用，否则容易出现错误，wait的作用是指让当前线程等待，等待时会自动解锁，直到该线程调用notify或者notifyAll为止才会重新在刚才的为止唤醒

### wait和sleep的区别

wait是object类的,sleep属于线程类,sleep不释放锁而wait会释放锁



notify是指唤醒wait等待的一个线程,notifyAll是指释放wait等待的所有线程

### 虚假唤醒

一个消费者线程抢到执行权，发现product是0，就等待，这个时候，另一个消费者又抢到了执行权，product是0，还是等待，此时两个消费者线程在同一处等待。然后当生产者生产了一个product后，就会唤醒两个消费者，发现product是1，同时消费，结果就出现了0和-1。这就是**虚假唤醒**。

### 用Lock锁实现等待唤醒

**Condition用来代替传统的wait和notify实现线程之间的协作**，相比与使用wait和notify使用Confition的await，singal这种方式实现线程间协作更加安全和高效，**阻塞队列实际上是用了Condition来模拟线程间的协作**



Condition是个接口基本得方法就是await和signal

Condition依赖于Lock接口生成一个Condition对象使用lock.newCondition()

Condition中的await对应wait(),signal对用notify,signalAll对应notifyAll

### 线程指定顺序执行，使用condition来实现线程中的通讯

```
public class TestLoopPrint {
    public static void main(String[] args) {
        AlternationDemo ad = new AlternationDemo();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    ad.loopA();
                }
            }
        }, "A").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    ad.loopB();
                }
            }
        }, "B").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    ad.loopC();
                }
            }
        }, "C").start();
    }
}

class AlternationDemo {
    private int number = 1;//当前正在执行的线程的标记
    private Lock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    Condition condition3 = lock.newCondition();

    public void loopA() {
        lock.lock();
        try {
            if (number != 1) { //判断
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName());//打印
            number = 2;
            condition2.signal();
        } catch (Exception e) {
        } finally {
            lock.unlock();
        }
    }

    public void loopB() {
        lock.lock();
        try {
            if (number != 2) { //判断
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName());//打印
            number = 3;
            condition3.signal();
        } catch (Exception e) {
        } finally {
            lock.unlock();
        }
    }

    public void loopC() {
        lock.lock();
        try {
            if (number != 3) { //判断
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName());//打印
            number = 1;
            condition1.signal();
        } catch (Exception e) {
        } finally {
            lock.unlock();
        }
    }
}
```

## ReadWriterLock读写锁

我们在读数据的时候可以多个线程同时读，但是在写数据的时候如果多个线程同时写数据那么到底是写入哪个线程的数呢？所以如果有两个线程，写写/读写需要互斥，读读不需要互斥，这个时候可以使用读写锁

```
public class TestReadWriterLock {
    public static void main(String[] args){
           ReadWriterLockDemo rw = new ReadWriterLockDemo();
           new Thread(new Runnable() {//一个线程写
               @Override
               public void run() {
                   rw.set((int)Math.random()*101);
               }
           },"write:").start();
           for (int i = 0;i<100;i++){//100个线程读
               Runnable runnable = () -> rw.get();
               Thread thread = new Thread(runnable);
               thread.start();
           }
    }
}

class ReadWriterLockDemo{
    private int number = 0;
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    //读(可以多个线程同时操作)
    public void get(){
        readWriteLock.readLock().lock();//上锁
        try {
            System.out.println(Thread.currentThread().getName()+":"+number);
        }finally {
            readWriteLock.readLock().unlock();//释放锁
        }
    }
    //写(一次只能有一个线程操作)
    public void set(int number){
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName());
            this.number = number;
        }finally {
            readWriteLock.writeLock().unlock();
        }
    }
}
```

ReadWriterLock读写锁通过使用readLock().lock()和writeLock.lock()来进行读操作和写操作的上锁，以及对应的unlonk来进行解锁操作

### 但是为了更加安全JAVA的并发包提供了ReentrantReadWriteLock

它表示两个锁一个是读操作的共享锁，一个是写操作的排他锁

**线程进入读锁的前提条件**：

> ```
> 没有其他线程的写锁
> 
> 没有写请求或者有写请求但是调用线程和持有锁的线程是同一个
> ```

**线程进入写锁的前提条件**：

```
没有其他线程的读锁
没有其他线程的写锁
```

**读写锁有以下三个重要的特性**

```
公平选择性：支持非公平（线程进入的顺序不固定）(默认)和公平（线程进入顺序从队列中获取，即串行的）的锁获取方式，吞吐量还是非公平优于公平

重进入：读锁和写锁都支持线程的重进入(即可以在读锁里调用写锁，写锁里调用读锁)

锁降级：遵循获取写锁，获取读锁再释放写锁的次序，写锁能够降级为读锁
```

源码：

```
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable {

    /** 读锁 */
    private final ReentrantReadWriteLock.ReadLock readerLock;

    /** 写锁 */
    private final ReentrantReadWriteLock.WriteLock writerLock;

    final Sync sync;//继承AbstractQueueSynchronizer
    
    /** 使用默认（非公平）的排序属性创建一个新的 ReentrantReadWriteLock */
    public ReentrantReadWriteLock() {
        this(false);
    }

    /** 使用给定的公平策略创建一个新的 ReentrantReadWriteLock */
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();//两者都继承Sync
        readerLock = new ReadLock(this);//ReadLock和WriteLock都继承Lock
        writerLock = new WriteLock(this);
    }

    /** 返回用于写入操作的锁 */
    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
    
    /** 返回用于读取操作的锁 */
    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }


    abstract static class Sync extends AbstractQueuedSynchronizer {}

    static final class NonfairSync extends Sync {}

    static final class FairSync extends Sync {}

    public static class ReadLock implements Lock, java.io.Serializable {}

    public static class WriteLock implements Lock, java.io.Serializable {}
}
```

ReentrantReadWriteLock的对象创造如果是无参的情况下会默认创造一个非公平策略，如果创造时带参数true就会创造一个公平策略其余操作和ReadWriteLock基本相同因为它是实现了ReadWriteLock的

## 线程池

我们使用线程的时候需要new一个线程用完了有需要销毁这样频繁的创建和销毁很消耗西元，所以就提供了线程池道理和Jedis连接池差不多，每次线程从线程池中拿出用完归还给线程池，线程池中有一个线程队列，里面保存着所有等待状态的线程



### 创建线程池

```
ExecutorService pool=Executors.newFixedThreadPool(5);
pool.submit(Thread)//为线程池中的线程分配任务
pool.shutdown();//关闭线程池
```

###  ThreadPoolExecutor类

java.util.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，因此如果要透彻了解线程池那么必须先了解这个类。

再ThreadPoolExecutor类中提供了四个构造方法

```
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```

参数说明：

corePoolSize:核心池的大小，这个参数和线程池的实现原理有非常大的关系，创建线程池之后默认没有任何线程 ，而是有任务来才创建线程去执行任务，除非调用预创建线程方法否则在任务来之前线程池内的线程数量都为0，当线程池中的线程数量达到corePoolSize后就会把后面的线程放到缓存队列当中

maximumPoolSize:线程池的最大线程数，这个参数表示线程池中最多能创建多少个线程

keepAliveTime:表示线程没有任务执行时最多保持多久时间会终止，只有当线程数大于corePoolSize时keepAliveTime才能起作用，大于corePoolSize之后如果有一个线程空闲时间达到keepAlive就会终止该线程，如果调用了allowCoreThreadTimeOut(Boolean)方法，在线程池线程总数不大于corePoolSize时keepAliveTime也会起作用直到线程池里的线程数量为0

unit:参数keepAliveTime的时间单位，比如TimeUnit.DAYS,或者TimeUnit.HOURS等等

workQueue:一个阻塞队列用于保存任务并将其传给works,如果需要取任务就从workQueue中取，works只是一个保存了所有任务的集合，一般来说有三种选择：

```
ArrayBlockingQueue;
LinkedBlockingQueue;//常用
SynchronousQueue;//常用
```

threadFactory:线程工厂，主要用来创建线程

handler:表示当拒绝处理任务时的策略，有以下四种取值;

```
ThreadPoolExecutor.AbortPolicy;//丢弃任务并抛出RejectedExecutionException异常
ThreadPoolExecutor.DiscardPolicy;//丢弃任务但是不跑出异常
ThreadPoolExecutor.DiscardOldestPolicy;//丢弃队列最前面的任务，然后重新尝试执行任务(重复此过程)
ThreadPoolExecutor.CallerRunsPolicy;//由调用线程处理该任务
```

由ThreadPoolExecutor类的源码可见它是继承了一个AbstractExecutorService

### AbstractExecutorService(暂不用探究其源码)

```
public abstract class AbstractExecutorService implements ExecutorService {
 
     
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) { };
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) { };
    public Future<?> submit(Runnable task) {};
    public <T> Future<T> submit(Runnable task, T result) { };
    public <T> Future<T> submit(Callable<T> task) { };
    private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks,
                            boolean timed, long nanos)
        throws InterruptedException, ExecutionException, TimeoutException {
    };
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
    };
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
    };
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
    };
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
    };
}
```

这里可以看到他是一个抽象类继承了ExecutorService,我们继续看下ExecutorService

### ExecutorService(暂不用探究其源码)

```
public interface ExecutorService extends Executor {
 
    void shutdown();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;
 
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

这里可以看到ExecutorService继承了Executor接口

### Executor

```
public interface Executor {
    void execute(Runnable command);
}
```

到这里大概可理出来ThreadPoolExecutor继承了AbstractExecutorService，AbstractExecutorService又继承了ExecutorService,最终ExecutorService又继承了Executor,即继承关系顶点为Executor

Executor只有一个接口execute参数为Runnable，所以executor的意思为执行传进去的任务

然后ExecutorService又继承了Executor接口并声明了一些方法submit(),invokeAll(),invokeAny(),shutdown()等

抽象类AbstractExecutorService实现了ExecutorService接口基本实现了ExecutorService中声明的所有方法

最后ThreadPoolExecutor继承了AbstractExecutorService,在ThreadPoolExecutor类中有几个非常重要的方法

```
execute(Runnable)
submit()
shutdown()
shutdownNow()
```

execute()方法实际上时Executor中声明的方法，在ThreadPoolExecutor进行了具体实现，这个方法是ThreadPoolExecutor类中的核心方法，**通过这个方法可以向线程池中提交一个任务让线程池去执行**

submit()方法实际上时Executor中声明的方法，在AbstractExecutorService中实现的ThreadPoolExecutor并没有对其进行重写，这个方法也是向线程池提交任务的但是他和execute()**不同的是它内部再execute()执行后利用Future来获取任务执行的结果，即submit可以返回任务执行的结果**

shutdown()和shutdownNow()是用来关闭线程池的

### 深入剖析线程池实现原理

#### 线程池状态

在ThreadPoolExecutor中定义了一个volatile变量，另外定义了几个static final变量表示线程池的各个状态

```
volatile int runState;
static final int RUNNING=0;
static final int SHUTDOWN=1;
static final int STOP=2;
static final int TERMINATED=3;
```

runState表示当前线程池的状态，他是一个volatile修饰的用来保证线程之间的可见性

下面的static final变量表示runState可能的几个取值

当创建线程池后初始化时线程池处于RUNNING状态

如果调用了shutdown()方法则线程池处于SHUTDOWN状态，此时线程池不能接受新的任务，它会等待所有任务执行完毕

如果调用shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会尝试终止正在执行的任务

当线程池处于SHUTDOWN或者STOP状态并且所有工作线程已经销毁，任务缓存队列(corePoolSize以外的线程放在任务缓存队列)已经清空或执行结束后，线程池被设置为TERMINATED状态

#### 任务的执行

在了解任务提交给线程池到任务完毕整个过程之前我们先来看一下ThreadPoolExecutor里的其他比较重要的成员变量

```
private final BlockingQueue<Runnable> workQueue;              //任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock();   //线程池的主要状态锁，对线程池状态（比如线程池大小
                                                              //、runState等）的改变都要使用这个锁
private final HashSet<Worker> workers = new HashSet<Worker>();  //用来存放工作集
 
private volatile long  keepAliveTime;    //线程没有执行任务时的存活时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
 
private volatile int   poolSize;       //线程池中当前的线程数
 
private volatile RejectedExecutionHandler handler; //任务拒绝策略
 
private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
 
private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数
 
private long completedTaskCount;   //用来记录已经执行完毕的任务个数
```

corePoolSize的理解：加入一个工厂原来有10个员工，谁有空闲就把工作给谁做如果10个员工都没空闲经理就多找4个人来做这样就能同时14个人干活，如果14个人都没有空闲就只有拒绝工作了，当这14个人当中有人有空闲时而新任务增长缓慢有部分工人可能长期接不到工作经历就会考虑辞退只保持原来的10个人，**在这里corePoolSize就是10个工人，maximumPoolSize就是14个工人，keepAliveTime就是最长可以空闲的时间，可以理解为maximumPoolSize比corePoolSize多的部分就是一个补救措施**



任务从提交到最终执行完毕经历了哪些过程：

在ThreadPoolExecutor类中最核心的任务提交方法就是execute()方法，submit()实际上还是执行的execute()方法，所以只需要研究execute方法的实现原理即可

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
        if (runState == RUNNING && workQueue.offer(command)) {//当前状态为RUNNING并且把当前任务放到任务缓存列表请求都成功时
            if (runState != RUNNING || poolSize == 0)//防止在任务添加到任务缓存队列中的同时别的线程调用shutdown()或者shutdownAll()时确保该任务一定放到任务缓存列表中
                ensureQueuedTaskHandled(command);
        }
        else if (!addIfUnderMaximumPoolSize(command))//如果该方法返回false(当前缓存任务列表已经打满了)那么执行reject()方法进行任务拒绝处理
            reject(command); // is shutdown or saturated
    }
}
```

上述代码中使用了两个方法addIfUnderCorePoolSize(command)和addIfUnderMaximumPoolSize(command),原理相差不大先来看看addIfUnderCorePoolSize(command)的源码：

```
private boolean addIfUnderCorePoolSize(Runnable firstTask) {
    Thread t = null;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (poolSize < corePoolSize && runState == RUNNING)
            t = addThread(firstTask);        //创建线程去执行firstTask任务   
        } finally {
        mainLock.unlock();
    }
    if (t == null)
        return false;
    t.start();
    return true;
}
```

这里可以看出他是判断当前线程数量是否大于corePoolSize，为什么在if的前一个条件判断了poolSize>=corePoolSize这里还要判断一次？

因为poolSize>=corePoolSize这个判断是没有加锁的如果在判断前刚好不满足这个条件这个时候别的线程又进来使得这个条件刚好满足了呢？所以这里需要进行第二次判断，第一个判断如果满足就无需执行第二个条件了加快效率

这里还得说一下addThread(firstTask)这个方法，他是利用参数的任务创建线程，在下面判断是否创建成功(poolSize>=corePoolSize或者runState!=RUNNING)时就不会成功下面会进行判断，为空则返回false,不为空就start()启动

这里来看一下addThread的源码：

```
private Thread addThread(Runnable firstTask) {
    Worker w = new Worker(firstTask);
    Thread t = threadFactory.newThread(w);  //利用线程池工厂拿出一个已经执行完任务的线程，执行任务   由于Work类继承了Runnable所以这里相当于new Thread(w);
    
    if (t != null) {
        w.thread = t;            //将创建的线程的引用赋值为w的成员变量       
        workers.add(w);//加入到任务集中去
        int nt = ++poolSize;     //当前线程数加1       
        if (nt > largestPoolSize)
            largestPoolSize = nt;
    }
    return t;
}
```

这里有个非常棒的点子是从线程池工厂中拿出一个已经执行完任务的线程再去执行该任务或者缓存任务列表中的任务，减少对任务分派线程管理的消耗

**这里还得说一下addIfUndercorePoolSize里面如果当前线程小于核心线程时，创建并启动的新建线程实际上是启动的Work内部的run方法**

Work内的run方法：

```
public void run() {
    try {
        Runnable task = firstTask;
        firstTask = null;
        while (task != null || (task = getTask()) != null) {
            runTask(task);//先执行Work构造方法传入的任务如果为null就去缓存任务列表中拿任务
            task = null;
        }
    } finally {
        workerDone(this);
    }
}
```





**这里重点说一下getTask()方法**

```
Runnable getTask() {
    for (;;) {//相当于while(true)
        try {
            int state = runState;
            if (state > SHUTDOWN)
                return null;
            Runnable r;
            if (state == SHUTDOWN)  //辅助清空任务缓存队列
                r = workQueue.poll();
            else if (poolSize > corePoolSize || allowCoreThreadTimeOut) //如果线程数大于核心池大小或者允许为核心池线程设置空闲时间，
                //则通过poll取任务，若等待一定的时间取不到任务，则返回null
                r = workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS);
            else
                r = workQueue.take();
            if (r != null)
                return r;//拿到任务之后就返回
            if (workerCanExit()) {    //如果没取到任务，即r为null，则判断当前的worker是否可以退出(当线程池处于STOP状态或者缓存任务队列已经为空或者允许为核心线程池设置空闲时间判定并且当前线程数大于一时就允许woker退出)
                if (runState >= SHUTDOWN) // Wake up others
                    interruptIdleWorkers();   //中断处于空闲状态的worker
                return null;
            }
            // Else retry
        } catch (InterruptedException ie) {
            // On interruption, re-check runState
        }
    }
}
```

getTask功能即从任务缓存队列中取任务，如果超过核心线程池数量或者允许核心池线程设置空闲时间过期机制那么就设定poll取任务如果一定时间取不到任务就返回null,虽然源码写的是for(;;)但是实际上不会产生死循环每次都会及时返回一个null或者一个被取出的任务

同样的addIfUndermaximumPoolSize也是相同的道理只不过判断对象从corePoolSize换成了maximumPoolSize



**总结线程的任务执行策略：**

如果当前线程池中的线程数目小于corePoolSize时通过addIfUnderCorePoolSize(Runnable)方法去创建一个线程来执行该任务

如果当前线程池中的线程数目>=corePoolSize,会尝试将其添加到任务缓存队列当中，若添加成功，则该任务会等待空闲线程将其取出去执行，若添加失败(一般来说是缓存任务队列满了的情况)，则会尝试创建新的线程来执行这个任务（addIfUndermaximumPoolSize方法）

如果当前线程池中的线程数目达到maximumPoolSize则会采取任务拒绝策略进行处理

如果线程池中的线程数量大于corePoolSize时如果某线程空闲时间超过keepAliveTime线程将被终止直到线程数量小于corePoolSize为止，如果允许核心池中的线程设置存活时间(allowCoreThreadTimeOut==true)的话那么就算是核心池内的线程超过keepAliveTime线程也会被终止

#### 线程池中的线程

默认情况下创建线程池之后线程池中是没有线程的，需要提交任务之后才会创建线程，在实际情况下如果需要线程池创建之后立即创建线程可以使用两个预创建方法：

```
prestartCoreThread():初始化一个核心线程
prestartAllCoreThreads():初始化所有核心线程
```

下面看一下两个方法的实现:

```
public boolean prestartCoreThread() {
    return addIfUnderCorePoolSize(null); //注意传进去的参数是null
}
 
public int prestartAllCoreThreads() {
    int n = 0;
    while (addIfUnderCorePoolSize(null))//注意传进去的参数是null
        ++n;
    return n;
}
```

addIfUnderCorePoolSize()如果传入的参数为null，并且当前线程数量小于核心线程池数量的时候会创建一个线程去执行这个空任务，实际上是执行的Work内的run()方法,Work内的run方法会首先去执行这个传进来的任务如果为空那么会在run里面的getTask()方法里面的r=workQueue.take()中进行获取任务即等待任务队列中有任务

#### 任务缓存队列及排队策略

前面我们多次提到了缓存任务队列workQueue,用来存放等待执行的任务

workQueue类型为BlockingQueue<Runnable>,通常可以取下面三种类型

```
ArrayBlockingQueue:基于数组的先进先出队列，此队列创建时必须指定大小，不常使用
LinkedBlockingQueue:基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE
synchronousQueue:这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行
```

### 使用示例

```
class WebsocketApplicationTests {
    public static void main(String[] args){
        ThreadPoolExecutor executor=new ThreadPoolExecutor(10,15,5,TimeUnit.MINUTES,new ArrayBlockingQueue<Runnable>(10));
        Run run=new Run();
        for(int i=0;i<25;i++)
        {
            executor.execute(new Run());
            System.out.println("当前有"+executor.getQueue().size()+"个待执行任务,"+"当前有"+executor.getPoolSize()+"个线程在内,"+"已经执行完的任务个数为"+executor.getCompletedTaskCount());
        }
        System.out.println("当前有"+executor.getQueue().size()+"个待执行任务,"+"当前有"+executor.getPoolSize()+"个线程在内,"+"已经执行完的任务个数为"+executor.getCompletedTaskCount());
        executor.shutdown();
        System.out.println(executor.getRejectedExecutionHandler());
    }
}
@Data
class Run implements Runnable
{
    public synchronized void method1(){
        System.out.println(Thread.currentThread().getName()+"method1");
    }

    public synchronized void method2(){
        System.out.println(Thread.currentThread().getName()+"method2");
    }

    @Override
    public void run() {
        try{
            Thread.sleep(4000);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        method1();
    }
}
```

这里最多可以同时打25个任务上去，由于我的workQueue设置为10，maximumPoolSize设置的15所以最多能同时打25个任务上去，这里没有写拒绝机制验证了之后，**默认的拒绝机制为ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。**



不过在实际项目中不提倡直接使用ThreadPoolExecutor,而是使用Executor的几个静态方法来创建线程池：

```
Executors.newCachedThreadPool();        //创建一个缓冲池，缓冲池容量大小为Integer.MAX_VALUE
Executors.newSingleThreadExecutor();   //创建容量为1的缓冲池
Executors.newFixedThreadPool(int);    //创建固定容量大小的缓冲池
```

这三个静态方法的具体实现

```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

**Question:**

这个个人觉得这三种静态方法的实现把corePoolSize和maximumPoolSize都固定了限制了变化性，有谁能说一下为什么推荐这种形式来创建线程池而不直接通过ThreadPoolExecutor来创建呢?



