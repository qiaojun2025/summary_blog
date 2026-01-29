

# 线程基础

## 线程方法

### Wait()

当一个线程调用一个共享变量的 wait（）方法时，该调用线程会被阻塞挂起，直到发生下面几件事情之一才返回： 
(1）其他线程调用了该共享对象的 notify（）或者 notifyAll（） 方法； 
(2）其他线程调用了该线程的 interrupt（）方法， 该线程抛出 InterruptedException 异常返回。


那么一个线程如何才能获取一个共享变量的监视器锁呢？
(1）执行 synchronized 同步代码块时，使用该共享变量作为参数。
(2）调用该共享变量的方法，并且该方法使用了 synchronized 修饰。

如果调用 wait（）方法的线程没有事先获取该对象的监视器锁，则调用 wait（）方法时调用线程会抛出 IllegalMonitorStateException 异常。

另外需要注意的是，一个线程可以从挂起状态变为可以运行状态（也就是被唤醒），即使该线程没有被其他线程调用 notify（）、notifyAll（）方法进行通知，或者被中断，或者等待超时，这就是所谓的虚假唤醒。

当前线程调用共享变量的 wait（）方法后只会释放当前共享变量上的锁，如果当前线程还持有其他共享变量的锁，则这些锁是不会被释放的。 

当一个线程调用共享对象的 wait（）方法被阻塞挂起后，如果其他线程中断了该线程，则该线程会抛出 InterruptedException 异常并返回。

### notify()

一个线程调用共享对象的 notify（）方法后，会唤醒一个在该共享变量上调用 wait 系列方法后被挂起的线程。 
一个共享变量上可能会有多个线程在等待，具体唤醒哪个等待的线 程是随机的。
此外，被唤醒的线程不能马上从 wait 方法返回并继续执行，它必须在获取了共享对象的监视器锁后才可以返回

### notifyAll()

notifyAll()方法则会唤醒所有在该共享变量上由于调用 wait 系列方法而被挂起的线程。



## 线程安全问题

所谓共享资源，就是说该资源被多个线程所持有或者说多个线程都可以去访问该资源。
线程安全问题是指当多个线程同时读写一个共享资源并且没有任何同步措施时，导致出现脏数据或者其他不可预见的结果的问题

是不是说多个线程共享了资源，当它们都去访问这个共享资源时就会产生线程安全问题呢？
答案是否定的，如果多个线程都只是读取共享资源，而不去修改，那么就不会存在线程安全问题，只有当至少一个线程修改共享资源时才会存在线程安全问题。

那么如何来解决这个问题呢？这就需要在线程访问共享变量时进行适当的同步，在 Java 中最常见的是使用关键宇 synchronized 进行同步，下面会有具体介绍。



## Java 中的 synchronized 关键字

synchronized 块是 Java 提供的一种原子性内置锁， Java 中的每个对象都可以把它当作一个同步锁来使用 ， 这些 Java 内置的使用者看不到的锁被称为内部锁，也叫作监视器锁。 线程的执行代码在进入 synchronized 代码块前会自动获取内部锁，这时候其他线程访问该同步代码块时会被阻塞挂起。拿到内部锁的线程会在正常退出同步代码块或者抛出异常后或者在同步块内调用了该内置锁资源的 wait 系列方法时释放该内置锁。 

进入 synchronized 块的内存语义是把在 synchronized 块内使用到的变量从线程的工作内存中清除，这样在 synchronized 块内使用到该变量时就不会从线程的工作内存中获取，而是直接从主内存中获取。退出 synchronized 块的内存语义是把在 synchronized 块内对共享变量的修改刷新到主内存。

使用 synchronized 关键宇的确可以实现线程安全性， 即内存可见性和原子性，但是 synchronized 是独占锁，没有获取内部锁的线程会被阻塞掉，而这里的 getCount 方法只是 读操作，多个线程同时调用不会存在线程安全问题。 但是加了关键宇 synchronized 后，同 一时间就只能有一个线程可以调用，这显然大大降低了并发性。  你也许会间，既然是只读 操作，那为何不去掉 getCount 方法上的 synchronized 关键字呢？其实是不能去掉的，别忘 了这里要靠 synchronized 来实现 value 的 内存可见性。

那么有没有更好的实现呢？答案是 肯定的，下面将讲到的在内部使用非阻塞 CAS 算法实现的原子性操作类 AtomicLong 就是 一个不错的选择。



## Java 中共享变量的内存可见性问题

Java 内存模型规定，将所有的变量都存放在主内存中，当线程使用变量时，会把主内存里面的变量复制到自己的工作空间或者叫作工作内存，线程读写变量时操作的是自己工作内存中的变量。 
当一个线程操作共享变量时， 它首先从主内存复制共享变量到自己的工作内存， 然后对工作内存里的变量进行处理，处理完后将变量值更新到主内存。

那么如何解决共享变量内存不可见问题？使用 Java 中的 volatile 关键字就可以解决这个问题



##  Java 中的 volatile 关键字

 volatile 关键字可以确保对一个变量的更新对其他线程马上可见。 当一个变量被声明为 volatile 时，线程在写入变量时不会把值缓存在寄存器或者其他地方，而是会把值刷新回主内存。 当其他线程读取该共享变量时，会从主内存重新获取最新值，而不是使用当前线程的工作内存中的值。 

 volatile 虽然提供了可见性保证，但并不保证操作的原子性。

## Java 中的 CAS 操作

在 Java 中，锁在并发处理中占据了一席之地，但是使用锁有一个不好的地方，就是当一个线程没有获取到锁时会被阻塞挂起，这会导致线程上下文的切换和重新调度开销。Java提供了非阻塞的 volatile 关键字来解决共享变量的可见性问题，这在一定程度上弥补了锁带来的开销问题，但是 volatile 只能保证共享变量的可见性，不能解决读-改-写等的原子性问题。 

 CAS 即 Compare and Swap，其是 JDK 提供的非阻塞原子性操作， 它通过硬件保证了比较更新操作的原子性。 JDK 里面的 Unsafe 类提供了一系列的 compareAndSwap方法



# JUC

JUC即JDK8新增并发包，包路径为java.util.concurrent

## JUC 原子操作类

JUC 并发包中包含有 Atomiclnteger、 AtomicLong 和 AtomicBoolean 等原子性操作类， 它们的原理类似，本章讲解 AtomicLong 类。 AtomicLong 是原子性递增或者递减类，其内部使用 Unsafe 来实现

JDK 8 新增的原子操作类 LongAdder
前面讲过， AtomicLong 通过 CAS 提供了非阻塞的原子性操作，相 比使用阻塞算法的 同步器来说它的性能己经很好了，但是 JDK 开发组并不满足于此。 使用 AtomicLong 时，在高并发下大量线程会同时去竞争更新同→个原子变量，但是由于同时只有一个线程的 CAS 操作会成功，这就造成了大量线程竞争失败后，会通过无限循环不断进行自旋尝试 CAS 的操作， 而这会白白浪费 CPU 资源。
因此 JDK 8 新增了一个原子性递增或者递减类 LongAdder 用来克服在高并发下使用 AtomicLong 的缺点。 既然 AtomicLong 的性能瓶颈是由于过多线程同时去竞争一个变量的 更新而产生的，那么如果把一个变量分解为多个变量，让同样多的线程去竞争多个资源，是不是就解决了性能问题？是的，LongAdder 就是这个思路。 

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicIntegerTest implements Runnable{

    static AtomicInteger atomicInteger = new AtomicInteger(0);

    static int commonInteger = 0;

    public void addAtomicInteger() {
        atomicInteger.getAndIncrement();
    }

    public void addCommonInteger() {
        commonInteger++;
    }

    @Override
    public void run() {
        //可以调大10000看效果更明显
        for (int i = 0; i < 10000; i++) {
            addAtomicInteger();
            addCommonInteger();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicIntegerTest atomicIntegerTest = new AtomicIntegerTest();
        Thread thread1 = new Thread(atomicIntegerTest);
        Thread thread2 = new Thread(atomicIntegerTest);
        thread1.start();
        thread2.start();
        //join()方法是为了让main主线程等待thread1、thread2两个子线程执行完毕
        thread1.join();
        thread2.join();
        System.out.println("AtomicInteger add result = " + atomicInteger.get());
        System.out.println("CommonInteger add result = " + commonInteger);
    }
}

```

**在高并发情况下，LongAdder(累加器)比AtomicLong原子操作效率更高**，LongAdder累加器是java8新加入的，参考以下压测代码：

```JAVA
public class AtomicLongTest implements Runnable {
 
    private static AtomicLong atomicLong = new AtomicLong(0);
 
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            atomicLong.incrementAndGet();
        }
    }
 
    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(30);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            es.submit(new AtomicLongTest());
        }
        es.shutdown();
        //保证任务全部执行完
        while (!es.isTerminated()) { }
        long end = System.currentTimeMillis();
        System.out.println("AtomicLong add 耗时=" + (end - start));
        System.out.println("AtomicLong add result=" + atomicLong.get());
    }
}

AtomicLong add 耗时=1893
AtomicLong add result=100000000
```

```JAVA
public class LongAdderTest implements Runnable {
 
    private static LongAdder longAdder = new LongAdder();
 
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            longAdder.increment();
        }
    }
 
    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(30);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            es.submit(new LongAdderTest());
        }
        es.shutdown();
        //保证任务全部执行完
        while (!es.isTerminated()) {
        }
        long end = System.currentTimeMillis();
        System.out.println("LongAdder add 耗时=" + (end - start));
        System.out.println("LongAdder add result=" + longAdder.sum());
    }
}

