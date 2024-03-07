# 论文翻译-AQS

```{note}
原文： http://gee.cs.oswego.edu/dl/papers/aqs.pdf
```

## ABSTRACT 摘要
Most synchronizers (locks, barriers, etc.) in the J2SE1.5
java.util.concurrent package are constructed using a small
framework based on class `AbstractQueuedSynchro-nizer`.
This framework provides common mechanics for
atomically managing synchronization state, blocking and
unblocking threads, and queuing. The paper describes the
rationale, design, implementation, usage, and performance of this
framework.

在 J2SE1.5 的 java.util.concurrent 包中，大多数同步器（locks、barriers 等）
都是使用一个基于`AbstractQueuedSynchro-nizer` 类的小型框架构建的。
这个框架为同步状态的原子化管理、线程的阻塞和解除、队列操作提供了通用机制。
这篇论文阐述了这个框架的原理、设计、实现、使用和性能。

## Categories and Subject Descriptors
D.1.3 [Programming Techniques]: Concurrent Programming – Parallel Programming

## General Terms
Algorithms, Measurement, Performance, Design.

## Keywords
Synchronization, Java

## 1. INTRODUCTION 介绍
Java tm release J2SE-1.5 introduces package java.util.concurrent, a
collection of medium-level concurrency support classes created
via Java Community Process (JCP) Java Specification Request
(JSR) 166. Among these components are a set of synchronizers –
abstract data type (ADT) classes that maintain an internal
synchronization state (for example, representing whether a lock
is locked or unlocked), operations to update and inspect that
state, and at least one method that will cause a calling thread to
block if the state requires it, resuming when some other thread
changes the synchronization state to permit it. Examples include
various forms of mutual exclusion locks, read-write locks,
semaphores, barriers, futures, event indicators, and handoff
queues.

Java 1.5 版本引入 java.util.concurrent 包，这里面包含了一些中级的并发支持类，在 JCP
JSR 166 中创建。）。在这些组件中是一组同步器——抽象数据结构类（ADT），维护内部的同步状态
（例如，代表一把锁是锁定状态还是解锁状态），更新和检查状态的操作，以及当状态需要同步器时，
特定的方法会导致调用线程阻塞，直到另一个线程更改同步状态，以允许其继续运行。示例来自各形式的
互斥锁，读写锁、信号量、屏障、futures、事件指示器、操作队列。

As is well-known (see e.g., [^2]) nearly any synchronizer can be
used to implement nearly any other. For example, it is possible to
build semaphores from reentrant locks, and vice versa. However,
doing so often entails enough complexity, overhead, and
inflexibility to be at best a second-rate engineering option.
Further, it is conceptually unattractive. If none of these constructs
are intrinsically more primitive than the others, developers
should not be compelled to arbitrarily choose one of them as a
basis for building others. Instead, JSR166 establishes a small
framework centered on class AbstractQueuedSynchro-
nizer, that provides common mechanics that are used by most
of the provided synchronizers in the package, as well as other
classes that users may define themselves.

众所周知(见例子, [^2]) ，大多数同步器之间可以互相实现。例如，可以使用重入锁实现信号量，反之亦然。然而，
这样做做往往相当复杂，开销过大，不灵活，只能作为次等的工程选择。此外，这在概念上也不吸引人。
如果每一本质上更基本的同步器，那么开发人员就不应该被强迫任意选择一种同步器作为构建其他同步器的基础。
所以，JSR166 建立了一个以 `AbstractQueuedSynchro-nizer` 类为中心的小型框架，改类为并发包中的
大多数同步器提供了一个通用的机制，也可被用户自定义类使用。

The remainder of this paper discusses the requirements for this
framework, the main ideas behind its design and implementation,
sample usages, and some measurements showing its performance
characteristics.

本文的其余部分讨论了这个框架的要求、其设计和实现背后的主要思想、使用示例，以及展示一些性能指标的测量结果。

