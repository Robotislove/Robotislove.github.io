---
layout: post
title: 深入并行编程锁优化笔记
date: 2022-08-24
author: lau
tags: [C++, Blog]
comments: true
toc: true
pinned: false
---
并行编程锁优化，这里记录它的原理和实践。

<!-- more -->

![](http://assets.processon.com/chart_image/6305c9f31efad40752c5cbeb.png)

## 优化方法分析
### 使用自旋锁
普通的锁，比如pthread的mutex基于OS提供的休眠和唤醒机制，如果锁的临界区很短，会造成CPU资源的浪费，并且延迟也比较高。这种情况我们可以考虑使用自旋锁——等待自旋锁的线程不会放弃CPU进入休眠，而是不断的检查锁占用标志，一旦持有锁的线程设置标志释放锁，等待线程可以立即进入临界区。

在测试环境上对pthread线程库的mutex和spinlock的做性能对比。

随着临界区越来越短，自旋锁的相对MUTEX的性能优势越来越明显，当临界区缩短到8us以后，自旋锁可以达到2倍以上的提升。

如果临界区持续时间很长，spinlock并不会比mutex性能差，但CPU占用率会很高，一直是400%（4个核，4\*100%）；而mutex逐渐接近于100%。主要是因为spinlock等锁时不放弃CPU，空转。所以spinlock应该在临界区比较短的情况下使用，否则很浪费CPU。

然而由于应用程序处于用户态，它的执行可被任意抢占，临界区的执行时间不能准确预期，有可能被耽搁“很久”。所以应用程序中使用自旋锁的实践并不多。

Facebook于今年六月份放出了他们针对多核环境的C++高性能库- folly，里面对自旋锁实现做了改进，更适用于用户态程序。这种改进非常简单，就是先自旋后休眠：

这个简单的改进，让自旋锁具有了很好的适应性，可以在应用程序中使用，多数情况下我可以很快，少数极端情况下临界区的执行被耽搁了，我也不至于傻转，浪费CPU。

simple::SpinLock是采用这种方式实现的一个C++自旋锁。和mutex的性能、CPU占用对比情况如下：

simple::SpinLock 在临界区时间很短的时候相对pthread_mutex的性能提升明显，和pthread_spinlock的表现一致。但CPU占用只有pthread_spinlock的一半左右！

simple::SpinLock 在临界区时间比较长的时候相对pthread_mutex的性能没有提升，和pthread_spinlock的表现一致。但CPU占用确很低，接近于pthread_mutex。

综上所述，**多核环境下对于锁争抢比较频繁、并且临界区很短的代码，可以使用simple::SpinLock这种带休眠的自旋锁代替pthread_mutex提升性能。**

### 使用读写锁

4个线程争抢一个锁，实际上同一时刻只有1个线程可以干活儿，无论我们怎么去优化锁的粒度（自旋是一种粒度优化），临界区的并行度仍然是1，白白浪费了3个CPU。如果CPU个数更多，参与争抢的线程数更多，这种浪费更明显，达到一种不可接受的程度。
从结构上对锁做出优化，是一种有效的思路。多数场景，对临界区同时读是没有问题的，也就是说只要没有人写临界区，读是没必要等待的，可以做到完全的并行。pthread_rwlock 就是这样一个锁。

随着写占用的比例越来越少，pthread_rwlock相对mutex提升越来越明显。最后可以达到3倍左右的提升。（理论上如果写是0%，应该是4倍）。

所以，**多核环境下对于锁争抢比较频繁、并且写明显少于读的代码，使用pthread_rwlock代替pthread_mutex可以明显提升性能。**

而同时如果临界区很短的话，**使用读写自旋锁代替pthread_mutex会有更多的性能提升。**

### 使用Double Check

考虑如下单例模式，实例初始化完毕以后，后面每次访问都做了一次根本不需要的锁操作。

```c++
static Tp* Instance() {
  Tp* newTp;
  locker.lock();
  if (inst_ == NULL) {
    newTp = new Tp;
    inst_ = newTp;
  }
  locker.unlock();
return inst_;
```

其实上面单例模式可以改为：

```c++
static Tp* Instance() {
  Tp* newTp;
  if (inst_ == NULL) {
    locker.lock();
  if (inst_ == NULL) {
    newTp = new Tp;
    inst_ = newTp;
  }
  locker.unlock();
  }
  return inst_;
}

```

这样一旦初始化之后，再也不需要获取锁了。

这种check-lock-check-do的方式称为Double Check。Double Check是一种锁的粒度优化。不只可以用于单例模式，其应对的基本模型是：
```c++
if (condition = true)
  do mutual work //change condition
```

### 使用原子操作
如果对共享数据的操作都是原子的那就不需要锁了。最简单的在x86_64上，如果多个线程同时访问一个对齐的long型是不需要锁的。
```c++
volatile unsigned long value_;
value_ = 18676783867
```

上面的赋值语句是原子的，所有的线程要么看到旧值，要么看到新值。即使多个线程同时赋值也没关系。

```c++
value++
```

但这条自增语句不是原子的，它一般需要读内存、改值、写内存三个指令。

一般的编译器都会针对这种操作，提供内建的原子方法，比如在GCC下，上面的value++可以修改为：
```c++
__sync_fetch_and_add(&value_, 1);
```

就是原子的了，它的实现如下：

```c++
40057c: 48 8d 45 f8 lea -0x8(%rbp),%rax
400580: f0 48 83 00 01 lock addq $0x1,(%rax)
```

直接对内存位置执行add指令，原来的3条变为1条，但这仍然不足以保证它的原子，因为别的线程可以可能正在操作这个内存位置，所以加了一个lock前缀，对于x86来说，lock前缀用于锁定总线，保证其后面指令对内存的独占访问。

gcc提供了一组原子方法，包含了：
```c++
type __sync_fetch_and_add (type *ptr, type value)
type __sync_fetch_and_sub (type *ptr, type value)
type __sync_fetch_and_or (type *ptr, type value)
type __sync_fetch_and_and (type *ptr, type value)
type __sync_fetch_and_xor (type *ptr, type value)
type __sync_fetch_and_nand (type *ptr, type value)
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval)
```

····更多的原子方法可以参考GCC手册。

无需声明直接使用即可。

这些原子方法的本质是用硬件层面的锁代替了软件层面的锁，从软件层面来看是原子化了。

使用原子操作时，可能会涉及到内存屏障的使用，后面将会介绍内存屏障。

### 免锁

#### CAS

CAS指compare and swap，整个过程是原子的，CAS原语的逻辑是：
```c++
bool CAS(Type* addr, const Type& compare, const Type& value)
{
  if (*addr == compare)
  {
    *addr = value;
    return true;
  }
  return false;
}
```

CAS依赖硬件实现，在X86上使用的是cmpxchg指令，cmpxchg支持最多64bit的比较交换操作。上节提到的GCC内建函数__sync_bool_compare_and_swap就是对cmpxchg的封装。

依靠CAS原语，我们可以实现完全无锁的单例模式：

```c++
static Tp* Instance() {
  Tp* newTp;
  if (inst_ == NULL) {
    newTp = new Tp;
  if (!__sync_bool_compare_and_swap(&inst_, NULL, newTp)) {
    delete newTp;
  }
  }
  return inst_;
}
```

CAS是很多无锁算法的核心。

#### SPSC

SPSC – single producer single consumer 单生产者单消费者模式，本质是一个FIFO的环形队列，一个读者从队列中取数据，一个写着往队列中添加数据，读和写都不需要加锁。SPSC的实现和维护都不难，是可以用于工程实践的一个免锁算法之一。

## 
