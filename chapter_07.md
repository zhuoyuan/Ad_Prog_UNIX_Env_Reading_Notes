
### 进程环境

#### **进程终止**
```
有8种方式使进程终止，其中有5种为正常终止
1. 从main返回
2. 调用exit
3. 调用_exit或_Exit
4. 最后一个线程从其启动例程返回
5. 从最后一个线程调用pthread_exit

异常终止有三种方式：
1. 调用abort
2. 接到一个信号
3. 最后一个线程对取消请求作出相应
```

##### **退出函数**
```
#include <stdlib.h>
void exit(int status);    /*先执行一些清理处理，然后返回内核*/
void _Exit(int status);   /*直接返回内核*/
#include <unistd.h>
void _exit(int status);   /*直接返回内核*/
```
*exit函数在返回内核前会对所有打开的流调用fclose，造成输出缓冲中数据冲洗(写文件)*

当出现下列情形之一时，进程的终止状态未定义：
```
1. 调用退出函数不带参数
2. main函数执行了一个无返回值的return语句
3. main函数没有声明返回类型为整型
```
##### **atexit函数**
ISO C规定，一个进程可以登记至多32个函数，这些函数由exit自动调用，这些函数通常被称为*终止处理程序*，通过调用atexit来登记这些函数
```
int atexit(void (*func)(void)); /* exit 执行登记函数的顺序与它们被登记的顺序相反*/
```
*exit返回内核前先执行退出函数，然后调用登记函数*

#### **环境表**
每个进程都接收到一张环境表，与参数表一样，环境表也是一个字符串指针数组，其中每个指针包含一个以NULL结束的c语言字符串地址，全局变量environ则包含了该指针数组的地址，通常使用getenv和putenv函数来访问特定的环境变量，但是如果要查看整个环境，则必须使用environ指针。
```
extern char **environ;
```
#### **C程序的存储空间布局**
```
从低地址到高地址：
1. 正文段(.text)：这是CPU执行的机器指令部分，通常正文段是可共享的
2. 初始化数据段(.data) : 由exec程序从文件中读出
3. 未初始化数据段(.bss) : 由exec初始化为0
4. 堆
5. 栈
```
#### **共享库**
共享库使得可执行文件不再需要包含共用的库函数，只需要在所有进程都可引用的存储区中保存这种库例程的一个副本，程序第一次执行或者第一次调用某个库函数时，用动态链接方法将程序与共享库函数相链接

```
优点
1. 减少可执行文件长度
2. 更新共享库无需重新编译使用该共享库的程序
缺点：
1. 增加了一些时间开销：这些开销发生在该程序第一次被执行时或者每个共享库函数第一次被调用时
```
#### **存储空间分配**
```
malloc:  void *malloc(size_t size);
calloc:  void *calloc(size_t nobj, size_t size);
realloc: void *realloc(void *ptr, size_t newsize);
```
#### **函数setjmp和longjmp**
实现函数间跳转

在有些嵌套层次较深的函数调用过程中发生错误时，很难直接跳转到一个较高层次的函数，上面两个函数实现了这个功能，这两个函数可以跳过若干调用帧，返回到当前函数调用路径上的某一个函数中。

第一次调用setjmp返回0，当调用longjmp时，返回到setjmp处，此时setjmp的返回值非0

由于调用longjmp后跳过了若干调用帧，那么当longjmp返回时，自动变量和寄存器的状态能够恢复吗？答案是”看情况”，令人沮丧的回答，大多数实现并不回滚这些自动变量和寄存器变量的值。如果你有一个自动变量，又不想让其回滚，可以声明它为volatile，声明为全局变量和静态变量的值在longjmp时保持不变

一般情况：CPU寄存器中的变量会回滚，而存放在存储器中的变量不回滚
```
    int             autoval;
    register int    regival;
    volatile int    volaval;
    static   int    statval;
    autoval = 1; regival = 2; volaval = 3; statval = 4;
    printf("");
    if(setjmp(jmpbuffer) != 0) {
        /* TODO:*/
    }
    autoval = 11; regival = 21; volaval = 31; statval = 41;
    f(autoval, regival, ..., statval);
    ...

    f(int i,...) {
        printf("");
        longjmp(jmpbuffer, 1);
    }
编译优化输出：
    1，2，3，4
    1，2，31，41
非编译优化输出：
    1，2，3，4
    1，2，3，4
    
```




