## 2.REQUIREMENTS 要求
### 2.1 Functionality 功能
Synchronizers possess two kinds of methods [^7]: at least one
acquire operation that blocks the calling thread unless/until the
synchronization state allows it to proceed, and at least one
release operation that changes synchronization state in a way that
may allow one or more blocked threads to unblock.

同步器要具备两种方法 [^7]：至少一种获取操作，用来阻塞调用线程，除非/直到
同步状态允许线程继续处理；以及至少一种释放操作，用来改变同步状态，从而允许一个或
多个阻塞的线程解除阻塞。

The `java.util.concurrent` package does not define a single unified
API for synchronizers. Some are defined via common interfaces
(e.g., Lock), but others contain only specialized versions. So,
acquire and release operations take a range of names and forms
across different classes. For example, methods `Lock.lock`,
`Semaphore.acquire`, `CountDownLatch.await`, and
`FutureTask.get` all map to acquire operations in the
framework. However, the package does maintain consistent
conventions across classes to support a range of common usage
options. When meaningful, each synchronizer supports:

- Nonblocking synchronization attempts (for example,
`tryLock``) as well as blocking versions.
- Optional timeouts, so applications can give up waiting.
- Cancellability via interruption, usually separated into one
version of acquire that is cancellable, and one that isn't.

`java.util.concurrent` 并没有为同步器定义一个统一的API。一些是通过通用接口
定义的（例如 Lock ），但其他只包含特定的版本。因此，获取和释放操作，在不同的类中
采用了不同的名称和形式。例如这些方法 `Lock.lock`，`Semaphore.acquire`，
`CountDownLatch.await` 和 `FutureTask.get` 都映射为框架中的获取操作。
然而，这个包在类之间保持了一致的约定，来支持一系列的使用选项。如果有意义，每个同步器都支持：
- 非阻塞的同步尝试（例如 `tryLock`）, 以及阻塞的版本
- 可选的超时机制，以便应用程序可以放弃等待。
- 通过中断实现取消功能，获取操作通常分为一个可取消的版本，以及一个不能取消的版本。

Synchronizers may vary according to whether they manage only
exclusive states – those in which only one thread at a time may
continue past a possible blocking point – versus possible shared
states in which multiple threads can at least sometimes proceed.
Regular lock classes of course maintain only exclusive state, but
counting semaphores, for example, may be acquired by as many
threads as the count permits. To be widely useful, the framework
must support both modes of operation.

同步器可能根据他们管理的状态是独占状态还是共享状态而有所不同。独占状态是指
一次只有一个线程可以继续通过可能的阻塞点，而共享状态则允许多个线程至少有时可以继续处理。
当然，常规的锁类只维护独占状态，但是，例如信号量，只要许可证数量足够，就可以被读多个线程获取。
为了广泛应用，框架必须支持这两种操作模式。

The java.util.concurrent package also defines interface
`Condition`, supporting monitor-style await/signal operations
that may be associated with exclusive `Lock` classes, and whose
implementations are intrinsically intertwined with their
associated `Lock` classes.

`java.util.concurrent` 包还定义了`Condition`接口，支持与独占锁`Lock`类关联的监视器
风格的 `await/signal` 操作，这些实现与它们关联的`Lock`类密切相关。

### 2.2 Performance Goals 性能目标
Java built-in locks (accessed using synchronized methods
and blocks) have long been a performance concern, and there is a
sizable literature on their construction (e.g., [^1], [^3]). However,
the main focus of such work has been on minimizing space
overhead (because any Java object can serve as a lock) and on
minimizing time overhead when used in mostly-single-threaded
contexts on uniprocessors. Neither of these are especially
important concerns for synchronizers: Programmers construct
synchronizers only when needed, so there is no need to compact
space that would otherwise be wasted, and synchronizers are
used almost exclusively in multithreaded designs (increasingly
often on multiprocessors) under which at least occasional
contention is to be expected. So the usual JVM strategy of
optimizing locks primarily for the zero-contention case, leaving
other cases to less predictable "slow paths" [^12] is not the right
tactic for typical multithreaded server applications that rely
heavily on java.util.concurrent.

Java 内置锁（使用`synchronized`方法和块）长期以来有性能问题，有大量关于它构造的
文献（例如 [^1], [^3]）。然而，这类工作的主要焦点在于，在单核心、单线程的上下文种，
最小化空间开销（因为任何Java 对象都可以作为一个锁）以及时间开销。这两点对于同步器
来说都不是特别地重要：程序员只有在需要的时候才会构造同步器，因此没有必要紧凑的使用
空间来避免浪费，并且同步器几乎专门为多线程设计（多处理器越来越常见），在这种情况下
偶尔就会发生竞争。通常 JVM 的锁优化策略主要是为零竞争情况优化的，将其他情况留给不太可
预测的“慢路径” [^12]，这对于依赖 java.util.concurrent 的典型的多线程服务器应用
来说并不是一个正确的策略。

Instead, the primary performance goal here is scalability: to
predictably maintain efficiency even, or especially, when
synchronizers are contended. Ideally, the overhead required to
pass a synchronization point should be constant no matter how
many threads are trying to do so. Among the main goals is to
minimize the total amount of time during which some thread is
permitted to pass a synchronization point but has not done so.
However, this must be balanced against resource considerations,
including total CPU time requirements, memory traffic, and
thread scheduling overhead. For example, spinlocks usually
provide shorter acquisition times than blocking locks, but usually
waste cycles and generate memory contention, so are not often
applicable.

相反，这里的主要性能目标是可伸缩性：即在同步器被争用时，能够可预测的保持效率。
理想情况下，无论多少线程正在尝试通过同步点，其所需要的开销应该是恒定的。主要目标之一
是尽量减少某个线程被允许通过同步点，但尚未这样做的总时间量（译注：竞态下获取资源的时间）。
然而，这些操作必须对资源利用均衡考量，包括需要 CPU 的总时间，内存流量和线程调度开销。
例如，自旋锁通常提高比阻塞锁更短的获取时间，但通常会浪费 CPU 周期和内存竞争，因此
并不普适。

These goals carry across two general styles of use. Most
applications should maximize aggregate throughput, tolerating, at
best, probabilistic guarantees about lack of starvation. However
in applications such as resource control, it is far more important
to maintain fairness of access across threads, tolerating poor
aggregate throughput. No framework can decide between these
conflicting goals on behalf of users; instead different fairness
policies must be accommodated.

这些目标贯穿两种基本的使用方式。大多数应用应该最大化总通吐量，最多只能容忍不发生饥饿
的概率上保证。然而，在资源控制等应用程序中，更重要的是在线程之间维持公平的资源访问，
可以容忍牺牲总通吐量。没有一个框架可以代表户在这些相互冲突的目标中作出决定；相反，
必须提供不同的公平策略来适应不同场景。

No matter how well-crafted they are internally, synchronizers
will create performance bottlenecks in some applications. Thus,
the framework must make it possible to monitor and inspect basic
operations to allow users to discover and alleviate bottlenecks.
This minimally (and most usefully) entails providing a way to
determine how many threads are blocked。

无论同步器内部设计的多么精良，他们都会成为某些应用的性能瓶颈。因此，框架必须提供监控和
检查内部操作的功能，以便用户能够发现和缓解瓶颈。至少应该（也是最有用的）包括提供一种
方法来确定有多少线程被阻塞。


## 3. DESIGN AND IMPLEMENTATION 设计和实现

The basic ideas behind a synchronizer are quite straightforward.
An acquire operation proceeds as:

同步器背后的基本思路非常简单。获取操作的过程如下：

```java
while (synchronization state does not allow acquire) {
    enqueue current thread if not already queued;
    possibly block current thread;
}
dequeue current thread if it was queued;
```

And a release operation is:
释放操作如下：
```java
update synchronization state;
if (state may permit a blocked thread to acquire)
    unblock one or more queued threads;
```
Support for these operations requires the coordination of three
basic components:
支持这些操作需要协调三个基本组件：
- Atomically managing synchronization state 原子的管理同步状态
- Blocking and unblocking threads 对线程进行阻塞和解除阻塞
- Maintaining queues 维护队列

It might be possible to create a framework that allows each of
these three pieces to vary independently. However, this would
neither be very efficient nor usable. For example, the information
kept in queue nodes must mesh with that needed for unblocking,
and the signatures of exported methods depend on the nature of
synchronization state.

也许可以创建一个框架，允许这三者独立变化。但是，这样性能不高，也无法使用。
例如，队列节点中保留的信息必须和解除阻塞所需要的信息所匹配，并且导出方法的
签名，取决于同步状态的性质。

The central design decision in the synchronizer framework was
to choose a concrete implementation of each of these three
components, while still permitting a wide range of options in
how they are used. This intentionally limits the range of
applicability, but provides efficient enough support that there is
practically never a reason not to use the framework (and instead
build synchronizers from scratch) in those cases where it does
apply.

同步框架设计的核心是如何具体实现这三个组件，同时允许它们在使用方式上有很大的灵活性。
这个设计故意限制了适用范围，但是提高了足够高效的支持，因此，在适合的情况下就应该使用
这个框架（而不是从头开始构建同步器）。

### 3.1 Synchronization State 同步状态

Class `AbstractQueuedSynchronizer` maintains synchro-
nization state using only a single (32bit) `int`, and exports
`getState`, `setState`, and `compareAndSetState`
operations to access and update this state. These methods in turn
rely on `java.util.concurrent.atomic` support providing JSR133
(Java Memory Model) compliant `volatile` semantics on reads
and writes, and access to native `compare-and-swap` or `load-
linked/store-conditional` instructions to implement `compare-
AndSetState`, that atomically sets state to a given new value
only if it holds a given expected value.

`AbstractQueuedSynchronizer` 类仅使用一个（32 位）`int`来维护同步状态，
并且通过对外暴露`getState`, `setState`, and `compareAndSetState`方法来
访问和改变这个状态。这些方法依赖于`java.util.concurrent.atomic`包，这个包
提供了 JSR133（Java 内存模型）兼容的`volatile`读、写语义，并且通过访问
`compare-and-swap` 或 `load-linked/store-conditional` 原生指令，来实现
`compare-AndSetState`，这个指令只有在状态值持有预期值时，才将状态原子地设置为
给定的新值。

Restricting synchronization state to a 32bit `int` was a pragmatic
decision. While JSR166 also provides atomic operations on 64bit
`long` fields, these must still be emulated using internal locks on
enough platforms that the resulting synchronizers would not
perform well. In the future, it seems likely that a second base
class, specialized for use with 64bit state (i.e., with `long` control
arguments), will be added. However, there is not now a
compelling reason to include it in the package. Currently, 32 bits
suffice for most applications. Only one `java.util.concurrent`
synchronizer class, `CyclicBarrier`, would require more bits
to maintain state, so instead uses locks (as do most higher-level
utilities in the package).

将同步状态限制为 32位的`int`是一个务实的决定。虽然 JSR166 也提供了对 64位`long`
字段的原子操作，但是，在部分平台上，仍然需要内部锁来模拟这些操作，从而导致生成的同步器
性能不佳。将来，可能会再添加一个基类，专门用来处理 64位状态（即具有`long`的控制参数）。
然而，目前还没有充分的理由将其包含在该包中。目前，32 位对于大多数应用程序来说已经足够。
`java.util.concurrent` 中只有`CyclicBarrier`一个同步器类，需要更多的位来维护状态，
因此它使用了锁（就像这个包中的大多数高级工具一样）。

Concrete classes based on `AbstractQueuedSynchronizer`
must define methods `tryAcquire` and `tryRelease` in terms
of these exported state methods in order to control the acquire
and release operations. The `tryAcquire` method must return
`true` if synchronization was acquired, and the `tryRelease`
method must return `true` if the new synchronization state may
allow future acquires. These methods accept a single `int`
argument that can be used to communicate desired state; for
example in a reentrant lock, to re-establish the recursion count
when re-acquiring the lock after returning from a condition wait.
Many synchronizers do not need such an argument, and so just
ignore it.

基于`AbstractQueuedSynchronizer`的具体类必须根据这些暴露的状态方法，
来定义方法 `tryAcquire` 和 `tryRelease` ，从而控制获取和释放操作。
`tryAcquire`必须在获取同步状态成功后返回`true`，`tryRelease`必须在
在将来可能获取新的同步状态时返回`true`。这些方法接受一个单独的`int`参数，
这个参数可以和所需的状态进行通信。例如，在可重入锁中，可以在条件等待返回后，
重新获取锁时重新建立递归计数。许多同步器不需要同步这个参数，忽视它即可。

### 3.2 Blocking 阻塞

Until JSR166, there was no Java API available to block and
unblock threads for purposes of creating synchronizers that are
not based on built-in monitors. The only candidates were
`Thread.suspend` and `Thread.resume`, which are
unusable because they encounter an unsolvable race problem: If
an unblocking thread invokes `resume` before the blocking
thread has executed `suspend`, the `resume` operation will have
no effect.

在 JSR166 以前，除了内置的监视器，没有可用的阻塞和解除阻塞线程的 Java API 来
创建同步器。唯一的选择是使用 `Thread.suspend` 和 `Thread.resume`，但是也不能
使用它们，因为会遇到一个无法解决的竞争问题：如果一个解除阻塞的线程先调用了 `resume`，
阻塞线程后执行`suspend` ，那么`resume`操作将没有任何作用。

The `java.util.concurrent.locks` package includes a `LockSup-port` 
class with methods that address this problem. Method
`LockSupport.park` blocks the current thread unless or until
a `LockSupport.unpark` has been issued. (Spurious wakeups
are also permitted.) Calls to `unpark` are not "counted", so
multiple `unparks` before a `park` only unblock a single `park`.
Additionally, this applies per-thread, not per-synchronizer. A
thread invoking `park` on a new synchronizer might return
immediately because of a "leftover" `unpark` from a previous
usage. However, in the absence of an `unpark`, its next
invocation will block. While it would be possible to explicitly
clear this state, it is not worth doing so. It is more efficient to
invoke `park` multiple times when it happens to be necessary.

`java.util.concurrent.locks` 包含一个名为 `LockSup-port` 的类，这个
类的方法解决了这个问题。`LockSupport.park` 方法会阻塞当前线程，直到发出
`LockSupport.unpark` 指令。（也允许虚假唤醒）对`unpark` 的调用不会“计数”，
因此在 `park` 之前多次调用`unparks`只会解除一个`park`。此外，这种情况是针对
每个线程，而不是每个同步器。一个线程在一个新的同步器上调用可能会立即返回，因为
之前使用的同步器上有一个“残留”的`unpark`。然而，如果没有调用`unpark`，下一次
调用就会阻塞。虽然可以显示的清除这个状态，但没有必要。必要时，多次调用`park`更高效。

This simple mechanism is similar to those used, at some level, in
the Solaris-9 thread library [^11], in WIN32 "consumable events",
and in the Linux NPTL thread library, and so maps efficiently to
each of these on the most common platforms Java runs on.
(However, the current Sun Hotspot JVM reference
implementation on Solaris and Linux actually uses a pthread
condvar in order to fit into the existing runtime design.) The
`park` method also supports optional relative and absolute
timeouts, and is integrated with JVM `Thread.interrupt`
support — interrupting a thread unparks it.

这种简单的机制类似于 Solaris-9 线程库[^11]、WIN32 的“可消耗事件”和 Linux 中的
NPTL 线程库，因此这在常见的 Java 运行平台上都可以高效运行。 （然而，当前的 Sun Hotspot JVM
实际上参考 Solaris 和 Linux 使用了一个 pthread condvar[^注1]，以适应当前的运行设计）
`park`操作还支持设置超时选项，可以是绝对时间和相对时间，并且于 JVM 的`Thread.interrupt`
集成——中断一个线程会解除其阻塞。

```{note}
**pthread condvar**: POSIX 线程库（Pthreads）中的条件变量（condition variable）。

条件变量是用于线程间同步的一种机制，通常与互斥锁一起使用。它允许线程在等待某个条件成立时进入阻塞状态，并在条件满足时唤醒等待的线程。
pthread condvar 提供了以下几个主要操作：
- pthread_cond_init: 初始化条件变量。
- pthread_cond_signal: 唤醒一个等待在条件变量上的线程。
- pthread_cond_broadcast: 唤醒所有等待在条件变量上的线程。
- pthread_cond_wait: 等待条件变量，同时释放相关的互斥锁，并在条件变量被信号唤醒时重新获取互斥锁。
- pthread_cond_timedwait: 类似于 pthread_cond_wait，但是可以设置超时。
```

### 3.3 Queues 队列

The heart of the framework is maintenance of queues of blocked
threads, which are restricted here to FIFO queues. Thus, the
framework does not support priority-based synchronization.

这个框架的核心是维护阻塞线程队列，这些队列在这里被限制为 FIFO 队列。因此，这个框架
不支持基于优先级的同步。

These days, there is little controversy that the most appropriate
choices for synchronization queues are non-blocking data
structures that do not themselves need to be constructed using
lower-level locks. And of these, there are two main candidates:
variants of Mellor-Crummey and Scott (MCS) locks [^9], and
variants of Craig, Landin, and Hagersten (CLH) locks [^5] [^8] [^10].
Historically, CLH locks have been used only in spinlocks.
However, they appeared more amenable than MCS for use in the
synchronizer framework because they are more easily adapted to
handle cancellation and timeouts, so were chosen as a basis. The
resulting design is far enough removed from the original CLH
structure to require explanation.

如今，普遍认为最适合同步队列的选择是非阻塞数据结构，非阻塞数据结构不需要使用较低级别
的锁来构建。在这些选择中，有两个主要的候选者：Mellor-Crummey 和 Scott (MCS)
锁的变体[^9]，以及 Craig、Landin 和 Hagersten (CLH) 锁的变体[^5] [^8] [^10]。
从历史上看，CLH 锁仅在自旋锁中被使用。然而，与 MCS 相比，CLH 似乎更适合用于同步器框架，
因为 CLH 更适合处理取消和超时操作，因此选择 CLH 作为同步框架实现的基础。这样得到的
设计与原始的 CLH 结构有很大的差异，因此需要额外解释说明。

A CLH queue is not very queue-like, because its enqueuing and
dequeuing operations are intimately tied to its uses as a lock. It is
a linked queue accessed via two atomically updatable fields,
`head` and `tail`, both initially pointing to a dummy node.

CLH 队列并不像传统的队列，因为它的入队和出队操作与它作为锁紧密相关。它是一个链式队列，
可以访问`head`和`tail`这两个字段，并且支持原子更新，这两个字段在初始化时指向一个虚拟节点。

![queues](./img/aqs/queues.png)

A new node, `node`, is enqueued using an atomic operation:

`node`是一个新节点，其原子化入队操作：
```java
do { pred = tail;
} while(!tail.compareAndSet(pred, node));
```

The release status for each node is kept in its predecessor node.
So, the "spin" of a spinlock looks like:

每个节点的释放状态都保存在前驱节点中。因此，自旋锁的“自旋”操作如下：
```java
while (pred.status != RELEASED) ; // spin
```

A dequeue operation after this spin simply entails setting the
`head` field to the node that just got the lock:

自旋之后的出队后，只需要将`head`字段设置为刚刚获得锁的节点即可。
```java
head = node;
```

Among the advantages of CLH locks are that enqueuing and
dequeuing are fast, lock-free, and obstruction free (even under
contention, one thread will always win an insertion race so will
make progress); that detecting whether any threads are waiting is
also fast (just check if `head` is the same as `tail`); and that
release status is decentralized, avoiding some memory
contention.

CLH 自旋锁的优点包括：入队和出队操作快、无锁且无阻塞（即使在竞争情况下，
总是会有一个线程会赢得插入的机会，因此整个任务一直可以取得进展）；检测是否有其他
线程正在等待也很快（只需要检查`head`与`tail`是否相同）；释放状态是分散的，
避免了一些内存竞争。

In the original versions of CLH locks, there were not even links
connecting nodes. In a spinlock, the `pred` variable can be held
as a local. However, Scott and Scherer[^10] showed that by
explicitly maintaining predecessor fields within nodes, CLH
locks can deal with timeouts and other forms of cancellation: If a
node's predecessor cancels, the node can slide up to use the
previous node's status field.

在最初的 CLH 锁中，节点之间甚至没有链接。在自旋锁中，`pred`变量可以作为局部变量存在。
然而， Scott 和 Scherer[^10] 表明，通过在节点之间明确维护前驱字段，CLH 锁可以处理
超时和其他形式的取消操作：如果一个节点的前驱节点取消了，这个节点就可以滑动上去使用前一个节点的状态字段。 

The main additional modification needed to use CLH queues for
blocking synchronizers is to provide an efficient way for one
node to locate its successor. In spinlocks, a node need only
change its status, which will be noticed on next spin by its
successor, so links are unnecessary. But in a blocking
synchronizer, a node needs to explicitly wake up (`unpark`) its
successor.

将 CLH 队列















### 3.4 Condition Queues 条件队列



[^1]: Agesen, O., D. Detlefs, A. Garthwaite, R. Knippel, Y. S.
Ramakrishna, and D. White. An Efficient Meta-lock for
Implementing Ubiquitous Synchronization. ACM OOPSLA
Proceedings, 1999.
[^2]: Andrews, G. Concurrent Programming. Wiley, 1991.
[^3]: Bacon, D. Thin Locks: Featherweight Synchronization for
Java. ACM PLDI Proceedings, 1998.
[^4]: Buhr, P. M. Fortier, and M. Coffin. Monitor Classification,
ACM Computing Surveys, March 1995.
[^5]: Craig, T. S. Building FIFO and priority-queueing spin locks
from atomic swap. Technical Report TR 93-02-02,
Department of Computer Science, University of
Washington, Feb. 1993.
[^6]: Gamma, E., R. Helm, R. Johnson, and J. Vlissides. Design
Patterns, Addison Wesley, 1996.
[^7]: Holmes, D. Synchronisation Rings, PhD Thesis, Macquarie
University, 1999.
[^8]: Magnussen, P., A. Landin, and E. Hagersten. Queue locks
on cache coherent multiprocessors. 8th Intl. Parallel
Processing Symposium, Cancun, Mexico, Apr. 1994.
[^9]: Mellor-Crummey, J.M., and M. L. Scott. Algorithms for
Scalable Synchronization on Shared-Memory
Multiprocessors. ACM Trans. on Computer Systems,
February 1991
[^10]: M. L. Scott and W N. Scherer III. Scalable Queue-Based
Spin Locks with Timeout. 8th ACM Symp. on Principles
and Practice of Parallel Programming, Snowbird, UT, June
2001.
[^11]: Sun Microsystems. Multithreading in the Solaris Operating
Environment . White paper available at
http://wwws.sun.com/software/solaris/whitepapers.html
2002.
[^12]: Zhang, H., S. Liang, and L. Bak. Monitor Conversion in a
Multithreaded Computer System. United States Patent
6,691,304. 2004.

