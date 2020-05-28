

[TOC]



# 周阳-面试题

> **1.理论 2.实践 3.小总结**
>
> 方法论 AB

## 一、多线程/高并发

### 1.volatile是什么

volatile是java虚拟机提供的轻量级的同步机制

1. 保证可见性   2.**不保证原子性**   3.禁止指令重排

### 2.JMM之可见性

​	JMM(java内存模型Java Memory Model 简称JMM)本身是一种抽象的概念**并不真实存在**，他描述的是一组规则或规范，通过这组规范定义了程序中各个变量(包括实例字段、静态字段和构成数组对象的元素)的访问方式。

JMM关于同步的规定:

1. 线程解锁前，必须把共享变量的值刷新回主内存。
2. 线程加锁前，必须读取主内存的最新值到自己的工作内存。
3. 加锁解锁是同一把锁

由于**JVM运行程序的实体是线程**，而每个线程创建时JVM都会为其创建一个工作内存(有的地方称为栈空间)，工作内存是每个线程的**私有数据区域**，而**java内存模型中规定所有变量都存储在主内存**中，主内存是共享内存区域，所有线程都可以访问，但线程对变量的操作(读取赋值等)必须在工作内存中进行，**首先**要将变量从主内存拷贝到自己的工作内存空间，然后对变量进行操作，操作完成后在将变量写回主内存，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的**变量副本拷贝**，因此不同的线程间无法访问对方的工作内存，线程间的通信(传值)必须通过主内存来完成，其简要访问过程如下

