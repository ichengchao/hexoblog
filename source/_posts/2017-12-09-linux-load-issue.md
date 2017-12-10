---
title: Linux load的问题
date: 2017-12-09 13:17:54
tags:
 - linux
 - tech

---

### 背景

>如果对linux load不熟悉的话,可以先看一下<a href="https://en.wikipedia.org/wiki/Load_(computing)">这里</a>

![](http://www.chengchao.name/resource-container/image/linux_cpu_load.jpg)

最近线上的系统(java应用)出现了一个很诡异的问题,就是cpu非常平稳的情况下load呈现出周期性的波动(如上图).如果把JVM停掉,又恢复正常了.花了不少时间排查,排除了是定时任务,IO.也排除了是JVM笨笨的问题.最后怀疑是内核的问题,于是准备写个测试程序验证一下.

### 测试程序

问题来了.怎么写一个消耗指定cpu百分比的测试?消耗cpu就是指的就是消耗cpu的时间.所以基本的原理就是按百分比吃掉cpu的时间.比如要消耗30%的cpu,假设单位时间是100ms,那就是跑满30ms,然后sleep 70ms.代码如下:

```java
/**
 * 指定cpu的使用率(单核版本)<br>
 * 原理:指定一个固定的时间单元,比如说是100ms,如果要跑30%的cpu,就需要跑30ms,休息70ms
 * 
 * @author charles
 */
public class CpuUtil_Single {

    public static final int TIME_UNIT_MS = 100;

    public static double    tmp;

    public static void main(String[] args) throws Exception {
        // default is 30%
        int cpuUtil = 30;
        if (args.length != 0) {
            cpuUtil = Integer.valueOf(args[0]);
        }
        if (cpuUtil < 5 || cpuUtil > 80) {
            throw new IllegalArgumentException("cpuUtil must be between 5 and 80");
        }
        int runTime = TIME_UNIT_MS * cpuUtil / 100;
        int sleepTime = TIME_UNIT_MS - runTime;
        System.out.println("runTime(ms):" + runTime + ",sleepTime(ms):" + sleepTime);
        while (true) {
            eatCpuTime(runTime);
            Thread.sleep(sleepTime);
        }
    }

    public static void eatCpuTime(long useTimeMs) {
        long start = System.nanoTime();
        long useTimeNano = useTimeMs * 1000000;
        while (true) {
            // eat cpu,这里的10000不宜过大,如果设置过大会导致控制的精度不够
            for (int i = 0; i < 10000; i++) {
                tmp = Math.sqrt(i) * Math.sqrt(i);
            }

            if ((System.nanoTime() - start) >= useTimeNano) {
                break;
            }

        }
    }

}


```

运行一下试试:

```
javac CpuUtil_Single.java

//后面的参数表示,把cpu跑到40%
java CpuUtil_Single 40

```

如果你的机器是单核的cpu,上面的代码没啥问题,还是比较准的.但如果是多核的就有问题了,需要改造一下.代码引起的变化可以用top命令看看,前面那个代码只能把其中一个核跑起来,后面这个则是每个核都跑起来了.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CpuUtil_Multi {

    public static final int       TIME_UNIT_MS = 100;

    public static double          tmp;

    public static final int       CpuCoreNum   = Runtime.getRuntime().availableProcessors();

    public static ExecutorService MyExecutor   = Executors.newFixedThreadPool(CpuCoreNum);

    public static void main(String[] args) throws Exception {
        // default is 30%
        int cpuUtil = 30;
        if (args.length != 0) {
            cpuUtil = Integer.valueOf(args[0]);
        }
        if (cpuUtil < 5 || cpuUtil > 80) {
            throw new IllegalArgumentException("cpuUtil must be between 5 and 80");
        }

        int runTime = TIME_UNIT_MS * cpuUtil / 100;
        int sleepTime = TIME_UNIT_MS - runTime;
        System.out.println("runTime(ms):" + runTime + ",sleepTime(ms):" + sleepTime + ",CpuCoreNum:" + CpuCoreNum);

        for (int i = 0; i < CpuCoreNum; i++) {
            MyExecutor.execute(() -> {
                while (true) {
                    try {
                        eatCpuTime(runTime);
                        Thread.sleep(sleepTime);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }

    }

    public static void eatCpuTime(long useTimeMs) {
        long start = System.nanoTime();
        long useTimeNano = useTimeMs * 1000000;
        while (true) {
            // eat cpu
            for (int i = 0; i < 10000; i++) {
                tmp = Math.sqrt(i) * Math.sqrt(i);
            }

            if ((System.nanoTime() - start) >= useTimeNano) {
                break;
            }

        }
    }

}

```


### 重现问题

![](http://www.chengchao.name/resource-container/image/linux_load_collect.png)

linux 的load其实是大概每隔5s看一次运行队列(在linux中是R+D)长度.系统的负载是很规律的波动的话,这个计算方式就有问题了.为了重现这个问题,需要对上面的代码再做一点改造.

```java
package name.chengchao.mylinuxtest.cpu;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class CpuUtil_Multi_Interval {

    public static final int                TIME_UNIT_MS             = 100;

    // 波动周期
    public static final int                CPU_WAVE_INTERVAL_SECOND = 5;

    public static double                   tmp;

    public static final int                CpuCoreNum               = Runtime.getRuntime().availableProcessors();

    public static ScheduledExecutorService MyScheduledExecutor      = Executors.newScheduledThreadPool(1);

    public static ExecutorService          MyExecutor               = Executors.newFixedThreadPool(CpuCoreNum);

    public static void main(String[] args) throws Exception {
        // default is 30%
        int cpuUtil = 30;
        if (args.length != 0) {
            cpuUtil = Integer.valueOf(args[0]);
        }
        if (cpuUtil < 5 || cpuUtil > 80) {
            throw new IllegalArgumentException("cpuUtil must be between 5 and 80");
        }

        int runTime = TIME_UNIT_MS * cpuUtil / 100;
        int sleepTime = TIME_UNIT_MS - runTime;
        System.out.println("runTime(ms):" + runTime + ",sleepTime(ms):" + sleepTime + ",CpuCoreNum:" + CpuCoreNum);

        int loopNum = 1000 / TIME_UNIT_MS;

        MyScheduledExecutor.scheduleAtFixedRate(() -> {
            for (int i = 0; i < CpuCoreNum; i++) {
                MyExecutor.execute(() -> {
                    for (int j = 0; j < loopNum; j++) {
                        try {
                            eatCpuTime(runTime);
                            Thread.sleep(sleepTime);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                });
            }
        }, 0, CPU_WAVE_INTERVAL_SECOND, TimeUnit.SECONDS);

    }

    public static void eatCpuTime(long useTimeMs) {
        long start = System.nanoTime();
        long useTimeNano = useTimeMs * 1000000;
        while (true) {
            // eat cpu,这里的10000不宜过大,如果设置过大会导致控制的精度不够
            for (int i = 0; i < 10000; i++) {
                tmp = Math.sqrt(i) * Math.sqrt(i);
            }

            if ((System.nanoTime() - start) >= useTimeNano) {
                break;
            }

        }
    }

}

```

![](http://www.chengchao.name/resource-container/image/linux_cpu_wave.jpg)

把这个代码跑一个一天,大概就能看到load是上面这个样子!原因就在于系统的load采样周期和cpu波动的周期会慢慢的偏移,重合.偏移,重合.最终导致了这个结果.当然可以把定时任务的周期改成3s试试,还是会有一样的结果只是波动的周期变了而已.
