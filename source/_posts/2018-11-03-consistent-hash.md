---
title: 一致性哈希
date: 2018-11-03 19:27:57
tags:

 - tech
 - java
 
---

![](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/consistent_hashing.png)

### 简介

> [wiki介绍](https://en.wikipedia.org/wiki/Consistent_hashing)

基本的原理就是将一堆任务尽量均匀的分给一堆节点.而且在节点不变的情况下,相同的任务始终分配给相同的节点.我们在项目中也用到了这个技术,最近发现了一个多线程的bug.下面就来看看具体的代码吧.

### 版本V1

这个版本基本上是网上最常见的做法.hash使用的是KETAMA_HASH算法.这个网上很多,具体就不贴出来了.主要看看hash环的实现.这个代码最大的问题就是线程安全问题(在最后面我会贴上测试代码).如果绝对的安全就需要在dispatchToNode,addNode,removeNode这几个方法上都加上synchronized.当然更优雅的方式是使用ReentrantReadWriteLock.把dispatchToNode加上读锁,在add和remove中加上写锁.

```java

package name.chengchao.myConsistentHash;

import java.util.List;
import java.util.SortedMap;
import java.util.TreeMap;

/**
 * 这个版本的实现是多线程不安全的.主要是使用了treemap,而且修改没有加锁
 * 
 * @author charles
 *
 */
public class ConsistentHashV1 implements ConsistentHash {

    /**
     * 将一个node分裂成多少个虚拟node
     */
    public final static int VIRTUAL_NODE_NUM = 1024;

    /**
     * 哈希环
     */
    private TreeMap<Long, String> hashCircle = new TreeMap<Long, String>();

    public ConsistentHashV1(List<String> nodeList) {
        Assert.notEmpty(nodeList, "nodeList can not be empty!");
        for (String node : nodeList) {
            addNode(node);
        }
    }

    /**
     * 新增node
     */
    public void addNode(String node) {
        Assert.hasText(node, "node can not be blank!");
        for (int i = 0; i < VIRTUAL_NODE_NUM; i++) {
            String tmpNode = node + "#" + i;
            hashCircle.put(HashCodeUtils.hashString(tmpNode), node);
        }
    }

    /**
     * 删除node
     */
    public void removeNode(String node) {
        Assert.hasText(node, "node can not be blank!");
        for (int i = 0; i < VIRTUAL_NODE_NUM; i++) {
            String tmpNode = node + "#" + i;
            hashCircle.remove(HashCodeUtils.hashString(tmpNode));
        }
    }

    /**
     * 分发
     * 
     * @param target
     * @return
     */
    public String dispatchToNode(String target) {
        Assert.hasText(target, "target can not be blank!");
        Long targetHashcode = HashCodeUtils.hashString(target);
        SortedMap<Long, String> tailMap = hashCircle.tailMap(targetHashcode);
        Long key = tailMap.isEmpty() ? (hashCircle.isEmpty() ? null : hashCircle.firstKey()) : tailMap.firstKey();
        String result = hashCircle.get(key);
        return result;
    }

    public int getHashCircleCount() {
        return hashCircle.size();
    }

}


```

### 版本V2

这个代码中把treeMap替换成了ConcurrentSkipListMap,而且用ceilingKey替换了tailMap的操作.使用ReentrantReadWriteLock来控制并发

```java
package name.chengchao.myConsistentHash;

import java.util.List;
import java.util.concurrent.ConcurrentSkipListMap;
import java.util.concurrent.locks.ReentrantReadWriteLock;

// 调整选取的策略,去除了tailMap的操作,改为使用ceiling,并增加读写锁
public class ConsistentHashV4 implements ConsistentHash {

	/**
	 * 将一个node分裂成多少个虚拟node
	 */
	public final static int VIRTUAL_NODE_NUM = 1024;

	/**
	 * 哈希环
	 */
	private ConcurrentSkipListMap<Long, String> hashCircle = new ConcurrentSkipListMap<Long, String>();

	// 公平策略为true,这样不会导致读多写少的时候,写拿不到锁的问题
	private ReentrantReadWriteLock lock = new ReentrantReadWriteLock(true);

	public ConsistentHashV4(List<String> nodeList) {
		Assert.notEmpty(nodeList, "nodeList can not be empty!");
		for (String node : nodeList) {
			addNode(node);
		}
	}

	/**
	 * 新增node
	 */
	public void addNode(String node) {
		Assert.hasText(node, "node can not be blank!");
		lock.writeLock().lock();
		try {
			for (int i = 0; i < VIRTUAL_NODE_NUM; i++) {
				String tmpNode = node + "#" + i;
				hashCircle.put(HashCodeUtils.hashString(tmpNode), node);
			}
		} finally {
			lock.writeLock().unlock();
		}
	}

	/**
	 * 删除node
	 */
	public void removeNode(String node) {
		Assert.hasText(node, "node can not be blank!");
		lock.writeLock().lock();
		try {
			for (int i = 0; i < VIRTUAL_NODE_NUM; i++) {
				String tmpNode = node + "#" + i;
				hashCircle.remove(HashCodeUtils.hashString(tmpNode));
			}
		} finally {
			lock.writeLock().unlock();
		}
	}

	/**
	 * 分发
	 * 
	 * @param target
	 * @return
	 */
	public String dispatchToNode(String target) {
		Assert.hasText(target, "target can not be blank!");
		lock.readLock().lock();
		try {
			Long targetHashcode = HashCodeUtils.hashString(target);
			Long key = hashCircle.ceilingKey(targetHashcode);
			if (null == key) {
				key = hashCircle.firstKey();
			}
			return hashCircle.get(key);
		} finally {
			lock.readLock().unlock();
		}
	}

	public int getHashCircleCount() {
		return hashCircle.size();
	}

}

```


### 测试用例

在多线程操作remove后,treeMap就有问题了.

```java
package name.chengchao.myConsistentHash;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import org.apache.commons.lang3.RandomStringUtils;

public class ConsistentHashTest {

    public final static int number = 2;
    public final static int dispatchNumber = 5;

    public static ExecutorService executorService = Executors.newFixedThreadPool(number);
    public static ExecutorService dispatchExecutorService = Executors.newFixedThreadPool(dispatchNumber);

    public static ConsistentHash buildConsistentHash(List<String> nodeList) {
        // return new ConsistentHashV1(nodeList); // 多运行几次就能出现异常了.
        // return new ConsistentHashV2(nodeList);
        // return new ConsistentHashV3(nodeList);
        return new ConsistentHashV4(nodeList);
    }

    public static void main(String[] args) throws Exception {
        test_balance();
        // test_multithread();
    }

    /**
     * 均衡性测试
     */
    public static void test_balance() {
        int taskCount = 100000;
        List<String> nodeList = Stream.of("node1", "node2", "node3", "node4").collect(Collectors.toList());
        Map<String, AtomicInteger> countMap =
            nodeList.stream().collect(Collectors.toMap(x -> x, x -> new AtomicInteger(0)));
        ConsistentHash consistentHash = buildConsistentHash(nodeList);
        for (int i = 0; i < taskCount; i++) {
            String target = consistentHash.dispatchToNode(RandomStringUtils.random(10));
            countMap.get(target).incrementAndGet();
        }

        System.out.println(countMap);
        // {node4=25084, node2=25610, node3=23930, node1=25376}

    }

    /**
     * 测试多线程安全
     * 
     * @throws Exception
     * 
     */
    public static void test_multithread() throws Exception {
        List<String> nodeList = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            nodeList.add("node" + i);
        }
        ConsistentHash consistentHash = buildConsistentHash(nodeList);
        for (int i = 0; i < 50; i++) {

            // 并发执行
            final CyclicBarrier barrier = new CyclicBarrier(number);
            final int index = i;
            for (int j = 0; j < number; j++) {
                executorService.execute(new Runnable() {

                    @Override
                    public void run() {
                        try {
                            barrier.await();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                        consistentHash.removeNode("node" + index);
                        System.out.println("remove:" + "node" + index);
                    }
                });

            }

            System.out.println("sleep 5s");
            Thread.sleep(5000);
            for (int j = 0; j < dispatchNumber; j++) {
                dispatchExecutorService.execute(new Runnable() {

                    @Override
                    public void run() {
                        consistentHash.dispatchToNode(RandomStringUtils.random(10));
                        System.out.println("dispatch done");
                    }
                });
            }
            System.out.println("--------------------------------" + consistentHash.getHashCircleCount());
        }

    }

}

```
