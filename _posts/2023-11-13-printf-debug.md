---
layout: post
title: C语言字符串、内存管理库函数和串口printf自实现
date: 2023-11-13
author: lau
tags: [C++, Archive]
comments: true
toc: false
pinned: false
---

C语言字符串和内存管理库函数自实现学习笔记。

<!-- more -->

## 1 前言

C语言学习最大的魅力就是库函数，可以通过自己写一些库函数实现，就能更深刻地理解C语言这门工具，同理，对于C++和Rust来说也是如此，可以时不时看看库函数源码，自己尝试将其写出来，语言底子能够提升不少。针对C语言来说，里面最值得学习的就是字符串和内存管理的实现，我们可以从C语言之父的那本书《C程序设计语言》里面看到很多关于C自身实现和库函数相关知识。然后将嵌入式领域的printf进行讲解，后续将会描述如何写自己的日志系统。

## 2 库函数实现

### 2.1 strlen

[strlen](https://legacy.cplusplus.com/reference/cstring/strlen/?kw=strlen)：计算字符串长度函数

`size_t strlen ( const char * str );`

**注意事项：**

- 字符串以`'\0'`作为结束标志，strlen函数返回值是在字符串中`'\0'`前面出现的字符个数（不包含`'\0'`）
- 参数是一个字符指针变量
- 参数指向的字符串必须要以`'\0'`结束，否则计算出的长度是随机值
- 注意函数的返回值为`size_t`，是无符号的

因为返回值是`size_t`，所以就要避免出现下面这样的代码：`strlen("abc") - strlen("abcde")`，`strlen("abc")`算出的结果是3， `strlen("abcde")`算出的结果是5，可能想着3-5得到-2，实际上并不是这样的，这里算出的3和5都是无符号整型，算出的-2也是一个无符号整型，-2在内存中以补码的形式存储，从无符号整型的视角看去，这串补码就表示一个很大的正数。

**模拟实现：**

```c
//常规实现
int my_strlen1(const char* arr)
{
	assert(arr != NULL);
	int count = 0;
	while (*arr)
	{
		count++;
		arr++;
	}
	return count;
}
//递归实现
int my_strlen2(const char* arr)
{
	assert(arr!=NULL);
	if (*arr == '\0')
	{
		return 0;
	}
	else
	{
		return 1 + my_strlen2(arr + 1);
	}
}
//指针减指针实现
int my_strlen3(const char* arr)
{
	assert(arr != NULL);
	const char* start = arr;
	while (*arr)
	{
		arr++;
	}
	return arr - start;
}
int main()
{
	char arr[] = "abcdef";
	int len = my_strlen3(arr);
	printf("%d\n", len);
	return 0;
}
```

### 2.2 strcpy

[strcpy](https://legacy.cplusplus.com/reference/cstring/strcpy/?kw=strcpy)：字符串拷贝函数，把源字符串拷贝到目标空间

`char * strcpy ( char * destination, const char * source );`

**注意事项：**

- 函数有两个参数，其中`source`指向待拷贝的字符串，也叫做源字符串，`destination`是目标空间的地址
- 源字符串必须以`’\0’`结束
- 会把源字符串中的 `‘\0’` 也拷贝到目标空间
- 目标空间必须足够大，以确保能存放源字符串，否则会出现非法访问
- 目标空间必须可变，例如把源字符串拷贝到一个字符串常量里面是不可取的

**模拟实现：**

```c
char* my_strcpy(char* dest, const char* src)
{
	assert(dest && src);
	char* ret = dest;
	while (*dest++ = *src++)
	{
		;
	}
	return ret;
}
int main()
{
	char arr1[20] = { 0 };
	char arr2[] = "hello world";
	my_strcpy(arr1, arr2);
	printf("%s\n", arr1);
	return 0;
}
```

### 2.3 strcat

[strcat](https://legacy.cplusplus.com/reference/cstring/strcat/?kw=strcat)：字符串追加函数，将源字符串追加到目标字符串后面，目标中的终止字符`’\0’`会被源字符串的第一个字符覆盖

`char * strcat ( char * destination, const char * source );`

**注意事项：**

- 函数有两个参数，其中`source`指向要追加的字符串，也叫做源字符串，`destination`是目标空间的地址
- 目标空间中必须要有`'\0'`，作为追加的起始地址
- 源字符串中也必须要有`'\0'`作为追加的结束标志
- 目标空间必须足够大，能容纳下源字符串的内容
- 目标空间必须可修改
- 自己给自己追加会陷入死循环

**模拟实现：**

```c
char* my_strcat(char* dest, const char* src)
{
	assert(dest && src);
	char* ret = dest;
	//找目标空间的'\0'
	while (*dest != '\0')
	{
		dest++;
	}
	//追加
	while (*dest++ = *src++)
	{
		;
	}
	return ret;
}
int main()
{
	char arr1[20] = "hello ";
	char arr2[] = "world";
	my_strcat(arr1, arr2);
	printf("%s\n", arr1);
	return 0;
}
```

### 2.4 strcmp

[strcmp](https://legacy.cplusplus.com/reference/cstring/strcmp/?kw=strcmp)：字符串比较函数

`int strcmp ( const char * str1, const char * str2 );`

**注意事项：**

- 这里比较的不是两个字符串的长度，而是对应位置上的ASCII值
- 第一个字符串大于第二个字符串，则返回大于0的数字
- 第一个字符串等于第二个字符串，则返回0
- 第一个字符串小于第二个字符串，则返回小于0的数字

**模拟实现：**

```c
int my_strcmp(const char* str1, const char* str2)
{
	assert(str1 && str2);
	while (*str1 == *str2)//如果相等就进去，两个指针加加，但是可能会出现两个字符串相等的情况，两个指针都指向'\0'，此时比较就结束了
	{
		if (*str1 == '\0')
		{
			return 0;
		}
		str1++;
		str2++;
	}
	if (*str1 > *str2)
	{
		return 1;
	}
	else
	{
		return -1;
	}
}
int main()
{
	char arr1[] = "abq";
	char arr2[] = "abq";
	int ret=my_strcmp(arr1, arr2);
	printf("%d\n", ret);
	return 0;
}
```

### 2.5 strncpy

[strncpy](https://legacy.cplusplus.com/reference/cstring/strncpy/?kw=strncpy)：长度受限的字符串拷贝函数

`char * strncpy ( char * destination, const char * source, size_t num );`

**注意事项：**

- 拷贝num个字符从源字符串到目标空间。
- 如果源字符串的长度小于num，则拷贝完源字符串之后，在目标的后边追加0，直到num个。

**模拟实现：**

```c
char* my_strncpy(char* dest, const char* src, int num)
{
	assert(dest && src);
	char* ret = dest;
	while (num)
	{
		if (*src == '\0')//此时说明src指针已经指向了待拷贝字符串的结束标志'\0'处，src指针就不用再++了
		{
			*dest = '\0';
			dest++;
		}
		else
		{
			*dest = *src;
			dest++;
			src++;
		}
		num--;
	}
	return ret;
}
int main()
{
	char arr1[20] = "xxxxxxxxxxxxxxxxxxx";
	my_strncpy(arr1, "abcdef", 10);
	printf("%s\n", arr1);
	return 0;
}
```

### 2.6 strncat

[strncat](https://legacy.cplusplus.com/reference/cstring/strncat/?kw=strncat)：长度受限的字符串追加函数

`char * strncat ( char * destination, const char * source, size_t num );`

**注意事项：**

- 从源字符串的第一个字符开始往后数num个字符追加到目标空间的后面，外加一个终止字符。
- 如果源字符串的长度小于 num，则仅复制终止字符之前的内容。

**模拟实现：**

```c
char* my_strncat(char* dest, const char* src, int sz)
{
	assert(dest && src);
	char* ret = dest;
	//找目标空间的\0
	while (*dest != '\0')
	{
		dest++;
	}
	//追加
	while (sz)
	{
		*dest++ = *src++;
		sz--;
	}
	*dest = '\0';
	return ret;
}
int main()
{
	char arr1[20] = "abc\0xxxxxxxxxxx";
	my_strncat(arr1, "defjhigk", 3);
	printf("%s\n", arr1);
	return 0;
}
```

### 2.7 strncmp

[strncmp](https://legacy.cplusplus.com/reference/cstring/strncmp/?kw=strncmp)：长度受限的字符串比较函数

`int strncmp ( const char * str1, const char * str2, size_t num );`

**注意事项：**

- 比较到出现另个字符不一样或者一个字符串结束或者num个字符全部比较完。

**模拟实现：**

```c
int my_strncmp(const char* str1, const char* str2, int sz)
{
	assert(str1 && str2);
	while (sz)
	{
		if (*str1 < *str2)
		{
			return -1;
		}
		else if (*str1 > *str2)
		{
			return 1;
		}
		else if(*str1 == '\0'||*str2 =='\0')//当有一个为'\0'，说明比较就可以结束了
		{
			if (*str1 == '\0' && *str2 == '\0')//如果二者都是'\0'，说明两个字符串相等
			{
				return 0;
			}
			else if(*str1 =='\0')//如果str1为'\0'，说明str1小，str2大
			{
				return -1;
			}
			else//如果src为'\0'，说明str1大，str2小
			{
				return 1;
			}
		}
		sz--;
		str1++;
		str2++;
	}
}
int main()
{
	int ret = my_strncmp("abcdef", "abcd", 5);
	printf("%d\n", ret);
	return 0;
}
```

### 2.8 strstr

[strstr](https://legacy.cplusplus.com/reference/cstring/strstr/?kw=strstr)：字符串查找函数

`const char * strstr ( const char * str1, const char * str2 );`

**注意事项：**

- 在str1指向的字符串中查找str2指向的字符串
- 返回一个指向str1中第一次出现的str2的指针
- 如果 str2 不是 str1 的一部分，则返回一个空指针
- 匹配过程不包括终止空字符，但它到此为止

**模拟实现：**

```c
char* my_strstr(char* str1, char* str2)
{
	assert(str1 && str2);
	if (*str2 == '\0')
	{
		return str1;
	}
	char* s1 = str1;
	char* s2 = str2;
	char* cp = str1;
	while (*cp)
	{
		s1 = cp;
		s2 = str2;
		while (*s1 != '\0' && *s2 != '\0' && *s1 == *s2)
		{
				s1++;
				s2++;
		}
		if (*s2 == '\0')
		{
			return cp;
		}
		cp++;
	}
	return NULL;
}
int main()
{
	char arr1[] = "abbvcbcbbdbbvbnui";
	char arr2[] = "bbvb";
	char* ret = my_strstr(arr1, arr2);
	if (ret == NULL)
	{
		printf("找不到\n");
	}
	else
	{
		printf("%s\n", ret);
	}
	
	return 0;
}
```

### 2.9 strtok

[strtok](https://legacy.cplusplus.com/reference/cstring/strtok/?kw=strtok)：字符串拆分函数

`char * strtok ( char * str, const char * sep );`

**注意事项：**

- sep参数是个字符串，定义了用作分隔符的字符集合
- 第一个参数指定一个字符串，它包含了0个或者多个由sep字符串中一个或者多个分隔符分割的标记
- strtok函数找到str中的下一个标记，并将其用 \0 结尾，返回一个指向这个标记的指针。（注：strtok函数会改变被操作的字符串，所以在使用strtok函数切分的字符串一般都是临时拷贝的内容并且可修改。）
- strtok函数的第一个参数不为 NULL ，函数将找到str中第一个标记，strtok函数将保存它在字符串中的位置
- strtok函数的第一个参数为 NULL ，函数将在同一个字符串中被保存的位置开始，查找下一个标记
- 如果字符串中不存在更多的标记，则返回 NULL 指针

**模拟实现：**

```c
int main()
{
	char arr[] = "liuning@qq.com";
	char* p = "@.";
	char buf[20] = { 0 };
	strcpy(buf, arr);
	char* ret=NULL;
	for (ret = strtok(buf, p); ret != NULL; ret = strtok(NULL, p))
	{
		printf("%s\n", ret);
	}
	return 0;
}
```

### 2.10 memcpy

[memcpy](https://legacy.cplusplus.com/reference/cstring/memcpy/?kw=memcpy)：内存拷贝函数

`void * memcpy ( void * destination, const void * source, size_t num );`

**注意事项：**

-  这里的`destination`指向要在其中赋值内容的目标数组，`source`指向要复制的数据源，num是要复制的字节数，注意这里前两个指针的的类型还有函数返回值都是`void*`，这是因为，`memcpy`这个函数是内存拷贝函数，它有可能拷贝整型，浮点型，结构体等等各种类型的数据……虽然返回类型是`void*`，但他也是必不可少的，`void*`也表示一个地址，用户可以把它强制转换成自己需要的类型去使用。

**应用：**

```c
//把arr1中的1、2、3、4、5拷贝到arr2数组中
int main()
{
	int arr1[] = { 1,2,3,4,5,6,7,8,9,10 };
	int arr2[10] = { 0 };
	memcpy(arr2, arr1, 20);//拷贝5个整型，就是20个字节
	return 0;
}
//把arr1中的3、4、5、6拷贝到arr2数组中
int main()
{
	int arr1[] = { 1,2,3,4,5,6,7,8,9,10 };
	int arr2[10] = { 0 };
	memcpy(arr2, arr1+2, 16);//此时只要改变参数中数据源的地址就可以，把3的地址传过去就行，复制4个整型就是1个字节
	return 0;
}
```

**模拟实现：**

```c
void* my_memcpy(void* dest, const void* src, size_t num)
{
	assert(dest && src);
	void* ret = dest;
	while (num)
	{
		*(char*)dest = *(char*)src;
		((char*)dest)++;
		((char*)src)++;
		num--;
	}
	return ret;
}
int main()
{
	int arr1[] = { 1,2,3,4,5,6,7,8,9,10 };
	int arr2[10] = { 0 };
	my_memcpy(arr2, arr1+2, 16);
	return 0;
}
```

### 2.11 memmove

[memmove](https://legacy.cplusplus.com/reference/cstring/memmove/?kw=memmove)：内存拷贝函数

`void * memmove ( void * destination, const void * source, size_t num );`

**注意事项：**

-  它的参数、返回值与`memcpy`函数一模一样。这里就不过多介绍。对于这两个函数来说，目标空间必须足够大，不然就会发生越界访问。

**模拟实现：**

```c
void* my_memmove(void* dest, const void* src, int num)
{
	assert(dest && src);
	void* ret = dest;
	if (dest < src)//目标空间的地址小，说们目标空间靠前
	{
		//从前向后
		while (num--)
		{
			*(char*)dest = *(char*)src;
			dest = (char*)dest + 1;
			src = (char*)src + 1;
		}
	}
	else
	{
		//从后往前
		while (num--)//num为1的时候，下面的num就是0
		{
			*((char*)dest + num) = *((char*)src + num);//通过num的减减就可以实现对每一个字节的访问
		}
	}
	return ret;
}
int main()
{
	int arr1[] = { 1,2,3,4,5,6,7,8,9,10 };
	my_memmove(arr1+2, arr1, 20);
	return 0;
}
```

### 2.12 memcmp

[memcmp](https://legacy.cplusplus.com/reference/cstring/memcmp/?kw=memcmp)：内存比较函数

`int memcmp ( const void * ptr1, const void * ptr2, size_t num );`

**注意事项：**

- 比较从ptr1和ptr2指针开始的num个字节
- 两个内存块中不匹配的第一个字节在 ptr1 中的值低于 ptr2 中的值返回一个小于零的数子，相等返回零，两个内存块中不匹配的第一个字节在 ptr1 中的值大于在 ptr2 中的值返回一个大于零的数子

**模拟实现：**
```c
int memcmp(const void *buffer1,const void *buffer2,int count) {
   if (!count)
        return(0);
   while ( --count && *(char *)buffer1 == *(char *)buffer2) {
        buffer1 = (char *)buffer1 + 1;
        buffer2 = (char *)buffer2 + 1;
   }
    return( *((unsigned char *)buffer1) - *((unsigned char *)buffer2) );
}

```


### 2.13 memset

[memset](https://legacy.cplusplus.com/reference/cstring/memset/?kw=memset)：内存设置函数

`void * memset ( void * ptr, int value, size_t num );`

**注意事项：**

- 以字节为单位来设置内存中的数据，把从ptr开始往后的num个字节设置成value
- 形参value也可以是字符，字符其实也是整型，因为字符在内存中存的是其ASCII
- value如果是整数的话，需要注意它的取值范围，因为一个字节最大可以存储255，超过255就会发生截断

**模拟实现：**

```c
void *(memset)(void *s, int c, size_t n)  
{  
    const unsigned char uc = c;  
    unsigned char *su;  
    for (su = s; 0 < n; ++su, --n)  
        *su = uc;  
    return (s);  
}
```
`memset`函数每次是以 一个字节为单位来进行赋值的，而不是一次性赋值4/8个字节，那么问题来了，当我们以int为单位的时候，它究竟是怎样进行的？例如以`memset(arr,1,sizeof(arr));` 来对数组进行初始化，int类型，那么就会导致一个结果，就是在以字节赋值的时候，int 类型每次调用4个字节(32bit)，他会将32bit 分为4*8个bit，每次将最低的bit位进行赋值。

```
使得二进制数变为         
实际的结果->00000001 00000001 00000001 00000001
想要的结果->00000000 00000000 00000000 00000001
```

很明显与我们想要赋值的1，也就是`00000000 00000000 00000000 00000001`是不匹配的，如果换算为10进制是一个非常大的值(16843009)是错误的赋值方法。

`memset`函数适用于将指定内存区域的每个字节设置为特定值。对于整型数组，可以使用`memset`函数来将每个元素设置为1，但是需要考虑到整型的大小。因为`memset`是按字节为单位进行操作的，所以在对整型数组赋值时，需要将要设置的值强制转换为`unsigned char`类型。

以下是使用`memset`给整型数组每个元素赋`1`的示例代码：

```c
#include <stdio.h>
#include <string.h>

int main() {
    int data[10];
    int value = 1;

    // 使用memset将整个数组设置为特定值
    memset(data, (unsigned char)value, sizeof(data));

    // 打印数组中的值进行验证
    for (int i = 0; i < 10; i++) {
        printf("%d ", data[i]);
    }

    return 0;
}
```

#### 2.13.1 初始化结构体

```c
struct sample_struct
　　{
　　char csName[16];
　　int iSeq;
　　int iType;
　　};
　　struct sample_strcut stTest;
　　//一般情况下，清空stTest的方法：
　　stTest.csName[0]='/0';
　　stTest.iSeq=0;
　　stTest.iType=0;
　　//用memset就非常方便，明显优于for循环
　　memset(&stTest,0,sizeof(struct sample_struct));
　　//如果是数组：
　　struct sample_struct test[10]；
　　memset(test,0,sizeof(struct sample_struct)*10);
```

#### 2.13.2 竞赛中Memset中无穷大常量的设定技巧

如果问题中各数据的范围明确，那么无穷大的设定不是问题，在不明确的情况下，很多程序员都取`0x7fffffff`作为无穷大，因为这是32-bit int的最大值。如果这个无穷大只用于一般的比较（比如求最小值时`min`变量的初值），那么`0x7fffffff`确实是一个完美的选择，但是在更多的情况下，`0x7fffffff`并不是一个好的选择。
很多时候我们并不只是单纯拿无穷大来作比较，而是会运算后再做比较，例如在大部分最短路径算法中都会使用的松弛操作：

```c
if (d[u]+w[u][v]<d[v]) d[v]=d[u]+w[u][v];
```

我们知道如果`u,v`之间没有边，那么`w[u][v]=INF`，如果我们的`INF`取`0x7fffffff`，那么`d[u]+w[u][v]`会溢出而变成负数，我们的松弛操作便出错了，更一般的说，`0x7fffffff`不能满足“无穷大加一个有穷的数依然是无穷大”，它变成了一个很小的负数。
除了要满足加上一个常数依然是无穷大之外，我们的常量还应该满足“无穷大加无穷大依然是无穷大”，至少两个无穷大相加不应该出现灾难性的错误，这一点上`0x7fffffff`依然不能满足我们。
所以我们需要一个更好的家伙来顶替`0x7fffffff`，最严谨的办法当然是对无穷大进行特别处理而不是找一个很大很大的常量来代替它（或者说模拟它），但是这样会让我们的编程过程变得很麻烦。在我读过的代码中，最精巧的无穷大常量取值是0x3f3f3f3f，我不知道是谁最先开始使用这个精妙的常量来做无穷大，不过我的确是从一位不认识的ACMer([ID:Staginner](https://www.cnblogs.com/staginner/))的博客上学到的，他/她的很多代码中都使用了这个常量，于是我自己也尝试了一下，发现非常好用，而当我对这个常量做更深入的分析时，就发现它真的是非常精巧了。`0x3f3f3f3f`的十进制是`1061109567`，也就是$10^9$级别的（和`0x7fffffff`一个数量级），而一般场合下的数据都是小于$10^9$的，所以它可以作为无穷大使用而不致出现数据大于无穷大的情形。另一方面，由于一般的数据都不会大于$10^9$，所以当我们把无穷大加上一个数据时，它并不会溢出（这就满足了“无穷大加一个有穷的数依然是无穷大”），事实上`0x3f3f3f3f+0x3f3f3f3f=2122219134`，这非常大但却没有超过32-bit int的表示范围，所以`0x3f3f3f3f`还满足了我们“无穷大加无穷大还是无穷大”的需求。 最后，`0x3f3f3f3f`还能给我们带来一个意想不到的额外好处：如果我们想要将某个数组清零，我们通常会使用`memset(a,0,sizeof(a))`这样的代码来实现（方便而高效），但是当我们想将某个数组全部赋值为无穷大时（例如解决图论问题时邻接矩阵的初始化），就不能使用memset函数而得自己写循环了（写这些不重要的代码真的很痛苦），我们知道这是因为`memset`是按字节操作的，它能够对数组清零是因为0的每个字节都是0，现在好了，如果我们将无穷大设为`0x3f3f3f3f`，那么奇迹就发生了，`0x3f3f3f3f`的每个字节都是`0x3f`！所以要把一段内存全部置为无穷大，我们只需要`memset(a,0x3f,sizeof(a))`。

### 2.14 printf

[printf](https://legacy.cplusplus.com/reference/cstdio/printf/)： 标准输入输出打印函数。 第一个参数是显示字符串的格式定义，的参数型为“...”也就是动态参数，说明printf在实现时可以接收不同数量的参数处理。返回值是int类型，表示它一共显示了多少个字节的字符。

`int printf ( const char * format, ... );`

**注意事项：**

- 以字节为单位来设置内存中的数据，把从ptr开始往后的num个字节设置成value
- 形参value也可以是字符，字符其实也是整型，因为字符在内存中存的是其ASCII
- value如果是整数的话，需要注意它的取值范围，因为一个字节最大可以存储255，超过255就会发生截断

**模拟实现：**

首先定义两个宏来实现动态不定参数：

```c
//定义动态参数地址号 
typedef u32 va_list; 
/*** 
 * 初始化动态参数地址 
 * v: 动态参数地址号 
 * a: 前一个参数变量 
 */ 
#define va_init(v, a)                        \ 
        ({                                   \ 
                v = (va_list)(&a);           \ 
        }) 
/*** 
 * 取得下一个参数的值 
 * v: 动态参数地址号 
 * t: 下一个参数的类型 
 * return: 返回下一个参数的值 
 */ 
#define va_arg(v, t)                         \ 
        ({                                   \ 
                v += 4;                      \ 
                (t)(*((t*)(v)));             \ 
        })
```

 以上两个就实现了动态参数的取值。对于`(t)(*((t*)(v)));`这一行语句做一下说明。`v`是一个u32类型，`(t*)(v)`就是把v强制类型转换为`(t*)`的指针类型，再将这个指针类型做取值运算`(*((t*)(v)))`即是取得这个参数的实际值，为了在赋值时进行安全的值传递，还要将这个结果进行一次强制类型转换`(t)(*((t*)(v)))`。我们来看一下在使用这个宏时它的宏展开：

 ```c
char ch = va_arg(args, char); 
char ch = ({args += 4; (char)(*((char*)(args)));});
 ```

 在能够取得动态参数之后，就可以根据`printf`的第1个参数的格式化字符串来处理显示程序了。C语言标准输出`printf`函数的格式有很多，我们不打算实现过多的复杂功能，在这里只实现它显示字符`(char: %c)`、字符串`(const char*: %s)`、整数`(int %d)`和无符号16进制整数`(u32: %x)`。在printf里读入格式化字符串：

 ```c
 //读到\0为结束 
while (*fmt != '\0') 
{ 
        //格式化标记% 
        if (*fmt == '%') 
        { 
                //显示一个字符 
                if ('c' == *(fmt + 1)) 
                { 
                        ch = va_arg(args, char); 
                        putchar(ch); 
                        count++; 
                        fmt += 2; 
                } 
                //显示字符串 
                else if ('s' == *(fmt + 1)) 
                { 
                        str = va_arg(args, char*); 
                        count += puts(str); 
                        fmt += 2; 
                } 
                //显示整数 
                else if ('d' == *(fmt + 1)) 
                { 
                        number_to_str(buff, va_arg(args, int), 10); 
                        count += puts(buff); 
                        fmt += 2; 
                } 
                //显示无符号16进制整数 
                else if ('x' == *(fmt + 1)) 
                { 
                        number_to_str(buff, va_arg(args, u32), 16); 
                        count += puts(buff); 
                        fmt += 2; 
                } 
        } 
        //显示普通字符 
        else 
        { 
                putchar(*fmt++); 
                count++; 
        } 
}
 ```

 其中还用到了两个函数：显示字符串`puts`和数字转字符串`number_to_str`。它们的实现如下：

 ```c
 /*
 * number_to_str : 将整数转为字符串
 *  - int tty_id : tty编号
 *  - char *buff : 数据地址
 *  - int number : 整数
 *  - int hex : 10进制或16进制
 * return : void
 */
void number_to_str(char *buff, int number, int hex)
{
	char temp[0x800];
	char num[0x20] = "0123456789ABCDEFG";

	int i = 0;
	int length = 0;
	int rem;
	char sign = '+';

	//反向加入temp
	temp[i++] = '\0';
	if (number < 0)
	{
		sign = '-';
		number = 0 - number;
	}
	else if (number == 0)
	{
		temp[i++] = '0';
	}

	//将数字转为字符串
	while (number > 0)
	{
		rem = number % hex;
		temp[i++] = num[rem];
		number = number / hex;
	}
	//处理符号
	if (sign == '-')
	{
		temp[i++] = sign;
	}
	length = i;

	//返向拷贝到buff缓冲区
	for (i = length - 1; i >= 0; i--)
	{
		*buff++ = temp[i];
	}
}

/*
 * puts : 显示字符串
 *  - int tty_id : tty编号
 *  - char *str : 字符串
 * return : void
 */
int puts(int tty_id, char *str)
{
	int count = 0;
	while (*str != '\0')
	{
		putchar(tty_id, *str++);
		count++;
	}
	return count;
}
 ```

参考：https://www.askpure.com/course_638FTDK7-A4EQ5DCZ-2R8Q7MLF-JUI0DFTK.html。

#### 2.14.1 嵌入式printf裸机串口

一共三个步骤，主要是初始化Usartinit()，重写putc()、getc()函数:

- Usartinit()
该函数主要配置UART的，波特率115200，数据位：8，奇偶校验位：0，终止位：1，不设置流控。

- putc()

该函数是向串口发送一个数据data，他的实现逻辑就是轮询检查寄存器UART2.UTRSTAT2 ，判断其bite【1】是否置1，如果置1，则向UART2.UTXH2存入要发送的数据即可。

- getc()
该函数是从串口接收一个数据data，他的实现逻辑就是轮询检查寄存器UART2.UTRSTAT2 ，判断其bite【0】是否置1，如果置1，说明数据准备好，则可以从寄存器UART2.URXH2取出数据。

```c
/*
 * UART2
 */
typedef struct {
				unsigned int ULCON2;
				unsigned int UCON2;
				unsigned int UFCON2;
				unsigned int UMCON2;
				unsigned int UTRSTAT2;
				unsigned int UERSTAT2;
				unsigned int UFSTAT2;
				unsigned int UMSTAT2;
				unsigned int UTXH2;
				unsigned int URXH2;
				unsigned int UBRDIV2;
				unsigned int UFRACVAL2;
				unsigned int UINTP2;
				unsigned int UINTSP2;
				unsigned int UINTM2;
}uart2;
#define UART2 ( * (volatile uart2 *)0x13820000 )
/* GPA1 */
typedef struct {
				unsigned int CON;
				unsigned int DAT;
				unsigned int PUD;
				unsigned int DRV;
				unsigned int CONPDN;
				unsigned int PUDPDN;
}gpa1;
#define GPA1 (* (volatile gpa1 *)0x11400020)
void uart_init()
{	/*UART2 initialize*/
	GPA1.CON = (GPA1.CON & ~0xFF ) | (0x22); //GPA1_0:RX;GPA1_1:TX
	UART2.ULCON2 = 0x3; //Normal mode, No parity,One stop bit,8 data bits
	UART2.UCON2 = 0x5;  //Interrupt request or polling mode
	//Baud-rate : src_clock:100Mhz
	UART2.UBRDIV2 = 0x35;
	UART2.UFRACVAL2 = 0x4;
}
void putc(const char data)
{	while(!(UART2.UTRSTAT2 & 0X2));
	UART2.UTXH2 = data;
	if (data == '\n')
			putc('\r');
}
char getc(void)
{	char data;
	while(!(UART2.UTRSTAT2 & 0x1));
	data = UART2.URXH2;
	if ((data == '\n')||(data == '\r'))
	{
			putc('\n');
			putc('\r');
	}else
			putc(data);
	return data;
}
```

然后`printf()`可以调用`vsprintf `来实现，需要借助头文件ctype.h、stdarg.h中一些宏。

```c
void printf (const char *fmt, ...)
{
	va_list args;
	unsigned int i;
	char printbuffer[100];
	va_start (args, fmt);

	/* For this to work, printbuffer must be larger than
	 * anything we ever want to print.
	 */
	i = vsprintf (printbuffer, fmt, args);//对输入的参数进行格式整理
	va_end (args);
	puts (printbuffer); //调用上一章我们封装的puts函数实现向串口打印书字符串
}
```

此外还可以结合DMA来进行传输，这里就不讲了。Debug日志系统贴在下边。

```c
#ifndef _DEBUG_H_
#define _DEBUG_H_

/* 
 * debug control, you can switch on (delete 'x' suffix)
 * to enable log output and assert mechanism
 */
#define CONFIG_ENABLE_DEBUG

/* 
 * debug level,
 * if is DEBUG_LEVEL_DISABLE, no log is allowed output,
 * if is DEBUG_LEVEL_ERR, only ERR is allowed output,
 * if is DEBUG_LEVEL_INFO, ERR and INFO are allowed output,
 * if is DEBUG_LEVEL_DEBUG, all log are allowed output,
 */
enum debug_level {
    DEBUG_LEVEL_DISABLE = 0,
    DEBUG_LEVEL_ERR,
    DEBUG_LEVEL_INFO,
    DEBUG_LEVEL_DEBUG
};

#ifdef CONFIG_ENABLE_DEBUG

/* it can be change to others, such as file operations */
#include <stdio.h>
#define PRINT               printf

/* 
 * the macro to set debug level, you should call it 
 * once in the files you need use debug system
 */
#define DEBUG_SET_LEVEL(x)  static int debug = x

#define ASSERT()                                        \
do {                                                    \
    PRINT("ASSERT: %s %s %d",                           \
           __FILE__, __FUNCTION__, __LINE__);           \
    while (1);                                          \
} while (0)

#define ERR(...)                                        \
do {                                                    \
    if (debug >= DEBUG_LEVEL_ERR) {                     \
        PRINT(__VA_ARGS__);                             \
    }                                                   \
} while (0)

#define INFO(...)                                       \
do {                                                    \
    if (debug >= DEBUG_LEVEL_INFO) {                    \
        PRINT(__VA_ARGS__);                             \
    }                                                   \
} while (0)

#define DEBUG(...)                                      \
do {                                                    \
    if (debug >= DEBUG_LEVEL_DEBUG) {                   \
        PRINT(__VA_ARGS__);                             \
    }                                                   \
} while (0)

#else   /* CONFIG_ENABLE_DEBUG  */

#define DEBUG_SET_LEVEL(x) 
#define ASSERT()
#define ERR(...)
#define INFO(...)
#define DEBUG(...)

#endif  /* CONFIG_ENABLE_DEBUG  */

#endif  /* _DEBUG_H_ */
```
## 参考文献

[1] [如何实现基于Cortex-A9 的UART裸机驱动并实现printf函数](https://blog.csdn.net/daocaokafei/article/details/108273041?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7ERate-5-108273041-blog-129898555.235%5Ev38%5Epc_relevant_sort&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7ERate-5-108273041-blog-129898555.235%5Ev38%5Epc_relevant_sort&utm_relevant_index=6)

[2] [手动写一个printf函数与spinrtf函数，基于Cortex-M处理器](https://blog.csdn.net/PaguC/article/details/129898555?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-129898555-blog-24582895.235^v38^pc_relevant_sort&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

[3] [【嵌入式C语言】可变参数 va_start、va_arg、va_end、va_list、stdarg.h 库详解](https://huaweicloud.csdn.net/6541fb455543f15fea1a1bcb.html?dp_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6NzI1ODYzLCJleHAiOjE3MDA0ODQxMjMsImlhdCI6MTY5OTg3OTMyMywidXNlcm5hbWUiOiJsZW1hZGVuNTIwIn0.gRgX4BK8Oh1bzJnsN145E82yxcN4eArqPSxhz482ZMw)

[4] https://github.com/magicworldos/lidqos/blob/master/lidqos/printf/printf.c

[5] [跟我一起写操作系统](https://www.askpure.com/course_638FTDK7-A4EQ5DCZ-2R8Q7MLF-JUI0DFTK.html)

[6] https://github.com/wowotech/wowolib/tree/master/devmem
