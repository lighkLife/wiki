# ZooKeeper 基础

## 节点类型

- 临时节点
- 临时有序节点
- 持久节点（支持TTL）
- 持久有序节点（支持TTL）
- 容器节点（新版本，没有子节点时自动消失）  

## 件事与通知


## curator 锁实现

### 独占锁
![alt text](img/image-4.png)

每个锁监听上一个锁，避免羊群效应


### 读写锁

![alt text](img/image-5.png)
