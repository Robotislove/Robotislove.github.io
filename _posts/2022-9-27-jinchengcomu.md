---
layout: post
title: Linux进程间通信方式总结学习笔记
date: 2022-09-27
author: lau
tags: [Linux, Blog]
comments: true
toc: false
pinned: false

---

Linux进程间通信方式总结学习笔记。

<!-- more -->

## 进程通信

### 概述

![](https://uploadfiles.nowcoder.com/images/20210505/427313909_1620218183328/C75529FD0328F3E14181DD53AE2DA817)

- 消息传递:  消息传递是进程间实现通信和同步等待的机制,使用消息传递,进程间的交流不需要共享变量，直接就可以进行通信;消息传递分为发送方和接收方。
- 先进先出队列:  先进先出队列指的是两个不相关进程间的通信,两个进程之间可以相互进程通信，这是一种全双工通信方式。
- 管道：管道用于两个相关进程之间的通信，这是一种半双工的通信方式，如果需要全双工，需要另外一个管道。
- 直接通信：在这种进程通信的方式中，进程与进程之间只存在一条链接，进程间要明确通信双方的命名。
- 间接通信：间接通信是通信双方不会直接建立连接，而是找到一个中介者，这个中介者可能是个对象等等，进程可以在其中放置消息，并且可以从中删除消息，以此达到进程间通信的目的。
- 消息队列:  消息队列是内核中存储信息的链表,它由消息队列标识符进行标识,这种方式能够在不同的进程之间提供全双工的通信连接。
- 共享内存:  共享内存是使用所有进程之间的内存来建立连接,这种类型需要同步进程访问来相互保护。

### 管道

UNIX 系统IPC最古老的形式。
**特点：**

1. 只支持半双工通信（单向交替传输）
2. 只能在父子进程或兄弟进程之间通信
3. 只存在于内存中，不属于任何文件系统

#### 命名管道（[FIFO](https://so.csdn.net/so/search?q=FIFO&spm=1001.2101.3001.7020)）

常用于C/S应用程序中，FIFO 用作汇聚点，在客户进程和服务器进程之间传递数据，它是一种文件类型。
**特点：**

1. 不再局限于父子进程和兄弟进程之间的通信，可以在任意进程之间通信
2. 有路径名与之关联，以一种特殊设备文件形式存在文件系统

#### 消息队列
消息队列，是消息的链表，存放在内核中。一个消息队列由一个标识符（即队列ID）来标识。
**特点：**

- 队列中的消息可以有特定的格式、类型和优先级
- 可独立于读写进程存在，进程终止时，队列中的消息也不会被删除
- 避免同步阻塞问题，不需要进程自己提供同步方法
- 实现消息的随机查询，读进程可根据消息类型有选择的接收消息
  

#### 信号量机制
信号量（semaphore）是一个计数器，用于实现进程间的互斥与同步，而不是用于存储进程间通信数据。

最简单的信号量是只能取 0 和 1 的变量，叫做二值信号量（Binary Semaphore），可以取正整数的信号量被称为通用信号量。
**注意**
Linux 下的信号量函数都是在通用信号量数组上进行操作，而不是在二值信号量上进行操作。

**特点：**

- 信号量用于进程间同步，若要在进程间传递数据需要用共享内存的方式
- 信号量基于操作系统的 PV 操作，对信号量的操作都是原子操作（底层通过屏蔽中断的方式实现）
- 每次对信号量的 PV 操作可以加减任意正整数（不局限于+1和-1）
- 通过信号量组多条件的同步机制

 信号量是用于提供不用进程或者不同线程之间的同步手段的原语，分为：

- Posix 有名信号量：可用于进程或者线程间的同步；
- Posix基于内存的信号量（无名信号量）：存放于共享内存中，可用于进程或者线程的同步；
- System V信号量：在内核中维护，可用于进程或线程间的同步。
    

另外信号量同时根据取值不同可分为：

- 二值信号量：取值只能是0或者1，可以实现类似互斥锁的跨进程的锁机制。
- 计数信号量：取值大于1。
    

计数信号量具备两种操作动作，称为V（signal()）与P（wait()）（即部分参考书常称的“PV操作”）。V操作会增加信号标S的数值，P操作会减少它。

- P操作，检测信标值S：
  - 如果S <= 0，则阻塞直到S > 0，并将信标值S=S-1；
  - 如果S > 0，则直接S=S-1；

- V操作：直接增加信标值`S=S+1`。

 PV操作某种程度上等价于如下代码只不过是原子的。

```c++
int p(int seq)
{
    while(seq <= 0) ;
    return --seq;
}

int v(int seq)
{
    return ++seq;
}
```

互斥锁，信号量和条件变量的区别：

1. 互斥锁强调的是资源的访问互斥，信号量强调进程或者线程之间的同步，条件变量常与互斥锁同时使用，达到线程同步的目的：条件变量通过允许线程阻塞
2. 等待另一个线程发送信号的方法弥补了互斥锁的不足；
3. 互斥锁总是必须由给其上锁的线程解锁，信号量的挂出确不必由执行过它的等待操作的同一线程执行；
4. 互斥锁要么被锁住，要么被解锁（二值状态，类似于二值信号量）；
5. 既然信号量有一个与之关联的状态（它的数值），那么信号量的挂出操作总是被记住。然而当向一个条件变量发送信号时，如果没有线程等待在该条件变量上，那么信号将丢失。

#### Posix信号量

##### 有名信号量

```c++
#include <semaphore.h>
sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value);
int sem_close(sem_t *sem);
int sem_unlink(const char *name);
int sem_post(sem_t *sem);
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_getvalue(sem_t *sem, int *sval);
```

- sem_open：创建或者打开一个信号量；
  - name：信号量在文件系统上的文件名；
  - oflag：可以为0,O_CREAT或者O_CREAT|O_EXCL；
  - mode：信号量的权限位；
  - value：信号量的初值不能大于SEM_VALUE_MAX；
  - 返回自sem_t是一个该信号量的标识符的指针；

- `sem_close`：关闭一个信号量，但是并不删除信号量，另外进程终止系统会自动关闭当前进程打开的信号量；
  - `sem`：信号量的指针，通过`sem_open`获得；
- `sem_unlink`：删除信号量，需要注意的是信号量同样维护了一个引用计数，只有当引用计数为0时才会显示的删除；
  - `name`：信号量的在文件系统上的唯一标示；
- `sem_post`：V操作，将信号量的值+1，并唤醒等待信号量的任意线程；
  - `sem`：信号量的指针，通过`sem_open`获得；
- `sem_wait`：P操作，如果当前信号量小于等于0则阻塞，将信号量的值-1，否则直接将信号量-1；
  - `sem`：信号量的指针，通过`sem_open`获得；
- `sem_trywait`：非阻塞的`sem_wait`；
  - `sem`：信号量的指针，通过`sem_open`获得；
- `sem_getvalue`：获取当前信号量的值，如果当前信号量上锁则返回0或者负值，其绝对值为信号量的值；
  - `sem`：信号量的指针，通过`sem_open`获得；
  - `sval`：信号量的值；
- 以上的函数除了`sem_open`失败返回`SEM_FAILED`，均是成功返回0，失败返回-1并且设置`errno`。

简单的消费者-生产者问题：

```c++
#define BUFF_SIZE 5
typedef struct named_share
{
    char buff[BUFF_SIZE];
    sem_t *lock;
    sem_t *nempty;
    sem_t *nstored;
    int items;
}named_share;

void* named_produce(void *arg)
{
    named_share *ptr = arg;
    for(int i = 0;i < ptr->items;i++)
    {
        lsem_wait(ptr->nempty);
        lsem_wait(ptr->lock);   //锁

        //生产物品
        ptr->buff[i % BUFF_SIZE] = rand() % 1000;
        printf("produce %04d into %2d\n", ptr->buff[i % BUFF_SIZE], i % BUFF_SIZE);
        sleep(rand()%2);
        lsem_post(ptr->lock);   //解锁
        lsem_post(ptr->nstored);
    }

    return NULL;
}

void* named_consume(void *arg)
{
    named_share *ptr = arg;
    for(int i = 0;i < ptr->items;i++)
    {
        lsem_wait(ptr->nstored);
        lsem_wait(ptr->lock);   //锁

        //生产物品
        printf("consume %04d at   %2d\n", ptr->buff[i % BUFF_SIZE], i % BUFF_SIZE);
        ptr->buff[i % BUFF_SIZE] = -1;
        sleep(rand()%2);
        lsem_post(ptr->lock);   //解锁
        lsem_post(ptr->nempty);
        //iterator
    }

    return NULL;
}

void named_sem_test()
{
    char *nempty_name = "nempty";
    char *nstored_name = "nstored";
    char *lock_name = "lock";
    
    int items = 10;
    int flag = O_CREAT | O_EXCL;

    named_share arg;
    srand(time(NULL));
    arg.items = items;
    memset(arg.buff, -1, sizeof(int) * BUFF_SIZE);
    arg.nempty = lsem_open(lpx_ipc_name(nempty_name), flag, FILE_MODE, BUFF_SIZE);
    arg.nstored = lsem_open(lpx_ipc_name(nstored_name), flag, FILE_MODE, 0);
    arg.lock = lsem_open(lpx_ipc_name(lock_name), flag, FILE_MODE, 1);
    
    pthread_t pid1, pid2;
    
    int val = 0;
    lsem_getvalue(arg.nstored, &val);
    lsem_getvalue(arg.nempty, &val);

    pthread_setconcurrency(2); 
    lpthread_create(&pid1, NULL, named_produce, &arg);
    lpthread_create(&pid2, NULL, named_consume, &arg);

    lpthread_join(pid1, NULL);
    lpthread_join(pid2, NULL);

    lsem_unlink(lpx_ipc_name(nempty_name));
    lsem_unlink(lpx_ipc_name(nstored_name));
    lsem_unlink(lpx_ipc_name(lock_name));
}
```

  结果：

```c++
➜  build git:(master) ✗ ./main
produce 0038 into  0
produce 0022 into  1
produce 0049 into  2
produce 0060 into  3
produce 0090 into  4
consume 0038 at    0
consume 0022 at    1
consume 0049 at    2
consume 0060 at    3
consume 0090 at    4
produce 0031 into  0
produce -056 into  1
produce -103 into  2
produce -047 into  3
produce -100 into  4
consume 0031 at    0
consume -056 at    1
consume -103 at    2
consume -047 at    3
consume -100 at    4
```

##### 无名信号量

```c++
int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_destroy(sem_t *sem);
```

- \```sem_init``：初始化一个无名信号量；
  - `sem`：信号量的指针；
  - `pshared`：标志是否共享：
    - `pshared==0`：该信号量只能在同一进程不同线程之间共享，当进程终止则消失；
    - `pshared!=0`：该信号量驻留与共享内存区，可以在不同进程之间进行共享；
  - `value`：信号量的初值；
  - 返回值出错返回-1，成功并不返回0；
- `sem_destroy`：销毁信号量。成功返回0，失败返回-1。

上面的程序的简易修改：

```c++
void unnamed_sem_test()
{
    int items = 10;

    named_share arg;
    srand(time(NULL));
    arg.items = items;
    memset(arg.buff, -1, sizeof(int) * BUFF_SIZE);

    arg.lock = (sem_t*)malloc(sizeof(sem_t));
    arg.nempty = (sem_t*)malloc(sizeof(sem_t));
    arg.nstored = (sem_t*)malloc(sizeof(sem_t));
    
    lsem_init(arg.lock, 0, 1);
    lsem_init(arg.nempty, 0, BUFF_SIZE);
    lsem_init(arg.nstored, 0, 0);
    
    pthread_t pid1, pid2;


    pthread_setconcurrency(2); 
    lpthread_create(&pid1, NULL, named_produce, &arg);
    lpthread_create(&pid2, NULL, named_consume, &arg);

    lpthread_join(pid1, NULL);
    lpthread_join(pid2, NULL);

    lsem_destroy(arg.lock);
    lsem_destroy(arg.nempty);
    lsem_destroy(arg.nstored);

    SAFE_RELEASE(arg.lock);
    SAFE_RELEASE(arg.nempty);
    SAFE_RELEASE(arg.nstored);
}
```

```c++
➜  build git:(master) ✗ ./main 
produce -120 into  0
produce -098 into  1
produce 0123 into  2
produce -058 into  3
produce 0028 into  4
consume -120 at    0
consume -098 at    1
consume 0123 at    2
consume -058 at    3
consume 0028 at    4
produce 0110 into  0
produce 0034 into  1
produce -068 into  2
produce -115 into  3
produce 0004 into  4
consume 0110 at    0
consume 0034 at    1
consume -068 at    2
consume -115 at    3
consume 0004 at    4
```

##### 多生产者单消费者

多生产者单消费者需要关注的是不同生产者之间的进度同步。

```c++
typedef struct multp_singc_share
{
    int i;
    sem_t nempty;
    sem_t nstored;
    sem_t lock;
    int items;
    char buff[BUFF_SIZE];
}multp_singc_share;

void* multp_singc_produce(void *arg)
{
    multp_singc_share *ptr = arg;
    for(;;)
    {
        lsem_wait(&(ptr->nempty));
        lsem_wait(&(ptr->lock));
        if(ptr->i >= ptr->items)
        {
            lsem_post(&(ptr->lock));
            lsem_post(&(ptr->nempty));
            return NULL;
        }
        
        ptr->buff[ptr->i % BUFF_SIZE] = rand() % 100;
        printf("produce %d at %d\n", ptr->buff[ptr->i * BUFF_SIZE], ptr->i % BUFF_SIZE);
        ptr->i++;
        lsem_post(&ptr->lock);
        lsem_post(&ptr->nstored);
    }
}

void* multp_singc_consume(void *arg)
{
    multp_singc_share *ptr = arg;
    for(int i = 0;i < ptr->items;i ++)
    {
        lsem_wait(&(ptr->nstored));
        lsem_wait(&(ptr->lock));
        
        printf("consume %d at %d\n", ptr->buff[i % BUFF_SIZE], i % BUFF_SIZE);

        lsem_post(&(ptr->lock));
        lsem_post(&(ptr->nempty));
    }
}

void multp_singc_test()
{
    multp_singc_share arg;
    pthread_t pro_th[THREAD_SIZE], con_th;

    arg.items = 10;
    arg.i = 0;
    memset(arg.buff, 0, BUFF_SIZE * sizeof(int));
    lsem_init(&(arg.lock), 0, 1);
    lsem_init(&(arg.nempty), 0, BUFF_SIZE);
    lsem_init(&(arg.nstored), 0, 0);

    pthread_setconcurrency(THREAD_SIZE + 1); 
    for(int i = 0;i < THREAD_SIZE;i ++)
    {
        lpthread_create(&pro_th[i], NULL, multp_singc_produce, &arg);
    }
    
    lpthread_create(&con_th, NULL, multp_singc_consume, &arg);

    for(int i = 0;i < THREAD_SIZE;i ++)
    {
        lpthread_join(pro_th[i], NULL);
    }

    lpthread_join(con_th, NULL);

    lsem_destroy(&(arg.lock));
    lsem_destroy(&(arg.nempty));
    lsem_destroy(&(arg.nstored));
}
```

```c++
➜  build git:(master) ✗ ./main   
produce 83 at 0
produce 0 at 1
produce 0 at 2
produce 0 at 3
consume 83 at 0
consume 86 at 1
produce 48 at 4
produce 127 at 0
produce 64 at 1
consume 77 at 2
consume 15 at 3
produce 0 at 2
consume 93 at 4
consume 35 at 0
produce -4 at 3
produce 31 at 4
consume 86 at 1
consume 92 at 2
consume 49 at 3
consume 21 at 4
```

##### 多生产者多消费者

多个生产者和消费者互相产生和读取数据。

```c++
typedef struct multp_multc_share
{
    int pi;
    int ci;
    sem_t nempty;
    sem_t nstored;
    sem_t lock;
    int items;
    char buff[BUFF_SIZE];
}multp_multc_share;

void* multp_multc_produce(void *arg)
{
    multp_multc_share *ptr = arg;
    for(;;)
    {
        lsem_wait(&(ptr->nempty));
        lsem_wait(&(ptr->lock));
        if(ptr->pi >= ptr->items)
        {
            lsem_post(&(ptr->nstored));
            lsem_post(&(ptr->nempty));
            lsem_post(&(ptr->lock));
            return NULL;
        }
        
        ptr->buff[ptr->pi % BUFF_SIZE] = rand() % 100;
        printf("produce %d at %d\n", ptr->buff[ptr->pi * BUFF_SIZE], ptr->pi % BUFF_SIZE);
        ptr->pi++;
        lsem_post(&ptr->lock);
        lsem_post(&ptr->nstored);
    }
}

void* multp_multc_consume(void *arg)
{
    multp_multc_share *ptr = arg;
    for(;;)
    {
        lsem_wait(&(ptr->nstored));
        lsem_wait(&(ptr->lock));
        if(ptr->ci >= ptr->items)
        {
            lsem_post(&(ptr->nstored));
            lsem_post(&(ptr->lock));
            return NULL;
        }

        printf("consume %d at %d\n", ptr->buff[ptr->ci % BUFF_SIZE], ptr->ci % BUFF_SIZE);
        ptr->ci++;
        lsem_post(&(ptr->lock));
        lsem_post(&(ptr->nempty));
    }
}

void multp_multc_test()
{
    multp_multc_share arg;
    pthread_t pro_th[THREAD_SIZE], con_th[THREAD_SIZE];

    arg.items = 10;
    arg.pi = 0;
    arg.ci = 0;
    memset(arg.buff, 0, BUFF_SIZE * sizeof(int));
    lsem_init(&(arg.lock), 0, 1);
    lsem_init(&(arg.nempty), 0, BUFF_SIZE);
    lsem_init(&(arg.nstored), 0, 0);

    pthread_setconcurrency(THREAD_SIZE + 1); 
    for(int i = 0;i < THREAD_SIZE;i ++)
    {
        lpthread_create(&pro_th[i], NULL, multp_multc_produce, &arg);
    }
    
    for(int i = 0;i < THREAD_SIZE;i ++)
    {
        lpthread_create(&con_th[i], NULL, multp_multc_consume, &arg);
    }
    

    for(int i = 0;i < THREAD_SIZE;i ++)
    {
        lpthread_join(pro_th[i], NULL);
    }

    for(int i = 0;i < THREAD_SIZE;i ++)
    {
        lpthread_join(con_th[i], NULL);
    }

    lsem_destroy(&(arg.lock));
    lsem_destroy(&(arg.nempty));
    lsem_destroy(&(arg.nstored));
}
```

```c++
➜  build git:(master) ✗ ./main
produce 83 at 0
produce 0 at 1
produce 0 at 2
produce 0 at 3
consume 83 at 0
consume 86 at 1
consume 77 at 2
produce 1 at 4
produce 0 at 0
produce 0 at 1
produce 0 at 2
consume 15 at 3
consume 93 at 4
consume 35 at 0
consume 86 at 1
produce -2 at 3
consume 92 at 2
produce 98 at 4
consume 49 at 3
consume 21 at 4
```

##### 多缓冲读取

通过多缓冲将数据从一个文件写入到另一个文件。

```c++
typedef struct 
{
    struct 
    {
        char buff[MAX_LEN + 1];
        int len;
    }buff[BUFF_SIZE];
    sem_t lock;
    sem_t nempty;
    sem_t nstored;
    int items;
    int readfd;
    int writefd;
}wr_share;

//将buffer中的数据写入文件
void *write_buff(void *arg)
{
    wr_share* ptr = arg;
    int i = 0;
    while(1)
    {
        lsem_wait(&ptr->lock);
        //获取当前缓冲区的操作
        lsem_post(&ptr->lock);

        lsem_wait(&ptr->nstored);
        lwrite(ptr->writefd, ptr->buff[i].buff, ptr->buff[i].len);
        lwrite(STDOUT_FILENO, ptr->buff[i].buff, ptr->buff[i].len);
        i++;
        if(i >= ptr->items)
        {
            i = 0;
        }

        lsem_post(&ptr->nempty);
    }
}

//从文件中读取数据
void *read_buff(void *arg)
{
    wr_share* ptr = arg;
    int i = 0;
    while(1)
    {
        lsem_wait(&ptr->lock);
        //获取当前缓冲区的操作
        lsem_post(&ptr->lock);

        lsem_wait(&ptr->nempty);
        int n = lread(ptr->readfd, ptr->buff[i].buff, MAX_LEN);
        ptr->buff[i].len = n;
        i++;
        if(i >= ptr->items)
        {
            i = 0;
        }

        lsem_post(&ptr->nstored);
    }
}


void read_write_test()
{
    wr_share arg;
    arg.items = BUFF_SIZE;
    #if 0
    char *readfile = "build/CMakeCache.txt";
    char *writefile = "build/mktmp";
    #else
    char *readfile = "CMakeCache.txt";
    char *writefile = "mktmp";
    #endif
    arg.readfd = lopen(readfile, O_RDONLY);
    arg.writefd = lopen(writefile, O_WRONLY | O_CREAT);
    lsem_init(&arg.lock, 0, 1);
    lsem_init(&arg.nempty, 0, arg.items);
    lsem_init(&arg.nstored, 0, 0);

    pthread_t read_th, write_th;
    lpthread_create(&read_th, 0, read_buff, &arg);
    lpthread_create(&write_th, 0, write_buff, &arg);

    lpthread_join(read_th, NULL);
    lpthread_join(write_th, NULL);

    lsem_destroy(&arg.lock);
    lsem_destroy(&arg.nempty);
    lsem_destroy(&arg.nstored);
}
```

##### System V信号量

上面提到的Posix信号量有二值信号量和计数信号量，而System V信号量是信号量集，即一个或者多个信号量构成的集合，这个集合中的信号量都是计数信号量。
对于信号量集，内核维护的信息结构如下：

```c++
/* Data structure describing a set of semaphores.  */
struct semid_ds 
{
    struct ipc_perm sem_perm;  /* Ownership and permissions */
    time_t          sem_otime; /* Last semop time */
    time_t          sem_ctime; /* Last change time */
    unsigned long   sem_nsems; /* No. of semaphores in set */
};

//每个信号量包含的值
unsigned short  semval;   /* semaphore value */
unsigned short  semzcnt;  /* # waiting for zero */
unsigned short  semncnt;  /* # waiting for increase */
pid_t           sempid;   /* ID of process that did last op */
```

上面的第一个结构是书上提到的，下面的第二个结构是我主机上的结构，我猜测原理都差不多只是实现进行了修改，因此以书本上的描述为主。

- `sem_perm`：用户的操作权限；
- `sem_nsems`：当前数据结构中信号量的数量；
- `sem_otime`：上一次调用api`sem_op`的时间；
- `sem_ctime`：信号量创建的时间或上次调用`IPC_SET`的时间。

```c++
int semget(key_t key, int nsems, int semflg);
int semop(int semid, struct sembuf *sops, size_t nsops);
int semctl(int semid, int semnum, int cmd, ...);
```

- `semget`：创建一个或者访问一个已经存在的信号量集；
  - `key`：键值；
  - `nsems`：希望初始化的信号量数目，初始化之后不可修改，如果只是访问已经存在的信号量集则设为0；
  - `semflg`：可以为`SEM_R,SEM_A`分别表示读和修改，也可以为`IPC_CREAT, IPC_EXCL`；
  - 返回值为信号量的标识符的整数；
  - 当创建新的信号量时，用户权限，读写权限，创建时间，信号量的数目都会设置，但是结构中的数组，即各个信号量并不会初始化。这本身是不安全的，即便创建之后立即初始化，因为操作是非原子性的，无法保证绝对的安全；

- `semop`：操作信号量集，内核能够保证当前操作的原子性;
  - `semid`：通过`sem_get`获得的信号量标识符；
  - `sops`：是一个如下结构数据的数组,因为有些系统不会定义该结构，因此有些时候需要用户自己定义：

```c++
struct sembuf
{
unsigned short int sem_num;	/* semaphore number */
short int sem_op;		/* semaphore operation */
short int sem_flg;		/* operation flag */
};
```

- `sem_num`：需要操作的信号量在信号量集中的下标；
- `sem_flg`：进行操作的设置：
  - `0`;
  - `IPC_NOWAIT`：不阻塞；
  - `SEM_UNDO`;
- `sem_op`：具体的操作方式
  - `sem_op > 0`：
    - 未设置`SEM_UNDO`：将当前值加到`sem_val`上，即`sem_val += sem_op`，等同于V操作，只不过释放线程量可能大于1；
    - 设置`SEM_UNDO`：从相应的信号量的进程调整值（由内核维护）中减去`sem_op`；
  - `sem_op == 0`：调用者希望`sem_val==0`：
    - 如果`sem_val == 0`：立即返回；
    - 如果`sem_val != 0`：对应信号量的`semzcnt += 1`，阻塞至为0为止，除非设置了`IPC_NOWAIT`，则不阻塞直接返回错误；
  - `sem_op < 0`：调用者希望等待`sem_val >= |sem_op|`用于等待资源：
    - 如果`sem_val >= |sem_op|`，则`sem_val -= |sem_op|`，若设置了`SEM_UNDO`，则将`|sem_op|`加到对应信号量的进程调整值上；
    - 如果sem_val < |sem_op|，则相应信号量的semncnt += 1，线程被阻塞到满足条件为止。等到解阻塞时semval-=|sem_op，并且semncnt -= 1，如果制定了SEM_UNDO则sem_op的绝对值加到对应信号量的调整值上。如果指定了IPC_NOWAIT则线程不会阻塞。
  - 如果一个被捕获的信号唤醒了`sem_op`或者信号量被删除，则该函数会过早的返回一个错误。

- `semctl`：控制信号量集；
  - `semid`：信号量集的标识符；
  - `semnum`：要操作的信号量的标号；
  - `cmd`：命令；
    - `GETVAL`：`semval`作为返回值返回，-1表示失败；
    - `SETVAL`：`semval`设置为`arg.val`，成功的话相应信号量在进程中的信号量调整值会设置为0；
    - `GETPID`：返回`sempid`；
    - `GETNCNT`：返回`semncnt`；
    - `GETZCNT`：返回`semzcnt`；
    - `GETALL`：返回所有信号量的`semval`，存入`arg.array`；
    - `SETALL`：按照`arg.array`设置所有信号量的`semval`；
    - `IPC_RMID`：删除指定信号量集；
    - `IPC_SET`：根据`arg.buf`设置信号量集的`sem_perm.uid,sem_perm.gid,sem_perm.mode`；
    - `IPC_STAT`：返回当前信号量集的`semid_ds`结构，存入`arg.buf`，空间需要用户分配。
  - `arg`：可选，根据`cmd`指定。

```c++
union semun {
           int              val;    /* Value for SETVAL */
           struct semid_ds *buf;    /* Buffer for IPC_STAT, IPC_SET */
           unsigned short  *array;  /* Array for GETALL, SETALL */
           struct seminfo  *__buf;  /* Buffer for IPC_INFO
                                       (Linux-specific) */
       };
```

存在的一些限制：

- semmni：系统范围内最大信号量集数；
- semmsl：每个信号量集最大信号量数；
- semmns：系统范围内最大信号量数；
- semopm：每个semop调用最大操作数；
- semmnu：系统范围内最大复旧结构数；
- semume：每个复旧结构最大复旧项数；
- semvmx：任何信号量的最大值；
- semaem：最大退出时调整值。

**示例：**

```c++
int vsem_create(key_t key)
{
    int semid = lsemget(key, 1, 0666 | IPC_CREAT | IPC_EXCL);
    return semid;
}
 
int vsem_open(key_t key)
{
    int semid = lsemget(key, 0, 0);//创建一个信号量集 
    return semid;
}
 
int vsem_p(int semid)//
{
    struct sembuf sb = {0, -1, /*IPC_NOWAIT*/SEM_UNDO};//对信号量集中第一个信号进行操作，对信号量的计数值减1，
    lsemop(semid, &sb, 1);//用来进行P操作 
    return 0;
}
 
int vsem_v(int semid)
{
    struct sembuf sb = {0, 1, /*0*/SEM_UNDO};
    lsemop(semid, &sb, 1);
    return 0;
}
 
int vsem_d(int semid)
{
    int ret = semctl(semid, 0, IPC_RMID, 0);//删除一个信号量集
    return ret;
}
 
int vsem_setval(int semid, int val)
{
    union semun su;
    su.val = val;
    semctl(semid, 0, SETVAL, &su);//给信号量计数值
    printf("value updated...\n");
    return 0;
}
 
int vsem_getval(int semid)//获取信号量集中信号量的计数值
{
    int ret = semctl(semid, 0, GETVAL, 0);//返回值是信号量集中
    printf("current val is %d\n", ret);
    return ret;
}
 
int vsem_getmode(int semid)
{
    union semun su;
    struct semid_ds sem;
    su.buf = &sem;
    semctl(semid, 0, IPC_STAT, su);
    printf("current permissions is %o\n", su.buf->sem_perm.mode);
    return 0;
}
 
int vsem_setmode(int semid, char *mode)
{
    union semun su;
    struct semid_ds sem;
    su.buf = &sem;
 
    semctl(semid, 0, IPC_STAT, su);

    printf("current permissions is %o\n", su.buf->sem_perm.mode);
    sscanf(mode, "%o", (unsigned int *)&su.buf->sem_perm.mode);
    semctl(semid, 0, IPC_SET, &su);
    printf("permissions updated...\n");
 
    return 0;
}
 
void usage(void)
{
    fprintf(stderr, "usage:\n");
    fprintf(stderr, "semtool -c\n");
    fprintf(stderr, "semtool -d\n");
    fprintf(stderr, "semtool -p\n");
    fprintf(stderr, "semtool -v\n");
    fprintf(stderr, "semtool -s <val>\n");
    fprintf(stderr, "semtool -g\n");
    fprintf(stderr, "semtool -f\n");
    fprintf(stderr, "semtool -m <mode>\n");
}
 
void v_test(int argc, char *argv[])
{
    int opt;
 
    opt = getopt(argc, argv, "cdpvs:gfm:");//解析参数
    if (opt == '?')
        exit(EXIT_FAILURE);
    if (opt == -1)
    {
        usage();
        exit(EXIT_FAILURE);
    }
 
    key_t key = ftok(".", 's');
    int semid;
    switch (opt)
    {
    case 'c'://创建信号量集
        vsem_create(key);
        break;
    case 'p'://p操作
        semid = vsem_open(key);
        vsem_p(semid);
        vsem_getval(semid);
        break;
    case 'v'://v操作
        semid = vsem_open(key);
        vsem_v(semid);
        vsem_getval(semid);
        break;
    case 'd'://删除一个信号量集
        semid = vsem_open(key);
        vsem_d(semid);
        break;
    case 's'://对信号量集中信号量设置初始计数值
        semid = vsem_open(key);
        vsem_setval(semid, atoi(optarg));
        break;
    case 'g'://获取信号量集中信号量的计数值
        semid = vsem_open(key);
        vsem_getval(semid);
        break;
    case 'f'://查看信号量集中信号量的权限
        semid = vsem_open(key);
        vsem_getmode(semid);
        break;
    case 'm'://更改权限
        semid = vsem_open(key);
        vsem_setmode(semid, argv[2]);
        break;
    }
 
    return;
}
```



#### 共享内存

两个或多个进程共享一个给定的存储区。
**特点：**

1. 最快的一种 IPC，因为进程是直接对内存进行存取，数据不需要在进程之间复制
2. 多个进程可以同时操作，需要使用信号量进行同步
3. 多个进程可以将同一个文件映射到它们的共享地址空间从而实现共享内存

#### Socket

套接字通信，不仅仅局限于单机进程间通信，更广泛用于不同机器之间的进程间通信。
**特点：**

1. 支持不同机器之间的通信
2. Socket不是协议，而是调用接口，通过Socket才能使用TCP/IP协议进行通信，过程需要经过“三次握手、四次挥手”

