---
layout: post
title: C++ 14标准学习笔记
date: 2022-03-11
author: lau
tags: [C++, Archive]
comments: true
toc: true
pinned: false


---

C++14 主要特性。

<!-- more -->

C++14整体来说只是发行的一个小版本，在C++11大版本的基础上做了一些优化和缺陷的修复。C++14在2014年8月18日正式批准宣布，同年12月15日正式发布release版本。本文中将就变动部分做一个总结，有需要改进和提升的地方希望大家批评指正。

**1 变量模板**

变量模板是C++14中新增的特性，可以将变量实例化成不同的类型，变量模板的定义方法如下所示：

```c++
template < 形参列表 > 变量声明
```

在上面的语法中，变量声明即为变量模板名，形参列表可以有一个或者多个，使用方法如下：

```c++
template<class T>
constexpr T pi = T(3.1415926535897932385L);  // 变量模板
```

在实际编码中，C++14以前如果要实现上面的功能，需要通过函数模板的方式得到。如下面的代码可以得到相同的功能：

```c++
template<class T>
6  std::make_unique方法 T Pi()
{
  return T(3.1415926535897932385L);
}
```

**2 变量模板**

**在编程时以auto为形参类型时，既可以认为是泛型lambda。**

在C++14中，泛型lambda是普通lambda的升级版，具体使用方法如下：

**2.1 有两个形参的**

```c++
int main () {
 auto glambda = [](auto a, auto&& b) { return a < b; };
 bool b = glambda(3, 3.14); 
 std::cout<<"b="<<b<<std::endl;
  return 0;
}
```

**2.2 有1个形参的**

```c++
int main () {
// 泛型 lambda，operator() 是有一个形参的模板
auto vglambda = [](auto printer) {
    return [=](auto&&... ts) // 泛型 lambda，ts 是形参包
    { 
        printer(std::forward<decltype(ts)>(ts)...);
        return [=] { printer(ts...); }; // 零元 lambda （不接受形参）
    };
};
auto p = vglambda([](auto v1, auto v2, auto v3) { std::cout << v1 << v2 << v3; });
auto q = p(1, 'a', 3.14); // 输出 1a3.14
//q();    
  return 0;
}
```

**2.3 无捕获的泛型**

```c++
int main () {
    auto func1 = [](auto i) { return i + 4; };
    std::cout << "func1: " << func1(6) << '\n';
    return 0;
}
```

代码输出值：10

上面的的代码也可以修改成lambda表达式形参中有默认值的类型，从C++14起，也支持这种写法，代码稍微修改后，程序如下：

```c++
int main () {
    auto func1 = [](int i = 6) { return i + 4; };
    std::cout << "func1: " << func1() << '\n';
    return 0;
}
```

运行此代码，可以运行结果和上面一致。

**3 constexpr放松限制**

使用**constexpr**-描述符后，指定的变量或函数的值可以在常量表达式中使用。

在C++11中，constexpr只能只用递归，C++14后进行了优化和提升，可以使用局部变量和循环。且不用将所有的语句放在一条return语句中进行编写。

```c++
constexpr int add(int a,int b)
{
    int sum = 0;
    if(a<=0) return -1;
    if(b<=0) return -1;
    sum = a+b;
    return sum;
}
int main () {
    std::cout<<add(4,6)<<std::endl;
    return 0;
}
```

如上面代码所示，在C++14之前constexpr是不支持的。

**4 二进制字面量**

C++14中新增了0b或者0B开头的字面量，使用方法如下：

```c++
int main () {
   int x=32;
   int y = 0b100000;
   std::cout<<(x==y)<<std::endl;
    return 0;
}
```

代码输出结果为：1

**5 函数返回值推导**

在C++11中使用后置类型推导函数返回值，C++14起，可以省略，返回值使用auto，编译器直接将函数体中的return语句值类型推导。使用时需要注意的是：

- 如果有多条return语句，每条的返回值类型需要保持一致。
- 如果没有返回语句或返回语句的实参是 void 表达式，那么所声明的返回类型，必须是 decltype(auto)或者auto，此时推导出的返回类型是void。
- 一旦在函数中见到一条返回语句，那么从该语句推导的返回类型就可以用于函数的剩余部分。
- 如果返回语句使用花括号初始化器列表（brace-init-list），那么不允许推导。

使用方法见如下代码：

```c++
//C++14中不需要后置
auto add(int x, int y)
{
  return x + y;
}
//多条返回语句返回值类型保持一致
auto f(bool val)
{
    if(val) return 123;   // 推导返回类型 int
    else return 3.14f;    // 错误：推导返回类型 float
}
//必须有条返回语句可以推导出来返回类型并用在后面的语句
auto sum(int i)
{
    if(i == 1)
        return i;              // sum 的返回类型是 int
    else
        return sum(i - 1) + i; // OK，sum 的返回类型已知
}
//返回值使用{}不允许推导
auto func () { return {1, 2, 3}; } // 错误
```

