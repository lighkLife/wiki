# ZooKeeper 论文翻译

**ZooKeeper: Wait-free coordination for Internet-scale systems**

```
Patrick Hunt and Mahadev Konar
Yahoo! Grid
{phunt,mahadev}@yahoo-inc.com

Flavio P. Junqueira and Benjamin Reed
Yahoo! Research
{fpj,breed}@yahoo-inc.com
```

## 摘要

In this paper, we describe ZooKeeper, a service for co-
ordinating processes of distributed applications. Since
ZooKeeper is part of critical infrastructure, ZooKeeper
aims to provide a simple and high performance kernel
for building more complex coordination primitives at the
client. It incorporates elements from group messaging,
shared registers, and distributed lock services in a repli-
cated, centralized service. The interface exposed by Zoo-
Keeper has the wait-free aspects of shared registers with
an event-driven mechanism similar to cache invalidations
of distributed file systems to provide a simple, yet pow-
erful coordination service.

本文介绍了 ZooKeeper，这是一个用于协调分布式应用程序的服务。
由于 ZooKeeper 是基础设施的部分，因此它旨在为客户端构建更复杂的协调原语提供简单且高性能的内核。它将组消息、共享寄存器和分布式锁服务的元素合并到一个复制的、集中式的服务中。ZooKeeper所提供的接口具有共享寄存器的无等待特性，同时采用类似于分布式文件系统的缓存失效机制的事件驱动机制，提供了简单而强大的协调服务。

The ZooKeeper interface enables a high-performance
service implementation. In addition to the wait-free
property, ZooKeeper provides a per client guarantee of
FIFO execution of requests and linearizability for all re-
quests that change the ZooKeeper state. These design de-
cisions enable the implementation of a high performance
processing pipeline with read requests being satisfied by
local servers. We show for the target workloads, 2:1
to 100:1 read to write ratio, that ZooKeeper can handle
tens to hundreds of thousands of transactions per second.
This performance allows ZooKeeper to be used exten-
sively by client applications.