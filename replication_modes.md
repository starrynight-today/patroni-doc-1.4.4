Replication modes
=================

Patroni uses PostgreSQL streaming replication. For more information about streaming replication, see the [Postgres documentation](http://www.postgresql.org/docs/current/static/warm-standby.html#STREAMING-REPLICATION).By default Patroni configures PostgreSQL for asynchronous replication.Choosing your replication schema is dependent on your business considerations. Investigate both async and sync replication, as well as other HA solutions, to determine which solution is best for you.

patroni使用Postgresql流复制。关于流复制的更多信息，可以参看[postgres手册](http://www.postgresql.org/docs/current/static/warm-standby.html#STREAMING-REPLICATION)。patroni默认配置的流复制集群是异步的。选择复制模式取决于业务考虑。调查异步复制和同步复制以及其他HA解决方案，以确定哪种解决方案最适合您。

Asynchronous mode durability
----------------------------

In asynchronous mode the cluster is allowed to lose some committed transactions to ensure availability. When the primary server fails or becomes unavailable for any other reason Patroni will automatically romote a sufficiently healthy standby to primary. Any transactions that have not been replicated to that standby remain in a "forked timeline" on the primary, and are effectively unrecoverable[^1].

在异步模式下，集群允许备库丢失一些提交的事务，与主库存在延迟，以提高数据库性能。当主服务器出现故障或由于任何其他原因而不可用时，Patroni将自动将足够健康的备用服务器提升到主服务器。没有被复制到备库的事务都保留在主库的“forked timeline”上，并且这些事务已经不能成功恢复到备库上[^1]。

The amount of transactions that can be lost is controlled via`maximum_lag_on_failover` parameter. Because the primary transaction log position is not sampled in real time, in reality the amount of lost data on failover is worst case bounded by `maximum_lag_on_failover` bytes of transaction log plus the amount that is written in the last `ttl` seconds (`loop_wait`/2 seconds in the average case). However typical steady state replication delay is well under a second.

可丢失的最大事务数由参数“maximum_lag_on_failover”指定控制。由于主库的事务日志位置不是实时采样的，事实上，在failover 是丢失的数据量最坏的值是事务日志maximum_lag_on_failover指定的字节数加上最后ttl秒（平均情况在loop_wait/2秒）写入的字节量。然而，典型的复制延迟要低于1s。

PostgreSQL synchronous replication
----------------------------------

You can use Postgres's [synchronous replication](http://www.postgresql.org/docs/current/static/warm-standby.html#SYNCHRONOUS-REPLICATION) with Patroni. Synchronous replication ensures consistency across a cluster by confirming that writes are written to a secondary before returning to the connecting client with a success. The cost of synchronous replication: reduced throughput on writes. This throughput
will be entirely based on network performance.

你可以在patroni集群中使用postgres数据库的[同步流复制模式](http://www.postgresql.org/docs/current/static/warm-standby.html#SYNCHRONOUS-REPLICATION)，同步流复制通过在成功返回连接客户端之前确认操作已经写入备库的方式确保了集群中各节点状态的一致性。同步复制的成本：写操作吞吐量降低。这种吞吐量将完全基于网络性能。

In hosted datacenter environments (like AWS, Rackspace, or any network you do not control), synchronous replication significantly increases the variability of write performance. If followers become inaccessible from the leader, the leader effectively becomes read-only.

在托管的数据中心环境（如AWS， Rackspace或者任何你无法访问的网络），同步流复制显著提升了写操作的性能可变性，如果备库与主库之间不可达，那么主库将变成只读库。

To enable a simple synchronous replication test, add the following lines to the `parameters` section of your YAML configuration files:

要启用一个简单的同步流复制的测试，在你的YAML配置文件中的`parameter`章节添加如下行：

``` {.sourceCode .YAML}
synchronous_commit: "on"
synchronous_standby_names: "*"
```

When using PostgreSQL synchronous replication, use at least three Postgres data nodes to ensure write availability if one host fails.

如果使用Postgresql同步流复制模式，使用至少3个Postgres数据节点确保当一个主机宕机后的写可用性。

Using PostgreSQL synchronous replication does not guarantee zero lost transactions under all circumstances. When the primary and the secondary that is currently acting as a synchronous replica fail simultaneously a third node that might not contain all transactions will be promoted.

使用Postgres同步流复制模式并不能保证在所有状态下事务零丢失。如果主库和当前的同步备库同时故障，这时可能不包含所有事务的潜在备库可能被提升为主库。

Synchronous mode
----------------

For use cases where losing committed transactions is not permissible you can turn on Patroni's `synchronous_mode`. When `synchronous_mode` is turned on Patroni will not promote a standby unless it is certain that the standby contains all transactions that may have returned a successful commit status to client[^2]. This means that the system may be unavailable for writes even though some servers are available. System administrators can still use manual failover commands to promote a standby even if it results in transaction loss.

对于不允许丢失已提交事务的用例，可以打开`synchronous_mode`。当`synchronous_mode`被配置打开后，Patroni 将不会提升一个备库除非确认备库中包含所有已经向客户端返回成功提交状态的事务[^2]。这意味着即使某些服务器可用，系统也可能无法进行写入。系统管理员仍然可以手动执行failover命令去提升备库的状态，即使这样可能会导致事务丢失。

Turning on `synchronous_mode` does not guarantee multi node durability of commits under all circumstances. When no suitable standby is available, primary server will still accept writes, but does not
guarantee their replication. When the primary fails in this mode no standby will be promoted. When the host that used to be the primary comes back it will get promoted automatically, unless system administrator performed a manual failover. This behavior makes synchronous mode usable with 2 node clusters.

打开synchronous_mode不能保证在所有情况下多节点提交的持久性，如果没有合适的备库可用，主库仍然可以接受写，但是不能保证他们的复制正常。在这种情形下，当主库出现异常后，备库将无法被提升，除非系统管理员手动执行failover。在没有人工干预的情况下，在旧主库恢复后，旧主库将会自动恢复为新主库。这种操作使得两节点集群变得可用。

When `synchronous_mode` is on and a standby crashes, commits will block until next iteration of Patroni runs and switches the primary to standalone mode (worst case delay for writes `ttl` seconds, average case
`loop_wait`/2 seconds). Manually shutting down or restarting a standby will not cause a commit service interruption. Standby will signal the primary to release itself from synchronous standby duties before
PostgreSQL shutdown is initiated.

当synchronous_mode被打开并且备库崩溃时，提交将阻塞，直到下一次Patroni运行的迭代（loop iteration），并且将集群主库模式（primary mode）切换成独立运行模式（standalone mode）（最坏的情况是延迟为写ttl秒，平均情况loop_wait/2秒）。手动关闭或重启备库将不会导致提交中断。在启动PostgreSQL关闭之前，备库将向主库发出解除其同步备库的信号。

You can ensure that a standby never becomes the synchronous standby by setting `nosync` tag to true. This is recommended to set for standbys that are behind slow network connections and would cause performance degradation when becoming a synchronous standby.

设置nosync标记为true能保证备库永远不会成为同步备库。建议在网络连接速度慢的备库上使用这个设置，这样能降低备库成为同步备库后性能下降的幅度。

Synchronous mode can be switched on and off via Patroni REST interface.See dynamic configuration &lt;dynamic\_configuration&gt; for instructions.

同步模式可以通过Patroni REST接口打开或关闭。详见[动态配置](./dynamic_configuration.md)

Synchronous mode implementation
-------------------------------

When in synchronous mode Patroni maintains synchronization state in the DCS, containing the latest primary and current synchronous standby. This state is updated with strict ordering constraints to ensure the following invariants:

在同步模式下，Patroni在DCS中保持同步状态，包含最新的主库和当前同步备库。这种状态通过严格的顺序约束进行更新来确保以下的不变量：

-   A node must be marked as the latest leader whenever it can accept write transactions. Patroni crashing or PostgreSQL not shutting down can cause violations of this invariant.
-   A node must be set as the synchronous standby in PostgreSQL as long as it is published as the synchronous standby.
-   A node that is not the leader or current synchronous standby is not allowed to promote itself automatically.
-   当一个节点可以接受写事务时，他必须被标记为最新的leader。Patroni崩溃或者PostgreSQL数据库没有关闭可能会引起违反一个约束
-   只要Postgresql节点作为同步备库存在，节点就必须被设置成同步备库
-   不是leader节点或者不是同步备库的节点不允许自动提升他自己

Patroni will only ever assign one standby to `synchronous_standby_names` because with multiple candidates it is not possible to know which node was acting as synchronous during the failure

Patroni只能在`synchronous_standby_names`参数上分配一个备库，因为对于多个候选节点，不可能知道哪个节点在故障期间是同步的。

On each HA loop iteration Patroni re-evaluates synchronous standby choice. If the current synchronous standby is connected and has not requested its synchronous status to be removed it remains picked.
Otherwise the cluster member available for sync that is furthest ahead in replication is picked.

在每个HA循环迭代中，Patroni重新评估选择同步备库。如果当前的同步备库是连接状态并且没有请求移除状态，那么他就会保持。否则，将选择其他集群成员作为同步备库，该成员是复制中优先级最高的成员。

[^1]: The data is still there, but recovering it requires a manual
[^1]: 数据仍然存在，但是恢复它需要数据库恢复专家的手动恢复。

    recovery effort by data recovery specialists. When Patroni is
    allowed to rewind with `use_pg_rewind` the forked timeline will be
    automatically erased to rejoin the failed primary with the cluster.
    当允许Patroni使用use_pg_rewind执行rewind，分叉的timeline将被擦除，这样失败的主节点将与集群重新连接起来。

[^2]: Clients can change the behavior per transaction using PostgreSQL's
[^2]: 客户端可以使用PostgreSQL的synchronous_commit设置来改变每个事物的行为

    `synchronous_commit` setting. Transactions with `synchronous_commit`
    values of `off` and `local` may be lost on fail over, but will not
    be blocked by replication delays.
    
    synchronous_commit 设置为off或者local在数据库集群进行故障转移时可能会导致事务丢失，但不会被复制延迟阻止。
