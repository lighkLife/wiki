# 事务

在数据系统的残酷现实中，很多事情都可能出错：

- 数据库软件、硬件可能在任意时刻发生故障（包括写操作进行到一半时）。
- 应用程序可能在任意时刻崩溃（包括一系列操作的中间）。
- 网络中断可能会意外切断数据库与应用的连接，或数据库之间的连接。
- 多个客户端可能会同时写入数据库，覆盖彼此的更改。
- 客户端可能读取到无意义的数据，因为数据只更新了一部分。
- 客户端之间的竞争条件可能导致令人惊讶的错误。

为了实现可靠性，系统必须处理这些故障，确保它们不会导致整个系统的灾难性故障。但是实现容错机制工作量巨大。需要仔细考虑所有可能出错的事情，并进行大量的测试，以确保解决方案真正管用。数十年来，事务（transaction） 一直是简化这些问题的首选机制。

## 概念

### 事务

**事务**是应用程序将多个读写操作组合成一个逻辑单元的一种方式。整个事务要么成功 提交（commit），要么失败 中止（abort）或 回滚（rollback）。如果失败，应用程序可以安全地重试（多次重试没有副作用）。

事务不是天然存在的；它们是为了 **简化应用编程模型** 而创建的。通过使用事务，应用程序可以自由地忽略某些潜在的错误情况和并发问题，因为数据库会替应用处理好这些。（我们称之为 **安全保证**，即 safety guarantees）。

**ACID** 就是来描述事务所提供的**安全保证**，ACID 代表:

- **原子性（Atomicity）**
- **一致性（Consistency）**
- **隔离性（Isolation）**
- **持久性（Durability）**

```{note}

但实际上，不同数据库的 ACID 实现并不相同。例如，我们将会看到，关于 隔离性 的含义就有许多含糊不清【8】。高层次上的想法很美好，但魔鬼隐藏在细节里。今天，当一个系统声称自己 “符合 ACID” 时，实际上能期待的是什么保证并不清楚。不幸的是，ACID 现在几乎已经变成了一个营销术语。

所以，我们必须深入了解原子性，一致性，隔离性和持久性的定义的细节，才能真正辨别数据库到底提供了哪些安全保证。
```

**BASE**[^1]

不符合 ACID 标准的系统有时被称为 BASE[^2]，它代表 基本可用性（Basically Available），软状态（Soft State） 和 最终一致性（Eventual consistency），这比 ACID 的定义更加模糊，似乎 BASE 的唯一合理的定义是 “不是 ACID”，即它几乎可以代表任何你想要的东西。

### ACID 的含义

#### 原子性 Atomicity

**原子性是指能够在错误时中止事务，丢弃该事务进行的所有写入变更的能力。** 或许 **可中止性（abortability）** 是更好的术语。

事务本质上是一系列读写操作的集合，原子性描述的是：从外界看来，这一系列操作表现就如一个操作，数据要么完全写入成功，要么数据完全没有写入，不存在部分写入成功，部分写入失败，也就是 “宁为玉碎，不为瓦全”。

#### 一致性 Consistency

**ACID 一致性的概念是，对数据的一组特定约束必须始终成立，即 **invariants**（在数据库中始终保持不变的约束） 。**

一致性是站在应用的角度，对数据库数据的描述，指数据库在应用程序的特定概念中处于 “良好状态”，而有些 invariants （例如唯一约束、外键约束）可以由数据库控制，但更多的状态不变性需要应用去控制（比如转账问题，子表记录数和主表的统计数）。

```{note}
原子性、隔离性和持久性是数据库的属性，而一致性（在 ACID 意义上）是应用程序的属性。应用可能依赖数据库的原子性和隔离性来实现一致性，但这并不仅取决于数据库。因此，字母 C 不属于 ACID，甚至有人指出，那时候大家都不怎么在乎一致性，“ACID 中的 C” 是被 “扔进去凑缩写单词的”。
```

#### 隔离性 Isolation

**ACID 意义上的隔离性意味着，同时执行的事务是相互隔离的：它们不能相互冒犯。** 

