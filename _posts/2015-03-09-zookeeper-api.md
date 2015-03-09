---
date: 2015/3/9 19:47:19 
layout: post
title: zookeeperAPI
categories: hadoop
tags:  zookeeper
---
##zookeeper API示例

http://zookeeper.apache.org/doc/r3.4.6/api/index.html

###zookeeper API示例

zookeeper代码示例：

	import java.io.IOException;
	import java.util.List;
	
	import org.apache.zookeeper.CreateMode;
	import org.apache.zookeeper.KeeperException;
	import org.apache.zookeeper.WatchedEvent;
	import org.apache.zookeeper.Watcher;
	import org.apache.zookeeper.ZooKeeper;
	import org.apache.zookeeper.ZooDefs.Ids;
	import org.apache.zookeeper.ZooKeeper.States;
	
	/**
	 * Zookeeper 工具类
	 * 
	 */
	public class ZookeeperClient {

	/*
	 * comma separated host:port pairs, each corresponding to a zk <p> server.
	 * e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002"
	 */
	public static final String ZOOKEEPER_CONNECTION_STRING = "hadoop-zengjr.xiaoqee.com:2181,"//
			+ "hadoop-slave01.xiaoqee.com:2181,"//
			+ "hadoop-slave02.xiaoqee.com:2181";

	public static final int ZOOKEEPER_CONNECTION_TIMEOUT = 2000;

	public static final int ZOOKEEPER_VERSION = -1;

	/**
	 * 获取Zookeeper链接
	 * 
	 * @return
	 * @throws IOException
	 * @throws InterruptedException
	 */
	public static ZooKeeper getZookeeper() {
		ZooKeeper zooKeeper = null;
		try {
			// 创建一个与服务器的连接
			zooKeeper = new ZooKeeper( //
					ZOOKEEPER_CONNECTION_STRING, ZOOKEEPER_CONNECTION_TIMEOUT, //
					new Watcher() { // 监控所有被触发的事件
						@Override
						public void process(WatchedEvent event) {
							// 打印事件的名称
							System.out.println(event.toString());
						}
					}//
			);

			// 等待连接
			while (zooKeeper.getState() != States.CONNECTED) {
				Thread.sleep(20);
			}

		} catch (Exception e) {
			e.printStackTrace();
		}
		return zooKeeper;
	}

	/**
	 * 关闭连接
	 * 
	 * @param zooKeeper
	 */
	public static void closeZookeeper(ZooKeeper zooKeeper) {
		if (null != zooKeeper) {
			try {
				zooKeeper.close();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * 判断目录节点是否存在
	 * 
	 * @param nodePath
	 * 		节点的路径,如/psb
	 * @return
	 * 		false表示节点不存在,true表示存在
	 */
	public static boolean isExists(String nodePath) {
		// 获取Zookeeper链接
		ZooKeeper zooKeeper = getZookeeper();

		try {
			// 判断节点是否存在
			return (null == zooKeeper.exists(nodePath, true)) ? false : true;
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (null != zooKeeper) {
				closeZookeeper(zooKeeper);
			}
		}
		return false;
	}

	/**
	 * 创建节点
	 * 
	 * @param nodePath
	 *            节点的路径,如/psb,目录一级一级的创建,不能一下创建多级节点目录
	 * @param nodeData
	 *            节点的数据
	 * @param createMode
	 *            节点的类型,有四种类型:PERSISTENT,PERSISTENT_SEQUENTIAL,EPHEMERAL,
	 *            EPHEMERAL_SEQUENTIAL
	 */
	public static void createNode(String nodePath, String nodeData,
			CreateMode createMode) {
		ZooKeeper zooKeeper = getZookeeper();

		try {
			// 判断节点是否存在
			boolean exists = isExists(nodePath);

			// 创建节点目录
			if (!exists) {
				zooKeeper.create(//
						nodePath,//
						nodeData.getBytes(),//
						Ids.OPEN_ACL_UNSAFE, //
						createMode);
			} else {
				System.out.println("[" + nodePath + "] 已经存在!");
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (null != zooKeeper) {
				closeZookeeper(zooKeeper);
			}
		}
	}

	/**
	 * 获取目录节点的数据
	 * 
	 * @param zooKeeper
	 * @param nodePath
	 * @throws KeeperException
	 * @throws InterruptedException
	 */
	public static String getNodeData(String nodePath) {
		ZooKeeper zooKeeper = getZookeeper();
		try {

			byte[] bytes = zooKeeper.getData(nodePath, false, null);
			String data = new String(bytes);
			return data;
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (null != zooKeeper) {
				closeZookeeper(zooKeeper);
			}
		}
		return "";
	}

	/**
	 *  设置目录节点的数据
	 * 
	 * @param zooKeeper
	 * @param nodePath
	 * @throws KeeperException
	 * @throws InterruptedException
	 */
	public static void setNodeData(String nodePath,String nodeData) {
		ZooKeeper zooKeeper = getZookeeper();
		try {
			// 修改节点数据
			zooKeeper.setData(nodePath,//
					nodeData.getBytes(),//
					ZOOKEEPER_VERSION);
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (null != zooKeeper) {
				closeZookeeper(zooKeeper);
			}
		}
	}
	
	/**
	 * 获取某个节点下面的所有子节点（属于儿子级别的）
	 * 
	 * @param nodePath
	 * @return
	 */
	public static List<String> getChildrenNodes(String nodePath) {
		ZooKeeper zooKeeper = getZookeeper();
		try {
			// 获取目录节点的所有子节点
			return zooKeeper.getChildren(nodePath, true);
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (null != zooKeeper) {
				closeZookeeper(zooKeeper);
			}
		}
		return null;
	}

	/**
	 * 刪除节点
	 * 
	 * @param nodePath
	 */
	public static void deleteNode(String nodePath) {
		ZooKeeper zooKeeper = getZookeeper();
		try {
			// 判断节点是否存在
			boolean exists = isExists(nodePath);
			
			if(exists){
				// 删除节点
				zooKeeper.delete(nodePath,ZOOKEEPER_VERSION);
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (null != zooKeeper) {
				closeZookeeper(zooKeeper);
			}
		}
	}

	// 测试
	public static void main(String[] args) {
		// createNode("/psb", "服务总线", CreateMode.PERSISTENT);
		// createNode("/test/hello", "测", CreateMode.PERSISTENT);

		// System.out.println(getNodeData("/psb"));;

		//	System.out.println(getChildrenNodes("/"));;
		//	deleteNode("/psb");
	}
	}