![jmm](e://pic/jmm内存模型图.png)

### 3.可见性代码实现

```java
class MyData{
    //每个线程都有一个自己主内存中值的一份拷贝，默认使用都是自己内存的值。
    //描述的场景是:在main线程中创建一个线程，通过启动调用修改mydata.value值 然后看主内存线程是否可以读取到
    //updateThread线程的值，如果不可以 说明线程之间的值是不可见的。
    //加上volatile关键字 就可以看到volatile保证了线程之间的可见性。
    volatile int value = 0;

    public void setValue(){
        value = 100;
    }

}

public class VolatileDemo {

    public static void main(String[] args) {
        MyData myData = new MyData();
        new Thread(new Runnable() {
            @Override
            public void run() {
                //线程加入
                System.out.println(Thread.currentThread().getName()+"update value Thread join!");
                try {
                    //线程睡眠3秒钟
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                myData.setValue();
                System.out.println(Thread.currentThread().getName()+"update value success!");
            }
        },"update Thread").start();

        while (myData.value == 0){
            //空等待
        }

        System.out.println(Thread.currentThread().getName()+" game over!");

    }

}
```

**总结**:通过前面对JMM的介绍，我们知道各个线程对主内存中共享变量的操作都是各个线程各自拷贝到自己的工作内存进行操作后在写回到主内存中的。

问题：这就可能存在一个线程AAA修改了共享变量X的值但还没写回到主内存时，另一个线程BBB又对主内存中同一个共享变量X进行操作，但此时A线程工作内存共享变量X对线程B来说是不可见的，这种工作内存与主内存同步延迟线程就造成了可见性问题。

### 4.volatile不保证原子性

代码展示：

```java
class MyData {

    volatile int value = 0;

    public void setValue() {
        value = 100;
    }

    public void addPlus(){
        value++;
    }

}

public class VolatileDemo {

    public static void main(String[] args) {
        MyData myData = new MyData();

        //开启20个线程 i++操作
        for (int i = 0; i < 20; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < 1000; j++) {
                        myData.addPlus();
                    }
                }
            }).start();
        }

        //如果当前线程大于2 说明除了主线程和垃圾回收线程还有别的线程在运行
        //因此mian线程等待其他线程执行完毕之后 在执行main线程的代码
        while (Thread.activeCount()>2){
            Thread.yield();//主线程让步
        }

        System.out.println(Thread.currentThread().getName()+"\t"+myData.value);

    }
```

**为什么不保证原子性操作** ？

![](e://pic/字节码.png)

如何解决？

```java
直接使用juc的atomic原子类 AtomicInteger atomicInteger = new AtomicInteger();//原子性操作
```

### 5.volatile禁止指令重排序

![有序性](e://pic/有序性.png)

![重排](e://pic/重排.png)



![](e://pic/禁止指令重排.png)

可能出现不一致现象。

### 6.禁止指令重排

volatile实现禁止指令重排序优化，从而避免多线程环境下程序出现乱序执行的现象。

先了解一个概念，内存屏障(Memory Barrier) 又称内存栅栏 是一个cpu指令，他的作用有两个

一是**保证特定操作的执行顺序**，

二是**保证某些变量的内存可见性**(利用该特性实现volatile的内存可见性)

由于编译器和处理器都能执行指令重排序优化，如果在指令间插入一条Memory Barrier 则会告诉编译器和cpu 不管什么指令都不能和这条Memory Barrier指令重排序，也就是说通过插入内存屏障禁止在内存屏障前后的指令重排序优化。内存屏障另一个作用是强制刷出各种CPU的缓存数据，因此任何cpu上的线程都能读取到这些数据的最新版本。

![volatile关键字](e://pic/volatile关键字.png)

小结：

> 工作内存与主内存同步延迟现象导致的可见性问题
>
> 可以使用synchroinzed和volatile关键字解决，他们都可以使得一个线程修改后的变量立即对其他线程可见。
>
> 对于指令重排序导致的可见性问题和有序性问题
>
> 可以利用volatile关键字解决，因为volatile的另外一个作用就是禁止重排序优化。

### 7.单例模式问题

1.在一般的单例模式下 单线程环境没有问题 但是在多线程环境下 会出现多次调用初始化构造函数。因此为了完成 可以使用双端检测机制，但是又由于指令重排序问题的存在 必须加上volatile 虽然使用syn可以解决 但是syn是重量级锁。

```java
public class SingleDemo {

    private static volatile SingleDemo instance = null;

    private SingleDemo(){
        System.out.println(Thread.currentThread().getName()+" new instance!");
    }

    //dcl 双端检查
    public static SingleDemo getInstance(){
        if (instance == null){
            synchronized (SingleDemo.class){
                if (instance == null){
                    instance = new SingleDemo();
                }
            }
        }
        return instance;
    }

    public static void main(String[] args) {
        //在单线程环境下 可以保证创建单例对象被唯一的初始化
//        System.out.println(SingleDemo.getInstance() == SingleDemo.getInstance());
//        System.out.println(SingleDemo.getInstance() == SingleDemo.getInstance());
//        System.out.println(SingleDemo.getInstance() == SingleDemo.getInstance());

        //多线程环境下 初始化构造方法可能被初始化了不止一次 使用双端检查
        //因此 就出现了问题
        // 0 new instance!
        //1 new instance!
        for (int i = 0; i < 10 ; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    SingleDemo.getInstance();
                }
            },i+"").start();
        }
    }

}
```

![](E:\pic\单例模式valatile分析.png)

> 但是指令重排序只会保证串行语义的执行的一致性(单线程) 但并不会关心多线程间的语义一致性。所以当一条线程访问instance不为null时，由于instance实例未必已经初始化完成，也就造成了线程安全问题。

### 8.CAS是什么

独占锁：是一种悲观锁，synchronized就是一种独占锁，会导致其他需要锁的线程挂起，等待持有锁的线程释放锁资源。

乐观锁：每次不加锁，假设没有冲突去完成操作，失败了就一直重试，知道成功。

1、CAS

乐观锁就是使用CAS这种机制(Compare And Set)

CAS中有三个数 一个内存中的值，一个预期的值，一个要修改的值，比较内存中的值和预期的值是否相等 如果相等说明在某一时间内没有其他线程修改这个值，说明数据一致性，因此，就进行更新操作。

2、非阻塞算法	

一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。

现代的CPU提供了特殊的指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。

比较并交换 根据当前版本号相同就修改成功 不相同就修改失败。

```java
AtomicInteger atomicInteger = new AtomicInteger(20);
        System.out.println(atomicInteger.compareAndSet(20, 200)+"\t"+atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(20, 100)+"\t"+atomicInteger.get());
```

### 9.CAS底层原理

![](e://pic/UnSafe.png)

![](e://pic/cas.png)

![cas2](e://pic/cas2.png)



3、AtomicInteger源码分析

在没有锁的机制下需要借助volatile原语，保证线程间的数据是可见的(共享)

```
public final int getAndAddInt(Object var1, long var2, int var4) {    
	int var5;    
do {        
	var5 = this.getIntVolatile(var1, var2);   
} while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));//失败重试 通过查看源码    return var5;}
```

### 10.CAS存在的问题

1. 循环时间长开销大
2. 只能保证一个共享变量的原子操作

> 当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作
>
> 但是 对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性。

 3.会有ABA问题

### 11.ABA问题

> CAS--->UnSafe------>CAS底层思想----->ABA------->原子引用更新----------》如果规避ABA问题

![](e://pic/ABA问题.png)

ABA的问题：假设当前有两个线程t1 t2  

t1读取内存的值这个过程会存在一个间隙时间，当t1读取到value=100  但是这个过程中t2读取了该值value=100 并且修改为101  在修改成100 这个过程对于t1线程来说是透明的，因此当t1进行修改的时候 就CAS就可以成功。这就是ABA问题

### 12.ABA问题的解决

对于ABA问题的解决，我们可以使用带有版本号的AtomicStampedReference 来解决 

<https://www.cnblogs.com/549294286/p/3766717.html>

```java
/**
 * @author i
 * @create 2019/10/29 20:17
 * @Description
 */
public class ABA {

    private static AtomicInteger atomicInt = new AtomicInteger(100);
    //维护带有整数“标志”的对象引用，可以用原子方式对其进行更新。
    private static AtomicStampedReference<Integer> atomicStampedRef = new AtomicStampedReference<>(100,0);

    public static void main(String[] args) throws InterruptedException {
        //该线程中修改内存中的值  在修改回来
        new Thread(new Runnable() {
            @Override
            public void run() {
                atomicInt.compareAndSet(100,101);//修改内存中
                atomicInt.compareAndSet(101,100);//在修改回来  这个过程对于其他线程是透明的过程
            }

        }).start();

        //该线程在不知道其他线程修改的前提下 看是否能修改成功
        new Thread(new Runnable() {
            @Override
            public void run() {
                boolean flag = atomicInt.compareAndSet(100, 200);
                System.out.println(flag);//true
            }

        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
//                如果当前引用 == 预期引用，并且当前标志等于预期标志，则以原子方式将该引用和该标志的值设置为给定的更新值。
                atomicStampedRef.compareAndSet(100,101,atomicStampedRef.getStamp(),atomicStampedRef.getStamp()+1);
                atomicStampedRef.compareAndSet(101,100,atomicStampedRef.getStamp(),atomicStampedRef.getStamp()+1);
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                int stamp = atomicStampedRef.getStamp();
                System.out.println("stamp:"+stamp);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                boolean flag = atomicStampedRef.compareAndSet(100,101,stamp,stamp+1);
                System.out.println("flag:"+flag);
            }

        }).start();

    }

}
```

```java
package com.ncst.t1;

import javax.sound.midi.Soundbank;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.atomic.AtomicStampedReference;

/**
 * @author i
 * @create 2019/12/27 16:59
 * @Description 模拟ABA问题
 */
public class ABADemo {

    private static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
    private static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100,1);

    public static void main(String[] args) {
        System.out.println("ABA问题产生");

        new Thread(new Runnable() {
            @Override
            public void run() {
                atomicReference.compareAndSet(100,101);//A
                atomicReference.compareAndSet(101,100);//B
                System.out.println("t1线程执行成功！");
            }
        },"t1").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                atomicReference.compareAndSet(100,200);
                System.out.println("t2线程执行成功!");
            }
        }).start();

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(atomicReference.get());


        System.out.println("ABA问题解决");

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int stamp = atomicStampedReference.getStamp();
                //获取当前版本号标志值 进行修改
                atomicStampedReference.compareAndSet(100,101,1,stamp+1);
                atomicStampedReference.compareAndSet(101,100,atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
                System.out.println(Thread.currentThread().getName()+"\t AB修改成功 当前版本号:"+atomicStampedReference.getStamp());
            }
        },"t3").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int stamp = atomicStampedReference.getStamp();
                System.out.println("t4获取版本号值为:"+stamp);
                boolean flag = atomicStampedReference.compareAndSet(100,2019,1,atomicStampedReference.getStamp()+1);
                System.out.println("修改成功与否:"+flag);
                int stamp2 = atomicStampedReference.getStamp();
                System.out.println("t4最后获取版本号值为:"+stamp2);
            }
        },"t4").start();

    }

}

```

> 总结:通过volatile关键字 保证了可见性，禁止指令重排序，不保证原子操作。引入JMM内存模型中拷贝值与主内存之间数据同步的问题。进而在单例模式中，我们引入volatile关键字来解决。所以为了保证原子操作 我们引入了AtomicXXX相关类，AtomicXXX类底层的实现是通过CAS机制来保证的，CAS中主要是通过期望值来判断其他线程是否修改过，但是这是存在问题的。由此引入了ABA问题。ABA问题的解决时使用AtomicStampedReference来解决，通过时间戳机制保证。类似于版本号机制。

## 二、集合类

### 1.List

```
List<String> list = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    list.add(UUID.randomUUID().toString().substring(0,8));
                    System.out.println(list.toString());
                }
            }).start();
        }

        /***
         * 1.出现的问题
         *   ConcurrentModificationException
```

- 解决方案

> 1.使用Vector
>
> 2.Collections.synchronizedList(new ArrayList<>())
>
> 3.new CopyOnWriteArrayList<String>();//写时复制

- 导致原因

  并发修改异常

2.CopyOnWriteArrayList源码解析

> 主要思想:CopyOnWrite通过加锁机制，复制当前的数组引用 将长度加1后 拷贝到一个新的数组中  在新的数组中进行修改操作。这样做的好处是可以对CopyOnWrite容器进行并发的读 而不需要加锁，因为当前容器不会添加任何元素，所以CopyOnWrite容器也是一种读写分离的思想。

```java
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

### 2.Set

特点

- 不是线程安全的

```java
//测试HashSet是否是并发安全的。
        //不是并发安全的 java.util.ConcurrentModificationException
        //解决办法
//        hashSet = new HashSet<>();
        Set<String> hashSet = new CopyOnWriteArraySet<String>();
        for (int i = 0; i < 20; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    hashSet.add(UUID.randomUUID().toString().substring(0,8));
                    System.out.println(hashSet.toString());
                }
            }).start();
        }

```

解决使用CopyOnWriteArraySet 

CopyOnWriteArraySet 底层是一个CopyOnWriteArrayList

```java
public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
```

HashSet底层源码

> 创建了一个 HashMap 但是HashMap是key - value键值对的结构
>
> Add方法添加使用key value为一个定制的object对象

```java
 public HashSet() {
        map = new HashMap<>();
    }
 public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
 private static final Object PRESENT = new Object();
```

### 3.Map

线程不安全  使用ConcurrentHashMap

```java
  //线程不安全  ConcurrentModificationException
       Map<String,String> map = new ConcurrentHashMap<>();
        for (int i = 0; i < 20; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0,8));
                    System.out.println(map.toString());
                }
            },i+" ").start();
        }
```

### 4.公平锁和非公平锁

>公平锁: 是指多个线程按照申请锁的顺序来获取锁，类似排队打饭，先来先到。
>
>非公平锁:是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。在高并发情况下，有可能会造成优先级反转或者饥饿现象。

![](e:\\pic/公平锁非公平锁.png)

Java ReentrantLock而言

通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。

对于Synchronized而言 就是非公平锁。

### 5.可重入锁(又名递归锁)

1.什么是可重入锁

> 指的是同一个线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码。
>
> 在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。
>
> 也就是说 线程可以进入任何一个它已经拥有的**锁所同步着的代码块**。
>
> **ReentrantLock/Synchronized就是一个典型的可重入锁**
>
> **可重入锁最大的作用是避免死锁**

2.代码演示

> 测试synchronized是否是可重入的

```java
 public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Person.sendMessage();
            }
        },"t1").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                Person.sendMessage();
            }
        },"t2").start();
    }

}


