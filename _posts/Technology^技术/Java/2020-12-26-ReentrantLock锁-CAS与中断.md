# ReentrantLock锁

ReentrantLock通过原子操作和阻塞实现锁原理，一般使用lock获取锁，unlock释放锁

lock的时候可能被其他线程获得所，那么此线程会阻塞自己，关键原理底层用到Unsafe类的API: CAS和park

## 使用方式

lock unlock对应

### lock

拿到锁，开始执行代码逻辑

### unlock

执行完代码后，释放锁，让其他线程去获取

```java
//创建锁对象
ReentrantLock lock = new ReentrantLock();
lock.lock(); //获取锁（锁定）
// 中间执行代码，保证同一时间只有一个线程能运行此处的代码
lock.unlock(); //锁释放
```

## 示例

为了体现锁的作用，这里sleep睡眠0.1秒，增加哪个线程获取锁的随机性
因为线程唤醒后，会开始尝试获取锁，多个线程下竞争一把锁是随机的

```java
package javabasis.threads;
import java.util.concurrent.locks.ReentrantLock;

public class LockTest implements Runnable {
    
	public static ReentrantLock lock = new ReentrantLock();//锁
	private int thold;
    
	public LockTest(int h) {
		this.thold = h;
	}
	
	public static void main(String[] args) {
		for (int i = 10; i < 15; i++) {
			new Thread(new LockTest(i),"name-" + i).start();
		}
	}

	@Override
	public void run() {
		try {
			Thread.sleep(100);
			lock.lock(); //获取锁
			System.out.println("lock threadName:" + Thread.currentThread().getName());
			{
				System.out.print(" writeStart ");
				for (int i = 0; i < 15; i++) {
						Thread.sleep(100);
					System.out.print(thold+",");
				}
				System.out.println(" writeEnd");
			}
			System.out.println("unlock threadName:" + Thread.currentThread().getName() + "\r\n");
			lock.unlock(); //锁释放 
		} catch (InterruptedException e) {	
		}		
	}	
}
```

运行main方法输出结果：

```java
lock threadName:name-10
 writeStart 10,10,10,10,10,10,10,10,10,10,10,10,10,10,10, writeEnd
unlock threadName:name-10

lock threadName:name-14
 writeStart 14,14,14,14,14,14,14,14,14,14,14,14,14,14,14, writeEnd
unlock threadName:name-14

lock threadName:name-13
 writeStart 13,13,13,13,13,13,13,13,13,13,13,13,13,13,13, writeEnd
unlock threadName:name-13

lock threadName:name-11
 writeStart 11,11,11,11,11,11,11,11,11,11,11,11,11,11,11, writeEnd
unlock threadName:name-11

lock threadName:name-12
 writeStart 12,12,12,12,12,12,12,12,12,12,12,12,12,12,12, writeEnd
unlock threadName:name-12
```

这体现在多线程情况下，锁能做到让线程之间有序运行，

如果没有锁，情况可能是 12,13,13,10,10,10,12，没有锁其他线程可能插队执行`System.out.print`



## 原理 

ReentrantLock主要用到unsafe的CAS和park两个功能实现锁（CAS + park ）

> 多个线程同时操作一个数N，使用原子（CAS）操作，原子操作能保证同一时间只能被一个线程修改，而修改数N成功后，返回true，其他线程修改失败，返回false，
> 这个原子操作可以定义线程是否拿到锁，返回true代表获取锁，返回false代表为没有拿到锁。
> 拿到锁的线程，自然是继续执行后续逻辑代码，而没有拿到锁的线程，则调用park，将线程（自己）阻塞。
> 线程阻塞需要其他线程唤醒，ReentrantLock中用到了链表用于存放等待或者阻塞的线程，每次线程阻塞，先将自己的线程信息放入链表尾部，再阻塞自己；之后需要拿到锁的线程，在调用unlock 释放锁时，从链表中获取阻塞线程，调用unpark 唤醒指定线程

### Unsafe

sun.misc.Unsafe是关键类，提供大量偏底层的API 包括CAS  park
`sun.misc.Unsafe` 此类在openjdk中可以查看

### CAS 原子操作

compare and swapz(CAS)比较并交换，是原子性操作，
原理：当修改一个(内存中的)变量o的值N的时候，首先有个期望值expected，和一个更新值x，先比较N是否等于expected，等于，那么更新内存中的值为x值，否则不更新。

```java
public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);
```

这里offset据了解，是对象的成员变量在内存中的偏移地址，
即底层一个对象object存放在内存中，读取的地址是0x2110，此对象的一个成员变量state的值也在内存中，但内存地址肯定不是0x2110

#### java中的CAS使用

`java.util.concurrent.locks.AbstractQueuedSynchronizer` 类