**6 std::make_unipue方法**

C++14提供了std::make_unique方法.使用方法如下：

 ```c++
#include <iostream>
#include <memory>
 
struct Vec3
{
    int x, y, z;
    // C++20 起不再需要以下构造函数
    Vec3(int x = 0, int y = 0, int z = 0) noexcept : x(x), y(y), z(z) { }
    friend std::ostream& operator<<(std::ostream& os, const Vec3& v)
    {
        return os << '{' << "x:" << v.x << " y:" << v.y << " z:" << v.z  << '}';
    }
};
 
int main()
{
    // 使用默认构造函数。
    std::unique_ptr<Vec3> v1 = std::make_unique<Vec3>();
    // 使用匹配这些参数的构造函数
    std::unique_ptr<Vec3> v2 = std::make_unique<Vec3>(0, 1, 2);
    // 创建指向 5 个元素数组的 unique_ptr 
    std::unique_ptr<Vec3[]> v3 = std::make_unique<Vec3[]>(5);
 
    std::cout << "make_unique<Vec3>():      " << *v1 << '\n'
              << "make_unique<Vec3>(0,1,2): " << *v2 << '\n'
              << "make_unique<Vec3[]>(5):   " << '\n';
    for (int i = 0; i < 5; i++) {
        std::cout << "     " << v3[i] << '\n';
    }
}
 ```

**7 std::shared_timed_mutex、std::share_lock**

共享互斥的使用使用场景是同一个数据资源，存在多个线程读，但只有一个线程可以进行修改的场景。如下面的代码，实现的功能是赋值运算符可以有多个读但是只有一个写能力

 ```c++
class R
{
    mutable std::shared_timed_mutex mut;
    /* 数据 */
public:
    R& operator=(const R& other)
    {
        // 要求排他性所有权以写入 *this
        std::unique_lock<std::shared_timed_mutex> lhs(mut, std::defer_lock);
        // 要求共享所有权以读取 other
        std::shared_lock<std::shared_timed_mutex> rhs(other.mut, std::defer_lock);
        std::lock(lhs, rhs);
        /* 赋值数据 */
        return *this;
    }
};
int main() {
    R r;
}
 ```

**8 std::integer_sequence类**

使用时需要包含头文件 <utility>，定义原型为：

```c++
template< class T, T... Ints >
struct integer_sequence;
```

有一个公开静态成员函数size,返回Ints的个数。改类使用方法为：

```c++
template<typename T, T... ints>
void print_sequence(std::integer_sequence<T, ints...> int_seq)
{
    std::cout << "The sequence of size " << int_seq.size() << ": ";
    ((std::cout << ints << ","),...);
    std::cout << '\n';
}
int main()
{
    print_sequence(std::integer_sequence<unsigned, 9, 2, 5, 1, 9, 1, 6>{});
    return 0;
}
```

代码输出结果为：The sequence of size 7: 9,2,5,1,9,1,6

**9 std::exchange类**

函数原型为：

```c++
template< class T, class U = T >
T exchange( T& obj, U&& new_value );
```

表示使用new_value替换obj，同时返回返回obj。如下面代码所示：

```c++
void f() { std::cout << "f()"; }
 
int main()
{
   std::vector<int> v;
   std::exchange(v, {1,2,3,4});
   std::copy(begin(v),end(v), std::ostream_iterator<int>(std::cout,", "));
   std::cout << "\n\n";
   void (*fun)();
   std::exchange(fun,f);
   fun();
}
```

上面代码运行结果为：

```c++
1, 2, 3, 4, 
f()
```

**10 std::quoted类**

函数原型为：

```c++
template< class CharT >
/*unspecified*/ quoted(const CharT* s,
                       CharT delim=CharT('"'), CharT escape=CharT('\\'));
```

实现功能是可以允许带有双引号或者其他符号字符串输入或者输出。如下面代码所示：

```c++
int main()
{
    std::string in = "String with spaces, and embedded \"quotes\" too";
    std::cout<<std::quoted(in)<<std::endl;
    in = "String with spaces, and embedded $quotes$ too";
    const char delim {'$'};
    const char escape {'%'};
    std::cout << std::quoted(in, delim, escape);
}
```

上述代码输出为：

```c++
"String with spaces, and embedded \"quotes\" too"
$String with spaces, and embedded %$quotes%$ too$
```

如上为C++14中新增的主要特性，当然，还有一些其它的提升，这里将不再进行阐述。如果大家有兴趣欢迎留言讨论。