class Person{

    public static synchronized void sendMessage(){
        System.out.println(Thread.currentThread().getName()+"\t sendMessage()");
        sendEmail();
    }

    private static synchronized void sendEmail() {
        System.out.println(Thread.currentThread().getName()+"\t sendEmail()");
    }

}
```

小结 syn是可重入的。

3.测试reentrantLock是否是可重入锁

```java
/**
 * @author i
 * @create 2019/12/28 15:00
 * @Description 测试ReentrantLock是否是可重入的
 */
public class ReentrantLockDemo {

    public static void main(String[] args) {
        LockDemo lockDemo = new LockDemo();
        new Thread(lockDemo,"t1").start();
        new Thread(lockDemo,"t2").start();
    }

}

class LockDemo implements Runnable {

    private Lock lock = new ReentrantLock();

    @Override
    public void run() {
        set();
    }

    private void set() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t set()");
            get();
        } finally {
            lock.unlock();
        }
    }

    private void get() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t get()");
        } finally {
            lock.unlock();
        }
    }
}
```

> 小结 ReentrantLock是可重入的。同一个资源可以加多个锁，但是多个锁必须是成对存在的。否则的话就会出现锁释放不了的情况 一直被阻塞。

### 6.自旋锁

含义:

![](e://pic/自旋锁.png)

代码：实现自旋锁

```java
/**
 * @author i
 * @create 2019/12/28 15:40
 * @Description 实现自旋锁
 */
public class SpinLockDemo {

    private AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void lock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"\t come in lock...");
        //当前引用为null 设置为thread 如果不为null就一直自旋中
        while (!atomicReference.compareAndSet(null,thread)){
            System.out.println(thread.getName()+"自旋中。。。。。。");
        }
    }

    public void unLock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"\t come in unLock...");
        //如果当前的值为thread 就修改成null
        atomicReference.compareAndSet(thread,null);
    }

    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();
        new Thread(new Runnable() {
            @Override
            public void run() {
                spinLockDemo.lock();
                try {
                    Thread.sleep(5000);
                    //t1线程占有
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                spinLockDemo.unLock();
            }
        },"t1").start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(new Runnable() {
            @Override
            public void run() {
                spinLockDemo.lock();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                spinLockDemo.unLock();
            }
        },"t2").start();
    }

}
```

### 7.独占锁(写锁)/共享锁(读锁)/互斥锁

> **独占锁**：指该锁一次只能被一个线程所持有，对ReentrantLock和Synchronized而言都是独占锁。
>
> **共享锁**：指该锁可以被多个线程所持有
>
> 对ReentrantReadWriteLock其读锁是共享锁，写锁是独占锁。
>
> 读锁的共享锁可以保证并发读是非常高效的。读写 写读 写写的过程是互斥的

**总结**

> 多个线程同时读一个资源类没有任何问题，所以为了满足并发量，读取共享资源应该可以同时进行。但是如果有一个线程想去写共享资源类，就不应该再有其他的线程可以对该资源进行读或写
>
>   **总结：读-读能共享    读-写不能共享   写-写不能共享**

2.读写锁代码演示

```java
package com.ncst.lock;

import jdk.internal.org.objectweb.asm.tree.TryCatchBlockNode;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * @author i
 * @create 2019/12/28 16:41
 * @Description  1.演示读写锁问题
 *               2.使用ReentrantReadWriterLock锁来保证读写分离
 */
public class ReadWriteLockDemo {

    //使用colatile关键字保证主内存和各个线程的工作内存保证可见性
    private volatile Map<String, Object> dataMap = new HashMap<>();
    private ReadWriteLock rwLock = new ReentrantReadWriteLock();//读写锁

    public void put(String key, Object value) {
        try {
            rwLock.writeLock().lock();

            System.out.println(Thread.currentThread().getName() + " 正在写入 " + key);
            try {
                TimeUnit.MILLISECONDS.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            dataMap.put(key, value);
            System.out.println(Thread.currentThread().getName() + " 写入完成 " + key);

        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public void get(String key) {
        try {
            rwLock.readLock().lock();
            System.out.println(Thread.currentThread().getName() + " 正在读取 " + key);
            try {
                TimeUnit.MILLISECONDS.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Object result = dataMap.get(key);
            System.out.println(Thread.currentThread().getName() + " 读取完毕 " + key);
        } finally {
            rwLock.readLock().unlock();
        }
    }


    public static void main(String[] args) {
        ReadWriteLockDemo rwkDemo = new ReadWriteLockDemo();
        //并发写
        for (int i = 0; i < 5; i++) {
            final Integer finalNumber = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    rwkDemo.put(finalNumber + "", finalNumber + "");
                }
            }, "t"+i).start();
        }

        //并发读
        for (int i = 0; i < 5; i++) {
            final Integer finalNumber = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    rwkDemo.get(finalNumber + "");
                }
            }, "t"+i).start();
        }

    }

}

```

> 总结:读锁是共享的，写锁是互斥的。虽然 使用syn可以保证读写之间数据的一致性，但是这个代价是比较高昂的。syn是更加粗粒度锁，而读写分离粒度更小。因此使用ReentrantReadWriteLock锁并发度更高。

### 8.CountDownLatch

> 让一些线程阻塞直到另一些线程完成一系列操作后才被唤醒。
>
> CountDownLatch主要有两个办法，当一个或多个线程调用await方法时，调用线程会被阻塞。其他线程调用countDown方法会将计数器减一(调用countDown方法的线程不会阻塞)当计数器的值变为零时，因调用await方法被阻塞的线程会被唤醒，继续执行。

```java
package com.ncst.lock;

import java.util.concurrent.CountDownLatch;

/**
 * @author i
 * @create 2019/12/28 17:30
 * @Description CountDownLatch
 */
public class CountDownLatchDemo {

    private static final Integer MAX_VALUE = 6;

    private static CountDownLatch countDownLatch = new CountDownLatch(MAX_VALUE);

    public static void main(String[] args) {
        for (int i = 1; i <= MAX_VALUE; i++) {
            int finalI = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + "xxxx");
                    countDownLatch.countDown();//计数器每次减一
                }
            }, CountryEnum.getIndex(i).getRetMessage()).start();
        }
        try {
            countDownLatch.await();//等到计数器为0时才执行否则一直处于等待状态
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("班长关门了");
    }

    public static void test1() {
        for (int i = 1; i <= 6; i++) {
            int finalI = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println("第" + finalI + "位同学走了");
                }
            }, i + "").start();
        }

        System.out.println("班长关门了");
    }


}

