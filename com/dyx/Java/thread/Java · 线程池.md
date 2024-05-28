## 什么是线程池？
线程池其实是一种池化的技术实现，池化技术的核心思想就是实现资源的复用，避免资源的重复创建和销毁带来的性能开销。线程池可以管理一堆线程，让线程执行完任务之后不进行销毁，而是继续去处理其它线程已经提交的任务。
使用线程池的好处

- 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。
## 线程池的构造
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

1. **corePoolSize**（核心线程数）
   - 核心线程数是指线程池中始终保持的线程数量，即使这些线程处于空闲状态。当提交任务时，如果当前运行的线程少于核心线程数，则会创建新线程来处理任务，即使其他核心线程处于空闲状态。
2. **maximumPoolSize**（最大线程数）
   - 最大线程数是指线程池中允许创建的最大线程数量。当任务队列已满并且运行的线程数少于最大线程数时，线程池会创建新线程来处理任务。
3. **keepAliveTime**（线程空闲保持时间）
   - 当线程池中的线程数量超过核心线程数时，多余的空闲线程的存活时间。超过这个时间，空闲线程会被终止和移除。该参数适用于非核心线程，即当线程池中的线程数量超过核心线程数时才会起作用。
4. **unit**（时间单位）
   - 用于指定 **keepAliveTime** 参数的时间单位。常用的单位有 **TimeUnit.SECONDS**、**TimeUnit.MILLISECONDS** 等。
5. **workQueue**（任务队列）
   - 用于存储等待执行的任务的队列。常见的队列类型有：
      - **ArrayBlockingQueue**：一个有界的阻塞队列。
      - **LinkedBlockingQueue**：一个无界的阻塞队列（实际上容量为 **Integer.MAX_VALUE**）。
      - **SynchronousQueue**：一个不存储元素的阻塞队列，每个插入操作必须等待一个对应的删除操作。
      - **PriorityBlockingQueue**：一个带优先级的无界阻塞队列。
6. **threadFactory**（线程工厂）
   - 用于创建新线程的工厂。可以通过实现 **ThreadFactory** 接口来自定义线程创建的方式。例如，可以为线程设置名称或设置为守护线程。
7. **handler**（拒绝策略）
   - 当线程池和队列都已满，且无法接受新的任务时，所采取的处理策略。常见的拒绝策略包括：
      - **AbortPolicy**：直接抛出 **RejectedExecutionException** 异常。
      - **CallerRunsPolicy**：由调用线程执行任务。
      - **DiscardPolicy**：丢弃无法处理的任务。
      - **DiscardOldestPolicy**：丢弃队列中最旧的任务，然后尝试重新提交任务。
## 线程池复用原理

1. **线程创建与初始化**：
   - 线程池在初始化时，会创建一定数量的核心线程（由 **corePoolSize** 指定），这些线程会一直存在，即使它们处于空闲状态。
2. **任务提交**：
   - 当有任务提交到线程池时，任务会被封装成一个 **Runnable** 对象，并放入任务队列中等待执行。
3. **任务调度**：
   - 如果有空闲的核心线程，这些线程会从任务队列中取出任务并执行。
   - 如果所有核心线程都在忙碌，并且任务队列未满，新任务会被放入任务队列中。
   - 如果任务队列已满且运行的线程数小于最大线程数，线程池会创建新的非核心线程来处理任务。
   - 如果任务队列已满且线程数已达到最大线程数，根据配置的拒绝策略处理新的任务。
4. **任务执行**：
   - 线程从任务队列中获取任务并执行。当任务执行完成后，线程不会被销毁，而是返回线程池中等待新的任务。
5. **线程复用**：
   - 当有新的任务提交到线程池时，空闲线程会再次被唤醒并执行新的任务。这种机制减少了频繁创建和销毁线程的开销，从而实现线程的复用。
6. **线程回收**：
   - 当线程池中的线程数量超过核心线程数，并且这些多余的线程空闲时间超过了设定的 **keepAliveTime**，这些空闲线程会被终止和移除。