传统的数据库教科书将隔离性形式化为 可串行化（Serializability），这意味着每个事务可以假装它是唯一在整个数据库上运行的事务。数据库确保当多个事务被提交时，结果与它们串行运行（一个接一个）是一样的，尽管实际上它们可能是并发运行的。然而实践中很少会使用可串行的隔离，因为它有性能损失。

为了提升性能，支持事务的主流的都定义一些**弱隔离级别**，比如 mysql 的：

- 读未提交（Read Uncommitted），可能出现：脏读、不可重复读和幻读的问题
- 读已提交（Read Committed），可能出现：不可重复读和幻读问题
- 可重复读（Repeatable Read），可能出现：幻读问题，通过快照隔离，可以避免一部分幻读问题
- 序列化（Serializable），可以避免脏读、不可重复读和幻读问题，但是会对性能产生较大的影响

#### 持久性 Durability

**持久性 是一个承诺，即一旦事务成功完成，即使发生硬件故障或数据库崩溃，写入的任何数据也不会丢失。**

完美的持久性是不存在的（比如发生地球大爆炸怎么办？），这里的持久性一般是指将数据从内存持久化到硬盘上。

### 单对象和多对象操作

当单个对象发生改变时，原子性和隔离性也是适用的。例如，假设你正在向数据库写入一个 20 KB 的 JSON 文档：

- 如果在发送第一个 10 KB 之后网络连接中断，数据库是否存储了不可解析的 10KB JSON 片段？
- 如果在数据库正在覆盖磁盘上的前一个值的过程中电源发生故障，是否最终将新旧值拼接在一起？
- 如果另一个客户端在写入过程中读取该文档，是否会看到部分更新的值？

这些问题非常让人头大，故存储引擎一个几乎普遍的目标是：对单节点上的单个对象（例如键值对）上提供**原子性**和**隔离性**。

原子性可以通过使用日志来实现崩溃恢复，并且可以使用每个对象上的锁来实现隔离（每次只允许一个线程访问对象），一些数据库也提供更复杂的原子操作，例如自增操作、比较和设置（CAS, compare-and-set）操作，但它们不是通常意义上的事务。

CAS 以及其他单一对象操作被称为 **“轻量级事务”**，甚至出于营销目的被称为 “ACID”，但是这个术语是误导性的。事务通常被理解为，将**多个对象**上的**多个操作**合并为**一个执行单元**的机制。

```{note}
尽管重试一个中止的事务是一个简单而有效的错误处理机制，但它并不完美：

- 如果事务实际上成功了，但是在服务器试图向客户端确认提交成功时网络发生故障（所以客户端认为提交失败了），那么重试事务会导致事务被执行两次 —— 除非你有一个额外的应用级去重机制。
- 如果错误是由于负载过大造成的，则重试事务将使问题变得更糟，而不是更好。为了避免这种正反馈循环，可以限制重试次数，使用指数退避算法，并单独处理与过载相关的错误（如果允许）。
- 仅在临时性错误（例如，由于死锁，异常情况，临时性网络中断和故障切换）后才值得重试。在发生永久性错误（例如，违反约束）之后重试是毫无意义的。
- 如果事务在数据库之外也有副作用，即使事务被中止，也可能发生这些副作用。例如，如果你正在发送电子邮件，那你肯定不希望每次重试事务时都重新发送电子邮件。如果你想确保几个不同的系统一起提交或放弃，两阶段提交（2PC, two-phase commit） 可以提供帮助（“原子提交与两阶段提交” 中将讨论这个问题）。
- 如果客户端进程在重试中失效，任何试图写入数据库的数据都将丢失。
```

[^1]: [亚马逊. 云计算概念中心/数据库. ACID 和 BASE 数据库有什么区别？ ](https://aws.amazon.com/cn/compare/the-difference-between-acid-and-base-database/)
[^2]: Armando Fox, Steven D. Gribble, Yatin Chawathe, et al.: [“Cluster-Based Scalable Network Services,”](http://www.cs.berkeley.edu/~brewer/cs262b/TACC.pdf) at 16th ACM Symposium on Operating Systems Principles (SOSP), October 1997.
