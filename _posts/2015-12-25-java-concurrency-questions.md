---
layout:     post
title:      Java Concurrency 40问
date:       2015-12-25
categories: Programming
---

1. 多线程有什么用？
2. 创建线程的方式
3. start方法和run方法的区别
4. Runnable接口和Callable接口的区别
5. CyclicBarrier和CountDownLatch的区别
6. Volatile关键字的作用
7. 什么是线程安全
8. Java中如何获取到线程dump文件
9. 一个线程如果出现了运行时异常会怎么样
10. 如何在两个线程之间共享数据
11. sleep方法和wait方法有什么区别
12. 生产者消费者模型的作用是什么
13. ThreadLocal有什么用
14. 为什么wait方法和notify/notifyAll方法要在同步块中被调用
15. wait方法和notify/notifyAll方法在放弃对象监视器时有什么区别
16. 为什么要使用线程池
17. 怎么检测一个线程是否持有对象监视器
18. synchronized和ReentrantLock的区别
19. ConcurrentHashMap的并发度是什么
20. ReadWriteLock是什么
21. FutureTask是什么
22. Linux环境下如何查找哪个线程使用CPU最长
23. Java编程写一个会导致死锁的程序
24. 怎么唤醒一个阻塞的线程
25. 不可变对象对多线程有什么帮助
26. 什么是多线程的上下文切换
27. 如果你提交任务时，线程池队列已满，这时会发生什么
28. Java中用到的线程调度算法是什么
29. Thread.sleep(0)的作用是什么
30. 什么是自旋
31. 什么是happens-before，即先行发生原则
32. 什么是CAS
33. 什么是乐观锁和悲观锁
34. 什么是AQS
35. 单例模式的线程安全性
36. Semaphore有什么作用
37. Hashtable的size方法中明明只有一条语句”return count”，为什么还要做同步？
38. 线程类的构造方法、静态块是被哪个线程调用的
39. 同步方法和同步块，哪个是更好的选择
40. 高并发、任务执行时间短的业务怎样使用线程池？并发不高、任务执行时间长的业务怎样使用线程池？并发高、业务执行时间长的业务怎样使用线程池？
