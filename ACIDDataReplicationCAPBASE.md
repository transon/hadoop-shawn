ACID
```
在传数据库系统中，事务具有ACID 4个属性(Jim Gray在《事务处理：概念与技术》中对事务进行了详尽的讨论)。

(1)原子性（Atomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。

(2)一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。

(3)隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。

(4)持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

对于单个节点的事务，数据库都是通过并发控制（两阶段封锁，two phase locking或者多版本，multiversioning）和恢复机制（日志技术）保证事务的ACID特性。对于跨多个节点的分布式事务，通过两阶段提交协议（two phase commiting）来保证事务的ACID。

可以说，数据库系统是伴随着金融业的需求而快速发展起来。对于金融业，可用性和性能都不是最重要的，而一致性是最重要的，用户可以容忍系统故障而停止服务，但绝不能容忍帐户上的钱无故减少(当然，无故增加是可以的)。而强一致性的事务是这一切的根本保证。
```


Data Replication
```
数据复制(data replication)属于分布式计算的范畴，它并不仅仅局限于数据库，但这里主要是指分布式数据库的复制。

在多副本构成的分布式数据库系统中， 其事务特性与单个数据库系统的差别主要表现在原子性和一致性两个方 面。在原子性方面， 要求同一分布式事务的所有操作在所有相关副本上要么提交， 要么回滚， 即除了保证原有的局部事务的原子性，还需要控制全局事务的原子性； 在一致性方面，多副本之间需要保证单一副本一致性。 

针对分布式事务的原子性和一致性这两个复制协议中的核心问题， 经过近20年的研究 ， 人们提出了各种各样的复制协议。这些协议在外在功能和内部实现两方面都有较大的差别。据此，我们可以从这两个大的方面进行分类说明。

从外在功能的角度看， 依据文献[1]， 可以从事务执行的地点和时间两个方面进行分类。从事务执行的地点，可以分为两类： 主从( Priamry / Copy)方式和更新所有( Update-Anywhere ) 方式。

前者的处理过程一般是系统中仅仅指定一个Primary节点接受更新请求 ， 在事务操作执行完毕后， 在事务提交前或后将操作广播到其他Copy节点。

后者的处理过程稍微复杂， 系统中的任何副本具有相同的地位，都可以接收Update请求 ，在检测事务冲突、 事务提交前或后将各个节点的Update传播到其他副本节点。

Primary / Copy方式并发控制较为简单， 由Primary本地的事务控制即可实现， 事务的原子性的实现也较为简单， 一般由Primary节点作为协调节点来实现。但是， 其缺陷也显而易见： 仅仅单个节点提供Update请求处理能力， 对于Update密集类型的应用， 如OLTP， 容易形成单点性能瓶颈。Update-Anywhere方式则与其相辅相成， 可以通过多点提高事务吞吐率， 但随之而来的是多个分布式事务之间复杂的并发控制和原子性问题。

      从事务提交的时间点看， 可以分为积极 (Eager)和消极(Lazy) 两类。其区别在于， 前者是在事务提交前传播更新，后者则是在提交之后才将事务操作传播到其他副本。实际上，前者即通常无谓的同步复制(synchronous replication)，后者即无谓的异步复制(asynchronous replication)。

异步复制的优点是可以提高响应速度， 但牺牲了一致性 ，一般实现该类协议的算法需要增加额外的补偿机制。同步复制的优点是可以保证一致性(一般通过两阶段提交协议)，但是开销较大，可用性不好(参见CAP部分)，带来了更多的冲突和死锁等问题。值得一提的是Lazy+Primary/Copy的复制协议在实际生产环境中是非常实用的，MySQL的复制实际上就属于这种。
```


CAP
```
在2000的PODC（Principles of Distributed Computing）会议上，Brewer提出了著名的CAP理论。2002年，Seth Gilbert和Nancy Lynch证明了这一理论。CAP指的是：Consistency、Availability和Partition Tolerance。

（1）Consistency（一致性）：一致性是说数据的原子性，这种原子性在经典的数据库中是通过事务来保证的，当事务完成时，无论其是成功还是回滚，数据都会处于一致的状态。在分布式环境中，一致性是说多个节点的数据是否一致。

（2）Availability（可用性）：可用性是说服务能一直保证是可用的状态，当用户发出一个请求，服务能在有限时间内返回结果。

（3）Partition Tolerance（分区容错性）：Partition是指网络的分区。可以这样理解，一般来说，关键的数据和服务都会位于不同的IDC。

CAP理论告诉我们，一个分布式系统不可能同时满足一致性，可用性和分区容错性这三个需求，三个要素中最多只能同时满足两点。三者不可兼顾，此所谓鱼与熊掌不可兼得也！而对于分布式数据系统而言，分区容错性是基本要求，否则就不称其为分布式系统了。因此架构设计师不要把精力浪费在设计如何能同时满足三者的完美分布式系统上，而是应该进行权衡取舍。这也意味着分布式系统的设计过程，也就是根据业务特点在C（一致性）和A（可用性）之间寻求平衡的过程，要求架构师真正理解系统需求，把握业务特点。
```


BASE
```
BASE来自于互联网的电子商务领域的实践，它是基于CAP理论逐步演化而来，核心思想是即便不能达到强一致性(Strong consistency)，但可以根据应用特点采用适当的方式来达到最终一致性(Eventual consistency)的效果。BASE是Basically Available、Soft state、Eventually consistent三个词组的简写，是对CAP中C & A的延伸。BASE的含义：

（1）Basically Available：基本可用；

（2）Soft-state：软状态/柔性事务，即状态可以有一段时间的不同步；

（3）Eventual consistency：最终一致性；

BASE是反ACID的，它完全不同于ACID模型，牺牲强一致性，获得基本可用性和柔性可靠性并要求达到最终一致性。

CAP、BASE理论是当前在互联网领域非常流行的NoSQL的理论基础。
```


主要参考

[1](1.md)J.N.Gray, P.Helland,and a.D.S.P.O’Neil. The dangers of replication and a solution. In Proceedings of the 1996 ACM SIGMOD International Conference on Management of Data, pages 173–182,Montreal, Canada, June 1996.SIGMOD.

[2](2.md)Gilbert , S., Lynch, N. 2002. Brewer’s conjecture and the feasibility of consistent, available, partition-tolerant Web services. ACM SIGACT News 33(2).

[3](3.md)http://www.allthingsdistributed.com/2008/12/eventually_consistent.html

[4](4.md)http://queue.acm.org/detail.cfm?id=1394128

[5](5.md)http://en.wikipedia.org/wiki/ACID