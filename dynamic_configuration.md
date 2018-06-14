Patroni configuration
=====================

Patroni configuration is stored in the DCS (Distributed Configuration
Store). There are 3 types of configuration:

# Patroni配置

Patroni配置存储在DCS(分布式配置存储)中。配置有三种类型：



- Dynamic configuration.

     These options can be set in DCS at any time. If the options changed are not part of the startup configuration, they are applied asynchronously (upon the next wake up cycle) to every node, which gets subsequently reloaded. If the node requires a restart to apply the configuration (for options with context postmaster, if their values have changed), a special flag,  `pending_restart` indicating this, is set in the members.data JSON. Additionally, the node status also indicates this, by showing `"restart_pending": true`.

    ## 动态配置

    这些选项能够在随时在DCS中被设置。如果更改的选项不是启动配置的一部分，则他们异步应用于每个节点(在下一个唤醒周期)，然后重新加载。如果节点需要重启才能在应用程序生效(如果带有上下文POST的选项的值发生了更改)，则使用pending_restart命令在embers.data的JSON中设置一个特殊的标志。此外，节点状态也通过显示"restart_pending": true来表示。

- Local configuration &lt;settings&gt; (patroni.yml).

    These options are defined in the configuration file and take recedence over dynamic configuration. patroni.yml could be changed and reload in runtime (without restart of Patroni) by sending SIGHUP to the Patroni process or by performing `POST /reload` REST-API request.

    ## 本地配置

    这些选项是在配置文件中定义的，优先于动态配置。通过将SIGHUP发送到Patroni进程或执行POST/RELOAD RET-API请求，可以在运行时更改并重新加载patroni.yml(无需重新启动patroni)。

    

- Environment configuration &lt;environment&gt;.

    It is possible to set/override some of the "Local" configuration parameters with environment variables. Environment configuration is very useful when you are running in a dynamic environment and you don't know some of the parameters in advance (for example it's not possible to know your external IP address when you are running inside `docker`).

    ## 环境配置

    可以用环境变量设置/覆盖一些"Local"配置参数。当您在动态环境中运行且事先不知道某些参数时(例如，在docker内运行时不可能知道外部IP地址)，环境配置非常有用。

    

Some of the PostgreSQL parameters must hold the same values on the master and the replicas. For those, values set either in the local patroni configuration files or via the environment variables take no effect. To alter or set their values one must change the shared configuration in the DCS. Below is the actual list of such parameters together with the default values:

PostgreSQL数据库的一些参数必须在主库和备库上配置的值相同。对于这种类型的参数，在本地配置文件中设置的值或通过环境变量设置的值都不起作用。要更改或设置它们的值，必须更改DCS中的共享配置。下面是这些参数的实际列表以及默认值：

-   max\_connections: 100
-   max\_locks\_per\_transaction: 64
-   max\_worker\_processes: 8
-   max\_prepared\_transactions: 0
-   wal\_level: hot\_standby
-   wal\_log\_hints: on
-   track\_commit\_timestamp: off

For the parameters below, PostgreSQL does not require equal values among the master and all the replicas. However, considering the possibility of a replica to become the master at any time, it doesn't really make sense to set them differently; therefore, Patroni restricts setting their values to the Dynamic configuration

对于以下参数，PostgreSQL并不要求在主库和备库上完全一致，但是要考虑到备库也会提升为主库，把这些值配置成为不同值也是没有意义的，因此，Patroni限制了将他们的值动态配置。

-   max\_wal\_senders: 5
-   max\_replication\_slots: 5
-   wal\_keep\_segments: 8

These parameters are validated to ensure they are sane, or meet a minimum value.

There are some other Postgres parameters controlled by Patroni:

这些参数需要被验证以确保它们是正常的或者满足最小值。

这还有一些由Patroni控制的PostgreSQL参数：

- listen\_addresses - is set either from `postgresql.listen` or from `PATRONI_POSTGRESQL_LISTEN` environment variable

- port - is set either from `postgresql.listen` or from `PATRONI_POSTGRESQL_LISTEN` environment variable

- cluster\_name - is set either from `scope` or from `PATRONI_SCOPE` environment variable

- hot\_standby: on

- listen_addresses:可以由postgresql.listen或PATRONI_POSTGRESQL_LISTEN环境变量控制。

- port:可以由postgresql.liste或PATRONI_POSTGRESQL_LISTEN环境变量控制

- cluster_name：可以由scope或PATRRONI_SCOPE环境变量控制

- hot_standby：on

    

