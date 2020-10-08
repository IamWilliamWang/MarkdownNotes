#Executors
**Executors**相当于ExecutorService的工厂类，包含各种常用执行器的工厂方法，可以直接创建常用的执行器。几种常用的执行器如下：
`Executors.newCachedThreadPool()`,根据需要可以创建新线程的线程池。线程池中曾经创建的线程，在完成某个任务后也许会被用来完成另外一项任务。
`Executors.newFixedThreadPool(int nThreads)` ,创建一个可重用固定线程数的线程池。这个线程池里最多包含nThread个线程。
`Executors.newSingleThreadExecutor()` ,创建一个使用单个 worker 线程的 Executor。即使任务再多，也只用1个线程完成任务。
`Executors.newSingleThreadScheduledExecutor()` ,创建一个单线程执行程序，它可安排在<u>给定延迟后</u>运行命令或者<u>定期</u>执行。

**1. 可缓存线程池(CachedThreadPool)**
创建的都是非核心线程，而且最大线程数为Integer的最大值，空闲线程存活时间是1分钟。如果有大量耗时的任务，则不适该创建方式。它只适用于生命周期短的任务。
```java
ExecutorService exec = Executors.newCachedThreadPool();
for (int i = 0; i < 5; i++) {//5个任务
    exec.submit(new Runnable() {
        @Override
        public void run() {            
            System.out.println(Thread.currentThread().getName()+" doing task");
         }
     });
}
exec.shutdown();  //关闭线程池
```
输出：
```
pool-1-thread-2 doing task
pool-1-thread-1 doing task
pool-1-thread-3 doing task
pool-1-thread-4 doing task
pool-1-thread-5 doing task
```

**2. 单线程池(SingleThreadExecutor)**
创建一个核心线程
```java
ExecutorService exec = Executors.newSingleThreadExecutor();
for (int i = 0; i < 5; i++) {
    exec.execute(new Runnable() {//execute方法接收Runnable对象，无返回值
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
        }
    });
}
exec.shutdown();
```
输出：
```
pool-1-thread-1
pool-1-thread-1
pool-1-thread-1
pool-1-thread-1
pool-1-thread-1
```

**3. 固定线程数线程池**
类似于单线程池的用法，如nThreads=3时，`Executors.newFixedThreadPool(3);`

**4. 固定线程数，支持定时和周期性任务**
```java
ExecutorService scheduledPool = Executors.newScheduledThreadPool(5);
```
可用于替代handler.postDelay和Timer定时器等延时和周期性任务。
```java
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
```
> **scheduleAtFixedRate**和**sheduleWithFixedDelay**的不同
scheduleAtFixedRate：创建并执行一个在给定初始延迟后的定期操作，也就是将在 initialDelay 后开始执行，然后在initialDelay+period 后下一个任务执行，接着在 initialDelay+2*period 后执行，依此类推，也就是只在第一次任务执行时有延时。
sheduleWithFixedDelay：创建并执行一个在给定初始延迟后首次启用的定期操作，随后，在每一次执行终止和下一次执行开始之间都存在给定的延迟，即总时间是(initialDelay+period)*n

帮助类
```java
import java.util.concurrent.*;

public final class LgExecutorService {

    private ConcurrentHashMap<String, Future> futureMap = new ConcurrentHashMap<>();
    private ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(5);
    private static final LgExecutorService INSTANCE = new LgExecutorService();

    private LgExecutorService() {
    }

    public static LgExecutorService getInstance() {
        return INSTANCE;
    }

    public ConcurrentHashMap<String, Future> getFutureMap() {
        return futureMap;
    }

    public void execute(Runnable runnable) {
        if (runnable != null) {
            executorService.execute(runnable);
        }
    }

    /**
     * @param runnable
     * @param delay    延迟时间
     * @param timeUnit 时间单位
     */
    public void sheduler(Runnable runnable, long delay, TimeUnit timeUnit) {
        if (runnable != null) {
            executorService.schedule(runnable, delay, timeUnit);
        }
    }

    /**
     * 执行延时周期性任务
     *
     * @param runnable {@code LgExecutorSercice.JobRunnable}
     * @param initialDelay 延迟时间
     * @param period       周期时间
     * @param timeUnit     时间单位
     */
    public <T extends JobRunnable> void sheduler(T runnable, long initialDelay, long period, TimeUnit timeUnit) {
        if (runnable != null) {
            var future = executorService.scheduleAtFixedRate(runnable, initialDelay, period, timeUnit);
            futureMap.put(runnable.getJobId(), future);
        }
    }

    public static abstract class JobRunnable implements Runnable {

        private String jobId;

        public JobRunnable(String jobId) {
            this.jobId = jobId;
        }

        /**
         * 强制终止定时线程
         */
        public void terminal() {
            try {
                var future = LgExecutorService.getInstance().getFutureMap().remove(jobId);
                future.cancel(true);
            } finally {
                System.out.println("jobId " + jobId + " had cancel");
            }
        }

        public String getJobId() {
            return jobId;
        }
    }
}
```

