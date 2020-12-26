# Tomcat如何使用线程池处理远程并发请求

通过了解学习tomcat如何处理并发请求，了解到线程池，锁，队列，unsafe类，下面的主要代码来自

java-jre：
`sun.misc.Unsafe`
`java.util.concurrent.ThreadPoolExecutor`
`java.util.concurrent.ThreadPoolExecutor.Worker`
`java.util.concurrent.locks.AbstractQueuedSynchronizer`
`java.util.concurrent.locks.AbstractQueuedLongSynchronizer`
`java.util.concurrent.LinkedBlockingQueue`

tomcat:
`org.apache.tomcat.util.net.NioEndpoint`
`org.apache.tomcat.util.threads.ThreadPoolExecutor`
`org.apache.tomcat.util.threads.TaskThreadFactory`
`org.apache.tomcat.util.threads.TaskQueue`


## ThreadPoolExecutor

是一个线程池实现类，管理线程，减少线程开销，可以用来提高任务执行效率，

构造方法中的参数有

```java
public ThreadPoolExecutor(
    int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler) {
    
}
```

corePoolSize 是核心线程数
maximumPoolSize 是最大线程数
keepAliveTime 非核心线程最大空闲时间（超过时间终止）
unit 时间单位
workQueue 队列，当任务过多时，先存放在队列
threadFactory 线程工厂，创建线程的工厂
handler 拒绝策略，当任务数过多，队列不能再存放任务时，该如何处理，由此对象去处理。这是个接口，你可以自定义处理方式

## ThreadPoolExecutor在Tomcat中http请求的应用

此线程池是tomcat用来在接收到远程请求后，将每次请求单独作为一个任务去处理使用，调用execute(Runnable)

### 初始化

`org.apache.tomcat.util.net.NioEndpoint`

#### 创建线程池

NioEndpoint初始化的时候，创建了线程池

```java
public void createExecutor() {
        internalExecutor = true;
        TaskQueue taskqueue = new TaskQueue();
        //TaskQueue无界队列，可以一直添加，因此handler 等同于无效
        TaskThreadFactory tf = new TaskThreadFactory(getName() + "-exec-", daemon, getThreadPriority());
        executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), 60, TimeUnit.SECONDS,taskqueue, tf);
        taskqueue.setParent( (ThreadPoolExecutor) executor);
    }
```

#### 创建工作线程worker

在线程池创建时，调用prestartAllCoreThreads(), 初始化核心工作线程worker，并启动

```java
public int prestartAllCoreThreads() {
        int n = 0;
        while (addWorker(null, true))
            ++n;
        return n;
    }
```

当addWorker 数量等于corePoolSize时，addWorker(null,ture)会返回false,停止worker工作线程的创建

addWorker时，会启动worker线程

<details>
	<summary>code:addWorker</summary>


```java
private boolean addWorker(Runnable firstTask, boolean core) {
    	//......省去判断代码（是否需要添加worker的判断）

        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            w = new Worker(firstTask);//1 创建worker线程
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                        workers.add(w);
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                } 	
                if (workerAdded) {
                    t.start(); //2 如果worker创建成功，启动这个工作线程
                    workerStarted = true; //返回true
                }
            }
        } finally {
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
    }
```

</details>

### 接收任务放入队列

每次客户端过来请求（http），就会提交一次处理任务，
poller对象的run方法中开始 -> processKey() -> processSocket() -> executor.execute()

<details>
	<summary>code:poller -> processKey -> processSocket -> execute</summary>

