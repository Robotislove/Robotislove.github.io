---
layout: post
title: C++性能优化常规方法讲解
date: 2022-08-10
author: lau
tags: [Archive]
comments: true
toc: false
pinned: false
---
C++性能优化常规方法讲解笔记。

<!-- more -->

## 评估性能目标

首先要根据当前机器的物理特性,明确可能达到性能的极限值，做到心里有数。一般的方法是用demo测试一下几个核心接口的性能，然后推断整体机器可运行的能力。在测试的时候，一定要注意查看cpu性能是否已经被榨干。
这一步可以避免不切实际的性能优化目标，浪费大量的时间资源。也可以用于评估目标性能，大致评估一下性能优化的难度和投入的时间。时间永远是最宝贵的资源。

## 性能瓶颈探寻
将当前程序跑在满负荷情况下，用top命令查看一下当前cpu的idle占比。如下图所示：
![](https://s1.328888.xyz/2022/08/10/4stGK.png)
如果cpu的idle很高，则此时cpu的利用率很低，此时很大的情况下是线程模型出了问题，cpu大部分时序都在等待中，需要重点排查程序中的整个线程模型。如果cpu的idle很低，那么cpu的负载确实很高，此时受限于cpu的性能了，这个时候就需要以下的方法去进一步确定低性能的代码位置。

### 1、使用火焰图或者类似工具(Intel VTune)，寻找热点函数
火焰图因其在查找热点函数时方便、直观而被广泛使用的一种方法。具体使用方法见https://github.com/brendangregg/FlameGraph。火焰图如下图所示：
![](https://s1.328888.xyz/2022/08/10/4Pryg.png)

横坐标表示时间轴，纵坐标表示函数调用堆栈。用FlameGraph生成的火焰图是svg格式的，一种可交互的矢量图，也可以搜索。在查看生成的火焰图时，我们重点关注跨度最大的块即可。
### 2、使用计时器细化瓶颈点
我们从火焰图上可以大致看到性能从高到低的情况，此时我们再用计时程序细化一下程序中的各个环节，时间越高，性能越差，需要优化。
### 3、获取系统内存带宽
#### 3.1 计算内存带宽的理论值
计算内存理论值公式：内存频率/8\*内存带宽。对于如下的硬件，MCS的内存带宽理论值如下：2400/8\*64 = 19200 MB/s
![](https://s1.328888.xyz/2022/08/10/4PA4p.png)
#### 3.2 使用PCM获取当前内存带宽实际值
这个工具是Intel官方推出的性能监测工具，包括内存带宽监测，功耗监测，pcie带宽监测等等，可以通过官网https://software.intel.com/en-us/articles/intel-performance-counter-monitor了解详细功能。PCM的内存监测界面如下：
![](https://s1.328888.xyz/2022/08/10/4fNwC.png)
用PCM跑实际内存带宽值的时候，必须要将性能跑满，这一步可以与评估性能目标一起做。
## 性能优化方法

### 1、优化代码逻辑

代码逻辑本身优化是性能优化的王道。要充分分析代码逻辑的正确性，减少程序无用功。以MCS中的CheckCamLocalMcs为例，此部分的优化方案见另一篇文档《广州PK版本优化总结——依图算法》。

### 2、减少数据拷贝操作
数据拷贝占用内存带宽，占用大量CPU指令。像MCS中一个明显的性能问题就是数据全部以一个巨大的连续内存空间去存储特征值，这样在内存不够的时候就需要将老数据拷贝到新空间。性能优化的方法就是特征块状空间存储，见R19性能优化。
另外还有一些变量传递的动作，尽量使用引用，减少拷贝。还有像容器的push_back等操作，实际会调用拷贝构造函数数，可以使用更高级的C++11右值移动语义，代码样例数下所示：

```c++
class obj
{
public:
    obj() { cout << ">> create obj " << endl; }
    obj(const obj& other) { cout << ">> copy create obj " << endl; }
};

template <class T>
class container
{
    public:
        T* value;

    public:
    container() :value(NULL) {}
    ~container() { delete value; }
    container(const container& other)
    {
        value = new T(*other.value);
    }

    const container& operator = (const container& other)
    {
        delete value;
        value = new T(*other.value);
        return *this;
    }
    container(container&& other)
    {
        value = other.value;
        other.value = NULL;
    }

    container& operator = (container&& other)
    {
        delete value;
        value = other.value;
        other.value = NULL;
        return *this;
    }

    void push_back(const T& item)
    {
        delete value;
        value = new T(item);
    }
};

vector<obj> foo()
{
    vector<obj> c;
    c.push_back(obj());

    cout << "---- exit foo ----" << endl;
    return c;
}

class bigobj
{
public:
    bigobj() { cout << ">> create obj " << endl; }
    bigobj(const bigobj& other) { cout << ">> copy create obj " << endl; }
    bigobj(bigobj&& other) { cout << ">> move create obj " << endl; }
};

int main()
{
    list<bigobj> list;
    for (int i = 0; i < 3; i++)
    {
        bigobj obj;
        list.push_back(std::move(obj));
    }
}

```

### 3、使用sse、avx指令集优化计算密集型操作
sse、avx都是CPU的SIMD指令集，使用这些指令集可以最大化的提升CPU单条指令提升操作数据量的能力。Intel平台上要想使用这些指令，要查看清楚CPU型号以便了解CPU支持的指令集版本。Intel提供了一个网址https://software.intel.com/sites/landingpage/IntrinsicsGuide/#techs=AVX&expand=118查询各指令的用法。使用这些命令，需要在代码中包含immintrin.h，在编译的时候增加-mavx,-mavx2,-msse等选项。需要注意的是这些指令对于计算密集型的操作性能有显著提升，其它则提升不明显。

### 4、用空间换时间
如果内存空间比较充足，这个操作在性能优化中是比较常见的手段。比如一块连续的内存空间存储着N个如下结构体的数据：
```c++
struct data
{
    uint64_t ObjectID;
    uint64_t Timestamp;
    uint8_t  Feature;
    size_t   FeatureSize;
}

```
如果我需要找到一个ObjectID为A的时间戳，那么我必须要做一个遍历。如果用空间换时间，我们另外再存储一个如下所示的映射表：
```c++
std::unordered_map<uint64_t,uint64_t> ObjectIDTimestampMapTable;
```

这样操作实际上是将最差为O(n)复杂度降为O(1),对于像MCS这样的大数据量的程序，提升性能还是很明显的。
### 5、多线程同步优化
#### 5.1 线程数量调优
在很多场景下，我们没有发挥好硬件性能，使得可以并发处理的计算没有很好的性能表现，比如MCS中的依图静态库特征获取操作。这个操作需要从16个哈希索引中获取特征的偏移位置，进一步找到特征值，这一个操作完全是可以并发处理的，同时考虑到不能无限大的使用cpu资源，因此，在优化的时候，设计了一个最大线程数为16的线程池来完成特征提取操作。
#### 5.2 合理使用互斥锁
在使用锁的时候，尽量最小化锁的作用域，也要注意不要出现锁重入导致死锁。
#### 5.3 使用原子操作
多线程下，尽量多地使用原子变量来替代锁。一般情况下，原子操作性能要比锁性能高,原子操作是非常底层的，涉及cpu指令集层。以下是原子变量与互斥锁的性能比对demo跑的结果：
![](https://s1.328888.xyz/2022/08/10/4f45X.png)
### 6、降低算法复杂度
对于数据量巨大的程序来说，算法的性能可能与数据量呈指数次方增加。比如选择排序算法时，快速排序(O(nlogn))就要优于就要比插入排序(O(n^2))
### 7、日志优化
去除无用的日志，不仅影响性能，而且还不利于问题定位。
### 8、编译器优化
#### 8.1 编译选项优化
使用-O3性能普遍要比默认优化等级要高，而且编出来的库要小很多。像下面这个程序，在windows下用release与debug编译，程序大小相差约10倍，性能相差2倍。性能如下图所示：
![](https://s1.328888.xyz/2022/08/10/4f3wr.png)
#### 8.2 更换编译器
如果经济条件允许，像Intel会提供针对Intel CPU的高性能编译器ICC，这个编译器在代码未改动的情况下，可以将性能提升1.5倍左右。
### 9、使用第三方高性能库
像Intel还提供了针对于自家处理器的一些高性能的数学库如MKL(https://software.intel.com/en-us/mkl)。
### 10、减少内存碎片
内存碎片对内存的使用也会带来性能影响，像MCS这种大规模使用内存的场景下，内存管理非常重要，尽量做一些内存池或者使用tcmalloc之类的库，降低内存碎片。