enum CountryEnum {

    ONE(1, "楚国"), TWO(2, "齐国"), THREE(3, "燕"), FOUR(4, "赵"), FIVE(5, "魏"), SIX(6, "韩");


    private Integer retCode;

    private String retMessage;

    CountryEnum(Integer retCode, String retMessage) {
        this.retCode = retCode;
        this.retMessage = retMessage;
    }

    public Integer getRetCode() {
        return retCode;
    }

    public void setRetCode(Integer retCode) {
        this.retCode = retCode;
    }

    public String getRetMessage() {
        return retMessage;
    }

    public void setRetMessage(String retMessage) {
        this.retMessage = retMessage;
    }

    //根据code 获取 对应的消息值 类似于key value一样
    public static CountryEnum getIndex(Integer index) {
        CountryEnum[] values = CountryEnum.values();
        for (CountryEnum result : values) {
            if (result.getRetCode() == index) {
                return result;
            }
        }
        return null;
    }
}
```

### 9.CyclicBarrier

cyclicBarrier 的字面意思**可循环(Cyclic)使用的屏障(Barrier)**,要做的事情是，让一组线程到达一个屏障(也可以叫做同步点)时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活，线程进入屏障通过CyclicBarrier的await方法。

- 集齐7龙珠使用

```java
private static CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
        System.out.println("集齐7龙珠！");
    });

    public static void main(String[] args) {
        for (int i = 1; i <= 7; i++) {
            int finalI = i;
            new Thread(()->{
                System.out.println("集齐第"+ finalI +"颗");

                try {
                    //线程阻塞
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },i+"").start();
        }
    }
```

### 10.Semaphore

> 信号量主要用于两个目的，一个是用于多个共享资源互斥使用，另一个用于并发线程数的控制。

```
public class SemaphoreDemo {

    private static Semaphore semaphore = new Semaphore(3);

    public static void main(String[] args) {
        for (int i = 0; i < 6 ; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();//从此信号量获取一个许可

                    System.out.println(Thread.currentThread().getName()+"---抢占到车位---");
                    TimeUnit.MILLISECONDS.sleep(3);
                    System.out.println(Thread.currentThread().getName()+"---离开到车位---");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            },i+"").start();
        }
    }

}
```

## 三、队列

### 1.阻塞队列

> 阻塞队列：顾名思义 首先它是一个队列，而一个阻塞队列在数据结构中所起的作用大致如下入所示。

![](e://pic/阻塞队列-2.png)

- 当阻塞队列是空时，从队列中获取元素的操作将会被阻塞。
- 当阻塞队列时满的时，往队列里添加元素的操作将会被阻塞。

试图从空的阻塞队列中获取元素的线程将会被阻塞，直到其他的线程往空的队列插入新的元素。

同样，试图往已满的阻塞队列中添加新元素的线程同样也会被阻塞，直到其他的线程从队列中移除一个元素才可以插入队列中。

### 2.为什么使用阻塞队列 好处?

在多线程领域:所谓阻塞，在某些情况下会挂起线程(即阻塞)，一旦条件满足，被挂起的线程又会自动被唤醒。****

**为什么需要BlockingQueue**

好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BolckingQueue都给你一手包办了，在concurrent包发布以前，在多线程环境下，我们每个程序员都必须去自己控制这些细节，尤其还要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。

### 3.阻塞队列接口和实现类

| **实现类**                 |                  **特点**                  |
| ----------------------- | :--------------------------------------: |
| **ArrayBlockingQueue**  |            **由数组结构组成的有界阻塞队列**            |
| **LinkedBlockingQueue** | **由链表结构组成的有界(但大小默认值有Integer.MAX_VALUE)阻塞队列** |
| PriorityBlockingQueue   |              支持优先级排序的无界阻塞队列              |
| DelayQueue              |            使用优先级队列实现的延迟无界阻塞队列            |
| **SynchronousQueue**    |         **不存储元素的阻塞队列，也既单个元素的队列**         |
| LinkedTransferQueue     |              由链表结构组成的无界阻塞队列              |
| LinkedBlockingDeque     |              由链表结构组成的双向阻塞队列              |

### 4.BlockingQueue的核心方法

![](e://pic/BlockingQueue核心方法.png)

### 5.SynchronousQueue

> SynchronousQueue没有容量。与其他的BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue 每一个put操作都要等到一个take操作。

```java
public class SynchronousQueueDemo {

    public static void main(String[] args) {
        BlockingQueue blockingQueue  = new SynchronousQueue();

        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"存储 a");
                blockingQueue.put("a");
                System.out.println(Thread.currentThread().getName()+"存储 b");
                blockingQueue.put("b");
                System.out.println(Thread.currentThread().getName()+"存储 c");
                blockingQueue.put("c");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        },"t1").start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(5);
                System.out.println(Thread.currentThread().getName()+"。。。获取了。。。"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(5);
                System.out.println(Thread.currentThread().getName()+"。。。获取了。。。"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(5);
                System.out.println(Thread.currentThread().getName()+"。。。获取了。。。"+blockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t2").start();
    }

}
```

### 6.生产者消费者

版本1 synchronized+notify+wait

```java
/**
 * @author i
 * @create 2019/12/29 14:57
 * @Description 生产者消费者 1 版本
 */
class MyShareData{

    private Integer count = 0;//资源

    public synchronized void increment()throws  Exception{
        while (count != 0){
            //如果count 不等于0 当前线程等待
            this.wait();
        }
        count++;
        System.out.println(Thread.currentThread().getName()+" 生产了一个"+count);
        this.notify();
    }

    public synchronized void decrement()throws  Exception{
        while (count == 0){
            //如果count 不等于0 当前线程等待
            this.wait();
        }
        count--;
        System.out.println(Thread.currentThread().getName()+" 消费了一个"+count);
        this.notify();
    }

}

public class ProducterAndCustomer01 {

    public static void main(String[] args){
        MyShareData myShareData = new MyShareData();
        //生产者
        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                try {
                    myShareData.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        },"t1").start();
        //消费者
        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                try {
                    myShareData.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        },"t2").start();
    }

}
```

版本2 lock+Condition+awiait+singal 

> Lock 替代了 synchronized 方法和语句的使用，Condition 替代了 Object 监视器方法的使用

```
/**
 * @author i
 * @create 2019/12/29 15:04
 * @Description 生产者 消费者 2版本
 *  Lock 替代了 synchronized 方法和语句的使用，Condition 替代了 Object 监视器方法的使用
 *
 */
class MyShareData {