LongAdder add 耗时=245
LongAdder add result=100000000
```



## JUC 并发List

### CopyOnWriteArrayList

并发包中的并发 List只有CopyOnWriteArrayList。 
CopyOnWriteArrayList 是一个线程 安全的 ArrayList，对其进行的修改操作都是在底层的一个复制的数组（快照）上进行的， 也就是使用了写时复制策略。
每个 CopyOnWriteArrayList 对象里面有一个 array 数组对象用来存放具体元素， ReentrantLock 独占锁对象用来保证同时只有－个线程进行修改。 这里只要记得 ReentrantLock 是独占锁，同时只有一个线程可以获取就可以了
CopyOnWriteArrayList 使用写时复制的策略来保证 list 的一致性，而获取一修改一写入三步操作并不是原子性的，所以在增删改的过程中都使用了独占锁，来保证在某个时间只有一个线程能对 list 数组进行修改。 另外 CopyOnWriteAnayList 提供了弱一致性的法代器，从而保证在获取迭代器后，其他线程对 list 的修改是不可见的，迭代器遍历的数组是一个快照。



## JUC 并发队列

JDK 中提供了一系列场景的并发安全队列。总的来说，按照实现方式的不同可分为阻塞队列和非阻塞队列，前者使用锁实现，而后者则使用 CAS 非阻塞算法实现。

### ConcurrentlinkedQueue

ConcurrentLinkedQueue 是线程安全的无界非阻塞队列，其底层数据结构使用单向链表实现，对于入队和出队操作使用 CAS 来实现线程安全。 

### LinkedBlockingQueue

使用独占锁实现的阻塞队列 LinkedB!ockingQueue
LinkedBlockingQueue 的内部是通过单向链表实现的，使用头、尾节点来进行入队和 出队操作，也就是入队操作都是对尾节点进行操作，出队操作都是对头节点进行操作。
对头、尾节点的操作分别使用了单独的独占锁从而保证了原子性， 所以出队和入队操作是可以同时进行的。 另外对头、 尾节点的独占锁都配备了一个条件队 列，用来存放被阻塞的线程，并结合入队、出队操作实现了一个生产消费模型。

### ArrayBlockingQueue

使用有界数组方式实现的阻塞队列 ArrayBlockingQueue
ArrayBlockingQueue 通过使用全局独占锁实现了同时只能有一个线 程进行入队或者出队操作，这个锁的粒度比较大，有点类似于在方法上添加 synchronized 的意思。
另 外，相比 LinkedBlockingQueue, Array B lockingQueue 的 size 操作的结果是精确的， 因为计算前加了全局锁。

### PriorityBlockingQueue

Priority B lockingQueue 是带优先级的无界阻塞队列，每次出队都返回优先级最高或者 最低的元素。其内部是使用平衡二叉树堆实现的，所以直接遍历队列元素不保证有序。默 认使用对象的 compareTo 方法提供比较规则
PriorityBlockingQueue 队列在内部使用二叉树堆维护元素优先级，使用数组作为元素 存储的数据结构，这个数组是可扩容的。当当前元素个数＞＝最大容量时会通过 CAS 算法 扩容，出队时始终保证出队的元素是堆树的根节点，而不是在队列里面停留时间最长的元 素。使用元素的 compareTo 方法提供默认的元素优先级比较规则，用户可以自定义优先级 的比较规则。
如图 7-33 所示， PriorityBlockingQueue 类似于 ArrayBlockingQueue，在内部使用一
第7章 Java并发包中并发队列原理剖析 I 217 
个独占锁来控制同时只有一个线程可以进行入队和出队操作。另外，前者只使用了一个 notEmpty 条件变量而没有使用 notFull，这是因为前者是无界队列，执行 put 操作时永远不 会处于 await 状态，所以也不需要被唤醒。而 take 方法是阻塞方法，并且是可被中断的。 当需要存放有优先级的元素时该队列比较有用 。

### DelayQueue

DelayQueue 并发队列是一个无界阻塞延迟队列，队列中的每个元素都有个过期时间， 当从队列获取元素时，只有过期元素才会出队列。队列头元素是最快要过期的元素。
本节讲解了 DelayQueue 队列（见图 7-34），其内部使用 PriorityQueue 存放数据，使 用 ReentrantLock 实现线程同步。另外队列里面的元素要实现 Delayed 接口，其中一个是 获取当前元素到过期时间剩余时间的接口，在出队时判断元素是否过期了，一个是元素之 间比较的接口 ，因为这是一个有优先级的队列。



## JUC 线程池ThreadPoolExecutor

线程池主要解决两个问题： 
一是当执行大量异步任务时线程池能够提供较好的性能。在不使用线程池时，每当需要执行异步任务时直接 new 一个线程来运行，而线程的创建和销毁是需要开销的。 线程池里面的线程是可复用的，不需要每次执行异步任务时都重新创建和销毁线程。
二是线程池提供了一种资源限制和管理的手段，比如可以限制线程的个数，动态新增线程等

