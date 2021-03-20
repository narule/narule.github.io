---
title: AbstractQueueSyncroniser抽象类-阻塞锁
author: Narule
date: 2021-03-20 00:10:00 +0800
categories: [Technology^技术, Java]
tags: [writing, Java，AbstractQueueSyncroniser]
---



# AbstractQueueSyncroniser抽象类-阻塞锁

AbstractQueuedSynchronizer 是多线程情况下保证代码快有序运行的一种设计，对多线程获取锁进行了抽象，

设计中包括： 

1.线程队列（阻塞的线程） 

当很多线程竞争锁的时候，排队一个个获取锁，先进先出

2.线程获取锁，成功就执行后面的代码

3.线程释放锁，如果等待队列中有线程，唤醒队列中的线程

关键方法有

addWaiter 添加阻塞线程到队列 

acquire 获取锁，

release 释放锁。



## 线程等待队列

### 线程队列结构

队列是双向链表，通过prev（前） 和 next （后）连接，

整个队列有两个初始变量，head 与 tail分别代表队列的头部和尾巴。唤醒线程就是通过prev寻找最前面的节点，添加阻塞线程就是通过.next加到尾部，先进先出的处理逻辑。





## 获取锁

acquire是一个获取锁的模板方法，逻辑字面上可以理解

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException(); //抛出异常
}
```

1  tryAcquire 尝试获取锁

2 addWaiter  获取锁失败，当前线程信息（自己）放入队列

3 acquireQueued 放入队列后，阻塞自己，等待其他线程释放锁后唤醒（自己）

tryAcquire 方法是抛出异常，是要我们自己写具体逻辑的，实现不同类型的个性化锁，ReentranLock类内部有实现逻辑的列子。



### 获取锁失败-放入队列

addWaiter方法是将创建一个节点，并且这个节点包含当前线程信息，整个是一个链表结构的队列。

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode); //1
    Node pred = tail;
    if (pred != null) { 
        node.prev = pred; //2
        if (compareAndSetTail(pred, node)) {
            pred.next = node; //3
            return node;
        }
    }
    enq(node);
    return node;
}
//在每次创建一个node节点的时候， 将尾部更新，

//node.prev = pred;  设置前一个节点

//compareAndSetTail(pred, node) 更新尾巴为新加入的节点（自己）

//pred.next = node;   尾巴的next更新
```

1 通过上面的代码能知道大致思想是，创建一个携带当前线程信息的node，

2 如果队列的尾部tail不为空，node的前节点设置为pred，`Node pred = tail; node.prev = pred;`

3 并且更新一下链表尾部，`compareAndSetTail`  实际的效果就是 tail = node; 更新尾部节点。



## 释放锁

release 与 acquire对应，是将锁释放，并且在锁释放之后，唤醒队列头部的线程



```java
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h); //唤醒线程
            return true;
        }
        return false;
    }

protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException(); //抛出异常
    }

```

### 唤醒线程

unparkSuccessor

```java
private void unparkSuccessor(Node node) {

    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);


    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev) //循环获取，找到队列里最前面的节点
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread); //唤醒队列中的线程
}
```





## CAS原子操作数

CAS 指的是 compareAndSwap 比较并交换技术，



```java
private volatile int state;
protected final int getState() {
        return state;
}

protected final void setState(int newState) {
    state = newState;
}

protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```



**简言之，类中state变量，对他的操作时，传入两个值，分别是期望原值，更新值，如果期望原值==原值，才能更新成功，否则更新失败，**



线程Thread1  调用 compareAndSetState(0,1) 如果成功，state的值将变为1，其他线程compareAndSetState(0,1)都会失败，因为其他线程期望原值 state = 0， 实际 state值已经被Thread1  线程修改为1。需要Thread1  执行完代码块之后，将state值改回0释放锁，其他线程才能修改成功，获取到锁。



