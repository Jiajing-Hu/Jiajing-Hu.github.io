---
title: redis源码阅读 - zmalloc
date: 2018-11-26 23:50:04
tags: redis
categories: redis源码阅读
---

在redis中，zmalloc模块负责redis内存的管理。他的底层本质实际上是malloc等库函数，但是作者对其进行了一些内容的增加，在其基础上增加了内存统计，错误处理等操作。

由于redis的跨平台性，很多函数都存在条件编译以适应不同的平台。这里仅仅讨论linux平台

### 一些宏函数

```c
//这里的两个宏的作用得到值为s的字符串
#define __xstr(s) __str(s)
#define __str(s) #s
```

举一个简单的例子
```c
//
//这个c语言程序的输出应该为123456 number。其中输出number将会直接输出123456，也就是number的值。但是对于str,对其进行赋值 str = __str(number) = #number = "number",所以就直接输出了number。
# include <stdio.h>

#define __str(x)  #x

int main()
{
	int number = 123456;
	char *str = __str(number);
	printf("%d\n",number);
	printf("%s\n",str);
}
``
```
两个重要的内存统计宏函数

 update_zmalloc_stat_alloc这个宏函数作用是统计已经分配的内存，并且加入了字节对其的功能，将其自动对其到long的长度
 例如，在32位系统下long的长度为8
 那么对于任意一个n, n += sizeof(long) - (n&(sizeof(long)-1)必定为8的倍数
 对于任意一个n,n & 8结果必定为n转换为二进制时最低三位的值
 所以sizeof(long) - (n & (sizeof(long) -1) 必定为缺少的对其字节数
 那么n += sizeof(long) - (n&(sizeof(Long)-1))便可以自动将其对其到long
 但是在记录的时候，redis依然会记录未对齐的内存大小（也就是分配的内存大小)
 另外，used_memory是全局中记录已使用内存的全局变量
 atomicIncr(used_memory,__n)则是对全局变量进行原子增加操作

 至于为什么redis要将内存对其到long，是因为在计算机对long的倍数的数据进行操作只需要一个总线周期，效率更高
```c
#define update_zmalloc_stat_alloc(__n) do { \

size_t _n = (__n); \
if (_n&(sizeof(long)-1)) _n += sizeof(long)-(_n&(sizeof(long)-1)); \
atomicIncr(used_memory,__n); \
// 源代码的一个小bug,应为 atomicIncr(used_memory,_n);\
} while(0)

// 下面这个宏函数同上，只是它是一个释放内存的操作。
// 最后也是对used_memory进行一次原则减少操作
#define update_zmalloc_stat_free(__n) do { \
    size_t _n = (__n); \
    if (_n&(sizeof(long)-1)) _n += sizeof(long)-(_n&(sizeof(long)-1)); \
    atomicDecr(used_memory,__n); \
	//源代码的一个小bug，应为 atomicDecr(used_memory,_n);\
} while(0)

```

**ps:上述代码来自redis-unstable，这两个宏函数貌似存在一些bug。因为最后不管是加还是减，都是对__n进行操作而不是_n，抱着这个问题我查询了一下redis的ISSUES，貌似作者也说这是一个小小的bug。
所以最后的的修正的代码我也写在上面了**
附:issue地址:https://github.com/antirez/redis/issues/4739

### 一些辅助函数
除了宏函数之外，redis也定义了一些内存辅助函数，用以辅助内存分配中的一些操作。
在redis中，利用zmalloc_default_oom函数在内存分配失败的时候抛出异常(因为本身在c语言中malloc函数无法抛出异常，只是返回NULL)
```c
static void zmalloc_default_oom(size_t size) {
    fprintf(stderr, "zmalloc: Out of memory trying to allocate %zu bytes\n",
        size);
    fflush(stderr);
    abort();
}
```
字符串复制函数,虽然不知道为什么在这里
```c
char *zstrdup(const char *s) {
    size_t l = strlen(s)+1;
    char *p = zmalloc(l);
//算出长度直接使用memcpy函数拷贝
    memcpy(p,s,l);
    return p;
}
```

### zmalloc
在redis中，内存的分配主要使用的是zmalloc函数，而zmalloc函数的本质其实就是malloc函数。只是作者对其进行了一些优化，加入了内存统计和异常处理
其中PREFIX_SIZE作用在于记录分配的内存块大小
因为在一些环境中无法直接使用库函数得到内存大小，所以需要在分配的内存之前分配一小块内存，用以记录其连接的内存的大小
这个思想在后续的sds中也有使用。
```c
void *zmalloc(size_t size) {
	// 分配的内存大小为size + PREFIX_SIZE
	// 对于PREFIX_SIZE,它由HAVE_MALLOC_SIZE宏控制
	// 在GUN/LINUX环境下为0
    void *ptr = malloc(size+PREFIX_SIZE);		
	
	// 如果内存分配失败（malloc函数内存分配失败的时候将会返回
	// NULL),那么将会转到异常处理函数。
    if (!ptr) zmalloc_oom_handler(size);

	// 内存分配成功之后将会更新内存统计
#ifdef HAVE_MALLOC_SIZE
    update_zmalloc_stat_alloc(zmalloc_size(ptr));
    return ptr;
#else
    *((size_t*)ptr) = size;
    update_zmalloc_stat_alloc(size+PREFIX_SIZE);
    return (char*)ptr+PREFIX_SIZE;
#endif
}
```

### zcalloc
zcalloc的原型为calloc
calloc与malloc的区别是分配的内存是否初始化，同理，zcalloc与zmalloc的区别也在此。
```c
void *zcalloc(size_t size) {
	//利用calloc分配内存
    void *ptr = calloc(1, size+PREFIX_SIZE);
	//如果分配失败，那么将会调用异常处理
    if (!ptr) zmalloc_oom_handler(size);
	//分配之后更新内存统计
#ifdef HAVE_MALLOC_SIZE
    update_zmalloc_stat_alloc(zmalloc_size(ptr));
    return ptr;
#else
    *((size_t*)ptr) = size;
    update_zmalloc_stat_alloc(size+PREFIX_SIZE);
    return (char*)ptr+PREFIX_SIZE;
#endif
}
```

### zrealloc
zrelloc的底层实现为realloc,即为对已分配好的内存进行重新分配
```c
void *zrealloc(void *ptr, size_t size) {
#ifndef HAVE_MALLOC_SIZE
    void *realptr;
#endif
    size_t oldsize;
    void *newptr;
//如果ptr为空，那么zrealloc的本质实际上是zmalloc，因此转到zmalloc
    if (ptr == NULL) return zmalloc(size);
#ifdef HAVE_MALLOC_SIZE

// 首先得到旧的大小oldsize
// 然后重新按照size分配一个新的指针
    oldsize = zmalloc_size(ptr);
    newptr = realloc(ptr,size);
    if (!newptr) zmalloc_oom_handler(size);

//更新内存统计（减去旧的大小，得到新的大小）
    update_zmalloc_stat_free(oldsize);
    update_zmalloc_stat_alloc(zmalloc_size(newptr));
    return newptr;
#else
// 如果没有定义HAVE_MALLOC_SIZE
// 此时的指针存在前导的大小，先算出真正内存的起始点后进行操作
    realptr = (char*)ptr-PREFIX_SIZE;
    oldsize = *((size_t*)realptr);
    newptr = realloc(realptr,size+PREFIX_SIZE);
    if (!newptr) zmalloc_oom_handler(size);

    *((size_t*)newptr) = size;
    update_zmalloc_stat_free(oldsize+PREFIX_SIZE);
    update_zmalloc_stat_alloc(size+PREFIX_SIZE);
    return (char*)newptr+PREFIX_SIZE;
#endif
}
```

### zfree
zfree的底层依旧是加了内存统计以及异常处理的free函数
```c
void zfree(void *ptr) {
//先得到其大小，并且在内存统计中将其减去
//然后free指针。这段代码比价简单
#ifndef HAVE_MALLOC_SIZE
    void *realptr;
    size_t oldsize;
#endif

    if (ptr == NULL) return;
#ifdef HAVE_MALLOC_SIZE
    update_zmalloc_stat_free(zmalloc_size(ptr));
    free(ptr);
#else
    realptr = (char*)ptr-PREFIX_SIZE;
    oldsize = *((size_t*)realptr);
    update_zmalloc_stat_free(oldsize+PREFIX_SIZE);
    free(realptr);
#endif
}
```

主要的函数大概就是这么多。
zmalloc.c下还有一些其他的函数，和一些在malloc中就有的一些操作，就不多说明了。不然篇幅将会太长了。

