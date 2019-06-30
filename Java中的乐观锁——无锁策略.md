---
title: "Java并发——乐观锁（无锁策略）"
date: 2018-11-25 18:41:23
tags: CAS
categories: 夯实基础
summary:  "题主在阅读《Java高并发程序设计》一书时，了解到了Java无锁的相关概念，在此记录下来以加深对其的理解，Java中的锁分为两种即为悲观锁和乐观锁，那么何为悲观锁和乐观锁呢？" 
---
题主在阅读《实战Java高并发程序设计》一书时，了解到了Java无锁的相关概念，在此记录下来以加深对其的理解，Java中的锁分为两种即为悲观锁和乐观锁，那么何为悲观锁和乐观锁呢？<!-- more -->
### 乐观锁与悲观锁
悲观锁是我们代码经常用到的，比如说Java中的`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现，它总是假设别的线程在拿数据的时候都会修改数据，所以在每次拿到数据的时候都会上锁，这样别的线程想拿这个数据就会被阻塞直到它拿到锁。  
乐观锁与之相反，它总是假设别的线程取数据的时候不会修改数据，所以不会上锁，但是会在更新的时候判断有没有更新过数据。也就是，乐观锁（无锁）使用一种比较交换的技术（`CAS` `Compare And Swap`）来鉴别线程冲突，一旦检测到冲突的产生，就重试当前操作直到没有冲突的产生。  
与锁相比，使用比较交换（CAS）会使代码看起来更加复杂一些。但由于其非阻塞性，它对死锁问题天生免疫，并且线程之间的相互影响也远远比基于锁的方式要小。更为重要的是，使用无锁的方式完全没有锁竞争带来的系统开销，也没有线程之间频繁调度带来的开销，因此，它要比基于锁的方式拥有更优越的性能。
### 乐观锁实现
乐视锁的实现之一就是CAS算法，CAS算法的过程大致是这样的：它包含三个参数CAS(V, E, N)。
- `V`表示要更新的变量
- `E`表示预期值
- `N`表示新值
仅当V等于E的时候，才会把V的值设置成N，否则不会执行任何操作（比较和替换是一个原子操作）。如果V值和E值不相等，则说明有其他线程修改过V值，当前线程什么都不做，最后返回当前V的真实值。CAS操作是抱着乐观的态度进行的，它总是认为自己可以成功的完成操作。当多个线程同时使用CAS操作一个变量时，只有一个会成功更新，其余都会失败。失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。

### 乐观锁在JDK中的应用
在`java.util.concurrent.atomic`包下面的原子变量类就是使用了CAS来实现的，下面我们重点看一下CAS在该包下面的AtomicInteger类实际应用，该类提供下面几个核心方法和属性：
- public final int incrementAndGet()  // 当前值加1，返回旧值
- public final int decrementAndGet() // 当前值减1，返回旧值
- public volatile int value // AtomicInteger对象当前实际取值

`incrementAndGet()`和`decrementAndGet()`方法类似，我们只看一下incrementAndGet方法就好，JDK1.7与JDK1.8在实现`incrementAndGet()`方法有所区别（[Java8中CAS的增强](http://ifeve.com/enhanced-cas-in-jdk8/)），下面给出的是在java8中的实现，可以看到incrementAndGet()实际调用的是`sun.misc.Unsafe.getAndAddInt`方法，Unsafe类可以理解为Java中指针，但是我们不可以直接使用，因为它是由Bootstrap类加载器加载，而非AppLoader加载。
```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;  // 
}
```
代码中的`valueOffset`代表value字段在AtomicInteger对象中的偏移量（到对象头部的偏移量），方便快速定位字段。
```java
public final int getAndAddInt(Object obj, long l, int i)
{
    int j;
    do
        j = getIntVolatile(obj, l);
    while(!compareAndSwapInt(obj, l, j, j + i));
    return j;
}
```
传入getAndAddInt方法的参数分别是obj(AtomicInteger对象)、l(对象内偏移量)、i(增加值)，可以看到getAndAddInt实际是一个循环，只有compareAndSwapInt返回true时，循环才能结束，并返回`j`(旧值)，下面是compareAndSwapInt方法签名，其中前面两个参数和传入getAndAddInt方法参数一致，后面expected的值是通过getIntVolatile获取的旧值，x是希望设置的新值。
```java
public final native boolean compareAndSwapInt(Object obj, long offset, int expected, int x);
```
与compareAndSwapInt方法类似，getIntVolatile()内部也是用原子操作获取AtomicInteger对象的value值，下面是该方法的签名
```java
public native int getIntVolatile(Object obj, long l);
```
CAS在JDK源码中应用广泛，下面给出其余的无锁的类：
- `AtomicReference`  无锁的对象引用
- `AtomicStampedReference` 带有标志的对象引用
- `AtomicIntegerArray` 无锁的数组
- `AtomicIntegerFieldUpdater` 无锁的普通变量
### 乐观锁的问题
**ABA问题**
如果一个变量V初次读取是A值，并且在准备赋值的时候也是A值，那就能说明A值没有被修改过吗？其实是不能的，因为变量V可能被其他线程改回A值，结果就是会导致CAS操作误认为从来没被修改过，从而赋值给V。
JDK 1.5以后提供了上文所说的`AtomicStampedReference类`来解决了这个问题，其中compareAndSet方法会首先检查当前引用是否等于预期引用，其次检查当前标志是否等于预期标志，如果都相等就会以原子的方式将引用和标志都设置为新值。
**自旋时间长**
CAS自旋就是上文说的getAndAddInt()方法内部do-while循环，如果compareAndSwapInt一直未设置成功，do-while一直循环下去，会给CPU带来非常大的执行开销。网上给出执行方法如下，unchecked（还没试过~）
>  如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

**只能保证单个共享变量**
CAS操作只对单个共享变量有用，涉及多个变量时无法使用CAS，同样在JDK 1.5之后，提供了AtomicReference对象引用，可以多个变量放到一个AtomicReference对象里。
### 使用场景
>  简单的来说CAS适用于写比较少的情况下（多读场景，冲突一般较少），synchronized适用于写比较多的情况下（多写场景，冲突一般较多）
### 参考文档
- 实战Java高并发程序设计
- [乐观锁与悲观锁](https://github.com/Snailclimb/JavaGuide/blob/master/%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87/%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87%E4%B9%8B%E4%B9%90%E8%A7%82%E9%94%81%E4%B8%8E%E6%82%B2%E8%A7%82%E9%94%81.md)
- [Java8中CAS的增强](http://ifeve.com/enhanced-cas-in-jdk8/)
