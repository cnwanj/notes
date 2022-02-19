# 一、Zookeeper是什么？

- 是一个观察者模式设计的分布式框架，负责协调客户端与服务端。
- 存储和管理服务端和客户端的注册信息。
- 当服务端注册信息发生变化，通知客户端（上线、下线）。
- 相当于文件系统 + 通知机制。

zookeeper官网下载：https://archive.apache.org/dist/zookeeper/

![image-20220215213615843](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220215213615843.png)

# 二、Zookeeper的选举机制

- SID（服务器ID）：服务器的唯一标识。
- ZXID（事务ID）：用来标识一次服务器状态变更；所有的节点ZXID不一定一致，与Zookeeper的处理逻辑有关。
- Epoch：Leader任期的代号。

## 1.服务器启动初始化

> 每台服务器有自己的唯一标识SID，优先选举SID最大的为Leader。

- 5台服务器，服务器1启动，发起选举，投自己1票，当票数到达一半以上（3票），选举结束。
- 服务器2启动，发起选举，SID比服务器1大，投票给服务器2，服务器2有2票。
- 服务器3启动，发起选举，服务器3的SID最大，获得3票（大于一半），成为Leader。

## 2.非第一次启动

> Zookeeper集群有两种情况会进入选举：
>
> - 服务器启动初始化。
> - 运行期间与Leader失去连接。

- 服务器与Leader断开连接时，想尝试选举，会被告知存在Leader信息，此时服务器需要建立联系，并更新状态即可。
- Leader挂了，5台服务器，SID为1、2、3、4、5，ZXID为6、6、6、5、5，Epoch都为1，节点3位Leader，3和5都挂了，进行选举：
  - 选举比较的优先级：Epoch > ZXID > SID；
    - Epoch大，直接胜出。
    - Epoch相同，ZXID大胜出。
    - ZXID相同，SID大胜出。
  - 1、2、4的投票情况：
    - 服务器1：1 6 1
    - 服务器2：1 6 2
    - 服务器4：1 5 4
  - 第一轮Epoch相同。
  - 第二轮服务器4淘汰（ZXID最小淘汰）。
  - 第三轮服务器2成为Leader（SID最大胜出）。



**启动hadoop102、hadoop103、hadoop104脚本**

```shell
#!/bin/bash
# 判断没有参数
if [$# -lt 1]
then
	echo "No Args Input..."
	exit;
fi

case $1 in
"start")
	for i in hadoop102 hadoop103 hadoop104
	do
		echo "----------------- zookeeper $i 启动 -----------------"
		ssh $i "/opt/install/zookeeper-3.5.7/bin/zkServer.sh start"
	done
;;
"stop")
	for i in hadoop102 hadoop103 hadoop104
	do
		echo "----------------- zookeeper $i 停止 -----------------"
		ssh $i "/opt/install/zookeeper-3.5.7/bin/zkServer.sh stop"
	done
;;
"status")
	for i in hadoop102 hadoop103 hadoop104
	do
		echo "----------------- zookeeper $i 状态 -----------------"
		ssh $i "/opt/install/zookeeper-3.5.7/bin/zkServer.sh status"
	done
;;
*)
	echo "Input Args Error..."
;;
esac
```



```shell
# 查看启动报错日志
bin/zkServer.sh start-foreground
```



## 动态监听服务器上下线

在分布式系统中，有多个节点，可以动态上下线，客户端能够感知节点的上下线情况。

## 五、Zookeeper分布式锁

> 分布式锁和分布式事务的区别：
>
> - 锁是一个过程，事务则是最终的结果状态

### 1.实现Zookeeper分布式锁

DistributedLock.java