**5. 手动创建线程池**
```java
private ExecutorService pool = new ThreadPoolExecutor(3, 10, 10L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(512), Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
```
#Executor(interface)
只有`void execute(Runnable command)`方法。里面具体怎么执行，是否调用线程执行由相应的Executor接口实现类决定。Executor将任务提交与每个任务如何运行（如何使用线程、调度）相分离。
#ExecutorService(interface)
该接口继承自Executor接口
* 用于执行任务的submit方法有两种
```java
Future<?> submit(Runnable task) 
Future<T> submit(Callable<T> task)
```
区别：一个接收Runnable类型入参，一个接收Callable类型入参。Callable入参允许任务返回值，而Runnable无返回值。

* **批量执行**任务的两种方法
```java
List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
T invokeAny(Collection<? extends Callable<T>> tasks)
```
区别：invokeAll是等所有任务完成后返回代表结果的Future列表。而invokeAny是等这一批任务中的任何一个任务完成后就返回。

* `void shutdown()`：启动一次顺序关闭，执行以前提交的任务，但不接受新任务。执行此方法后，线程池等待任务结束后就关闭，同时不再接收新的任务。
<u>原则</u>：只要ExecutorService(线程池)不再使用，就应该关闭，以回收资源。
#Callable(interface)
Callable代表有返回值的任务。使用方法如下：
```java
class CalcTask implements Callable<String> {
    @Override
    public String call() throws Exception {
        return Thread.currentThread().getName();
    }
}
```
```java
ExecutorService exec = Executors.newCachedThreadPool();
List<Callable<String>> taskList = new ArrayList<Callable<String>>();
/* 往任务列表中添加5个任务 */
for (int i = 0; i < 5; i++) {
    taskList.add(new CalcTask());
}
/* 结果列表:存放任务完成返回的值 */
List<Future<String>> resultList = new ArrayList<Future<String>>();
try {
    /*invokeAll批量运行所有任务, submit提交单个任务*/
    resultList = exec.invokeAll(taskList);
} catch (InterruptedException e) {
    e.printStackTrace();
}
try {
    /*从future中输出每个任务的返回值*/
    for (Future<String> future : resultList) {
        System.out.println(future.get());//get方法会阻塞直到结果返回
    }
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (ExecutionException e) {
    e.printStackTrace();
}
```
#Future(interface)
`get()`：获取其结果
`isDone()`：用来查询任务是否做完

```java
/*新建一个Callable任务*/
Callable<Integer> callableTask = new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
        System.out.println("Calculating!");
        TimeUnit.SECONDS.sleep(2);//休眠2秒
        return 2;
    }
}; 
ExecutorService executor = Executors.newCachedThreadPool();
Future<Integer> result = executor.submit(callableTask);
executor.shutdown();
while(!result.isDone()){//isDone()方法可以查询子线程是否做完
    System.out.println("子线程正在执行");
    TimeUnit.SECONDS.sleep(1);//休眠1秒
}
try {
    System.out.println("子线程执行结果:"+result.get());
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
```
#FutureTask
FutureTask类实现了Runnable接口和Future接口，所以FutureTask可以作为Runnable被线程执行，也可以作为Future得到传入的Callable对象的返回值
```java
FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {
   @Override
   public Integer call() throws Exception {
        System.out.println("futureTask is wokring 1+1!");
        return 2;
   }
});
Thread t1 = new Thread(futureTask);//1.可以作为Runnable类型对象使用
t1.start();
try {
   System.out.println(futureTask.get());//2.可以作为Future类型对象得到线程运算返回值
} catch (ExecutionException e) {
   e.printStackTrace();
}
```
#Semaphore
`public Semaphore(int permits,boolean fair)`
&emsp;permits:初始化可用的许可数目
&emsp;fair: 若该信号量保证在征用时按FIFO的顺序授予许可，则为true，否则为false
> `void acquireUninterruptibly(int permits)`：从信号量中获取多个许可，并且在获得许可之前，一直将线程阻塞
`void release()`：添加一个许可/释放一个许可，并将其返回给信号量。从而可能释放一个正在阻塞的获取者。 
`void acquire()`：从信号量获得许可

