Watchdog support
================

# 支持Watchdog

Having multiple PostgreSQL servers running as master can result in transactions lost due to diverging timelines. This situation is also called a split-brain problem. To avoid split-brain Patroni needs to ensure PostgreSQL will not accept any transaction commits after leader key expires in the DCS. Under normal circumstances Patroni will try to achieve this by stopping PostgreSQL when leader lock update fails for
any reason. However, this may fail to happen due to various reasons:

如果有多个PostgreSQL服务器作为主服务器运行，可能会由于不同的时间线而导致事务丢失。这种情况也被称为脑分裂问题。为了避免大脑分裂，Patroni需要确保PostgreSQL不会在DCS中的LeaderKey过期后接受任何事务提交。在正常情况下，Patroni会尝试在领导者锁更新由于任何原因失败时停止PostgreSQL来实现这一点。然而，由于各种原因，这种情况可能无法实现：

-   Patroni has crashed due to a bug, out-of-memory condition or by being accidentally killed by a system administrator.
-   Shutting down PostgreSQL is too slow.
-   Patroni does not get to run due to high load on the system, the VM being paused by the hypervisor, or other infrastructure issues.
-   Patroni因错误、内存不足或被系统管理员意外杀死而崩溃
-   关闭PostgreSQL太慢了
-   Patroni因为系统负载过高、虚拟机管理程序暂停或其他基础设施问题而无法运行

To guarantee correct behavior under these conditions Patroni supports watchdog devices. Watchdog devices are software or hardware mechanisms that will reset the whole system when they do not get a keepalive heartbeat within a specified timeframe. This adds an additional layer of fail safe in case usual Patroni split-brain protection mechanisms fail.

为了保证在这些条件下的正确行为，Patroni支持Watchdog设备。Watchdog设备是一种软件或硬件机制，当它们在指定的时间范围内无法保持心跳时，它们将重置整个系统。这增加了一个额外的故障安全层，以防Patroni分裂脑保护机制失效。

Patroni will try to activate the watchdog before promoting PostgreSQL to master. If watchdog activation fails and watchdog mode is `required` then the node will refuse to become master. When deciding to participate in leader election Patroni will also check that watchdog configuration will allow it to become leader at all. After demoting PostgreSQL (for example due to a manual failover) Patroni will disable the watchdog again. Watchdog will also be disabled while Patroni is in paused state.

Patroni在将PostgreSQL提升为MASTER之前尝试激活Watchdog设备。如果Watchdog设备激活失败，并且Watchdog 的模式为`required`，那么节点将拒绝成为主节点。在决定参加领导人选举时，Patroni还将检查Watchdog的配置是否允许它成为领导者。降级PostgreSQL之后(例如，由于手动故障转移)，Patroni将再次禁用Watchdog。在Patroni处于暂停状态时，Watchdog也将被禁用。

By default Patroni will set up the watchdog to expire 5 seconds before TTL expires. With the default setup of `loop_wait=10` and `ttl=30` this gives HA loop at least 15 seconds (`ttl` - `safety_margin` -`loop_wait`) to complete before the system gets forcefully reset. By default accessing DCS is configured to time out after 10 seconds. This means that when DCS is unavailable, for example due to network issues, Patroni and PostgreSQL will have at least 5 seconds (`ttl` - `safety_margin` - `loop_wait` - `retry_timeout`) to come to a state where all client connections are terminated.

默认情况下，Patroni将设置Watchdog在TTL过期前5秒过期。默认设置 `loop_wait`=10 并且 `ttl`=30，这样HA循环至少需要15秒（`ttl` - `safety_margin` - `loop_wait`）才能完成，然后系统才能重启。默认情况下，访问DCS被配置为10秒后超时。这意味着，当DCS不可达，如网络故障，Patroni和PostgreSQL将至少有5秒（`ttl` - `safety_margin` - `loop_wait` - `retry_timeout`）到达所有客户端被终止这个状态。

Safety margin is the amount of time that Patroni reserves for time between leader key update and watchdog keepalive. Patroni will try to send a keepalive immediately after confirmation of leader key update. If Patroni process is suspended for extended amount of time at exactly the right moment the keepalive may be delayed for more than the safety margin without triggering the watchdog. This results in a window of time where watchdog will not trigger before leader key expiration, invalidating the guarantee. To be absolutely sure that watchdog will trigger under all circumstances set up the watchdog to expire after half of TTL by setting `safety_margin` to -1 to set watchdog timeout to `ttl / 2`. If you need this guarantee you probably should increase `ttl` and/or reduce `loop_wait` and `retry_timeout`.

安全阈值（safety margin）是指Patroni在领导者秘钥（leader key）更新和看Watchdog存活之间预留的时间。Patroni在确认领导者密钥更新后，尝试立即发送一个保持活动的消息。如果Patroni进程在计划内的时间暂停了很长一段时间，那么在不触发Watchdog的情况下，保持生命可能会被延迟超过安全阈值。这会导致一个时间窗口，在领导者密钥到期之前，Watchdog不会触发，从而使保证无效。为了保证Watchdog在任何情况下都会被触发，通过把`safety_margin`设置为-1，将Watchdog超时设置为`ttl/2`，从而将Watchdog设置为TTL的一半之后过期。如果你需要这个保证，你应该调高`ttl` and/or 减少`loop_wait` 和 `retry_timeout`.

Currently watchdogs are only supported using Linux watchdog device interface.

目前，Watchdog只支持使用Linux Watchdog设备接口的设备。

Setting up software watchdog on Linux
-------------------------------------

## 在Linux上安装Watchdog软件

Default Patroni configuration will try to use `/dev/watchdog` on Linux if it is accessible to Patroni. For most use cases using software watchdog built into the Linux kernel is secure enough.

Patroni的默认配置将尝试去Linux的/dev/watchdog路径去找Watchdog设备。从大多数使用Linux内核内置的软件看门狗的使用情况来说，安全性已经足够。

To enable software watchdog issue the following commands as root before starting Patroni:

要启用软件Watchdog，在启动Patroni之前以root身份发出以下命令：

``` {.sourceCode .bash}
modprobe softdog
# Replace postgres with the user you will be running patroni under
chown postgres /dev/watchdog
```

For testing it may be helpful to disable rebooting by adding `soft_noboot=1` to the modprobe command line. In this case the watchdog will just log a line in kernel ring buffer, visible via dmesg.

为了进行测试，可以通过将`soft_noboot=1`添加到modprobe命令行中来禁用重新引导。在这种情况下，看门狗只会记录内核环缓冲区中的一行，可以通过dmesg看到。

Patroni will log information about the watchdog when it is successfully enabled.

成功启用时，Patroni将记录有关Watchdog的信息。