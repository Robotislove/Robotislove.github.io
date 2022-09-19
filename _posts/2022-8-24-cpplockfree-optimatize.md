---
layout: post
title: C++锁机制学习笔记
date: 2022-08-24
author: lau
tags: [C++, Blog]
comments: true
toc: false
pinned: false
---
多线程并发时对锁机制需要有一定了解，这里记录学习笔记。

<!-- more -->
## C++锁机制
![](http://assets.processon.com/chart_image/63059b555653bb0715db5eec.png)

c++的锁包含两部分：**同步原语**和**RAII封装器**。

同步原语有std::mutex、std::shared_mutex、std::timed_mutex、std::recursive_mutex、std::shared_timed_mutex 和 std::recursive_timed_mutex等；

RAII 封装器包含 std::unique_lock、std::lock_guard 和 std::shared_lock等。


std::mutex 是能用于保护共享数据免受从多个线程同时访问的同步原语，也就是最常见的互斥锁概念。

std::shared_mutex 同样也可用于保护共享数据不被多个线程同时访问。但是与 std::mutex 等其他互斥类型不同，std::shared_mutex 拥有二个访问级别：
- 共享 - 多个线程能共享同一互斥的所有权。
- 独占性 - 仅一个线程能占有互斥。

std::shared_mutex 比一般的 std::mutex 多了函数 lock_shared() / unlock_shared()，允许多个（读者）线程同时加锁、解锁，而 std::shared_lock 则相当于共享版的 std::lock_guard。
对 std::shared_mutex 使用 std::lock_guard 或 std::unique_lock 就达到了写者独占的目的。

因此 std::shared_mutex 是C++中实现读写锁最简单有效的办法。示例如下（Get() 实现读共享，Increase() 实现写独占）：
```c++
class Counter {
public:
    Counter() : value_(0) {}

    // Multiple threads/readers can read the counter's value at the same time.
    std::size_t Get() const
    {
        std::shared_lock<std::shared_mutex> lock(mutex_);
        return value_;
    }

    // Only one thread/writer can increment/write the counter's value.
    void Increase()
    {
        // You can also use lock_guard here.
        std::unique_lock<std::shared_mutex> lock(mutex_);
        value_++;
    }

    // Only one thread/writer can reset/write the counter's value.
    void Reset()
    {
        std::unique_lock<std::shared_mutex> lock(mutex_);
        value_ = 0;
    }

private:
    mutable std::shared_mutex mutex_;
    std::size_t value_;
};
```

std::recursive_mutex 与 std::mutex 一样，也是一种可以被上锁的对象，但是和 std::mutex 不同的是，std::recursive_mutex 允许同一个线程对互斥量多次上锁（即**递归锁**），来获得对互斥量对象的多层所有权，std::recursive_mutex 释放互斥量时需要调用与该锁层次深度相同次数的 unlock()，可理解为 lock() 次数和 unlock() 次数相同，除此之外，std::recursive_mutex 的特性和 std::mutex 大致相同。

std::timed_mutex 相对 std::mutex 多了两个成员函数，try_lock_for()，try_lock_until()，可以指定尝试加锁的时长。其他如 std::shared_timed_mutex、std::recursive_timed_mutex 也都是类似的组合。

以上是C++的提供的同步原语，为便于操作，C++还提供了相关的RAII包装器：std::shared_lock、std::unique_lock、std::lock_guard。其中 std::shared_lock 主要和 std::shared_mutex 配合使用，通过 lock_shared 函数实现共享锁定，上文已有示例。std::unique_lock 和 std::lock_guard 的区别，我们通过以下示例说明：
```c++
template <typename T>
class ThreadSafeQueue {
public:
    void Insert(T value)
    {
        std::lock_guard<std::mutex> lk(mut_);
        que_.push_back(value);
        cond_.notify_one();
    }

    void Popup(T &value)
    {
        std::unique_lock<std::mutex> lk(mut_);
        cond_.wait(lk, [this]{return !que_.empety();});
        value = que_.front();
        que_.pop();
    }

    bool Empety()
    {
        std::lock_guard<std::mutex> lk(mut_);
        return que_.empty();
    }

private:
    mutable std::mutex mut_;
    std::queue<T> que_;
    std::condition_variable cond_;
};
```

上面代码只实现了关键的几个函数，并使用了 std::condition_variable 条件变量。从Popup与Inert两个函数看 std::unique_lock 相对 std::lock_guard 更灵活的地方在于在等待中的线程如果在等待期间需要解锁mutex，并在之后重新将其锁定。而 std::lock_guard 却不具备这样的功能。

std::condition_variable 就对应我们常说的 linux 中的**条件锁**，实际上是用来等待线程而不是上锁的，通常和互斥锁一起使用（互斥锁的一个明显的特点就是某些业务场景中无法借助系统来唤醒，仍然需要业务代码使用while来判断，这样效率本质上比较低。而条件变量通过允许线程阻塞和等待另一个线程发送信号来弥补互斥锁的不足，所以互斥锁和条件变量通常一起使用，来让条件变量异步唤醒阻塞的线程），因为它解决的问题不是「互斥」，而是「等待」。

至此，linux 常见的几种锁的概念：互斥锁、自旋锁、条件锁、读写锁和递归锁，除了**自旋锁**外，我们在C++中都可以找到现成库实现。自旋锁与互斥锁的相比，在获取锁失败的时候不会使得线程阻塞而是一直自旋尝试获取锁。当线程等待自旋锁的时候，CPU不能做其他事情，而是一直处于轮询忙等的状态。自旋锁主要适用于被持有时间短，线程不希望在重新调度上花过多时间的情况。实际上许多其他类型的锁在底层使用了自旋锁实现，例如多数互斥锁在试图获取锁的时候会先自旋一小段时间，然后才会休眠。如果在持锁时间很长的场景下使用自旋锁，则会导致CPU在这个线程的时间片用尽之前一直消耗在无意义的忙等上，造成计算资源的浪费。C++标准库当前并没有自旋锁的实现，如果一定要用，可以通过原子操作来实现，示例如下：
```c++
class SpinLock {
public:
    SpinLock() : flag_(false) {}

    void Lock()
    {
        bool expect = false;
        while (!flag_.compare_exchange_weak(expect, true))
        {
            // 这里一定要将expect复原，执行失败时expect结果是未定的
            expect = false;
        }
    }

    void Unlock()
    {
        flag_.store(false);
    }

private:
    std::atomic<bool> flag_;
};
```

这里既然用到了原子操作，就再多说一句，虽然当前的C++标准库提供了强大的锁操作支持，但是锁本身就是低效的，因此我们在设计上要尽量多做无锁设计，而原子操作就是实现无锁操作的一种常见方法（也可以实现读写锁）。所谓的原子操作，取的就是“原子是最小的、不可分割的最小个体”的意义，它表示在多个线程访问同一个全局资源的时候，能够确保所有其他的线程都不在同一时间内访问相同的资源。也就是他确保了在同一时刻只有唯一的线程对这个资源进行访问。这有点类似互斥对象对共享资源的访问的保护，但是原子操作更加接近底层，因而效率更高。C++11 之后标准库提供了强大的原子操作库 std::atomic, 有许多值得探究的地方。

## 参考文献

[1] [Lock-Free Single-Producer - Single Consumer Circular Queue](https://www.codeproject.com/script/Articles/ListVersions.aspx?aid=43510#Main)

[2] Memory Ordering in Modern Microprocessors Paul E. McKenney http://www.linuxjournal.com/article/8211

[3] Facebook folly source code https://github.com/facebook/folly/

[4] Intel 64 and IA-32 Architectures Software Developer's Manual http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html

[5] GCC online doc http://gcc.gnu.org/onlinedocs/gcc-4.1.1/gcc/Atomic-Builtins.html

[6] Linux Source Code http://www.kernel.org/
