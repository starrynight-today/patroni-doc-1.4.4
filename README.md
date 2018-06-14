Introduction
============

Patroni originated as a fork of [Governor](https://github.com/compose/governor), the project from Compose. It includes plenty of new features.

For an example of a Docker-based deployment with Patroni, see [Spilo](https://github.com/zalando/spilo), currently in use at Zalando.

For additional background info, see:

-   [PostgreSQL HA with Kubernetes and Patroni](https://www.youtube.com/watch?v=iruaCgeG7qs), talk by Josh Berkus at KubeCon 2016 (video)
-   [Feb. 2016 Zalando Tech blog post](https://tech.zalando.de/blog/zalandos-patroni-a-template-for-high-availability-postgresql/)

# 简介

Patroni起源于Compose公司的governor（https://github.com/compose/governor）项目，作为其一个分支，它包含了大量的新特性。

对于使用patroni进行基于Docker的部署的示例，spilo（https://github.com/zalando/spilo）当前正在Zalando中使用。

有关其他背景信息，请参见：

· Josh Berkus 在KubeCon 2016的演讲视频：https://www.youtube.com/watch?v=iruaCgeG7qs

· https://tech.zalando.de/blog/zalandos-patroni-a-template-for-high-availability-postgresql/



Development Status
------------------

Patroni is in active development and accepts contributions. See our Contributing &lt;contributing&gt; section below for more details. We report new releases information here &lt;releases&gt;.[]()

## 发展状况

Patroni正在积极开发和接受贡献。有关更多细节，请参见下面我们的[贡献部分](#CONTRIBUTING)。



Technical Requirements/Installation
-----------------------------------

**Pre-requirements for Mac OS**

To install requirements on a Mac, run the following:

    brew install postgresql etcd haproxy libyaml python

**General installation for pip**

Patroni can be installed with pip:

    pip install patroni[dependencies]

where dependencies can be either empty, or consist of one or more of the
following:

etcd

:   python-etcd module in order to use Etcd as DCS

consul

:   python-consul module in order to use Consul as DCS

zookeeper

:   kazoo module in order to use Zookeeper as DCS

exhibitor

:   kazoo module in order to use Exhibitor as DCS (same dependencies as
    for Zookeeper)

kubernetes

:   kubernetes module in order to use Kubernetes as DCS in Patroni

aws

:   boto in order to use AWS callbacks

For example, the command in order to install Patroni together with
dependencies for Etcd as a DCS and AWS callbacks is:

    pip install patroni[etcd,aws]

Note that external tools to call in the replica creation or custom
bootstrap scripts (i.e. WAL-E) should be installed independently of
Patroni.

## 技术要求/安装

在mac操作系统安装时的先决条件。

要在Mac上安装需求，请运行以下命令：

```
brew install postgresql etcd haproxy libyaml python
```

pip的一般安装

Patroni可以与pip一起安装：

```
pip install patroni[dependencies]
```

其中依赖项可以是空的，也可以由下列一个或多个元素组成：

etcd

​        python-etcd 模块主要用于将etcd用作dcs

consul

​        python-consul模块主要用于将consul用作dcs

zookeeper

​         kazoo模块主要用于将zookeeper用作dcs

exhibitor

​         kazoo模块主要用于将Exhibitor用作dcs（和Zooeeper相同的依赖包）

kubernetes

​          kubernetes 模块主要用于在Patroni集群中将Kubernetes 用作dcs

aws

​         boto模块主要用于AWS回调

例如，为了安装Patroni以及etcd作为DCS和AWS回调的依赖项，可以执行如下命令：

```
pip install patroni[etcd,aws]
```

注意，用于调用副本创建或者自定义引导脚本（如WAL-E）的外部工具，应该独立于Patroni安装



Running and Configuring
-----------------------

The following section assumes Patroni repository as being cloned from
<https://github.com/zalando/patroni>. Namely, you will need example
configuration files postgres0.yml and postgres1.yml. If you installed
Patroni with pip, you can obtain those files from the git repository and
replace ./patroni.py below with patroni command.

To get started, do the following from different terminals: :

    > etcd --data-dir=data/etcd
    > ./patroni.py postgres0.yml
    > ./patroni.py postgres1.yml

You will then see a high-availability cluster start up. Test different
settings in the YAML files to see how the cluster's behavior changes.
Kill some of the components to see how the system behaves.

Add more `postgres*.yml` files to create an even larger cluster.

Patroni provides an [HAProxy](http://www.haproxy.org/) configuration,
which will give your application a single endpoint for connecting to the
cluster's leader. To configure, run:

    > haproxy -f haproxy.cfg
    
    > psql --host 127.0.0.1 --port 5000 postgres

## 运行和配置

在开始之前，我们需要从不同的终端执行以下的操作：

```
> etcd --data-dir=data/etcd
> ./patroni.py postgres0.yml
> ./patroni.py postgres1.yml
```

然后，你将看到一个高可用集群启动。测试YAML文件中的不同设置，以查看集群中的行为如果更改，杀死一些组件以查看系统的行为

添加更多的postgres*.yml文件以创建更大的集群

Patroni提供了一个HAProxy的配置，他将为应用程序提供一个连接到集群领导者的端点。若要配置，请运行：

```
> haproxy -f haproxy.cfg
```

```
> psql --host 127.0.0.1 --port 5000 postgres
```

YAML Configuration
------------------

Go here &lt;settings&gt; for comprehensive information about settings for etcd, consul, and ZooKeeper. And for an example, see[postgres0.yml](https://github.com/zalando/patroni/blob/master/postgres0.yml).

请到[settings](./SETTINGS.md)去查看有关etcd, consul, and ZooKeeper设置的全面信息。有关示例参看[postgres0.yml](./postgres0.yml)

Environment Configuration
-------------------------

Go here &lt;environment&gt; for comprehensive information about configuring(overriding) settings via environment variables.

访问[environment](./ENVIRONMENT.md)去查找有关通过环境变量配置(覆盖)设置的全面信息

Replication Choices
-------------------

Patroni uses Postgres' streaming replication, which is asynchronous by default. Patroni's asynchronous replication configuration allows for `maximum_lag_on_failover` settings. This setting ensures failover will not occur if a follower is more than a certain number of bytes behind the leader. This setting should be increased or decreased based on business requirements. It's also possible to use synchronous replication for better durability guarantees. See replication modes documentation &lt;replication\_modes&gt; for details.

Patroni使用Postgres的流复制，默认情况下它是异步的。Patroni的异步复制配置通过maximum_lag_on_failover选项控制。这个设置确保如果后面的跟随者和领导者之间的差异超过特定字节数，则不会发生故障转移。应根据业务需求增加或减少此设置。还可以使用同步复制来更好地保证持久性。有关详细信息，请参阅[复制模式](./replication_modes.md)文档



Applications Should Not Use Superusers
--------------------------------------

When connecting from an application, always use a non-superuser. Patroni requires access to the database to function properly. By using a superuser from an application, you can potentially use the entire
connection pool, including the connections reserved for superusers, with the `superuser_reserved_connections` setting. If Patroni cannot access the Primary because the connection pool is full, behavior will be undesirable.

## 复制用户不用使用超级用户

在应用程序连接数据库是，要始终使用非超级用户。Patroni需要访问数据库才能正常工作，从一个应用程序使用超级用户，你可能使用掉所有的连接，包括为超级用户保留的连接`superuser_reserved_connections`。应用由于连接数已经被占满导致无法连接主库，这样的行为是不可取的

