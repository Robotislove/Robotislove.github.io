---
layout: post
title: 深入并行编程锁优化笔记
date: 2022-08-24
author: lau
tags: [C++, Blog]
comments: true
toc: false
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

## 具体实现

### 自旋锁

simple::SpinLock完全采用了folly:: MicroSpinLock实现方式，先自旋后休眠。目前仅支持GCC+X86_64。

simple::SpinLock 核心使用CAS原语，改变一个1字节的内容实现加锁。
```c++
__sync_bool_compare_and_swap(&lock_, FREE, LOCKED);
```

解锁时，进行一个内存写操作。由于单字节是必然对齐的，必定是原子的。并插入了优化屏障确保其他线程看到锁FREE以后，拿锁的已经离开了临界区（所有对共享数据的读写已经完成）。目前x86上并不需要硬件屏障。但如果担心特殊环境或未来变化，换成硬件屏障肯定是没问题的。Glibc的和folly以及内核的自旋锁都没有针对x86用硬件屏障。

```c++
void UnLock() {
  asm volatile("" : : : "memory");
  lock_ = FREE;
}
```
### 读写自旋锁

simple::RWSpinLock，完全采用了folly:: RWSpinLock的实现，做了简化并去除了对C++0x atomic原子对象的依赖。使用GCC内建原子操作。
使用一个int型记录读者和写者状态。
```c++
volatile unsigned int lock_ __attribute__ ((aligned (4)));
```
读者对lock_原子加2，可以满足2^31个并发读。

写着对lock_ 执行CAS（&lock_, 0, 1）。

### SPSC

simple:: SPSC内部采用一个vector存储环形队列，并且需要预先制定vector大小：

SPSC<string, 1000> spsc; 定义了一个单进单出的环形队列，大小是1000，存储对象是string。

两个游标p_和c_分别记录生产者和消费者的位置，初始都指向0。此时队列为空。

Enqueue入队，如果返回false说明队列满了，由调用者选择处理方式，或休眠或做其他事情。

```c++
bool Enqueue(const Tp& v) {
  if (p_ + 1 == c_) return false;
    queue_[p_.cur_] = v;
    ++p_;
    return true;
  }
```

入队之后，更新游标p_之前，需要加入内存屏障，确保消费者看到p_更新时，元素已经在队列里面了。

Dequeue出队，如果返回false说明队列为空，由调用者选择处理方式，或休眠或做其他事情。

```c++
bool Dequeue(Tp& v) {
  if (c_ == p_) return false;
    v = queue_[c_.cur_];
    ++c_;
    return true;
  }
```

入队之后，更新游标c_之前，需要加入内存屏障，确保生产者看到c_更新时，元素不在队列里面了。

由于是环形的p_和c_到vector尾端时需要从头来，通过一个Iter对象将这个过程封装了。

显然游标值必须是内存变量，所以声明为volatile，而且应该保持对齐，确保改写它是原子的。

```c++
volatile size_t cur_ __attribute__ ((aligned (8)));
```

对它改写时，先在**临时变量上生成最终新值（包括了到尾端后的调整）**，然后写进内存，写之前加入屏障。这是所有SPSC实现最容易出错的地方。直接对游标修改，中间值会被对方看到。

```c++
Iter& operator++() {
  Iter tmp = *this + 1;
  asm volatile("" : : : "memory");
  cur_ = tmp.cur_; //atomic
  return *this;
}
```

### Singleton
演示CAS的使用而已，实际上单例也就第一次创建时存在竞争，使用Double Check之后，是否免锁没有什么区别。

## 附录：内存屏障

考虑如下代码：
```c++
线程A:        线程B:
a=malloc();   if(b==true)
b=true;          {access a}
```
a和b均为内存变量，代码逻辑上并没有问题，但遗憾的是，当b为true时，a不一定已经被赋值为一个合法的内存地址。也就是说对a和b指向的两个内存位置的访问顺序并不是严格按照代码顺序来的。