```java
//org.apache.tomcat.util.net.NioEndpoint.Poller.run() 
@Override
public void run() {
    // Loop until destroy() is called
    while (true) {
        //...............
            NioSocketWrapper socketWrapper = (NioSocketWrapper) sk.attachment();
            if (socketWrapper != null) {
                //1调用processKey方法
                processKey(sk, socketWrapper);
            }
        //.............
        }
    }

//org.apache.tomcat.util.net.NioEndpoint.Poller.processKey(SelectionKey, NioSocketWrapper)
protected void processKey(SelectionKey sk, NioSocketWrapper socketWrapper) {
            try {
                    //....................
					// 2调用processSocket方法
                   processSocket(socketWrapper, SocketEvent.OPEN_WRITE, true))
                    //..................
        	}
}
    
//org.apache.tomcat.util.net.AbstractEndpoint.processSocket(SocketWrapperBase<S>, SocketEvent, boolean)
public boolean processSocket(SocketWrapperBase<S> socketWrapper,
            SocketEvent event, boolean dispatch) {
        try {
            //...............
            Executor executor = getExecutor();
            if (dispatch && executor != null) {
                executor.execute(sc); // 3调用ThreadPoolExecutor.execute提交新请求任务
            } else {
                sc.run();
            }
            //.....................
        return true;
    }
```

</details>

#### ThreadPoolExecutor.execute

worker 从队列中获取任务运行，下面是将任务放入队列的逻辑代码

ThreadPoolExecutor.execute(Runnable) 提交任务：

```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
       
        int c = ctl.get();
    	// worker数 是否小于 核心线程数   tomcat中初始化后，一般不满足第一个条件，不会addWorker
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
    	// workQueue.offer(command)，将任务添加到队列
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```

workQueue.offer(command) 最终完成了任务的提交(在tomcat处理远程http请求时)。

#### workQueue.offer

TaskQueue 是 BlockingQueue 具体实现类，workQueue.offer(command)实际代码：

```java
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    final AtomicInteger count = this.count;
    if (count.get() == capacity)
        return false;
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        if (count.get() < capacity) {
            enqueue(node); //此处将任务添加到队列
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        }
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
    return c >= 0;
}

// 添加任务到队列
/**
     * Links node at end of queue.
     *
     * @param node the node
     */
private void enqueue(Node<E> node) {
    // assert putLock.isHeldByCurrentThread();
    // assert last.next == null;
    last = last.next = node; //链表结构 last.next = node; last = node
}
```

之后是worker的工作，worker在run方法中通过去getTask()获取此处提交的任务，并执行完成任务。



### 线程池如何处理新提交的任务

添加worker之后，提交任务，因为worker数量达到corePoolSize，任务都会将放入队列，而worker的run方法则是循环获取队列中的任务（不为空时），

worker run方法：

```java
/** Delegates main run loop to outer runWorker  */
        public void run() {
            runWorker(this);
 }
```

#### 循环获取队列中的任务

runWorker(worker)方法 循环部分代码：

```java
final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) { //循环获取队列中的任务
                w.lock(); // 上锁
                try {
                    // 运行前处理
                    beforeExecute(wt, task);
                    // 队列中的任务开始执行
                    task.run();
                    // 运行后处理
                    afterExecute(task, thrown);
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock(); // 释放锁
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
```

task.run()执行任务



### 锁运用

锁用于保证过程的有序，一般一段代码上锁后，同一时间只允许一个线程去操作

ThreadPoolExecutor 使用锁主要保证两件事情，
1.给队列添加任务，释放锁之前，保证其他线程不能操作队列-添加队列任务）
2.获取队列的任务，释放锁之前，保证其他线程不能操作队列-取出队列任务）
在高并发情况下，锁能有效保证请求的有序处理，不至于混乱

#### 给队列添加任务时上锁

```java
public boolean offer(E e) {
        if (e == null) throw new NullPointerException();
        final AtomicInteger count = this.count;
        if (count.get() == capacity)
            return false;
        int c = -1;
        Node<E> node = new Node<E>(e);
        final ReentrantLock putLock = this.putLock;
        putLock.lock();  //上锁
        try {
            if (count.get() < capacity) {
                enqueue(node);
                c = count.getAndIncrement();
                if (c + 1 < capacity)
                    notFull.signal();
            }
        } finally {
            putLock.unlock();  //释放锁
        }
        if (c == 0)
            signalNotEmpty();
        return c >= 0;
    }
```



#### 获取队列任务时上锁