```java
public class ThreadPoolReuseExample {
    public static void main(String[] args) {
        // 创建一个固定大小的线程池，包含 4 个线程
        ExecutorService executorService = new ThreadPoolExecutor(
                4, // corePoolSize
                8, // maximumPoolSize
                60L, // keepAliveTime
                TimeUnit.SECONDS, // unit
                new LinkedBlockingQueue<Runnable>(), // workQueue
                Executors.defaultThreadFactory(), // threadFactory
                new ThreadPoolExecutor.AbortPolicy() // handler
        );

        // 提交任务给线程池
        for (int i = 0; i < 20; i++) {
            final int taskNumber = i;
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println("Task " + taskNumber + " executed by " + Thread.currentThread().getName());
                    try {
                        Thread.sleep(1000); // 模拟任务执行时间
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
            });
        }

        // 关闭线程池
        executorService.shutdown();
    }
}

//运行结果
Task 0 executed by pool-1-thread-1
Task 2 executed by pool-1-thread-3
Task 1 executed by pool-1-thread-2
Task 3 executed by pool-1-thread-4

Task 4 executed by pool-1-thread-4
Task 5 executed by pool-1-thread-1
Task 7 executed by pool-1-thread-3
Task 6 executed by pool-1-thread-2

Task 8 executed by pool-1-thread-3
Task 10 executed by pool-1-thread-4
Task 11 executed by pool-1-thread-1
Task 9 executed by pool-1-thread-2

Task 13 executed by pool-1-thread-4
Task 14 executed by pool-1-thread-2
Task 12 executed by pool-1-thread-3
Task 15 executed by pool-1-thread-1

Task 17 executed by pool-1-thread-3
Task 19 executed by pool-1-thread-1
Task 18 executed by pool-1-thread-4
Task 16 executed by pool-1-thread-2
```
## 线程是如何获取任务以及如何实现超时的
### 线程获取任务的过程

1. **线程启动**：
   - 线程池在初始化时，会创建核心线程并启动这些线程，这些线程会调用内部的 **run** 方法开始运行。
2. **获取任务**：
   - 每个线程会循环执行，从任务队列中获取任务。当任务队列中有任务时，线程会从队列中取出任务并执行。如果任务队列为空，线程会进入等待状态，直到有新任务提交到队列中。
3. **执行任务**：
   - 线程获取到任务后，会调用任务的 **run** 方法来执行任务。
### 线程超时机制的实现
线程池中的超时机制主要通过 **keepAliveTime** 和 **TimeUnit** 参数来实现。这两个参数决定了线程在空闲时保持存活的时间。当线程空闲超过指定的时间后，如果线程池中的线程数量超过核心线程数，线程将被终止并从线程池中移除。
### 详细实现过程
以下是 **ThreadPoolExecutor** 中获取任务和实现超时的详细实现过程：
```java
public class ThreadPoolTimeoutExample {
    public static void main(String[] args) {
        // 创建一个自定义的线程池
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2, // corePoolSize
            4, // maximumPoolSize
            10L, // keepAliveTime
            TimeUnit.SECONDS, // unit
            new LinkedBlockingQueue<Runnable>(), // workQueue
            Executors.defaultThreadFactory(), // threadFactory
            new ThreadPoolExecutor.CallerRunsPolicy() // handler
        );

        // 提交一些任务给线程池
        for (int i = 0; i < 10; i++) {
            executor.submit(new Task(i));
        }

        // 关闭线程池
        executor.shutdown();
    }
}

class Task implements Runnable {
    private final int taskId;

    public Task(int taskId) {
        this.taskId = taskId;
    }

    @Override
    public void run() {
        System.out.println("Task " + taskId + " executed by " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000); // 模拟任务执行时间
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```
## 线程池的五种状态
线程池内部有5 个常量来代表线程池的5 种状态
```java
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716345375648-bc8a956d-ca80-46f7-890a-a66f7ff03556.png#averageHue=%23f1ecec&clientId=u8d9ea835-669e-4&from=paste&id=u72246ec5&originHeight=294&originWidth=1080&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u41711f21-5708-4280-8cc6-096d51c060b&title=)
### 1. RUNNING

- **状态描述**：线程池处于运行状态，可以接收新任务并处理任务队列中的任务。
- **状态转换**：线程池在创建时初始为 RUNNING 状态。
- **关键特征**：新任务可以提交，已有任务会被执行。
### 2. SHUTDOWN

- **状态描述**：线程池不再接收新任务，但仍会处理任务队列中的任务。
- **状态转换**：调用 **shutdown()** 方法后，线程池从 RUNNING 状态转换为 SHUTDOWN 状态。
- **关键特征**：不接收新任务，继续执行队列中的任务。
### 3. STOP

- **状态描述**：线程池不再接收新任务，也不再处理任务队列中的任务，并且会中断正在执行的任务。
- **状态转换**：调用 **shutdownNow()** 方法后，线程池从 RUNNING 或 SHUTDOWN 状态转换为 STOP 状态。
- **关键特征**：不接收新任务，不执行队列中的任务，中断正在执行的任务。
### 4. TIDYING

- **状态描述**：线程池中的所有任务都已终止，且线程池中的线程数量为 0，进入整理状态。
- **状态转换**：当线程池在 SHUTDOWN 状态下，任务队列已空，且线程池中的所有任务都已完成，或在 STOP 状态下，所有任务已终止并清空队列，线程池转换为 TIDYING 状态。
- **关键特征**：所有任务已终止，线程数量为 0。
### 5. TERMINATED

- **状态描述**：线程池彻底终止。
- **状态转换**：**terminated()** 钩子方法执行完成后，线程池从 TIDYING 状态转换为 TERMINATED 状态。
- **关键特征**：线程池完全终止，不再处理任何任务。
## 线程池的关闭
线程池提供了 `**shutdown**` 和 `**shutdownNow**` 两个方法来关闭线程池。
**shutdown** 方法：就是将线程池的状态修改为 SHUTDOWN，然后尝试打断空闲的线程。
```java
/**
 * 启动一次顺序关闭，在这次关闭中，执行器不再接受新任务，但会继续处理队列中的已存在任务。
 * 当所有任务都完成后，线程池中的线程会逐渐退出。
 */
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock; // ThreadPoolExecutor的主锁
    mainLock.lock(); // 加锁以确保独占访问

    try {
        checkShutdownAccess(); // 检查是否有关闭的权限
        advanceRunState(SHUTDOWN); // 将执行器的状态更新为SHUTDOWN
        interruptIdleWorkers(); // 中断所有闲置的工作线程
        onShutdown(); // ScheduledThreadPoolExecutor中的挂钩方法，可供子类重写以进行额外操作
    } finally {
        mainLock.unlock(); // 无论try块如何退出都要释放锁
    }

    tryTerminate(); // 如果条件允许，尝试终止执行器
}

