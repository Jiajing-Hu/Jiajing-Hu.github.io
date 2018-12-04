---
title: redis源码阅读-sds
date: 2018-12-04 23:56:53
tags: redis
catories: redis源码阅读
---

在redis中，作者自己实现了一个动态字符串，并且使其二进制安全。其具体的实现方式存放在sds文件中。

> 附：
> **二进制安全**
> 二进制安全是一种主要用于字符串操作函数相关的计算机编程术语。一个二进制安全功能（函数），其本质上将操作输入作为原始的、无任何特殊格式意义的数据流。其在操作上应包含一个字符所能有的256种可能的值（假设为8比特字符） 

### sds结构定义
从sds.h文件中我们可以看到这么一行代码

```c
typedef char *sds;
```

这里说明了sds的底层储存的方式其实就是char *，但是char *本身不是二进制安全的。例如字符串"123\0123"，如果直接使用char *存放的话，我们对其进行输出，strlen等操作的时候，遇到第一个\0符号便会终止。所以这里必须要使用一些其他的辅助结构来帮助我们实现二进制安全。

这里也就是sds头部(sdshdr)，其定义如下

```c++
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
}

#define SDS_TYPE_5  0
#define SDS_TYPE_8  1
#define SDS_TYPE_16 2
#define SDS_TYPE_32 3
#define SDS_TYPE_64 4
```
在sds中，由flag声明了类型。len代表了已经使用的长度，alloc代表了声明的长度。这样我们在使用sds的时候，可以直接通过len来判断字符串是否结束。避免了遇到'\0'符号强制结束导致的二进制不安全。

> 使用**__attribute__ ((__packed__))**可以防止编译器自动内存对其

### 一些辅助函数

```c
#define SDS_HDR_VAR(T,s) struct sdshdr##T *sh = (void*)((s)-(sizeof(struct sdshdr##T)));	//获取头指针
#define SDS_HDR(T,s) ((struct sdshdr##T *)((s)-(sizeof(struct sdshdr##T))))  //获取头指针
```