**有四个层次的顺序**

1. 语句顺序：就是我们C/C++语句的顺序，这是我们要的正确的逻辑顺序。
2. 指令顺序：编译器生成的指令顺序，这个顺序可能会和语句顺序不一致，编译器出于优化的目的可能会把没有依赖关系的指令打乱。
3. 执行顺序：CPU执行指令的顺序，这个顺序可能会和指令顺序不一致，CPU出于优化的目的可能会把没有依赖关系的指令打乱。
4. 感知顺序：CPU感知到的自己或其他CPU对内存操作的顺序。这个顺序可能和执行顺序不一样，Cache机制、内存系统方面的优化可能会导致先对内存的操作却被后生效（感知）。

我们需要的是感知顺序和语句顺序一致。而内存屏障就是做这个事情的。

内存屏障分为两种:
1. 优化屏障，优化屏障语句提示编译器不要做优化，确保屏障语句前面的所有对内存的操作均在其后面语句对内存的操作之前完成。GCC插入优化屏障的方法是：

```c++
a=malloc(); if(b==true)
asm volatile("" : : : "memory");
b=true;
```

`asm volatile("" : : : "memory");` 是一条内联汇编语句，起到屏障的作用。

优化屏障确保指令顺序和语句顺序的一致。

2. 硬件屏障， 硬件屏障是具体平台架构提供的特殊指令，硬件屏障通常分为读屏障、写屏障、读写屏障。 顾名思义，读屏障确保读顺序、写屏障确保写顺序、读写屏障确保读写顺序。

X86提供的三个屏障指令。

- LFENCE instructions cannot pass earlier reads.
- SFENCE instructions cannot pass earlier writes.
- MFENCE instructions cannot pass earlier reads or writes.
使用X86硬件屏障也很简单。

```c++
asm volatile("mfence " : : : "memory");
```

也可以借鉴内核代码，把这些内联汇编定义成可读性好的宏。

```c++
#define mb() asm volatile("mfence":::"memory")
#define rmb() asm volatile("lfence":::"memory")
#define wmb() asm volatile("sfence" ::: "memory")
```

硬件屏障确保感知顺序和指令顺序一致。但硬件屏障对应的汇编语句，也包括了优化屏障，所以硬件屏障语句可以确保感知顺序和语句顺序一致。

回到我们上面一开始提到的例子，实际上在已有的x86架构上只要插入了优化屏障就可以了，

```c++
线程A:                                   线程B:
a=malloc();                             if(b==true)
asm volatile("sfence " : : : "memory");
b=true;                                 {access a}
```
只要编译器不乱序，对于一般的写操作x86是不会做乱序处理的。

Writes to memory are not reordered with other writes, with the following
exceptions:

— writes executed with the CLFLUSH instruction;

— streaming stores (writes) executed with the non-temporal move instructions
(MOVNTI, MOVNTQ, MOVNTDQ, MOVNTPS, and MOVNTPD); and

— string operations (see Section 8.2.4.1).

但是INTEL在他的手册里面也说了，内存顺序的机制随着x86的架构演进可能会变化。总之要是实在摸不准，我们直接使用硬件屏障语句就好了。

**如果你使用mutex spinlock这样的锁机制，你不需要担心内存屏障，这些锁的底层实现隐含了内存屏障的处理。**

**如果在X86上使用__sync_fetch_and_add这一组原子方法，是不需要屏障的，因为lock前缀隐含了内存屏障的处理。**

## 参考
[1] Linux Source Code http://www.kernel.org/

[2] Memory Ordering in Modern Microprocessors Paul E. McKenney http://www.linuxjournal.com/article/8211

[3] Facebook folly source code https://github.com/facebook/folly/

[4] Intel 64 and IA-32 Architectures Software Developer's Manual http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html

[5] GCC online doc http://gcc.gnu.org/onlinedocs/gcc-4.1.1/gcc/Atomic-Builtins.html
