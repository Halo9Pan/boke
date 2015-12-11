---
layout: post
title:  "Advanced Memcache!"
date:   2015-12-11 15:00:00
categories: Development
---

[memcached 源码阅读笔记](https://github.com/daoluan/decode-memcached)
[Memcached二三事儿](http://huoding.com/2012/12/30/205)

[Memcached的线程模型及状态机](http://basiccoder.com/thread-model-and-state-machine-of-memcached.html)
Memcached使用libevent实现事件循环，libevent在Linux环境下默认采用epoll作为IO多路复用方法。
Memcached采用了很典型的Master-Worker模型，采用的是多线程而不是多进程，而线程之间没有冗余的共享数据，这样便降低了多线程进行线程同步的开销，核心的共享数据是消息队列，主线程会把收到的事件请求放入队列，随后调度程序会选择一个空闲的Worker线程来从队列中取出事件请求进行处理。
在main()函数里面，Memcached为主线程调用event_init()创建了一个libevent base对象，随后调用thread_init()来初始化线程。

[Memcached内存管理机制浅析](http://basiccoder.com/memcached-memory-mamagement.html)
从main()函数入口进去之后便是几个模块的初始化函数，和内存管理相关的主要有两个函数，一个是assoc_init()，这个是用来初始化哈希表的，另一个是slabs_init()，该函数用来初始化slab。  
Memcached把内存分为一个个的slab，每个slab又分成一个个的chunk，系统会定义一个slab_class数组，其中每个元素是都是一个对该slab的描述，包括这个slab里面的每个chunk的大小，这个slab里面包含多少个chunk等信息。  
如果开启了prealloc功能的话，那么很有可能在有空闲内存的情况下分配内存失败，另外提前为slabs分配内存也有可能会造成内存的浪费，有可能所有的item都不会使用某个slab class，这样这个slab class里面分配的内存就浪费掉了，DONT_PREALLOC_SLABS 在1.4.7里面是默认定义的，也就是说prealloc功能是默认关闭的。  
创建新的slab，slabs_preallocate()函数只不过是对每一个已初始化的slab_class调用do_slabs_newslab()函数为其分配一块slab内存空间。  
在slab上分配内存，在某个slab_class上分配size大小的内存的函数是do_slabs_alloc()，这个函数有两个参数，要分配的内存字节数size，和该内存应该存在于哪个slab class上的slab class id. 这两个参数在是有相关性的，在调用该函数的时候class id一般是通过size来计算得出来的。
存入系统的每个key-value对都会被转换成一个item，这个item中保存了相关的状态标志信息，当服务器收到一个set请求时便需要在内存中创建一个item，item的内存理所当然是在上面讨论过的slab分配器上分配的。item的存储使用了LRU的方法，把item链入一个链表中。

[Jenkins hash function](https://en.wikipedia.org/wiki/Jenkins_hash_function)
[MurmurHash])(https://en.wikipedia.org/wiki/MurmurHash)