```
**shotdownNow **方法：就是将线程池的状态修改为 STOP，然后尝试打断所有的线程，从阻塞队列中移除剩余的任务。
```java
/**
 * 尝试停止所有正在执行的任务，停止处理等待的任务，
 * 并返回等待处理的任务列表。
 *
 * @return 从未开始执行的任务列表
 */
public List<Runnable> shutdownNow() {
    List<Runnable> tasks; // 用于存储未执行的任务的列表
    final ReentrantLock mainLock = this.mainLock; // ThreadPoolExecutor的主锁
    mainLock.lock(); // 加锁以确保独占访问

    try {
        checkShutdownAccess(); // 检查是否有关闭的权限
        advanceRunState(STOP); // 将执行器的状态更新为STOP
        interruptWorkers(); // 中断所有工作线程
        tasks = drainQueue(); // 清空队列并将结果放入任务列表中
    } finally {
        mainLock.unlock(); // 无论try块如何退出都要释放锁
    }

    tryTerminate(); // 如果条件允许，尝试终止执行器

    return tasks; // 返回队列中未被执行的任务列表
}

```
### 两者区别

1. **shutdown**：
   - 不再接受新任务，但继续处理队列中的任务。
   - 中断空闲线程，但不会中断正在执行的任务。
2. **shutdownNow**：
   - 不再接受新任务，并立即停止所有任务。
   - 中断所有线程（包括正在执行任务的线程）。
   - 清空任务队列并返回未执行的任务列表。
### 案例实现
```java
public class ThreadPoolShutdownExample {
    public static void main(String[] args) throws InterruptedException {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                2, 4, 10, TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>(),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.CallerRunsPolicy()
        );

        // 提交一些任务给线程池
        for (int i = 0; i < 10; i++) {
            executor.submit(new Task(i));
        }

        // 调用shutdown方法，等待所有任务完成
        executor.shutdown();
        System.out.println("Shutdown 被调用");
        if (executor.awaitTermination(1, TimeUnit.MINUTES)) {
            System.out.println("所有任务已经结束");
        }

        // 再次提交任务会抛出RejectedExecutionException
        try {
            executor.submit(new Task(10));
        } catch (RejectedExecutionException e) {
            System.out.println("Task 被拒绝: " + e.getMessage());
        }

        // 再次创建线程池并调用shutdownNow方法
        executor = new ThreadPoolExecutor(
                2, 4, 10, TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>(),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.CallerRunsPolicy()
        );

        // 提交一些任务给线程池
        for (int i = 0; i < 10; i++) {
            executor.submit(new Task(i));
        }