    private Integer count = 0;

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    //生产
    public void increment() throws Exception {
        try {

            lock.lock();
            while (count != 0) {
                condition.await();
            }
            count++;
            System.out.println(Thread.currentThread().getName() + " 生产了一个" + count);
            condition.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    //消费
    public void decrement() throws Exception {
        try {
            lock.lock();
            while (count == 0) {
                condition.await();
            }
            count--;
            System.out.println(Thread.currentThread().getName() + " 消费了一个" + count);
            condition.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

}

public class ProducterAndCustomer02 {

    public static void main(String[] args) {
        MyShareData myShareData = new MyShareData();
        //生产者
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    myShareData.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "t1").start();
        //消费者
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    myShareData.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "t2").start();
    }

}

```

版本3  生产者消费者  volatile/BlockingQueue/AtomicInteger

>版本1中通过syn来保证原子操作，但是锁的粒度比较大。重量级。使用notify和wait 并且不确定唤醒哪一个线程有一定的随机性。
>
>版本2中使用Lock 和 condition来进行控制 
>
>版本3 使用volatile BlockingQueue AtomicInteger 来实现组合使用。



```java
/**
 * @author i
 * @create 2019/12/29 16:42
 * @Description 生产者消费者 volatile/BlockingQueue/AtomicInteger
 *  
 */
public class ProdConsumer_BlockQueueDemo {

    private volatile boolean flag = true;//标志位
    private BlockingQueue blockingQueue = null;
    private AtomicInteger atomicInteger = new AtomicInteger();

    //构造
    public ProdConsumer_BlockQueueDemo(BlockingQueue blockingQueue) {
        this.blockingQueue = blockingQueue;
    }

    //生产者
    public void pro() throws InterruptedException {
        String str = null;
        while (flag) {
            str = atomicInteger.incrementAndGet()+"";//原子操作增加值
            System.out.println(Thread.currentThread().getName() + "\t 生产了 "+str);
            blockingQueue.offer(str,1,TimeUnit.SECONDS);//生产
            TimeUnit.SECONDS.sleep(1);
        }
        System.out.println(Thread.currentThread().getName()+" boss叫停了服务 生产者退出。。。");
    }

    //消费者
    public void con()throws  Exception{
        String str = null;
        while (flag){
            str = (String) blockingQueue.poll(2L, TimeUnit.SECONDS);
            if (null == str || str.equalsIgnoreCase("")){
                flag = false;
                System.out.println(Thread.currentThread().getName()+"消费者退出。。。");
                return;
            }
            System.out.println(Thread.currentThread().getName()+" \t 消费了 "+str);
        }
    }

    public void stop(){
        flag = false;
    }

    public static void main(String[] args) throws InterruptedException {
        BlockingQueue blockingQueue = new ArrayBlockingQueue(10);
        ProdConsumer_BlockQueueDemo p = new ProdConsumer_BlockQueueDemo(blockingQueue);

        new Thread(()->{
            try {
                p.pro();
            } catch (Exception e) {
                e.printStackTrace();
            }
        },"p").start();

        new Thread(()->{
            try {
                p.con();
            } catch (Exception e) {
                e.printStackTrace();
            }
        },"c").start();

        TimeUnit.SECONDS.sleep(5);
        p.stop();
    }

}
```



### 7.Synchronized和Lock的区别

> **1.原始构成**
>
> synchronized是关键字属于jvm层面。
>
>   monitorenter(底层是通过monitor对象来完成的，其实wait/notify等方法也依赖于monitor对象只有在同步块或方法中才能调用wait/notify等方法)
>
>  monitorexit
>
>  Lock是具体的类(java.util.concurrent.locks.Lock)是api层面的锁
>
> **2.使用方法**
>
> synchronized 不需要用户去手动释放锁，当synchronized代码执行完后系统会自动让线程释放对锁的占用。
>
> ReentrantLock则需要用户去手动释放锁若没有主动释放锁，就有可能导致出现死锁现象。
>
> 需要lock()和unlock()方法配合try{}finally语句块来完成。
>
> **3.等待是否可中断**
>
> synchronized不可中断，除非抛出异常或正常运行完成。
>
> ReentrantLock可中断。 1.设置超时方法 tryLock(long timeout,TimeUnit unit)
>
> ​				           2.LockInterruptibly()放代码块中，调用interrupt()方法可中断
>
> **4.加锁是否公平**
>
> synchronized非公平锁
>
> ReentrantLock两者都可以，默认非公平锁，构造方法可以传入boolean值 true为公平锁，false为非公平锁。
>
> **5.锁绑定多个条件Condition**
>
> synchronized没有
>
> ReentrantLock用来实现分组唤醒需要唤醒的线程们，可以精确唤醒，而不是想synchonized要么随机唤醒一个线程，要么唤醒全部线程。

### 8.3个线程依序打印ABC

>大体思路:借助juc中Lock+Condition实现，通过一个变量来标志执行哪一个线程 condition中唤醒对应的线程。

```java
/**
 * @author i
 * @create 2019/12/29 16:03
 * @Description  ABC线程每次依序打印 ABCABC循环 10次
 */
class MyShareData{

    private Integer count = 0;
    private Lock lock = new ReentrantLock();
    private Condition cA = lock.newCondition();
    private Condition cB = lock.newCondition();
    private Condition cC = lock.newCondition();

    public void printA(){
        try {
            //1.判断
            lock.lock();
            while (count != 0){
                cA.await();//等待
            }
            //2.工作
            System.out.println(Thread.currentThread().getName()+"\t A");
            //3.唤醒其他线程
            count = 1;
            cB.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public void printB(){
        try {
            //1.判断
            lock.lock();
            while (count != 1){
                cB.await();//等待
            }
            //2.工作
            System.out.println(Thread.currentThread().getName()+"\t B");
            //3.唤醒其他线程
            count = 2;
            cC.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public void printC(){
        try {
            //1.判断
            lock.lock();
            while (count != 2){
                cC.await();//等待
            }
            //2.工作
            System.out.println(Thread.currentThread().getName()+"\t C");
            //3.唤醒其他线程
            count = 0;
            cA.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}

public class SynAndReentrantLockDemo {

    public static void main(String[] args) {
        MyShareData myShareData = new MyShareData();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                myShareData.printA();
            }
        },"t1").start();

        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                myShareData.printB();
            }
        },"t2").start();

        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                myShareData.printC();
            }
        },"t3").start();
    }

}
```

## **四、线程池**

### 1.Callable使用

```java
public class CallableDemo {

    public static void main(String[] args) throws Exception, InterruptedException {
        FutureTask futureTask = new FutureTask(new MyThread());
        new Thread(futureTask).start();

        Integer result = (Integer) futureTask.get();
        
        System.out.println(Thread.currentThread().getName()+" result:"+(result+1));;
    }

}

class MyThread implements Callable<Integer>{

    @Override
    public Integer call() throws Exception {
        System.out.println("xxx");
        return 1;
    }
}
```

### 2.线程池的优势

> 线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量超出数量的线程排队等候，等其他线程执行完毕，在从队列中取出任务来执行。
>
> 主要特点是 线程复用 控制最大并发数 管理线程
>
> 第一：降低资源消耗，通过重复利用已经创建的线程降低线程创建和销毁造成的消耗。
>
> 第二：提供响应速度，当任务到达时，任务可以不需要的等到线程创建就可能立即执行。
>
> 第三：提高线程的客观理性，线程是稀缺资源 如果无限制的创建 不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

### 3.线程池架构图

![](e://pic/线程池架构图.png)

### 4.线程池的创建

- newFixedThreadPool-----------**执行长期的任务，性能好很多**

  ![](e://pic/newFixedThreadPool.png)

- newCachedThreadPool-------------**执行很多短期异步的小程序或者负载较轻的服务器**

- ![](e://pic/newCachedThreadPool.png)

- newSingleThreadExecutor------------- **一个任务一个任务执行的场景**

- ![](e://pic/newSingleThreadExecutor.png)

  

```java
/**
 * @author i
 * @create 2019/12/29 20:10
 * @Description  创建线程的方式
 * 1.实现Runnable中的run方法
 * 2.继承Thread类
 * 3.使用Callable 中call
 * 4.使用ThreadPool
 */
public class ThreadPoolDemo {