```java
private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?
		// ...省略
        for (;;) {
            try {
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take(); //获取队列中一个任务
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }
public E take() throws InterruptedException {
        E x;
        int c = -1;
        final AtomicInteger count = this.count;
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lockInterruptibly(); // 上锁
        try {
            while (count.get() == 0) {
                notEmpty.await(); //如果队列中没有任务，等待
            }
            x = dequeue();
            c = count.getAndDecrement();
            if (c > 1)
                notEmpty.signal();
        } finally {
            takeLock.unlock(); // 释放锁
        }
        if (c == capacity)
            signalNotFull();
        return x;
    }
```

## 其他

### volatile

在并发场景这个关键字修饰成员变量很常见，

主要目的公共变量在被某一个线程修改时，对其他线程可见（实时）

### sun.misc.Unsafe 高并发相关类API

线程池使用中，有平凡用到Unsafe类，这个类在高并发中，能做一些原子CAS操作，锁线程，释放线程等。

`sun.misc.Unsafe`  类是底层类，openjdk源码中有

#### 原子操作数据


java.util.concurrent.locks.AbstractQueuedSynchronizer 类中就有保证原子操作的代码，

```java
protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

对应Unsafe类的代码:

```java
//对应的java底层，实际是native方法，对应C++代码
/**
* Atomically update Java variable to <tt>x</tt> if it is currently
* holding <tt>expected</tt>.
* @return <tt>true</tt> if successful
*/
public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);
```

方法的作用简单来说就是 更新一个值，保证原子性操作
当你要操作一个对象`o`的一个成员变量`offset`时,修改o.offset，
高并发下为保证准确性，你在操作o.offset的时候，读应该是正确的值，并且中间不能被别的线程修改来保证高并发的环境数据操作有效。

即 expected 期望值与内存中的值比较是一样的expected == 内存中的值 ，则更新值为 x，返回true代表修改成功

否则，期望值与内存值不同，说明值被其他线程修改过，不能更新值为x，并返回false，告诉操作者此次原子性修改失败。

注意一下能知道这是locks包下的类，ReentrantLock锁的底层原理就与unsafe类有关，以及下面的park，unpark。线程可以通过这个原子操作放回true或者false的机制，定义自己获取锁成功还是失败。

#### 阻塞和唤醒线程 

ThreadPoolExecute设计在请求队列任务为空时，worker线程可以是等待或者中断的（非销毁状态）。
这种做法避免了没必要的循环，节省了硬件资源，提高线程使用效率，

线程池的worker角色循环获取队列任务，如果队列中没有任务，worker.run 还是在等待的，不会退出线程，代码中用了`notEmpty.await() ` 中断此worker线程，放入一个等待线程队列（区别去任务队列）；当有新任务需要时，再`notEmpty.signal()`唤醒此线程

底层分别是 

#### park

unsafe.park() 阻塞(停止)当前线程
public native void park(boolean isAbsolute, long time); 

#### unpark

unsafe.unpark() 唤醒(取消停止)线程
public native void unpark(Object thread);

这个操作是对应的，
阻塞时，先将thread放入队列,再park，
唤醒时，从队列拿出被阻塞的线程，unpark(thread)唤醒指定线程。 

`java.util.concurrent.locks.AbstractQueuedLongSynchronizer.ConditionObject` 类中

通过链表存放线程信息

```java
// 添加一个阻塞线程
private Node addConditionWaiter() {
            Node t = lastWaiter;
            // If lastWaiter is cancelled, clean out.
            if (t != null && t.waitStatus != Node.CONDITION) {
                unlinkCancelledWaiters();
                t = lastWaiter;
            }
            Node node = new Node(Thread.currentThread(), Node.CONDITION);
            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node; //将新阻塞的线程放到链表尾部
            return node;
        }

// 拿出一个被阻塞的线程
 public final void signal() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter; //链表中第一个阻塞的线程
            if (first != null)
                doSignal(first);
        }

// 拿到后，唤醒此线程
final boolean transferForSignal(Node node) {
            LockSupport.unpark(node.thread);
        return true;
    }
public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }
```



这里要区分park 和 compareAndSwapInt是两个完全不同的东西，可以单独或者组合使用，
比如ReentrantLock实现锁功能这两个都需要