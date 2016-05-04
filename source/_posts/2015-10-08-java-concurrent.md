---
layout: post
title: java并发入门
date: 2015-10-08 21:41
tags:
  - tech
  - java

---

###为什么会有并发问题

- 并发程序的关键在于对共享的，可变的状态进行访问管理
- 不可变对象永远是线程安全的
- 因此，即使多个线程之间没有使用同步，也不保证一定会发生内存可见性问题（指线程读取到过期的值），java内存模型仅仅是允许内存可见性问题发生而已。在很多当前的JVM实现和java执行平台中，甚至是在那些使用多处理器的JVM和平台中，也很少出现内存可见性问题。
- 这使得测试基于内存可见性的错误是不切实际的，因为这样的错误极难发生。
- 如果把一个字段声明为volatile型，线程对这个字段写入后，在执行后续的内存访问之前，线程必须刷新这个字段且让这个字段对其他线程可见（即该字段立即刷新）。每次对volatile字段的读访问，都要重新装载字段的值。



###为什么volatile要和compareAndSet配合起来使用?

```java
    public final long getAndIncrement() {
        while (true) {
            long current = get();
            long next = current + 1;
            if (compareAndSet(current, next))
                return current;
        }
    }
```

- 如果long current = get()中get()的值不是volatile的话,有可能会导致死循环
- 因为只有是volatile的变量才满足下面两点,这两点对于cas操作非常重要
- 在读之前需要强制更新成主内存中的数据
- 在写之后需要强制更新到主内存中的数据
- 所以在getAndIncrement操作中cas保证原子性,volatile保证内存可见性,两者共同完成无锁的原子操作


###使用AtomicXXX的两个重要原因

- 性能好很多
- 使用方便
可以参见: http://stackoverflow.com/questions/11670687/when-is-atomicinteger-preferrable-over-synchronized
