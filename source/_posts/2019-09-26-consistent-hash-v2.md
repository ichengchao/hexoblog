---
title: 一致性哈希V2
date: 2019-09-26 15:34:08
tags:

 - tech
 - java
 
---

### 背景

之前写了一个[一致性hash的实现](http://blog.chengchao.name/2018/11/03/consistent-hash/),在线上跑得挺稳定的.不过在极端压力的情况下还是有bug.这次趁着双11把这个问题给fix了,性能提升也非常明显(可以跑一下后面的测试代码).主要归功于google guava中的实现.


### 代码

```java
package name.chengchao.myConsistentHash;

import java.util.HashSet;
import java.util.List;
import java.util.Set;

import com.google.common.collect.ImmutableList;
import com.google.common.hash.Hashing;

/**
 * 去除读锁,改用google guava实现
 * 
 * @author charles
 * @date 2019-09-26
 */
public class ConsistentHashV5 implements ConsistentHash {

	private Set<String> nodeSet = new HashSet<String>();
	ImmutableList<String> immutableNodeList;

	public ConsistentHashV5(List<String> nodeList) {
		Assert.notEmpty(nodeList, "nodeList can not be empty!");
		nodeSet.addAll(nodeList);
		immutableNodeList = ImmutableList.copyOf(nodeSet);
	}

	/**
	 * 核心方法,将target路由到node
	 */
	@Override
	public String dispatchToNode(String target) {
		Assert.hasText(target, "target can not be blank!");
		Long targetHashcode = HashCodeUtils.hashString(target);
		ImmutableList<String> tmpList = immutableNodeList;
		int index = Hashing.consistentHash(targetHashcode, tmpList.size());
		return tmpList.get(index);
	}

	/**
	 * 新增node
	 */
	@Override
	synchronized public void addNode(String node) {
		Assert.hasText(node, "node can not be blank!");
		if (nodeSet.contains(node)) {
			return;
		}
		nodeSet.add(node);
		immutableNodeList = ImmutableList.copyOf(nodeSet);
	}

	/**
	 * 删除node
	 */
	@Override
	synchronized public void removeNode(String node) {
		Assert.hasText(node, "node can not be blank!");
		if (!nodeSet.contains(node)) {
			return;
		}
		nodeSet.remove(node);
		immutableNodeList = ImmutableList.copyOf(nodeSet);
	}

	@Override
	public int getHashCircleCount() {
		// TODO Auto-generated method stub
		return 0;
	}

}


```


### 测试代码


```java
package name.chengchao.myConsistentHash;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import org.apache.commons.lang3.RandomStringUtils;

public class ConsistentHashTestV5 {

	public final static int number = 2;
	public final static int dispatchNumber = 5;

	public static ExecutorService executorService = Executors.newFixedThreadPool(number);
	public static ExecutorService dispatchExecutorService = Executors.newFixedThreadPool(dispatchNumber);

	public static ConsistentHash buildDispatchHashV5(int nodeCount) {
		List<String> nodeList = new ArrayList<String>();
		for (int i = 0; i < nodeCount; i++) {
			nodeList.add("node-" + i);
		}
		ConsistentHash dispatchHash = new ConsistentHashV5(nodeList);
		return dispatchHash;
	}

	public static ConsistentHash buildDispatchHashV4(int nodeCount) {
		List<String> nodeList = new ArrayList<String>();
		for (int i = 0; i < nodeCount; i++) {
			nodeList.add("node-" + i);
		}
		ConsistentHash dispatchHash = new ConsistentHashV4(nodeList);
		return dispatchHash;
	}

	public static void main(String[] args) throws Exception {
		test_perfermance();
//		test_multithread();
//		test_balance();
//		test_consistent();

	}

	public static void test_consistent() {
		ConsistentHash dispatchHash = buildDispatchHashV5(100);
		for (int i = 0; i < 3; i++) {
			System.out.println(dispatchHash.dispatchToNode("127.0.0.1"));
		}
		System.out.println("==========remove node 10");
		dispatchHash.removeNode("node-10");
		System.out.println(dispatchHash.dispatchToNode("127.0.0.1"));
		System.out.println("==========remove node 99");
		dispatchHash.removeNode("node-99");
		System.out.println(dispatchHash.dispatchToNode("127.0.0.1"));
		System.out.println("==========add node 99");
		dispatchHash.addNode("node-101");
		System.out.println(dispatchHash.dispatchToNode("127.0.0.1"));
		System.out.println("==========add node 200");
		dispatchHash.addNode("node-200");
		System.out.println(dispatchHash.dispatchToNode("127.0.0.1"));
	}

	public static void test_balance() {
		int count = 1;
		int nodeCount = 5;
		long start = System.currentTimeMillis();
		for (int i = 0; i < count; i++) {
			System.out.println("*************start************");
			ConsistentHash dispatchHash = buildDispatchHashV5(nodeCount);
			testDispatch(dispatchHash, 10000);
			System.out.println("==remove node==");
			dispatchHash.removeNode("node-0");
			testDispatch(dispatchHash, 10000);

			System.out.println("==add node==");
			dispatchHash.addNode("node-add-0");
			testDispatch(dispatchHash, 10000);
			System.out.println("**************end*************");
		}
		long end = System.currentTimeMillis();
		System.out.println("耗时(ms):" + (end - start));

	}

	// 性能测试
	public static void test_perfermance() {
		int count = 1000;
		int nodeCount = 20;

		long start = System.currentTimeMillis();
		for (int i = 0; i < count; i++) {
			ConsistentHash dispatchHash = buildDispatchHashV4(nodeCount);
			dispatchHash.dispatchToNode("127.0.0.1");
		}
		long end = System.currentTimeMillis();
		System.out.println("V4 耗时(ms):" + (end - start));

		start = System.currentTimeMillis();
		for (int i = 0; i < count; i++) {
			ConsistentHash dispatchHash = buildDispatchHashV5(nodeCount);
			dispatchHash.dispatchToNode("127.0.0.1");
		}
		end = System.currentTimeMillis();
		System.out.println("V5 耗时(ms):" + (end - start));
	}

	public static void testDispatch(ConsistentHash dispatchHash, int taskNum) {

		// ConsistentHash dispatchHash = new LoopHash(nodeList);

		Map<String, Long> map = new HashMap<String, Long>();
		for (int i = 0; i < taskNum; i++) {
			String target = RandomStringUtils.randomAlphanumeric(20);
			String node = dispatchHash.dispatchToNode(target);
			Long count = map.get(node);
			if (count == null) {
				count = 0L;
			}
			count++;
			map.put(node, count);
		}
		for (Map.Entry<String, Long> entry : map.entrySet()) {
			System.out.println(entry.getKey() + ":" + entry.getValue());
		}

	}

	public static void test_multithread() throws Exception {
		List<String> nodeList = new ArrayList<String>();
		for (int i = 0; i < 100; i++) {
			nodeList.add("node" + i);
		}
		final ConsistentHash consistentHash = buildDispatchHashV5(100);
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

			System.out.println("sleep 1s");
			Thread.sleep(1000);
			for (int j = 0; j < dispatchNumber; j++) {
				dispatchExecutorService.execute(new Runnable() {

					@Override
					public void run() {
						String nodeStr = consistentHash.dispatchToNode(RandomStringUtils.random(10));
						System.out.println("dispatch done,node:" + nodeStr);
					}
				});
			}
		}

	}

}

```
