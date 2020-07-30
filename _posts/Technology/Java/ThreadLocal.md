# ThreadLocal-源码阅读

## 关键

通过阅读源码发现，**ThreadLocal是与Thread类共存关系**，ThreadLocal的出现就是为Thread服务的，Thread的成员变量**ThreadLocals** 的类型就是ThreadLocal的内部类**ThreadLocalMap**

ThreadLocal的内部类 ThreadLocal$ThreadLocalMap 以及 Threadlocal$ThreadLocalMp$Entry

Entry 类继承WeakReference类，泛型为ThreadLocal<?>

### 多线程引用清除

Entry类继承WeakReference类，目的是为了在多线程情况下，对ThreadLocal对象回收时，达到简单，搞笑的效果，是对jvm内存优化的一个技术方案。

通过 Reference的clear方法使引用为空，达到多线程清空效果。

### ThreadLocalMap-关键子类

通过此类达到 ThreadLocal与线程绑定存取值的功能



### Entry

内部成员变量 Object value 存放多线程场景下私有安全变量

### WeakReference 

弱引用，Entry继承此类，在多线程环境下，当ThreadLocal对像需要回收时，此类的引用机制起到作用。

## 疑问，如果Entry不继承WeakRefrence 会有什么后果。

？？？？





### Reference 类 内部有静态代码块，并且启动了线程，可能是垃圾回收