    public static void main(String[] args) {
//        ExecutorService threadPool = Executors.newFixedThreadPool(10);//创建一个固定大小的线程池
        ExecutorService threadPool = Executors.newCachedThreadPool();//
        try {
            //模拟20个用户request请求
            for (int i = 1; i <= 20; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"\t 处理了请求");
                });
                TimeUnit.SECONDS.sleep(1);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            threadPool.shutdown();
        }
    }
}
```

### 5.线程池参数详解

1.corePoolSize：线程池中的常驻核心线程数

> 1.在创建了线程池后，当有请求任务来了之后，就会安排池中的线程去执行请求任务，近似理解为今日当值线程
>
> 2.当线程池中的线程数达到corePoolSize后，就会把到达的任务放到缓存队列当中

2.maxmumPoolSize：线程池能容纳同时执行的最大线程数，此值必须大于等于1

3.keepAliveTime：多余的空闲线程的存储时间 

> 当前线程池数量超过了corePoolSize时，当空闲时间达到keepAliveTime值时，多余空闲线程会被销毁直到只剩下corePoolSize个线程为止时。
>
> 默认情况下：只有当线程池中的线程数大于corePoolSize时keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize

4.unit：keepAliveTime的单位

5.workQueue：任务队列，被提交但尚未被执行的任务。

6.threadFactory：表示生产线程池中工作线程的线程工厂，用于创建线程一般用默认的即可。

7.handler：拒绝策略 表示当队列满了并且工作线程大于等于线程池的最大线程数(maxumumPoolSize)

![](e://pic/线程池参数讲解.png)

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
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

### 6.线程池的底层原理

- 图解

  ![](e://pic/线程池底层原理-1.png)

主要流程

> 1.创建了线程池后，等待提交过来的任务请求
>
> 2.当调用execute()方法添加一个请求任务时，线程池会做如下判断
>
>   2.1 如果正在运行的线程数量小于corePoolSize 那么马上创建线程运行这个任务
>
>   2.2 如果正在运行的线程数量大于或等于corePoolSize 那么将这个任务**放入队列**
>
>   2.3 如果这时候队列满了且正在进行的线程数量还小于maxmumPoolSize 那么还是要创建非核心线程立即运行这个任务。
>
>   2.4 如果队列满了且正在运行的线程数量大于或等于maximumPoolSize 那么线程池会**启动饱和拒绝策略来执行**
>
> 3.当一个线程完成任务时，他会从队列中取下一个任务来执行。
>
> 4.当一个线程无事可做超过了一定的时间(keepAliveTime)时，线程池会判断
>
>    如果当前运行的线程数大于corePoolSize 那么这个线程就会停掉。
>
>    所以线程池的所有任务完成后他最终会**收缩到corePoolSize的大小**

### 7.线程池的拒绝策略

#### 7.1 是什么

> 等待队列已经满了，再也塞不下新任务了，同时，线程池中的max线程也达到了，无法继续为新任务服务。这个时候我们就需要拒绝策略机制合理的处理这个问题。

#### 7.2 JDK内置策略模式

> 1.AbortPlicy(**默认**) ：直接抛出RejectedExecutionException异常阻止系统正常运行。
>
> 2.CallerRunsPolicy：调用者运行  一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，
>
> 3.DiscardOldestPolicy：抛弃队列中等待最久的任务，然后将当前任务加入队列中尝试再次提交当前任务。
>
> 4.DiscardPolicy：直接丢弃任务，不予任何处理也不抛出异常，如果允许任务丢失，这是最好的一种方案。

#### 7.3 生产中使用哪一个线程池

![](e://pic/线程池工作.png)

#### 7.4 自定义线程池

```java
public static void main(String[] args) {
        ExecutorService threadPool =
                new ThreadPoolExecutor(3,//corePoolSize 核心处理线程数
                        5,//最大线程处理数
                        1L,//等待时间
                        TimeUnit.SECONDS,//时间单位
                        new LinkedBlockingQueue(3),//链表阻塞队列  不赋值默认为最大值
                        Executors.defaultThreadFactory(),//创建一个默认线程池
                       // new ThreadPoolExecutor.AbortPolicy()//超出最大线程池数直接抛出异常
                       // new ThreadPoolExecutor.CallerRunsPolicy()//超出线程回退到调用者线程
                       // new ThreadPoolExecutor.DiscardOldestPolicy()//抛弃任务
                        new ThreadPoolExecutor.DiscardPolicy()//直接抛弃任务
        );
        try {
            //模拟20个用户request请求
            for (int i = 1; i <= 9; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"\t 处理了请求");
                });
//                TimeUnit.MILLISECONDS.sleep(200);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            threadPool.shutdown();
        }
    }
```

#### 7.5 线程池参数如何配置

##### 7.5.1 获取当前机器的核心数

```
Runtime.getRuntime().availableProcessors()
```

##### 7.5.2 CPU密集型

> CPU密集的意思是该任务需要大量的运算，而没有阻塞。cpu会一直全速运行。
>
> CPU密集任务只有在真正的多个CPU上才可能得到加速(通过多线程)
>
> 而在单核CPU上() 无论你开几个模拟的多线程任务都不可能得到加速，因为cpu总的运算能力就这些。
>
> CPU密集型任务配置尽可能少的线程数量。****
>
> **一般公式：CPU核数+1个线程的线程池**

##### 7.5.3 I/O密集型

> 1.由于IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如CPU*2
>
> 2.IO密集型 即该任务需要大量的IO 即大量的阻塞
>
> 在单线程上运行的IO密集型的任务会导致浪费大量的CPU运算能力浪费在等待。
>
> 所以在IO密集型任务中使用多线程可以大大的加速程序的运行 即使在单核cpu上，这种加速主要是利用被浪费掉的阻塞时间。
>
> IO密集型时，大部分线程都阻塞，故需要多配置线程数。
>
> 参考公式 cpu核数/1-阻塞系数   阻塞系数在0.8-0.9之间
>
> 比如8核CPU   8/1-0.9 = 80

### 8.死锁编码以及定位分析

#### 8.1  什么是死锁

![](e://pic/死锁是什么.png)

#### 8.2  死锁代码

```java
package com.ncst.jvm;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author i
 * @create 2019/12/30 11:15
 * @Description 死锁出现必定是两个或多个线程竞争同一资源造成的。
 * A持有1资源 去获取
 */

class MyShareData implements Runnable{

    private AtomicInteger atomicInteger = new AtomicInteger();
    private Object lock1 = null;
    private Object lock2 = null;


    public MyShareData(Object lock1, Object lock2) {
        this.lock1 = lock1;
        this.lock2 = lock2;
    }



    @Override
    public void run() {
        synchronized (lock1){
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lock2){
                atomicInteger.incrementAndGet();
            }
        }
    }
}

public class DeadLockDemo {

    public static void main(String[] args) {
        Object lock1 = new Object();
        Object lock2 = new Object();
        new Thread(new MyShareData(lock1,lock2),"t1").start();

        new Thread(new MyShareData(lock2,lock1),"t2").start();

    }

}

