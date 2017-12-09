---
title: Linux load的问题
date: 2017-12-09 13:17:54
tags:
 - linux
 - tech

---

### 背景

最近线上的系统(java应用)出现了一个很诡异的问题,就是cpu非常平稳的情况下load呈现出周期性的抖动.如果把JVM停掉,又恢复正常了.花了不少时间排查,排除了是定时任务,IO.也排除了是JVM的问题.最后怀疑是内核的问题,于是写了一个测试的程序来验证

### 测试程序

问题来了.需要写一个消耗指定cpu百分比的测试代码怎么写?

```java
public class CpuTest {

    public static double tmp = 0;

    public static void main(String[] args) throws Exception {
        // 20%
        int percent = 2;
        if (args.length != 0) {
            percent = Integer.valueOf(args[0]);
        }
        int sleepTime = 10 - percent;
        int runNano = 1000000 * percent;
        long useTime = 0;
        while (true) {
            useTime += runTaskUnit();
            if (useTime >= runNano) {
                Thread.sleep(sleepTime);
                useTime = 0;
            }
        }
    }

    public static long runTaskUnit() {
        long start = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            tmp = Math.sqrt(i) * Math.sqrt(i);
        }
        long end = System.nanoTime();
        return (end - start);
    }

}


```

运行一下试试:

```
javac CpuTest.java

//后面的参数表示,把cpu跑到40%
java CpuTest 4

```

如果你的机器是单核的cpu,上面的代码应该没啥问题,还是比较准的.但如果是多核的就有问题了,需要稍加改造适应多核.(代码引起的变化可以用top命令看看,前面那个代码只把其中一个cpu跑起来了,后面这个则是每个核都起来了)

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CpuTest {

    public static double          tmp             = 0;

    public static final int       CpuCoreCount    = Runtime.getRuntime().availableProcessors();

    public static ExecutorService executorService = Executors.newFixedThreadPool(CpuCoreCount);

    public static void main(String[] args) throws Exception {
        // 20%
        int percent = 2;
        if (args.length != 0) {
            percent = Integer.valueOf(args[0]);
        }
        final int finalPercent = percent;
        for (int i = 0; i < CpuCoreCount; i++) {
            executorService.execute(() -> {
                int sleepTime = 10 - finalPercent;
                int runNano = 1000000 * finalPercent;
                long useTime = 0;
                while (true) {
                    useTime += runTaskUnit();
                    if (useTime >= runNano) {
                        try {
                            Thread.sleep(sleepTime);
                        } catch (Exception e) {
                        }
                        useTime = 0;
                    }
                }
            });
        }

    }

    public static long runTaskUnit() {
        long start = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            tmp = Math.sqrt(i) * Math.sqrt(i);
        }
        long end = System.nanoTime();
        return (end - start);
    }

}

```
