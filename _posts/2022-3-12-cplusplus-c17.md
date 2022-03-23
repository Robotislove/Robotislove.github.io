---
layout: post
title: C++ 17标准学习笔记
date: 2022-03-12
author: lau
tags: [C++, Archive]
comments: true
toc: true
pinned: false
---

C++17 主要特性。

<!-- more -->

**1 C++17 介绍**

每次C++版本的发布都会带来很多新的特性，C++17也不例外，虽然有很多期待的特性没有包含进来，但是新增的特性依然挡不住它独特的魅力。

C++17发布于2017年，ISO C++ 委员会将其正式命名为：ISO/IEC 14882:2017。

**2 C++17新特性**

**2.1 折叠表达式**

从C++17开始，可以使用二元操作符对形参包中的参数进行计算，这一特性主要针对可变参数模板进行提升,可以分为左折叠和右折叠。支持的二元操作符多达32个。有一点需要注意的是，如果形参包为空包，那么展开式逻辑与的值为true，逻辑或的值为false，逗号表达式的值为void()。

```c++
template<typename ... T>
auto sum_right(T ... arg)
{
    return (arg + ...);//右折叠
}
 
template<typename ... T>
double sum_left(T ... arg)
{
    return ((8*2) + ... + arg);//左折叠
}

int main()
{
    int sum1 = sum_right(1,2,3);
    std::cout<<"sum1="<<sum1<<std::endl;
    int sum2 = sum_left();
    std::cout<<"sum2="<<sum2<<std::endl;
    return 0;
}
```

运行结果：

```c++
sum1=6
sum2=16
```

**2.2 类模板实参推导**

对模板进行实例化时，不需要指定模板参数，编译器会根据传入的实参进行类型推导。

- 根据变量及变量模板的初始化或者声明进行推导

 ```c++
std::pair p(2, 4.5);     // 推导出 std::pair<int, double> p(2, 4.5);
std::tuple t(4, 3, 2.5); // 同 auto t = std::make_tuple(4, 3, 2.5);
std::less l;             // 同 std::less<void> l;
 ```

- new 表达式推导

```c++
template<class T> struct A { A(T,T); };
auto y = new A{1,2}; // 分配的类型是 A<int>
```

- 函数转型表达式推导

```c++
auto lck = std::lock_guard(mtx); // 推导出 std::lock_guard<std::mutex>
std::copy_n(vi1, 3, std::back_insert_iterator(vi2)); // 或 std::back_inserter(vi2)
std::for_each(vi.begin(), vi.end(), Foo([&](int i) {...})); // 推导 Foo<T>，其中 T 
                                                            // 是独有的 lambda 类型
```

**2.3 用auto作为非类型模板参数**

在模板参数中使用auto作为关键字时，模板实例化传入非类型值，auto可以推导出参数类型。

```c++
template<auto T1,auto T2>
auto sum()
{
    return (T1 * T2);
}

int main()
{
    float sum1 = sum<1,2>();
    std::cout<<"sum1="<<sum1<<std::endl;
    return 0;
}
```

代码运行结果为：3；

需要注意的是C++17目前还不支持参数类型是浮点型的推导。不过这一特性在C++20中已经被支持进来。**C++17支持的类型包括：左值引用，整数，指针类型，成员指针类型，枚举。**

**2.4 在if语句中使用constexpr**

使用后，如果if语句中表达式为true，它所对应的else分支就不会被编译出汇编语句，反之亦然

 ```c++
template<bool bFlag>
constexpr void test()
{
    if constexpr (bFlag == true)
{
        std::cout << bFlag << std::endl;
    }
    else
    {
        std::cout << "false" << std::endl;
    }
}
 ```

**上面的代码生成汇编后如下图所示：被标注的代码没有生成对应的汇编语句。**