        // 调用shutdownNow方法，立即停止所有任务
        List<Runnable> notExecutedTasks = executor.shutdownNow();
        System.out.println("ShutdownNow 被调用");
        System.out.println("没有被执行任务的数量: " + notExecutedTasks.size());
    }

    static class Task implements Runnable {
        private final int taskId;

        public Task(int taskId) {
            this.taskId = taskId;
        }

        @Override
        public void run() {
            System.out.println("Task " + taskId + " executed by " + Thread.currentThread().getName());
            try {
                Thread.sleep(2000); // 模拟任务执行时间
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println("Task " + taskId + " was interrupted");
            }
        }
    }
}
```
## 线程池的监控
在项目中使用线程池的时候，一般需要对线程池进行监控，方便出问题的时候快速定位。线程池本身提供了一些方法来获取线程池的运行状态。

- getCompletedTaskCount：已经执行完成的任务数量
- getLargestPoolSize：线程池里曾经创建过的最大的线程数量。这个主要是用来判断线程是否满过。
- getActiveCount：获取正在执行任务的线程数据
- getPoolSize：获取当前线程池中线程数量的大小

除了线程池提供的上述已经实现的方法，同时线程池也预留了很多扩展方法。比如在 runWorker 方法里面，执行任务之前会回调 beforeExecute 方法，执行任务之后会回调 afterExecute 方法，而这些方法默认都是空实现，小伙伴们可以自己继承 ThreadPoolExecutor 来重写这些方法，实现自己想要的功能
## 线程池的使用场景
### 1. Web服务器

- **场景描述**：处理大量并发的客户端请求，如 HTTP 请求。
- **优势**：通过线程池管理请求处理线程，避免频繁创建和销毁线程带来的开销，提高服务器的响应速度和吞吐量。
### 2. 数据库连接池

- **场景描述**：管理与数据库的并发连接。
- **优势**：通过连接池管理数据库连接，可以复用连接，减少连接创建和销毁的开销，提高数据库操作的效率。
### 3. 并发任务处理

- **场景描述**：处理需要并发执行的大量任务，如批处理、数据分析等。
- **优势**：通过线程池管理任务执行线程，可以高效调度任务，充分利用多核处理器，提高任务处理效率。
### 4. 异步任务执行

- **场景描述**：在 GUI 应用程序中执行异步任务，如文件上传、下载等。
- **优势**：通过线程池管理异步任务，可以避免阻塞主线程，提高应用程序的响应速度和用户体验。
### 5. 周期性任务调度

- **场景描述**：需要定时执行或周期性执行的任务，如日志记录、数据备份等。
- **优势**：通过 **ScheduledThreadPoolExecutor** 管理定时任务，可以精确调度任务执行，确保任务按时执行。
### 6. 任务队列处理

- **场景描述**：处理消息队列中的任务，如消息中间件的消费者处理消息。
- **优势**：通过线程池管理任务处理线程，可以高效处理队列中的任务，提高系统吞吐量。
### 7. 文件处理

- **场景描述**：并发处理大量文件，如文件扫描、转换、压缩等。
- **优势**：通过线程池管理文件处理线程，可以提高文件处理的并发能力和效率。
### 8. 复杂计算

- **场景描述**：并发执行复杂计算任务，如图像处理、科学计算等。
- **优势**：通过线程池管理计算任务线程，可以充分利用多核处理器的计算能力，提高计算效率。
### 案例演示
以下是一个使用线程池处理并发任务的示例代码：
```java
public class ThreadPoolAsynchronousExample {
    public static void main(String[] args) {
        // 创建一个固定大小的线程池，包含 4 个线程
        ExecutorService executorService = Executors.newFixedThreadPool(4);

        // 提交 10 个任务给线程池
        for (int i = 0; i < 10; i++) {
            executorService.submit(new Task(i));
        }

        // 关闭线程池
        executorService.shutdown();
    }

    static class Task implements Runnable {
        private final int taskId;

        public Task(int taskId) {
            this.taskId = taskId;
        }

        @Override
        public void run() {
            System.out.println("Task " + taskId + " executed by " + Thread.currentThread().getName());
            try {
                Thread.sleep(2000); // 模拟任务执行时间
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}

//执行结果
Task 1 executed by pool-1-thread-2
Task 3 executed by pool-1-thread-4
Task 2 executed by pool-1-thread-3
Task 0 executed by pool-1-thread-1

Task 4 executed by pool-1-thread-2
Task 6 executed by pool-1-thread-3
Task 5 executed by pool-1-thread-4
Task 7 executed by pool-1-thread-1

Task 8 executed by pool-1-thread-3
Task 9 executed by pool-1-thread-1
```
> 部分内容转载至：[二哥进阶之路 · 线程池](https://www.javabetter.cn/thread/pool.html)