例子：
```java
import java.util.concurrent.*;
/**
 * 线程信号量Semaphore的运用
 */
public class SemaphoreThreadTest {

    private int threadId = 0;
    /**
     * 银行存钱类
     */
    class Bank {
        private int account = 100;
        public int getAccount() {
            return account;
        }
        public void save(int money) {
            account += money;
        }
    }

    /**
     * 线程执行类，每次存10块钱
     */
    class SaveMoneyTask implements Runnable {
        private Bank bank;
        private Semaphore semaphore;

        public SaveMoneyTask(Bank bank, Semaphore semaphore) {
            this.bank = bank;
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            int threadId = SemaphoreThreadTest.this.threadId++;
            if (semaphore.availablePermits() > 0) {
                System.out.println("线程" + threadId + "启动，进入银行,有位置立即去存钱");
            } else {
                System.out.println("线程" + threadId + "启动，进入银行,无位置，去排队等待等待");
            }
            try {
                semaphore.acquire();
                bank.save(10);
                System.out.println(threadId + "账户余额为：" + bank.getAccount());
                TimeUnit.SECONDS.sleep(1);
                System.out.println("线程" + threadId + "存钱完毕，离开银行");
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 建立线程，调用内部类，开始存钱
     */
    public void start() {
        Bank bank = new Bank();
        // 定义2个信号量
        Semaphore semaphore = new Semaphore(2);
        // 建立一个缓存线程池
        ExecutorService executorService = Executors.newCachedThreadPool();
        // 建立10个线程
        for (int i = 0; i < 10; i++) {
            // 执行一个线程
            executorService.execute(new SaveMoneyTask(bank, semaphore));
        }
        // 关闭线程池
        executorService.shutdown();

        // 从信号量中获取两个许可，并且在获得许可之前，一直将线程阻塞
        semaphore.acquireUninterruptibly(2);
        System.out.println("到饭点了，窗口缩减为2个。");
        // 释放两个许可，并将其返回给信号量
        semaphore.release(2);
    }

    public static void main(String[] args) {
        SemaphoreThreadTest test = new SemaphoreThreadTest();
        test.start();
    }
}
```
#Lock
`Lock lock = new ReentrantLock();`
#Condition
接口内方法：
1. `lock.newCondition()` 构建新Condition
`Condition`依赖于`Lock`接口，lock是Lock的实例。调用`Condition`的`await()`和`signal()`方法，都必须在`lock`保护之内，就是说必须在`lock.lock()`和`lock.unlock`之间才可以使用！
2. `await()`
Conditon中的await()对应Object的wait()
3. `signal()`
Condition中的signal()对应Object的notify()
4. `signalAll()`
Condition中的signalAll()对应Object的notifyAll()

