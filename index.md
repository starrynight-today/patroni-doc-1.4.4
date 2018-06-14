Introduction
============

Patroni is a template for you to create your own customized,high-availability solution using Python and - for maximum accessibility - a distributed configuration store like [ZooKeeper](https://zookeeper.apache.org/), [etcd](https://github.com/coreos/etcd), [Consul](https://github.com/hashicorp/consul) or [Kubernetes](https://kubernetes.io). Database engineers, DBAs, DevOps engineers, and SREs who are looking to quickly deploy HA PostgreSQL in the datacenter-or anywhere else-will hopefully find it useful.

We call Patroni a "template" because it is far from being a one-size-fits-all or plug-and-play replication system. It will have its own caveats. Use wisely. There are many ways to run high availability with PostgreSQL; for a list, see the [PostgreSQL Documentation](https://wiki.postgresql.org/wiki/Replication,_Clustering,_and_Connection_Pooling).

**Note to Kubernetes users**: Patroni can run natively on top of
Kubernetes. Take a look at the Kubernetes &lt;kubernetes&gt; chapter of
the Patroni documentation.

```
.. toctree::
   :maxdepth: 2
   :caption: Contents:

   README
   dynamic_configuration
   ENVIRONMENT
   SETTINGS
   replica_bootstrap
   replication_modes
   pause
   kubernetes
   watchdog
   releases
   CONTRIBUTING
```

介绍

Patroni是一个模板，您可以使用Python创建自己定制的、高可用性的解决方案，并且为了获得最大的可访问性，可以创建一个分布式配置存储，比如ZooKeeper, etcd, Consul or Kubernetes。数据库工程师、DBA、DevOps工程师和SRE希望在数据中心或其他任何地方快速部署HA PostgreSQL。 

我们称Patroni为一个“模板”，是因为他不是一个专门定制的复制系统。他有自己的警告。好好利用，这里又很多方式是实现运行Postgresql的高可用。有关列表，参见[PostgreSQL文档](https://wiki.postgresql.org/wiki/Replication,_Clustering,_and_Connection_Pooling) 

Kubernetes的用户请注意，Patroni可以在Kubernetes之上本地运行。 查看Patroni文档中的[Kubernetes](./kubernetes.md)章节

Indices and tables
==================

-   genindex
-   modindex
-   search

