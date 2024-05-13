## 多线程带来的问题
在多线程编程中，为了保证数据的一致性和线程的正确交互，有三个关键的概念：原子性、可见性和活跃性。下面我会分别解释这三个概念，并提供一些现实中的例子来帮助理解。
### 原子性（Atomicity）
原子性意味着在一个操作中的所有步骤都是不可分割的，要么全部完成，要么全部不发生，不会出现中间状态。这在多线程环境中非常重要，因为它防止了多个线程同时修改同一数据时可能导致数据不一致的问题。
**例子：**
在银行系统中，当你从一个账户向另一个账户转账时，这个操作需要两个步骤：1) 从源账户扣款；2) 向目标账户加款。这两个步骤必须作为一个原子操作完成，这样就可以确保不会只完成一个步骤（例如，钱被扣除了但没有被存入目标账户）。
### 可见性（Visibility）
可见性确保一个线程对共享变量的修改能够被其他线程看到，这是在多核处理器系统中尤其重要。没有适当的同步措施，一个线程的更改可能仅仅在本地缓存中进行，而不是写回主内存，导致其他线程看不到更新。
**例子：**
在一个多线程应用程序中，如果一个线程更新了共享变量的值，而这个更新没有及时反映到主内存中，其他依赖该变量的线程可能会读取到旧值，从而导致错误的计算或决策。
### 活跃性（Liveness）
活跃性问题描述的是一个系统能够继续执行，进展其工作的能力。在多线程环境中，常见的活跃性问题包括死锁、活锁和饥饿。

- **死锁**：多个线程相互等待对方持有的资源，导致都无法继续执行。
- **活锁**：线程虽然在执行，但操作却没有向前推进（例如，两个线程不停地尝试重试但一直失败）。
- **饥饿**：一个或多个线程因为无法获得必要的资源而无法进行处理。

**例子：**
假设有两个线程，每个线程都需要两把锁（Lock1 和 Lock2）来完成其任务。如果线程A持有Lock1并等待Lock2，同时线程B持有Lock2并等待Lock1， 这就形成了一个典型的**死锁**情况。在这种场景中，两个线程互相等待对方释放锁，但都不放弃自己已经持有的锁，导致两者都无法继续执行，从而使程序无法正常运行。
## happens-before
在并发编程中，**happens-before** 关系是一个非常重要的概念，用来确保内存的可见性和有序性，防止指令重排带来的问题。**happens-before** 原则定义了一种规则，用于确定在一个线程中对数据的修改如何对其他线程可见，即在什么条件下一个线程的操作结果对其他线程是可见的。
### happens-before的基本规则包括：

1. **程序顺序规则**：在同一个线程中，按照程序代码顺序，前面的操作 happens-before 后面的操作。
2. **监视器锁规则**：对一个锁的解锁 happens-before 随后对这个锁的加锁。
3. **volatile变量规则**：对一个volatile字段的写操作 happens-before 任何后续对这个volatile字段的读操作。
4. **传递性**：如果操作A happens-before 操作B，且操作B happens-before 操作C，则操作A happens-before 操作C。
### 解释
假设有一个接力赛，分为两个阶段：

1. **第一阶段**：选手A跑步并传递接力棒给选手B。
2. **第二阶段**：选手B接到接力棒后开始跑。

在这个例子中：

- **选手A的跑步和传递接力棒** happens-before **选手B接棒和开始跑**。
#### 概念对应

- **选手A跑步和传递接力棒** 类似于线程中的某个操作（比如修改一个变量的值）。
- **选手B接棒和开始跑** 类似于另一个线程中的后续操作（比如读取之前线程修改的变量值）。

在接力赛中，选手A传递接力棒给选手B是一个关键事件，它保证了选手B不能在接到棒之前开始跑。这与 happens-before 原则相似，后者确保在一个线程中对数据的修改在另一个线程中变得可见，必须通过某种形式的通信（比如通过锁、或者写入读取一个 volatile 变量）。
在这个接力赛的比喻中：