```

#### 8.3 死锁解决

> jps命令定位进程号   jstack找到死锁查看

## **五、JVM**

### 1.jvm前期知识

#### 1.1 JVM体系结构

![](e://pic/jvm体系结构.png)

#### 2.GC作用域

![](e://pic/gc作用域.png)

#### 3.常见的4种垃圾回收算法

##### 3.1引用计数

![](e://pic/引用计数.png)

##### 3.2 复制

![](e://pic/复制算法.png)

##### 3.3 标记清除

![](e://pic/垃圾收集算法.png)

##### 3.4 标记整理

![](e://pic/标记压缩.png)

### 2.GC Roots

#### 2.1 什么是垃圾

> 简单的说就是内存中已经不再被使用到的空间就是垃圾。

#### 2.2  判断一个对象是否可回收

1.引用计数法

> java中，引用和对象是有关联的，如果要操作对象就必须用引用进行。
>
> 因此，很显然一个简单的办法就是通过引用计数来判断一个对象是否是可回收的，简单的说，给对象中添加一个引用计数器
>
> 每当有一个地方引用它，计数器值加1
>
> 每当有一个引用失效，计数器减1 
>
> 任何时刻计数器值为零的对象就是不可能再被使用的，那么这个对象就是可回收对象。
>
> 那为什么主流的java虚拟机里面没有选用这种算法呢 
>
> 其中最主要的原因就是他很难解决对象之间相互循环引用的问题。

2.枚举根节点做可达性分析

![](e://pic/可达性分析.png)

![](e://pic/case.png)

#### 2.3 可以作为GC Roots

> 1.虚拟机栈(栈帧中的局部变量区，也叫做局部变量表)中引用的对象。
>
> 2.方法区中的类静态属性引用的对象
>
> 3.方法区中常量引用的对象
>
> 4.本地方法栈中JNI（Navite方法）引用的对象

### 3.jvm调参

3.1 jvm参数类型 

**1.标准参数**

```
-version   -help java -showversion
```

**2.X参数**

```
-Xint -- 解释执行
-Xcomp  -- 第一次使用就编译成本地代码
-Xmixed -- 混合模式
```

![](e://pic/x参数.png)

**3.xx参数**

**1.Boolean** 

```java
-XX:+ 或者 -某个属性值 
+表示开启   -表示关闭
-XX:+PrintGCDetails //是否打印GC收集细节
-XX:+UseSerialGC //是否使用串行垃圾收集器
```

![](e://pic/-gc.png)

![](e://pic/+gc.png)

**2.kv设值类型**

```
-XX:属性key=属性值value
-XX:MetaspaceSize = 128m  //元空间大小
-XX:MaxTenuringThreshold=15  //多少岁移动到老年代 默认15
```

![](e://pic/MetaspaceSize.png)

![](e://pic/MetaspaceSize.png)

**jinfo -flags**

![](e://pic/-flags.png)

> 两个经典参数 -Xms和-Xmx
>
> -Xms 等价于 -XX:InitialHeapSize //初始化堆空间
>
> -Xmx 等价于 -XX:MaxHeapSize  //最大堆空间

```java
java -XX:+PrintFlagsInitial //查看默认初始化值
java -XX:+PrintFlagsFinal //主要查看修改更新  = 表示默认的  ：= 表示 jvm或者人为修改过
```

![](e://pic/查看jvm默认初始化值.png)

![](e://pic/查看jvm最终配置.png)

### 4.工作中常用的jvm参数

基础知识

![](e://pic/元空间.png)

```java
long totalMemory = Runtime.getRuntime().totalMemory();//返回java虚拟机中内存总量  最大	堆内存的空间 为内存的16/1
        long maxMemory = Runtime.getRuntime().maxMemory();//返回java虚拟机试图使用的最大内存容量  最大jvm堆内存空间 为内存的4/1
        System.out.println("Total_MEMORY(-Xms)="+totalMemory+"字节、"+(totalMemory/(double)1024/1024)+"MB");
        System.out.println("MAX_MEMORY(-Xmx)="+maxMemory+"字节、"+(maxMemory/(double)1024/1024)+"MB");
```

常用参数

```java
-Xms //初始化大小内存 默认为物理内存1/64 等价于 -XX:InitialHeapSize
-Xmx //最大分配内存  默认为物理内存1/4  等价于 -XX:MaxHeapSize
-Xss //设置单个线程的大小  一般默认为512k~1024k  等价于 -XX:ThreadStackSize
-Xmn //设置年轻代大小 
-XX:MetaspaceSize //设置元空间大小  元空间的本质和永久代类似，都是对jvm规范中方法区的实现。
 //不过元空间与永久代之间最大的区别在于 元空间并不在虚拟机中 而是使用本地内存 
 //因此 默认情况下  元空间的大小受本地内存限制
  -Xms10m -Xmx10m -XX:MetaspaceSize=1024m -XX+PrintFlagsFinal
-Xms128m -Xmx4096m -Xss1024k -XX:MetaspaceSize=512m  -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseSerialGC //设置初始化内存 128 最大堆内存4g  单线程数1024k
//元空间大小512m  打印日志  
-XX:SurvivorRatio //设置	新生代中eden和s0/s1空间的比例 默认-XX:SurvivorRatio=8,Eden:S0:S1=8:1:1 
//假如 -XX:SurvivorRatio = 4,Eden:S0:S1=4:1:1
//SurvivorRatio 值就是设置eden区的比例占多少 S0/S1相同
-XX:NewRatio //配置年轻代与老年代在堆结构的占比默认 -XX:newRatio=2新生代占1 老年代2 年轻代占整个堆的1/3 -XX:NewRatio=4新生代占1 老年代4 年轻代占整个堆的1/5 NewRatio值就是设置老年代的占比 剩下的1给新生代
-XX:MaxTenuringThreshold//设置垃圾最大年龄
```

-XX:+PrintGCDetails //输出详细的GC收集日志信息

![](e://pic/1.png)

![](e://pic/2.png)

![](e://pic/3.png)

### 5.引用类型

![](e://pic/4.png)

#### 1.强引用(默认支持模式)

![image-20200527102522023](e:\pic\image-20200527102522023.png)

```java
	//强引用
    public static void main(String[] args) {
        Object obj1 = new Object();//这样定义的默认就是强引用
        Object obj2 = obj1;
        obj1 = null;
        obj2 = null;
        System.gc();
        System.out.println(obj2);
    }
```

#### 2.软引用

![image-20200527102615616](e:\pic\image-20200527102615616.png)

```java
	//内存够用的情况
    public static void softRef_Memory_Enough(){
        Object obj1 = new Object();
        SoftReference<Object> softReference = new SoftReference<>(obj1);
        System.out.println(obj1);
        System.out.println(softReference);

        obj1 = null;//回收obj1
        System.out.println(obj1);
        System.out.println(softReference);//内存够用 不回收软引用
    }

    //内存不够用的情况
    //jvm配置 故意产生大对象并配置小的内存，让它内存不够用了导致OOM 看软引用的回收情况
    //-Xms5m  -Xmx5m -XX:+PrintGCDetails 最大堆内存5M
	//-Xms 初始化内存大小 -Xmx 最大内存
    public static void softRef_Memory_NotEnough(){
        Object obj1 = new Object();
        SoftReference<Object> softReference = new SoftReference<>(obj1);
        System.out.println(obj1);
        System.out.println(softReference.get());

        obj1 = null;

        try {
            byte[] bytes = new byte[30 * 1024 * 1024];
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            System.out.println(obj1);
            System.out.println(softReference.get());//内存不够 回收软引用
        }
    }
```

#### 3.弱引用

![image-20200527102808902](e:\pic\image-20200527102808902.png)

```java
	//对于软引用来说，只要垃圾回收机制一运行，不管jvm的内存是否足够，都会回收该对象占用的内存。
    public static void main(String[] args) {
        Object obj1 = new Object();
        WeakReference<Object> weakReference = new WeakReference<>(obj1);
        System.out.println(obj1);
        System.out.println(weakReference.get());
        obj1 = null;
        System.gc();
        System.out.println("============");

        System.out.println(obj1);
        System.out.println(weakReference.get());
    }
