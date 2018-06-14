Pause/Resume mode for the cluster
=================================

# 集群的暂停/恢复模式

The goal
--------

## 目标

Under certain circumstances Patroni needs to temporary step down from managing the cluster, while still retaining the cluster state in DCS. Possible use cases are uncommon activities on the cluster, such as major
version upgrades or corruption recovery. During those activities nodes are often started and stopped for the reason unknown to Patroni, some nodes can be even temporary promoted, violating the assumption of
running only one master. Therefore, Patroni needs to be able to "detach" from the running cluster, implementing an equivalent of the maintenance mode in Pacemaker.

在某些情况下，Patroni需要暂时退出对集群的管理，同时在DCS中保留集群状态。可能的用例是集群中不常见的活动，例如主要版本升级或损坏恢复。在这些活动中，由于Patroni不知道的原因，节点经常被启动和停止，有些节点甚至可以被临时提升，这违反了只运行一个master的假设。因此，Patroni需要能够从正在运行的集群中“分离”出来，实现与Pacemaker等效的维护模式。



The implementation
------------------

## 实施

When Patroni runs in a paused mode, it does not change the state of PostgreSQL, except for the following cases：

当Patroni以暂停模式运行时，它不会更改PostgreSQL的状态，但下列情况除外：

-   For each node, the member key in DCS is updated with the current information about the cluster. This causes Patroni to run read-only queries on a member node if the member is running.
-   For the Postgres master with the leader lock Patroni updates the lock. If the node with the leader lock stops being the master (i.e. is demoted manually), Patroni will release the lock instead of promoting the node back.
-   Manual unscheduled restart, reinitialize and manual failover are allowed. Manual failover is only allowed if the node to failover to is specified. In the paused mode, manual failover does not require a
    running master node.
-   If 'parallel' masters are detected by Patroni, it emits a warning, but does not demote the masters without the leader lock.
-   If there is no leader lock in the cluster, the running master acquires the lock. If there is more than one master node, then the first master to acquire the lock wins. If there are no masters altogether, Patroni does not try to promote any replicas. There is an exception in this rule: if there is no leader lock because the old master has demoted itself due to the manual promotion, then only the candidate node mentioned in the promotion request may take the leader lock. When the new leader lock is granted (i.e. after promoting a replica manually), Patroni makes sure the replicas that were streaming from the previous leader will switch to the new one.
-   When Postgres is stopped, Patroni does not try to start it. When Patroni is stopped, it does not try to stop the Postgres instance it is managing.
-   对于每个节点，DCS中的成员键都会使用集群的当前信息进行更新。这会导致Patroni在正在运行的成员节点上执行只读查询
-   对于有领导锁的Postgres主库，Patroni将更新锁。如果有领导锁的Postgres主库不在是主库（比如手动降级），Patroni将释放锁而不是将他重新提升为主库
-   允许手动进行非计划重新启动、重新初始化和手动故障转移。只有在指定了要故障转移到的节点时，才允许手动故障转移。在暂停模式下，手动故障转移不需要正在运行的主节点
-   如果‘parallel’的主库被Patroni检测到，他会发出一个警告，但是不会在没有领导锁的情况下降级主程序
-   如果集群中没有领导锁，则运行的主机将获取锁。如果有多个主节点，则获得锁的第一个主节点获胜。如果集群中没有主节点，Patroni不会尝试去提升任何备节点。这一规则中有一个例外：如果由于手动操作引起主节点被降级导致的没有领导锁，那么只有提升请求中提到的候选节点才能获得领导锁。当一个新的领导锁被授予（比如手动提升了一个候选节点），Patroni来确保其他备库的流复制从旧主库切换成新主库
-   当PostgreSQL进程被停止，Patroni不会尝试去启动它。当Patroni进程被停止，不会尝试去停止他正在管理的Postgres实例

User guide
----------

## 用户手册

`patronictl` supports `pause` and `resume` commands.

One can also issue a `PATCH` request to the`{namespace}/{cluster}/config` key with `{"pause": true/false/null}`

patronict支持pause和resume命令。

你还可以向{namespace}/{cluster}/config键加上{"pause": true/false/null}发出PATCH请求。