- 如果选手A没有正确传递接力棒给选手B（即没有正确使用同步机制），那么选手B可能无法按预期开始比赛（即可能读取到旧的数据，导致错误）。
- 正确的传递接力棒（合适的同步操作）确保了比赛的顺利进行，类似于多线程程序中正确的内存可见性和操作顺序。
## volatile 关键字
### volatile 会禁止指令重排
> 注：不了解指令重排可以学习该文章（[二哥进阶之路·JMM内存模型](https://javabetter.cn/thread/jmm.html)）

当我们使用 volatile 关键字来修饰一个变量时，Java 内存模型会插入内存屏障（一个处理器指令，可以对 CPU 或编译器重排序做出约束）来确保以下两点：

- 写屏障（Write Barrier）：当一个 volatile 变量被写入时，写屏障确保在该屏障之前的所有变量的写入操作都提交到主内存。
- 读屏障（Read Barrier）：当读取一个 volatile 变量时，读屏障确保在该屏障之后的所有读操作都从主内存中读取。

换句话说：

- 当程序执行到 volatile 变量的读操作或者写操作时，在其前面操作的更改肯定已经全部进行，且结果对后面的操作可见；在其后面的操作肯定还没有进行；
- 在进行指令优化时，不能将 volatile 变量的语句放在其后面执行，也不能把 volatile 变量后面的语句放到其前面执行
- 也就是说，执行到 volatile 变量时，其前面的所有语句都必须执行完，后面所有得语句都未执行。且前面语句的结果对 volatile 变量及其后面语句可见。
#### 演示案例
```java
public class ReorderExample {
  int a = 0;
  boolean flag = false;
  public void writer() {
      a = 1;                   //1
      flag = true;             //2
  }
  Public void reader() {
      if (flag) {                //3
          int i =  a * a;        //4
          System.out.println(i);
      }
  }
}

```
因为重排序影响，所以最终的输出可能是 0，如果引入 volatile，我们再看一下代码：
```java
public class ReorderExample {
  int a = 0;
  boolean volatile flag = false;
    
  public void writer() {
      a = 1;                   //1
      flag = true;             //2
  }

  Public void reader() {
      if (flag) {                //3
          int i =  a * a;        //4
          System.out.println(i);
      }
  }

}

```
这时候，volatile 会禁止指令重排序，这个过程建立在 happens before 关系的基础上：

1. 根据程序次序规则，1 happens before 2;  3 happens before 4。
2. 根据 volatile 规则，2 happens before 3。
3. 根据 happens before 的传递性规则，1 happens before 4。

上述 happens before 关系的图形化表现形式如下：
![](https://cdn.nlark.com/yuque/0/2024/jpeg/22796888/1715599471799-13b260e3-b916-4a0e-af9c-9012813da237.jpeg#averageHue=%23f9f9f9&clientId=uceb37488-c87a-4&from=paste&id=uf656895a&originHeight=784&originWidth=1080&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ufdd20518-647b-4ddc-9e01-bd5e84c1c13&title=)
在上图中，每一个箭头链接的两个节点，代表了一个 happens before 关系:

- 黑色箭头表示程序顺序规则；
- 橙色箭头表示 volatile 规则；
- 蓝色箭头表示组合这些规则后提供的 happens before 保证。

这里 A 线程写一个 volatile 变量后，B 线程读同一个 volatile 变量。A 线程在写 volatile 变量之前所有可见的共享变量，在 B 线程读同一个 volatile 变量后，将立即变得对 B 线程可见。
### volatile 不适应场景
#### 需要原子性的操作
当多个操作需要作为单个不可中断的单位执行时，**volatile** 关键字是不够的。例如，对于增加操作（如 **count++**），这实际上是三个独立的操作：读取 **count** 的值，增加这个值，然后写回新值。即使 **count** 被声明为 **volatile**，这三个操作步骤之间仍然可能被其他线程的操作打断，导致竞争条件和数据不一致。

- 在这个程序中，我们启动了两个线程，每个线程都尝试将计数器 **count** 增加1000次。理想情况下，因为总共增加了2000次，所以最终的 **count** 值应该是2000。然而，你会发现实际运行的结果可能会小于2000。
- 问题在于 **count++** 实际上是由三个独立的步骤组成：读取 **count** 的当前值，增加这个值，然后将新值写回 **count**。这三个步骤并不是原子性的，**volatile** 只能保证每个单独操作的可见性和部分顺序性，但不能保证整个序列的原子性。因此，当两个线程同时执行 **increment()** 方法时，它们可能会读到相同的 **count** 值，导致一些增加操作被覆盖，最终结果就会出现不一致。
- 上方问题可以通过`sychronized、AtomicInteger、Lock`解决
```java
public class Counter {
    private volatile int count = 0;

    public void increment() {
        count++;  // 这不是一个原子操作
    }

    public int getCount() {
        return count;
    }

    public static void main(String[] args) throws InterruptedException {
        final Counter counter = new Counter();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        t1.start();
        t2.start();

        // 等待两个线程都执行完毕
        t1.join();
        t2.join();

        // 输出最终的计数值
        System.out.println("Final count: " + counter.getCount());
    }
}

```
### 小结
volatile 可以保证线程可见性且提供了一定的有序性，但是无法保证原子性。在 JVM 底层 volatile 是采用“内存屏障”来实现的。
观察加入 volatile 关键字和没有加入 volatile 关键字时所生成的汇编代码就能发现，加入 volatile 关键字时，会多出一个 lock 前缀指令，lock 前缀指令实际上相当于一个内存屏障（也称内存栅栏），内存屏障会提供 3 个功能：

- 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
- 它会强制将对缓存的修改操作立即写入主存；
- 如果是写操作，它会导致其他 CPU 中对应的缓存行无效。
## sychronized 关键字
在 Java 中，关键字 `synchronized` 可以保证在同一个时刻，只有一个线程可以执行某个方法或者某个代码块(主要是对方法或者代码块中存在共享数据的操作)，同时我们还应该注意到 `synchronized` 的另外一个重要的作用，`synchronized` 可保证一个线程的变化 (主要是共享数据的变化) 被其他线程所看到（保证可见性，完全可以替代 volatile 功能）。
`synchronized` 关键字最主要有以下 3 种应用方式：

- 同步方法，为当前对象（`this`）加锁，进入同步代码前要获得当前对象的锁；
- 同步静态方法，为当前类加锁（锁的是 `Class对象`），进入同步代码前要获得当前类的锁；
- 同步代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。
### sychronized 同步方法
通过在方法声明中加入 `synchronized` 关键字，可以保证在任意时刻，只有一个线程能执行该方法；
> 注意：一个对象只有一把锁，当一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，所以无法访问该对象的其他 synchronized 方法，但是其他线程还是可以访问该对象的其他非 synchronized 方法。

```java
public class SynchronizedDemo01 implements Runnable{
    static int i = 0;
    public synchronized void increase(){
        i++;
    }

    public static void main(String[] args) throws InterruptedException {
        SynchronizedDemo01 demo = new SynchronizedDemo01();
        Thread t1 = new Thread(demo);
        Thread t2 = new Thread(demo);

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("input result：" + i);
    }

    @Override
    public void run() {
        for (int j = 0; j < 10000; j++) {
            increase();
        }
    }
}

/**
* 输出结果
* input result：20000
*/
```
但是，当两个线程分别访问两个不同对象的 synchronized 方法时，每个线程都会锁定它们各自正在访问的对象。由于这些对象（及其锁）是独立的，因此这两个线程可以并发地执行各自的方法，而不会因为等待对方释放锁而阻塞。

1. **synchronized 方法的锁定对象：** 当一个方法被声明为 **synchronized**，它需要锁定当前的对象实例（如果是实例方法）或者锁定该类的 Class 对象（如果是静态方法）。这意味着任何线程要执行这个方法，必须先获得对应对象的锁。
2. **线程 A 访问 obj1 的 synchronized 方法 f1：** 线程 A 在调用 obj1 的 synchronized 方法 f1 时，必须先获得 obj1 的锁。只有获得了 obj1 的锁，线程 A 才能执行 f1 方法。
3. **线程 B 访问 obj2 的 synchronized 方法 f2：** 同时，线程 B 在调用 obj2 的 synchronized 方法 f2 时，需要获得 obj2 的锁。这个锁与 obj1 的锁是独立的。
4. **锁的独立性：** 因为 obj1 和 obj2 是两个不同的对象，它们的锁是互不相关的。线程 A 持有 obj1 的锁并不影响线程 B 获得 obj2 的锁。因此，这两个线程可以同时执行各自的 synchronized 方法，互不干扰。
```java
public class SynchronizedDemo01 implements Runnable{
    static int i = 0;
    public synchronized void increase(){
        i++;
    }

    public static void main(String[] args) throws InterruptedException {
        // SynchronizedDemo01 demo = new SynchronizedDemo01();
        // Thread t1 = new Thread(demo);
        // Thread t2 = new Thread(demo);

        Thread t1 = new Thread(new SynchronizedDemo01());
        Thread t2 = new Thread(new SynchronizedDemo01());

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("input result：" + i);
    }

    @Override
    public void run() {
        for (int j = 0; j < 10000; j++) {
            increase();
        }
    }
}

/**
* 输出结果
* input result：16694
*/
```
### synchronized 同步静态方法
当 `synchronized` 同步静态方法时，锁的是当前类的 Class 对象，不属于某个对象。当前类的 Class 对象锁被获取，不影响实例对象锁的获取，两者互不影响，本质上是 this 和 Class 的不同。
由于静态成员变量不专属于任何一个对象，因此通过 Class 锁可以控制静态成员变量的并发操作。
需要注意的是如果线程 A 调用了一个对象的非静态 `synchronized` 方法，线程 B 需要调用这个对象所属类的静态 `synchronized` 方法，是不会发生互斥的，因为访问静态 `synchronized` 方法占用的锁是当前类的 Class 对象，而访问非静态 `synchronized` 方法占用的锁是当前对象（this）的锁。

1. **静态 synchronized 方法：** 锁定的是类的 Class 对象，即所有实例共享的锁。因为静态方法属于类，而不属于任何特定的实例。
2. **非静态 synchronized 方法：** 锁定的是调用该方法的实例对象，即每个实例有自己的锁。
```java
public class SynchronizedDemo02 implements Runnable{
    static int i = 0;
    public static synchronized void increase(){
        i++;
    }

    public synchronized void increaseObj(){
        i++;
    }

    @Override
    public void run() {
        for (int j = 0; j < 10000; j++) {
            increase(); //同步锁定当前类
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new SynchronizedDemo02());
        Thread t2 = new Thread(new SynchronizedDemo02());

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("input result：" + i);
    }
}
```
### synchronized 同步代码块
某些情况下，当我们编写的方法代码量比较多，存在一些比较耗时的操作，而需要同步的代码块只有一小部分，如果直接对整个方法进行同步，可能会得不偿失，此时我们可以使用同步代码块的方式对需要同步的代码进行包裹。
```java
public class SynchronizedDemo03 implements Runnable{
    static SynchronizedDemo03 instance = new SynchronizedDemo03();

    static int i = 0;

    @Override
    public void run() {
        synchronized (instance){
            for (int j = 0; j < 10000; j++) {
                i++;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("instance i = " + i);
    }
}
```
当然除了用 instance 作为对象外，我们还可以使用 this 对象(代表当前实例)或者当前类的 Class 对象作为锁：

- 当 **synchronized** 关键字使用 **this** 作为锁对象时，它锁定的是当前实例。这意味着任何尝试进入任何以 **this** 为锁的**synchronized** 代码块的线程必须等到没有其他线程持有这个锁。
- 当使用类的 Class 对象作为锁时，即使用 **ClassName.class** 作为锁对象，锁定的是类的所有实例。
```java
synchronized (this){
    for (int j = 0; j < 10000; j++) {
        i++;
    }
}

synchronized (SynchronizedDemo.class){
    for (int j = 0; j < 10000; j++) {
        i++;
    }
}
```
### synchronized 属于可重入锁
**synchronized** 关键字确实实现了一个可重入锁的功能。所谓的“可重入锁”，意味着一个线程可以多次获得同一把锁。这是非常重要的一个特性，因为它解决了递归调用或者一个类中多个 synchronized 方法相互调用时可能产生的死锁问题。
假设有一个类，它有两个 synchronized 方法：**outer** 和 **inner**。**outer** 方法在执行过程中会调用 **inner** 方法。如果 synchronized 不是可重入的，那么在 **outer** 方法持有锁的情况下调用 **inner** 方法时，会因为尝试再次获取已经被持有的锁而导致死锁。但由于 synchronized 是可重入的，这种调用是允许且安全的。
```java
public class ReentrantExample {
    public synchronized void outer() {
        System.out.println("Inside outer method");
        inner();  // 调用另一个 synchronized 方法
    }

    public synchronized void inner() {
        System.out.println("Inside inner method");
    }

    public static void main(String[] args) {
        ReentrantExample example = new ReentrantExample();
        example.outer();
    }
}

```


> 转载：
>
> 1. [二哥进阶之路 · 多线程带来了哪些问题？](https://javabetter.cn/thread/thread-bring-some-problem.html)
> 2. [二哥进阶之路 · volatile关键字](https://javabetter.cn/thread/volatile.html)
> 3. [二哥进阶之路 · synchronized关键字](https://javabetter.cn/thread/synchronized-1.html)