```

##### 软引用和弱应用的适用场景

![image-20200527102911951](e:\pic\image-20200527102911951.png)

##### 你知道弱引用的话，能谈谈WeakHashMap吗？

```java
	//弱引用 一旦发生gc 就回收
    private static void myWeakHashMap() {
        WeakHashMap<Integer,String> hashMap = new WeakHashMap<>();
        Integer integer = new Integer(1);
        hashMap.put(integer,"hashmap");

        System.out.println(hashMap);

        //null
        integer = null;
        System.out.println(hashMap);

        //gc
        System.gc();
        System.out.println(hashMap);
    }

    //分析 因为是强引用 所以integer null 和 gc不会发生回收。
    private static void myHashMap() {
        HashMap<Integer,String> hashMap = new HashMap<>();
        Integer integer = new Integer(1);
        hashMap.put(integer,"hashmap");

        System.out.println(hashMap);

        //null
        integer = null;
        System.out.println(hashMap);

        //gc
        System.gc();
        System.out.println(hashMap);
    }
```

#### 4.虚引用

![image-20200527103028697](e:\pic\image-20200527103028697.png)

##### 引用队列

![image-20200527103053443](e:\pic\image-20200527103053443.png)

```java
	//分析 弱引用只有在发生gc的时候 才会被回收。而gc之后会进入到引用队列中。
    public static void main(String[] args) {
        ReferenceQueue<Integer> referenceQueue = new ReferenceQueue();
        Integer integer = new Integer(1);
        WeakReference<Integer> weakReference = new WeakReference<>(integer,referenceQueue);

        System.out.println(integer);
        System.out.println(weakReference.get());
        System.out.println(referenceQueue.poll());

        //integer == null
        integer = null;

        System.out.println("=====================");
        System.out.println(integer);
        System.out.println(weakReference.get());
        System.out.println(referenceQueue.poll());

        System.gc();
        System.out.println("=====================");
        System.out.println(integer);
        System.out.println(weakReference.get());
        System.out.println(referenceQueue.poll());
    }
```

#### 5.GCRoots和四大引用的小总结

![image-20200527103142271](e:\pic\image-20200527103142271.png)

### 6.OOM

#### 1.Java.lang.StackOverflowError

```java
	public static void main(String[] args) {
        stackOverflowError();
    }

    private static void stackOverflowError() {
        stackOverflowError(); //Exception in thread "main" java.lang.StackOverflowError
    }
//递归调用 超出栈深度
```

#### 2.Java. heap space

```java
 	public static void main(String[] args) {
        byte [] bytes = new byte[100*1024 * 1024];
        //java.lang.OutOfMemoryError: Java heap space
    }
```

#### 3.GC overhead limit exceeded

![image-20200527105547316](e:\pic\image-20200527105547316.png)

```java
/**
     * JVM参数配置演示
     * -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize =5m
     *
     * GC回收时间过长会抛出OutOfMemoryError 过长的定义时 超过%98的时间来做GC 并且回收了不到2%的堆内存。
     * 连续多次GC都只回收了不到2%的极端情况才会抛出，假如不抛出GC overhead limit 错误会发生什么情况呢
     * 那就是GC清理的这么点内存很快再次填满，迫使GC再次执行，这样就形成恶性循环。
     * cpu实用率一直是100% 而GC确没有任何成果。
     * @param args
     */
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        int i = 0;
        try {
            while (true){
                list.add(new String(""+i++).intern());
            }
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("=========================== i=:"+i);
            throw  e;
        }
    }
```

#### 4.Direct buffer memory

![image-20200527113104334](e:\pic\image-20200527113104334.png)

```java
	//参数配置 -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize = 5m
    public static void main(String[] args) {
        //最大堆外内存
        System.out.println("配置的maxDirectMemory:"+ (sun.misc.VM.maxDirectMemory()/(double)1024/1024)+"MB");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //实际分配5mb堆外内存
        ByteBuffer bb = ByteBuffer.allocateDirect(6*1024*1024);
    }
```

#### 5.unable to create new native thread 

![image-20200527115145083](e:\pic\image-20200527115145083.png)

```java
	public static void main(String[] args) {
        for (int i = 0;  ; i++) {
            System.out.println(i);
            new Thread(new Runnable() {
                @Override
                public void run() {

                }
            }).start();
        }
    }
```

##### OOM之unable to create new native thread上限调整

1.非root用户登录Linux系统测试

![image-20200527173445525](e:\pic\image-20200527173445525.png)

![image-20200527173617828](e:\pic\image-20200527173617828.png)

#### 6.Metaspace

![image-20200527173929556](e:\pic\image-20200527173929556.png)

使用Java -XX:+PrintFlagsInitial命令查看本机的初始化参数，-XX:MetaspaceSize为21810376B(约20M)

![image-20200527174640286](e:\pic\image-20200527174640286.png)

### 7.G垃圾回收算法和垃圾收集器的关系?分别是什么请你谈谈

GC算法（引用计数/复制/标清/标整）是内存回收的方法论，垃圾收集器就是算法落地实现

因为目前为止还没有完美的收集器出现，更加没有万能的收集器，只是针对具体应用最合适的收集器，进行分代收集。

4种主要垃圾收集器

![image-20200527180245019](e:\pic\image-20200527180245019.png)

![image-20200527180301447](e:\pic\image-20200527180301447.png)

![image-20200527181047302](e:\pic\image-20200527181047302.png)























































## 六、github



### 1.常用词含义**

```
watch ：会持续收到该项目的动态

fork：复制某个项目到自己的github仓库中

star：可以理解为点赞

clone  ：将项目下载至本地

follow ：关注你感兴趣的作者，会收到他们的动态
```

### **2.in关键字限制搜素范围**

```
公式：xxx关键字 in：name 或description 或readme

xxx in : name 项目名包含xxx的

xxx in : description 项目描述包含xxx

xxx in : readme 项目的 readme文件中包含xxx的

组合使用： 搜索项目名或者readme中包含秒杀的项目

seckill in : name ,readme;
```

### **3.star 和 fork范围搜索**

```
公式    xxx关键字  stars 通配符  :>  或者 :>=

区间范围数字  数字1..数字2
eg: springboot stars:>=5000 
springboot forks:>500
组合使用 查找fork在100到200之间并且stars数在80到100之间的springboot项目
springboot forks:100..200 stars:80..100
```

### **4. awesome 加强搜索**

```
awesome xxx  //awesome 一般是用来收集学习 工具 书籍类相关的项目
搜索优秀的redis相关的项目  包括框架 教程等
```

### **5.高亮显示某一行代码**

```
公式  1行 地址后面紧跟#L数字   多行 地址后面紧跟#L数字-L数字2
给别人指出关键代码的行号
地址+#l 
xxx//OnLinkedList.java#L8-L20
```

### **6.项目内搜索**

```
英语字母 t 
```

### **7.搜索某个地区的大佬**

```
公式  location 地区 language 语言
地区北京的java方向的用户 location:beijing language:java
```

## 七、linux

### **1.生产环境服务器变慢，诊断思路和性能评估谈谈**

```
1、整机
1.top //看cpu和内尺暂用率 
uptime//系统性能精简版
```

#### 2.查看CPU

```
[root@nsrfgobohh1t8oym-0312025 ~]# vmstat -n 2 3	
查看额外
- 查看所有cpu核信息  mpstat -P ALL 2 
- 每个进程使用cpu的用量分解信息 pidstat -u 1 p [进程编号]
```