使用synchronized/wait()实现生产者消费者模式例子：
```java
import java.util.Date;
import java.util.LinkedList;
import java.util.List;

//模拟生产和消费的对象
class Buffer {
	private int maxSize;
	private List<Date> storage;

	Buffer(int size) {
		maxSize = size;
		storage = new LinkedList<>();
	}

	// 生产方法
	public synchronized void put() {
		try {
			while (storage.size() == maxSize) {// 如果队列满了
				System.out.print(Thread.currentThread().getName() + ": wait \n");
				;
				wait();// 阻塞线程
			}
			storage.add(new Date());
			System.out.print(Thread.currentThread().getName() + ": put:" + storage.size() + "\n");
			Thread.sleep(1000);
			notifyAll();// 唤起线程
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	// 消费方法
	public synchronized void take() {
		try {
			while (storage.size() == 0) {// 如果队列满了
				System.out.print(Thread.currentThread().getName() + ": wait \n");
				;
				wait();// 阻塞线程
			}
			Date d = ((LinkedList<Date>) storage).poll();
			System.out.print(Thread.currentThread().getName() + ": take:" + storage.size() + "\n");
			Thread.sleep(1000);
			notifyAll();// 唤起线程
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}

//生产者
class Producer implements Runnable {
	private Buffer buffer;

	Producer(Buffer b) {
		buffer = b;
	}

	@Override
	public void run() {
		while (true) {
			buffer.put();
		}
	}
}

//消费者
class Consumer implements Runnable {
	private Buffer buffer;

	Consumer(Buffer b) {
		buffer = b;
	}

	@Override
	public void run() {
		while (true) {
			buffer.take();
		}
	}
}

public class Main {
	public static void main(String[] arg) {
		Buffer buffer = new Buffer(10);
		Producer producer = new Producer(buffer);
		Consumer consumer = new Consumer(buffer);
		// 创建线程执行生产和消费
		for (int i = 0; i < 3; i++) {
			new Thread(producer, "producer-" + i).start();
		}
		for (int i = 0; i < 3; i++) {
			new Thread(consumer, "consumer-" + i).start();
		}
	}
}
```
将以上代码转换为使用lock/condition实现，代码如下
```java
import java.util.Date;
import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Buffer {
	private final Lock lock;
	private final Condition notFull;
	private final Condition notEmpty;
	private int maxSize;
	private List<Date> storage;

	Buffer(int size) {
		// 使用锁lock，并且创建两个condition，相当于两个阻塞队列
		lock = new ReentrantLock();
		notFull = lock.newCondition();
		notEmpty = lock.newCondition();
		maxSize = size;
		storage = new LinkedList<>();
	}

	public void put() {
		lock.lock();
		try {
			while (storage.size() == maxSize) {// 如果队列满了
				System.out.print(Thread.currentThread().getName() + ": wait \n");
				;
				notFull.await();// 阻塞生产线程
			}
			storage.add(new Date());
			System.out.print(Thread.currentThread().getName() + ": put:" + storage.size() + "\n");
			Thread.sleep(1000);
			notEmpty.signalAll();// 唤醒消费线程
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}

	public void take() {
		lock.lock();
		try {
			while (storage.size() == 0) {// 如果队列满了
				System.out.print(Thread.currentThread().getName() + ": wait \n");
				;
				notEmpty.await();// 阻塞消费线程
			}
			Date d = ((LinkedList<Date>) storage).poll();
			System.out.print(Thread.currentThread().getName() + ": take:" + storage.size() + "\n");
			Thread.sleep(1000);
			notFull.signalAll();// 唤醒生产线程

		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
}

class Producer implements Runnable {
	private Buffer buffer;

	Producer(Buffer b) {
		buffer = b;
	}

	@Override
	public void run() {
		while (true) {
			buffer.put();
		}
	}
}

class Consumer implements Runnable {
	private Buffer buffer;

	Consumer(Buffer b) {
		buffer = b;
	}

	@Override
	public void run() {
		while (true) {
			buffer.take();
		}
	}
}

public class Main {
	public static void main(String[] arg) {
		Buffer buffer = new Buffer(10);
		Producer producer = new Producer(buffer);
		Consumer consumer = new Consumer(buffer);
		for (int i = 0; i < 3; i++) {
			new Thread(producer, "producer-" + i).start();
		}
		for (int i = 0; i < 3; i++) {
			new Thread(consumer, "consumer-" + i).start();
		}
	}
}
```
#BlockingQueue阻塞队列
阻塞队列 (`BlockingQueue`)是`java.util.concurrent`包下重要的数据结构，`BlockingQueue`提供了**线程安全的队列**访问方式：当阻塞队列进行插入数据时，如果队列已满，线程将会阻塞等待直到队列非满；从阻塞队列取数据时，如果队列已空，线程将会阻塞等待直到队列非空。

接口内方法：
||抛异常|特定值|阻塞|超时
|:-:|-|-|-|-
**插入**|add(o)|offer(o)|put(o)|offer(o, timeout, timeunit)
**移除**|remove(o)|poll(o)|take(o)|poll(timeout, timeunit)
**检查**|element(o)|peek(o)||

抛异常：如果试图的操作无法立即执行，抛一个异常。
特定值：如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。
阻塞：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
超时：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是true / false)。

**BlockingQueue 的实现类**
1. `ArrayBlockingQueue`：`ArrayBlockingQueue`是一个有界的阻塞队列，其内部实现是将对象放到一个数组里。有界也就意味着，它不能够存储无限多数量的元素。它有一个同一时间能够存储元素数量的上限。你可以在对其初始化的时候设定这个上限，但之后就无法对这个上限进行修改了(译者注：因为它是基于数组实现的，也就具有数组的特性：一旦初始化，大小就无法修改)。

2. `DelayQueue`：`DelayQueue`对元素进行持有直到一个特定的延迟到期。注入其中的元素必须实现`java.util.concurrent.Delayed`接口。

3. `LinkedBlockingQueue`：`LinkedBlockingQueue`内部以一个链式结构(链接节点)对其元素进行存储。如果需要的话，这一链式结构可以选择一个上限。如果没有定义上限，将使用 Integer.MAX_VALUE 作为上限。

4. `PriorityBlockingQueue`：`PriorityBlockingQueue`是一个无界的并发队列。它使用了和类`java.util.PriorityQueue`一样的排序规则。你无法向这个队列中插入 null 值。所有插入到`PriorityBlockingQueue`的元素必须实现`java.lang.Comparable`接口。因此该队列中元素的排序就取决于你自己的`Comparable`实现。

5. `SynchronousQueue`：`SynchronousQueue`是一个特殊的队列，它的内部同时只能够容纳单个元素。如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。据此，把这个类称作一个队列显然是夸大其词了。它更多像是一个汇合点。