[![q1akwV.png](https://s1.ax1x.com/2022/03/23/q1akwV.png)](https://imgtu.com/i/q1akwV)

**2.5 inline**

可以将变量定义成为内联变量，内联变量不能用户函数定义中，使用时避免重复定义。使用方法如下：

```c++
#include<iostream>
inline int iCount=9;
inline int  sum(int a)
{
    int iSum=iCount+a;
    return iSum;
}
int main()
{
    float sum1 = sum(5);
    std::cout<<"sum1="<<sum1<<std::endl;
    return 0;
}
```

**2.6 结构化绑定**

- 绑定数组

将数组绑定的制定标识符列表中，每个列表元素都是一个数组元组，如下：

```c++
int a[2] = {1,2}; 
auto [x,y] = a; // 创建 e[2]，复制 a 到 e，然后 x 指代 e[0]，y 指代 e[1]
auto& [xr, yr] = a; // xr 指代 a[0]，yr 指代 a[1]
```

- 绑定元组类型

```c++
float x{};
char  y{};
int   z{};
std::tuple<float&,char&&,int> tpl(x,std::move(y),z);
const auto& [a,b,c] = tpl;
// a 指名指代 x 的结构化绑定；decltype(a) 为 float&
// b 指名指代 y 的结构化绑定；decltype(b) 为 char&&
// c 指名指代 tpl 的第 3 元素的结构化绑定；decltype(c) 为 const int
```

- 绑定成员变量

```c++
struct Elem {
   int x1 ;
   double y1;
};
S f(); 
//x指向int左值，y指向double左值
const auto [x, y] = f();
```

**2.7 if和switch语句中包含初始化语句**

```c++
int main()
{
   if(int i=-1;i<=0)
   {
       std::cout<<"i>=0"<<std::endl;
   }
   else
   {
       std::cout<<"i<0"<<std::endl;
   }
   
   switch(int k =2;k)
   {
        case 1:
            std::cout<<k<<std::endl;
            break;
        case 2:
            std::cout<<k<<std::endl;
            break;
        default:
             std::cout<<"default"<<std::endl;
   }
    return 0;
}
```

**2.8 u8' c-字符 '**

```c++
std::cout<<u8'你'<<std::endl;
```

**2.9 简化的嵌套命名空间**

```c++
namespace A {
    void g();
}
namespace X {
    using A::g, A::g; // (C++17) OK：命名空间作用域允许双重声明
}

namespace A::B::C::D{
}
```

**2.10 noexcept**

从C++17起noexcept被当做系统类型的一部分，可以用作任何函数的声明。

在C++17中，noexcept(true)相当于之前的throw();

```c++
void f() noexcept;
void f() noexcept(false);
```

**2.11 lambda表达式捕获\*this的值**

 ```c++
class Test {
public:
  int m_iValue;
  void foo() {
    auto lamfoo = [*this]() { std::cout << m_iValue << std::endl; };
    lamfoo();
  }
};

int main() {
  Test test;
  test.m_iValue=10;
  test.foo();
  return 0;
}
 ```

上面代码运行结果为：10，在C++17之前，`auto lamfoo = [*this]() { std::cout << m_iValue << std::endl; };`这么写会报语法错误。

**2.12 fallthrough**

用在switch语句中，如果case语句不需要使用break希望继续执行下一个case时使用此关键字。可以避免编译器产生告警。

 ```c++
int main() {
  switch(int k=2;k)
  {
      case 1:
        k++;
      case 2:
        k--;
        [[fallthrough]]
      case 3:
        k--;
      default:
        break;
  }
  return 0;
}
 ```

**2.13 nodiscard**

可以用在类，枚举，结构体，函数定义，但是只有使用在函数定义时效果比较明显，主要作用是如果调用了函数没有检查返回值的话，使得编译器产生告警。

 ```c++
[[nodiscard]] int sum()
{
    return 3;
}
int main() {
  sum();
  return 0;
}
 ```

编译时编译器告警如下：

```c++
main.cpp:16:4: warning: ignoring return value of ‘int sum()’, declared with attribute nodiscard [-Wunused-result]
 sum();
 ~~~^~
main.cpp:11:19: note: declared here
```

**2.14 maybe_unused**

这个属性可以在类、结构体、共同体、函数、非静态成员变量、枚举等定义前。如果已经定义但是没有实现，可以禁止编译器告警。

```c++
[[maybe_unused]] class Test {};
[[maybe_unused]] enum WEEK {};
[[maybe_unused]] int iValue;
[[maybe_unused]] void Sum();
```

**2.15 __has_include**

功能是判断有没有包含头文件

```c++
#if __has_include(<optional>)
#  include <optional>
#  define have_optional 1
namespace guard { using std::optional; }
#elif __has_include(<experimental/optional>)
#  include <experimental/optional>
#  define have_optional 1
#  define experimental_optional 1
namespace guard { using std::experimental::optional; }
#else
#  define have_optional 0
#endif
 
#include <iostream>
 
int main()
{
    if (have_optional)
        std::cout << "<optional> 存在。\n";
 
    int x = 42;
#if have_optional == 1
    guard::optional<int> i = x;
#else
    int* i = &x;
#endif
    std::cout << "i = " << *i << '\n';
    return 0;
}
```

代码运行结果为：

```c++
<optional> 存在。
i = 42
```

**3 总结**

对于C++17新增特性很多编译器已经都能够进行支持，当然C++17版本中规划的内容也不止上面说的这些，如果大家有需要补充或者对上述内容进行指正的欢迎大家留言。

**4 参考**

- https://zh.cppreference.com/w/cpp/17
- https://blog.csdn.net/qq811299838/article/details/90371604