这两个宏定义函数均为获取sds的头部指针。从sds的指针地址减去sizeof(sdshdr##T)的地址，便可以得到sdshdr的头部地址

> 注：在C语言宏定义中##表示连接，所以sdshdr##T最后可以得到sdshdr8/16/32/64

另外的一些函数，例如sdslen，sdsavail等，都是十分简单的函数，简明易懂。这里不多赘言

### sds构造函数
sds构造函数通过不同的initlen来判断创建的动态字符串的类型，从而创建不同的sds，代码以及解析如下。

```c
sds sdsnewlen(const void *init, size_t initlen) {
    void *sh;       //
    sds s;
    //根据长度来判断类型，如果是sds5类型的话，那么会重新设定为sds8
    //根据不同的长度设定hdr，从而分配空间
    char type = sdsReqType(initlen);    		// 获取类型
    if (type == SDS_TYPE_5 && initlen == 0) type = SDS_TYPE_8;       // SDS_TYPE5不使用
    int hdrlen = sdsHdrSize(type);		// 判断了type之后得到头部大小
    unsigned char *fp; /* flags pointer. */
	
    //分配空间
    sh = s_malloc(hdrlen+initlen+1);		//分配空间,空间 = 字符串 + 头部空间 + 1（尾部终止符)
    if (init==SDS_NOINIT)		
        init = NULL;
    else if (!init)
        memset(sh, 0, hdrlen+initlen+1);
    if (sh == NULL) return NULL;
    //sh + hdrlen 即为数据部分的启始指针
    s = (char*)sh+hdrlen;
    //fp = flags指针
    fp = ((unsigned char*)s)-1;
    //根据不同的类型来设定hdr中的内容、
    switch(type) {
        case SDS_TYPE_5: {
            *fp = type | (initlen << SDS_TYPE_BITS);
            break;
        }
        case SDS_TYPE_8: {
            SDS_HDR_VAR(8,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_16: {
            SDS_HDR_VAR(16,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_32: {
            SDS_HDR_VAR(32,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_64: {
            SDS_HDR_VAR(64,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
    }
    if (initlen && init)
        memcpy(s, init, initlen);//拷贝数据内容并且加入终止符（兼容C）
    s[initlen] = '\0';
    return s;		//直接返回数据部分的指针，那么如果想要得到头部数据，需要数组通过辅助函数得到头部指针
}
```

### sds调整空间
sds既然是变长字符串，那么关键就是在于它可以自动调整长度，下面就是sds自动调整长度的函数

sds空间调整规则如下：

１．如果原字符串大小小于１Ｍ的话，那么将字符串扩展为原来的两倍

２．如果字符串大小大于１Ｍ的话，那么在原有基础上，扩展１Ｍ的空间

```c
//sds调整空间的函数
sds sdsMakeRoomFor(sds s, size_t addlen) {
    void *sh, *newsh;
    size_t avail = sdsavail(s);		//剩余空间
    size_t len, newlen;
    char type, oldtype = s[-1] & SDS_TYPE_MASK;		//获取type和旧的type
    int hdrlen;

    //在空间足够的情况下，那么就直接返回sds即可
    if (avail >= addlen) return s;

    len = sdslen(s);
    sh = (char*)s-sdsHdrSize(oldtype);
    newlen = (len+addlen);
    //在字符串长度小于1m(1024 * 1024)的情况下，将字符串空间扩展为原来的两倍，否则将字符串长度增加1m
    if (newlen < SDS_MAX_PREALLOC)
        newlen *= 2;
    else
        newlen += SDS_MAX_PREALLOC;

    //根据新长度来判断新的类型
    type = sdsReqType(newlen);

    //依旧不适用sds5
    if (type == SDS_TYPE_5) type = SDS_TYPE_8;
	
    //重新判断hdrlen
    hdrlen = sdsHdrSize(type);
    if (oldtype==type) {
        //如果类型没有变化的话，直接重新省内请内存既可以
        newsh = s_realloc(sh, hdrlen+newlen+1);
        if (newsh == NULL) return NULL;
        s = (char*)newsh+hdrlen;
    } else {
        //如果是hdr类型变化的话，后面的字符串将会被移动位置，那么就不能使用recalloc函数重新分配内存了
        newsh = s_malloc(hdrlen+newlen+1);
        if (newsh == NULL) return NULL;
        memcpy((char*)newsh+hdrlen, s, len+1);
        s_free(sh);
        s = (char*)newsh+hdrlen;
        s[-1] = type;
        sdssetlen(s, len);
    }
    sdssetalloc(s, newlen);
    return s;
}
```

### 回收多余空间的函数
一些情况下，我们可能给sds分配了过多的空间，但是这些空间我们可能一直用不上，这个时候就可以使用sdsRemoveFreeSpace函数来回收掉一部分的空间。

```c
sds sdsRemoveFreeSpace(sds s) {
    void *sh, *newsh;
    //刚开始跟扩展函数一样，先判断类型，从而判断以前的头长度
    char type, oldtype = s[-1] & SDS_TYPE_MASK;
    int hdrlen, oldhdrlen = sdsHdrSize(oldtype);
    size_t len = sdslen(s);
    sh = (char*)s-oldhdrlen;
//新的类型
    type = sdsReqType(len);
    hdrlen = sdsHdrSize(type);

    //如果类型依旧没有变化的话，那么直接将原来的内存重新分配一下就可以了，否则的话就要重新分配，然后重新设置属性了。
    //最后将alloc和len都设置为len长度，这样的话，多余的空间都可以被回收
    if (oldtype==type || type > SDS_TYPE_8) {
        newsh = s_realloc(sh, oldhdrlen+len+1);
        if (newsh == NULL) return NULL;
        s = (char*)newsh+oldhdrlen;
    } else {
        newsh = s_malloc(hdrlen+len+1);
        if (newsh == NULL) return NULL;
        //复制
        memcpy((char*)newsh+hdrlen, s, len+1);
        s_free(sh);
        //新的s，等于hdr + hdrlen
        s = (char*)newsh+hdrlen;
        s[-1] = type;//设置type
        sdssetlen(s, len);
    }
    sdssetalloc(s, len);
    return s;
}
```

### 一些其他的小函数

```c
//创建一个空的sds
//其实就是创建一个initlen为0的
sds sdsempty(void) {
    return sdsnewlen("",0);
}

//从一个const char创建
sds sdsnew(const char *init) {
    size_t initlen = (init == NULL) ? 0 : strlen(init);
    return sdsnewlen(init, initlen);
}

//可以理解为复制构造函数
sds sdsdup(const sds s) {
    return sdsnewlen(s, sdslen(s));
}
//释放空间
void sdsfree(sds s) {
    if (s == NULL) return;
    s_free((char*)s-sdsHdrSize(s[-1]));
}
//将sds当做char *处理
//将len设置为第一个'\0'的位置
void sdsupdatelen(sds s) {
    size_t reallen = strlen(s);
    sdssetlen(s, reallen);
}

//将sds清零
void sdsclear(sds s) {
    sdssetlen(s, 0);
    s[0] = '\0';
}
```