To be on the safe side parameters from the above lists are not written into `postgresql.conf`, but passed as a list of arguments to the `pg_ctl start` which gives them the highest precedence, even above [ALTER SYSTEM](https://www.postgresql.org/docs/current/static/sql-altersystem.html)

为了安全起见，上面列表中的参数没有写入postgresql.conf中，而是作为参数的形式传递给pg_ctl start，这以为着他们有最高的优先级，甚至优于[ALTER SYSTEM](https://www.postgresql.org/docs/current/static/sql-altersystem.html)

When applying the local or dynamic configuration options, the following actions are taken:

在应用本地或动态配置选项时，将采取以下操作：

- The node first checks if there is a postgresql.base.conf or if the `custom_conf` parameter is set.

- If the custom\_conf parameter is set, it will take the file specified on it as a base configuration, ignoring postgresql.base.conf and postgresql.conf.

- If the custom\_conf parameter is not set and postgresql.base.conf exists, it contains the renamed "original" configuration and it will be used as a base configuration.

- If there is no custom\_conf nor postgresql.base.conf, the original postgresql.conf is taken and renamed to postgresql.base.conf.

- The dynamic options (with the exceptions above) are dumped into the postgresql.conf and an include is set in postgresql.conf to the used base configuration (either  ostgresql.base.conf or what is on `custom_conf`). Therefore, we would be able to apply new options without re-reading the configuration file to check if the include is
    present not.

- Some parameters that are essential for Patroni to manage the cluster are overridden using the command line.

- If some of the options that require restart are changed (we should look at the context in pg\_settings and at the actual values of those options), a pending\_restart flag of a given node is set. This flag is reset on any restart.

- 节点首先检查是否有postgresql.base.conf，或者是否设置了custom_conf参数

- 如果设置了custom_conf参数，它将把在其上指定的文件作为基本配置，而忽略postgresql.base.conf和postgresql.conf

- 如果没有设置custom_conf参数并且存在postgresql.base.conf文件，他将包含一个重命名的“原始”文件，并且被用作基本配置

- 如果没有设置custom_conf并且不存在postgresql.base.conf文件，原始配置文件postgresql.conf将被使用并且被复制一份命名为postgresql.base.conf的配置文件

- 动态配置项（除上述例外）被存储在postgresql.conf文件中，并在postgresql.conf文件中设置一个include选项来指定数据库的配置参数（postgresql.base.conf或者custom_conf）。因此我们可以通过不重读配置文件的来应用新的配置项来检查include是否生效

- Patroni管理集群所必须的一些参数可以通过命令行被重写

- 如果一些必须重启生效的选项在修改后（我们可以查看pg_settings中那些选项的含义以及当前的实际配置），指定节点的pending_restart标记将会被设置。这个标记在重启后会重置

    

The parameters would be applied in the following order (run-time are given the highest priority):

配置的参数将按以下的顺序应用(运行时设置的参数（run-time parameter）被赋予最高的优先级)

1. load parameters from file postgresql.base.conf (or from a custom\_conf file, if set)

2. load parameters from file postgresql.conf

3. load parameters from file postgresql.auto.conf

4. run-time parameter using -o --name=value

5. 从配置文件postgresql.base.conf中加载参数（如果设置了custom_conf，也可能从custom_conf的配置中加载）

6. 从配置文件postgresql.conf中加载参数

7. 从配置文件postgresql.auto.conf中加载参数

8. 在程序运行时通过 -o –name=value 指定的参数

    

This allows configuration for all the nodes (2), configuration for a specific node using ALTER SYSTEM (3) and ensures that parameters essential to the running of Patroni are enforced (4), as well as leaves room for configuration tools that manage postgresql.conf directly without involving Patroni (1)

优先级二的配置对所有节点生效，使用ALTER SYSTEM（优先级三）对某些特定节点进行配置，Patroni管理集群所必须的一些参数（优先级四）是强制指定的，优先级一的参数存储在独立的配置文件，修改时不需要修改postgresql.conf 文件



Also, the following Patroni configuration options can be changed only dynamically:

此外，只能动态更改下列Patroni配置选项：

-   ttl: 30
-   loop\_wait: 10
-   retry\_timeouts: 10
-   maximum\_lag\_on\_failover: 1048576
-   postgresql.use\_slots: true

Upon changing these options, Patroni will read the relevant section of the configuration stored in DCS and change its run-time values.

Patroni nodes are dumping the state of the DCS options to disk upon for every change of the configuration into the file `patroni.dynamic.json` located in the Postgres data directory. Only the master is allowed to restore these options from the on-disk dump if these are completely
absent from the DCS or if they are invalid.

更改这些选项后，Patroni将读取存储在DCS中配置的相关部分并动态更改他运行时的值。

Patroni节点在实时地将DCS的状态信息转储到磁盘上，以便将每次修改写入到位于数据库数据目录的patroni.dynamic.json文件中。



REST API
========

We provide a REST API endpoint for working with dynamic configuration.

# REST API

我们提供一个接口用于动态配置参数。

GET /config
-----------

Get current version of dynamic configuration.

## GET /config

获取当前版本的动态配置。

``` {.sourceCode .bash}
$ curl -s localhost:8008/config | jq .
{
  "ttl": 30,
  "loop_wait": 10,
  "retry_timeout": 10,
  "maximum_lag_on_failover": 1048576,
  "postgresql": {
    "use_slots": true,
    "use_pg_rewind": true,
    "parameters": {
      "hot_standby": "on",
      "wal_log_hints": "on",
      "wal_keep_segments": 8,
      "wal_level": "hot_standby",
      "max_wal_senders": 5,
      "max_replication_slots": 5,
      "max_connections": "100"
    }
  }
}
```

PATCH /config
-------------

Change existing configuration.

## PATCH /config

修改已经存在的配置

``` {.sourceCode .bash}
$ curl -s -XPATCH -d \
    '{"loop_wait":5,"ttl":20,"postgresql":{"parameters":{"max_connections":"101"}}}' \
    http://localhost:8008/config | jq .
{
  "ttl": 20,
  "loop_wait": 5,
  "maximum_lag_on_failover": 1048576,
  "retry_timeout": 10,
  "postgresql": {
    "use_slots": true,
    "use_pg_rewind": true,
    "parameters": {
      "hot_standby": "on",
      "wal_log_hints": "on",
      "wal_keep_segments": 8,
      "wal_level": "hot_standby",
      "max_wal_senders": 5,
      "max_replication_slots": 5,
      "max_connections": "101"
    }
  }
}
```

The above REST API call patches the existing configuration and returns the new configuration.

Let's check that the node processed this configuration. First of all it should start printing log lines every 5 seconds (loop\_wait=5). The change of "max\_connections" requires a restart, so the
"restart\_pending" flag should be exposed:

上面的接口调用修改了现有的配置并且返回了新的配置信息

我们来检查一下节点是否处理了这个配置，首先应该配置每5秒打印一次日志行(loop_wait=5)。参数“max_connections”的改变需要重启，因此“restart_pending”应该被标记为true

``` {.sourceCode .bash}
$ curl -s http://localhost:8008/patroni | jq .
{
  "pending_restart": true,
  "database_system_identifier": "6287881213849985952",
  "postmaster_start_time": "2016-06-13 13:13:05.211 CEST",
  "xlog": {
    "location": 2197818976
  },
  "patroni": {
    "scope": "batman",
    "version": "1.0"
  },
  "state": "running",
  "role": "master",
  "server_version": 90503
}
```

Removing parameters:

If you want to remove (reset) some setting just patch it with `null`:

移除参数：

如果你需要移除或者重置一些配置只需要给他赋值为null

``` {.sourceCode .bash}
$ curl -s -XPATCH -d \
    '{"postgresql":{"parameters":{"max_connections":null}}}' \
    http://localhost:8008/config | jq .
{
  "ttl": 20,
  "loop_wait": 5,
  "retry_timeout": 10,
  "maximum_lag_on_failover": 1048576,
  "postgresql": {
    "use_slots": true,
    "use_pg_rewind": true,
    "parameters": {
      "hot_standby": "on",
      "unix_socket_directories": ".",
      "wal_keep_segments": 8,
      "wal_level": "hot_standby",
      "wal_log_hints": "on",
      "max_wal_senders": 5,
      "max_replication_slots": 5
    }
  }
}
```

Above call removes `postgresql.parameters.max_connections` from the dynamic configuration.

上面的调用从动态配置中删除postgresql.parameters.max_connections

PUT /config
-----------

It's also possible to perform the full rewrite of an existing dynamic configuration unconditionally:

## PUT /config

还可以无条件地执行现有动态配置的全部重写：

``` {.sourceCode .bash}
$ curl -s -XPUT -d \
    '{"maximum_lag_on_failover":1048576,"retry_timeout":10,"postgresql":{"use_slots":true,"use_pg_rewind":true,"parameters":{"hot_standby":"on","wal_log_hints":"on","wal_keep_segments":8,"wal_level":"hot_standby","unix_socket_directories":".","max_wal_senders":5}},"loop_wait":3,"ttl":20}' \
    http://localhost:8008/config | jq .
{
  "ttl": 20,
  "maximum_lag_on_failover": 1048576,
  "retry_timeout": 10,
  "postgresql": {
    "use_slots": true,
    "parameters": {
      "hot_standby": "on",
      "unix_socket_directories": ".",
      "wal_keep_segments": 8,
      "wal_level": "hot_standby",
      "wal_log_hints": "on",
      "max_wal_senders": 5
    },
    "use_pg_rewind": true
  },
  "loop_wait": 3
}
```