```java
protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

在Java中，这个操作如果更新成功，返回true,失败返回false，通过这个机制，可以定义锁（乐观锁）。
如三个线程A，B，C，在目标值为0的情况下，同时执行`compareAndSetState(0,1) ` 去修改它
期望值是0，更新值是1，因为是原子操作，在第一个线程操作成功之后目标值变为1，返回true
所以另外两个线程就因为期望值为0不等于1，返回false。
我们可以理解为，返回true的线程拿到了锁。

最终调用的Java类是`sun.misc.Unsafe`

### park 阻塞

Java中可以通过unsafe.park()去阻塞（停止）一个线程，也可以通过unsafe.unpark()让一个阻塞线程恢复继续执行

#### unsafe.park() 

阻塞(停止)当前线程

```java
public native void park(boolean isAbsolute, long time); 
```



根据debug测试，此方法能停止线程自己，最后通过其他线程唤醒

#### unsafe.unpark() 

取消阻塞(唤醒)线程

```java
public native void unpark(Object thread);
```



根据debug测试，此方法可以唤醒其他被park调用阻塞的线程

#### park与interrupt的区别

interrupt是Thread类的的API，park是Unsafe类的API，两者是有区别的。
测试了解，Thread.currentThread().interrupt(),线程会继续运行，而Unsafe.park(Thread.currentThread())就是直接阻塞线程，不继续运行代码。

### 获取锁

线程cas操作失败，可以park阻塞自己，让其他拥有锁的线程在unlock的时候释放自己，达到锁的效果

java.util.concurrent.locks.ReentrantLock的lock方法是

```java
public void lock() {
        sync.lock();
    }
```

而sync的实现类其中一个是java.util.concurrent.locks.ReentrantLock.NonfairSync 不公平锁，它的逻辑比较直接

```java
/**
NonfairSync
*/
final void lock() {
    if (compareAndSetState(0, 1))//cas操作，如果true 则表示操作成功，获取锁
        setExclusiveOwnerThread(Thread.currentThread()); //设置获取锁拥有者为当前线程
    else
        acquire(1);//获取锁失败，锁住线程(自己)
}
```

#### 获取失败后阻塞线程

如果获取锁失败，会再尝试一次，失败后，将线程（自己）阻塞

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) { 
			//如果期望值为0，内存值也为0，再次尝试获取锁（此时其他线程也可能尝试获取锁）
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current); //第二次获取成功，放回true
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false; //没有获取到锁，返回false，则 !tryAcquire(arg) 为true，执行acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        }

```

获取锁失败，线程会进入循环，acquireQueued 方法中for是个无限循环，除非获取锁成功后，才会return。

```java
//获取锁失败后，准备阻塞线程（自己）
//阻塞之前，添加节点存放到链表，其他线程可以通过这个链表唤醒此线程
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode); 
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {//cas操作
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }

// 在此方法直到获取锁成功才会跳出循环
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted; //获取锁成功之后才会return跳出此方法
                }
                if (shouldParkAfterFailedAcquire(p, node) && //如果满足阻塞条件
                    parkAndCheckInterrupt()) 
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);//停止线程（自己）
        return Thread.interrupted();
    }
```



### 释放锁

一个线程拿到锁之后，执行完关键代码，必须unlock释放锁的，否则其他线程永远拿不到锁

```java
public void unlock() {
        sync.release(1);
    }

public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
//java.util.concurrent.locks.ReentrantLock.Sync 的tryRelease
 protected final boolean tryRelease(int releases) {
            int c = getState() - releases; //这里一般是 1 - 1 = 0
            if (Thread.currentThread() != getExclusiveOwnerThread()) //只能是锁的拥有者释放锁
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c); //设置state为0，相当于释放锁，让其他线程compareAndSetState(0, 1)可能成功
			
            return free;
        }

protected final void setState(int newState) {
        state = newState; //没有cas操作
    }
```

setState不做cas操作是因为，只有拥有锁的线程才调用unlock，不存才并发混乱问题

其他线程没拿到锁不会设值成功，其他线程在此线程设置state为0之前，compareAndSetState(0, 1)都会失败，拿不到锁，此线程设置state为0之后，其他线程compareAndSetState(0, 1)才有可能成功，返回true从而拿到锁

#### 释放线程

线程在获取锁失败后，有可能阻塞线程（自己），在阻塞之前把阻塞线程信息放入链表的
释放锁之后，线程会尝试通过链表释放其他线程（一个），让一个阻塞线程恢复运行

```java
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
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t; //循环，找到链表最前面需要被唤醒的线程
        }
        if (s != null)
            LockSupport.unpark(s.thread); //唤醒（释放）被阻塞的线程
    }
```



### 阻塞线程被取消阻塞后如何拿到锁(ReentrantLock中)

有时候线程被中断后，唤醒继续执行后面的代码，
线程没有拿到锁之后主动阻塞自己的，但所还没拿到，被唤醒之后怎么去尝试重新获取锁呢？ 里面有一个for循环

```java
final void lock() {
            if (compareAndSetState(0, 1)) 
                setExclusiveOwnerThread(Thread.currentThread());//拿到锁
            else
                acquire(1); //没有拿到锁
        }
// 上锁失败，会添加一个节点，节点包含线程信息，将此节点放入队列
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

// 存好节点后，将线程（自己）中断，等其他线程唤醒（自己）
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {//循环 被唤醒后线程还是在此处循环
                
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {//尝试获取锁
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted; //如果拿到锁了，才会return
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt()) //没拿到锁时，主动中断Thread.currentThread()
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

被唤醒后继续执行`compareAndSetState(0, 1)`返回false没拿到锁，则继续循环或阻塞

`compareAndSetState(0, 1)` 这个操作是获取锁的关键