```java
package com.cnwanj.distributed;

import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.io.IOException;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;

/**
 * @author: cnwanj
 * @date: 2022-02-19 15:55:41
 * @version: 1.0
 * @desc: Zookeeper分布式锁实现
 */
public class DistributedLock {

	private final String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
	private final int sessionTimeout = 2000;
	private final ZooKeeper zk;

	// 等待连接完毕
	private CountDownLatch connectLatch = new CountDownLatch(1);
	// 等待监听上一个节点完毕
	private CountDownLatch waitLatch = new CountDownLatch(1);
	// 前一个节点路径
	private String waitPath;
	private String currentMode;

	public DistributedLock() throws IOException, InterruptedException, KeeperException {
		// 建立连接
		zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
			public void process(WatchedEvent watchedEvent) {
				// 释放连接等待（若已建立连接）
				if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
					connectLatch.countDown();
				}
				// 释放监听等待（存在释放锁 && 路径是前一个节点）
				if (watchedEvent.getType() == Event.EventType.NodeDeleted && watchedEvent.getPath().equals(waitPath)) {
					waitLatch.countDown();
				}
			}
		});
		// 线程等待，建立连接再执行后面
		connectLatch.await();

		// 判断根目录是否存在
		Stat stat = zk.exists("/locks", false);
		if (stat == null) {
			// 创建根节点（参数1：目录名称，参数2：目录下内容，参数3：对外开放，参数4：永久创建）
			zk.create("/locks", "locks".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		}
	}

	// 加锁
	public void zkLock() throws KeeperException, InterruptedException {
		// 创建临时带序号节点
		currentMode = zk.create("/locks/" + "seq-", null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
		// 校验节点是否第一个，是的话直接加锁，若不是，需要监听前一个是否解锁
		List<String> childrenList = zk.getChildren("/locks", false);
		// 如果只有一个节点，直接获取锁
		if (childrenList.size() == 1) {
			return;
		} else {
			// 从小到大排序
			Collections.sort(childrenList);
			// 当前节点名称
			String thisNode = currentMode.substring("/locks/".length());
			// 获取节点位置
			int index = childrenList.indexOf(thisNode);
			if (index == -1) {
				System.out.println("数据异常");
			} else if (index == 0) {
				// thisNode为最小，直接获取锁
				return;
			} else {
				// 获取前一个节点路径
				waitPath = "/locks/" + childrenList.get(index - 1);
				// 获取前一个节点，初始化监听
				zk.getData(waitPath, true, null);
				// 等待监听
				waitLatch.await();
				return;
			}
		}

	}

	// 解锁
	public void unZkLock() throws KeeperException, InterruptedException {
		// 删除节点
		zk.delete(currentMode, -1);
	}
}
```

DistributedLockTest.java

```java
package com.cnwanj.distributed;

import org.apache.zookeeper.KeeperException;

import java.io.IOException;

/**
 * @author: cnwanj
 * @date: 2022-02-19 21:50:39
 * @version: 1.0
 * @desc: 测试Zookeeper分布式锁
 */
public class DistributedLockTest {

	public static void main(String[] args) throws InterruptedException, IOException, KeeperException {

		// 创建分布式锁1
		final DistributedLock lock1 = new DistributedLock();
		// 创建分布式锁2
		final DistributedLock lock2 = new DistributedLock();

		// 创建线程1
		new Thread(() -> {
			// 获取锁对象
			try {
				lock1.zkLock();
				System.out.println("线程1获取锁");
				Thread.sleep(3 * 1000);
				lock1.unZkLock();
				System.out.println("线程1释放锁");
			} catch (KeeperException | InterruptedException e) {
				e.printStackTrace();
			}
		}).start();
		// 创建线程2
		new Thread(() -> {
			try {
				lock2.zkLock();
				System.out.println("线程2获取锁");
				Thread.sleep(3 * 1000);
				lock2.unZkLock();
				System.out.println("线程2释放锁");
			} catch (KeeperException | InterruptedException e) {
				e.printStackTrace();
			}
		}).start();
	}
}
```



### 2.Curator实现分布式锁

官网：https://curator.apache.org/

Apache Curator is a Java/JVM client library for [Apache ZooKeeper](https://zookeeper.apache.org/), a distributed coordination service. It includes a highlevel API framework and utilities to make using Apache ZooKeeper much easier and more reliable. It also includes recipes for common use cases and extensions such as service discovery and a Java 8 asynchronous DSL.

意思是：Curator是Zookeeper的一个Java/Jvm客户端库，也是一个分布式协调服务，Curator成为Zookeeper更简单可靠的一个高可用框架和工具，它也包含了一些常用的用例和扩展方法，如Java8和异步DSL。

