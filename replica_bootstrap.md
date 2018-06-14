Replica imaging and bootstrap
=============================

Patroni allows customizing creation of a new replica. It also supports defining what happens when the new empty cluster is being bootstrapped. The distinction between two is well defined: Patroni creates replicas
only if the `initialize` key is present in DCS for the cluster. If there is no `initialize` key - Patroni calls bootstrap exclusively on the first node that takes the initialize key lock.

Patroni支持创建一个新的备库。他还支持定义当一个空示例被引导时会发生什么。两者之间的区别是很明确的：Patroni仅在集群的DCS中存在初始化键时才创建副本，如果没有初始化键，Patroni会接受经过初始化键初始化的第一个节点上的引导程序。

Bootstrap
---------

PostgreSQL provides `initdb` command to initialize a new cluster and Patroni calls it by default. In certain cases, particularly when creating a new cluster as a copy of an existing one, it is necessary to replace a built-in method with custom actions. Patroni supports executing user-defined scripts to bootstrap new clusters, supplying some required arguments to them, i.e. the name of the cluster and the path to the data directory. This is configured in the `bootstrap` section of the Patroni configuration. For example:

PostgreSQL提供initdb命令去初始化新的集群，并且Patroni默认是调用这个命令进行集群初始化的。在某些情况下，特别是在创建现有集群副本的新集群时，必须用自定义操作替换内置方法。Patroni支持执行用户自定义脚本去引导新的集群，为它们提供一些必需的参数，例如集群的名称和数据目录的路径。这些是在引导项部分（bootstrap）中配置的，例如：

``` {.sourceCode .YAML}
bootstrap:
    method: <custom_bootstrap_method_name>
    <custom_bootstrap_method_name>:
        command: <path_to_custom_bootstrap_script> [param1 [, ...]]
        keep_existing_recovery_conf: False
        recovery_conf:
            recovery_target_action: promote
            recovery_target_timeline: latest
            restore_command: <method_specific_restore_command>
```

Each bootstrap method must define at least a `name` and a `command`. A special `initdb` method is available to trigger the default behavior, in which case `method` parameter can be omitted altogether. The `command`
can be specified using either an absolute path, or the one relative to the `patroni` command location. In addition to the fixed parameters defined in the configuration files, Patroni supplies two cluster-specific ones:

每个引导方法必须定义name和command，可以使用一个特殊的initdb方法来触发默认行为，在这种情况下，可以完全省略method参数。command参数可以使用绝对路径或相对于 patroni命令所在位置的相对路径。除了配置文件中定义的固定参数外，Patroni还提供了两个特定于集群的参数：

| --scope   | Name of the cluster to be bootstrapped  Name of the cluster to be bootstrapped |
| --------- | ------------------------------------------------------------ |
| --datadir | Path to the data directory of the cluster instance to be bootstrapped |

If the bootstrap script returns 0, Patroni tries to configure and start the PostgreSQL instance produced by it. If any of the intermediate steps fail, or the script returns a non-zero value, Patroni assumes that the bootstrap has failed, cleans up after itself and releases the initialize lock to give another node the opportunity to bootstrap.

如果引导脚本返回0，Patroni将尝试配置并启动由它生成的PostgreSQL实例。如果任何中间步骤失败，或者脚本返回一个非零值，Patroni就假定引导失败，将对自己进行清理并释放初始化锁，以便为另一个节点提供引导的机会。

If a `recovery_conf` block is defined in the same section as the custom bootstrap method, Patroni will generate a `recovery.conf` before starting the newly bootstrapped instance. Typically, such recovery.conf
should contain at least one of the `recovery_target_*` parameters, together with the `recovery_target_timeline` set to `promote`.

如果`recovery_conf`部分与自定义引导方法在同一节中定义，Patroni在启动新引导的实例之前将生成`recovery.conf`文件。通常，`recovery.conf`文件应该包含至少一个`recovery_target_*`参数，和`recovery_target_timeline`参数一起设置到promote参数。

If `keep_existing_recovery_conf` is defined and set to `True`, Patroni will not remove the existing  `recovery.conf` file if it exists. This is useful when bootstrapping from a backup with tools like pgBackRest that generate the appropriate `recovery.conf` for you.

如果keep_existing_recovery_conf参数被定义并且被设置为True，如果recovery.conf文件存在，patroni将不会移除他。当使用pgBackRest这样的工具从备份中引导时，这是非常有用的，这些工具为您生成适当的recovery.conf。

> <div class="admonition note">
>
> Bootstrap methods are neither chained, nor fallen-back to the default one in case the primary one fails
>
> 引导方法既不链接，也不退回默认节点以防主节点失效。
>
> </div>

Building replicas
-----------------

Patroni uses tried and proven `pg_basebackup` in order to create new replicas. One downside of it is that it requires a running master node. Another one is the lack of 'on-the-fly' compression for the backup data
and no built-in cleanup for outdated backup files. Some people prefer other backup solutions, such as `WAL-E`, `pgBackRest`, `Barman` and others, or simply roll their own scripts. In order to accommodate all those use-cases Patroni supports running custom scripts to clone a new replica. Those are configured in the `postgresql` configuration block:

Patroni使用`pg_basebackup`方法备份来创建新的副本。他的一个限制就是创建备份需要一个正在运行的master节点，另一个问题在于对备份数据的“实时”压缩，也没有对过时的备份文件进行内置清理。有些人喜欢其他备份解决方案，如`WAL-E`、`pgBackRest`、`Barman`或者其他备份方案。为了适用所有这些用例，Patroni支持运行自定义脚本来克隆一个新的副本。哪些选项配置在postgresql块。

``` {.sourceCode .YAML}
postgresql:
    create_replica_method:
        - wal_e
        - basebackup
    wal_e:
        command: patroni_wale_restore
        no_master: 1
        envdir: {{WALE_ENV_DIR}}
        use_iam: 1
    basebackup:
        max-rate: '100M'
```

The `create_replica_method` defines available replica creation methods and the order of executing them. Patroni will stop on the first one that returns 0. Each method should define a separate section in the
configuration file, listing the command to execute and any custom parameters that should be passed to that command. All parameters will be passed in a `--name=value` format. Besides user-defined parameters,
Patroni supplies a couple of cluster-specific ones:

`create_replica_method`配置项可以定义可用的副本创建方法和执行它们的顺序。Patroni在第一步执行返回0后会停止。基本备份是内置方法，不需要任何配置。其余的方法应该在配置文件中定义一个单独的部分，列出要执行的命令和应该传递给该命令的任何自定义参数。所有参数将以`--name=value`格式传递。除了用户定义的参数之外，Patroni还提供了几个特定于集群的参数：

| --scope      | Which cluster this replica belongs to                        |
| ------------ | ------------------------------------------------------------ |
| --datadir    | Path to the data directory of the replica                    |
| --role       | Always 'replica'                                             |
| --connstring | Connection string to connect to the cluster member to clone from    (master or other replica). The user in the connection string can    execute SQL and replication protocol commands. |



| --scope      | 备库所属集群名称                                             |
| ------------ | ------------------------------------------------------------ |
| --datadir    | 备库数据目录位置                                             |
| --rolee      | 一直是 ‘replica’                                             |
| --connstring | 连接字符串，连接到要从其中克隆的群集成员(主库或其他备库)。连接字符串中的用户可以执行SQL和复制协议命令。 |

A special `no_master` parameter, if defined, allows Patroni to call the replica creation method even if there is no running master or replicas. In that case, an empty string will be passed in a connection string.This is useful for restoring the formerly running cluster from the binary backup.

如果定义了特殊的参数`no_master`，即使没有正在运行的主库或者其他备库，Patroni也可以调用该复制创建方法。在这种情况下，将在连接字符串中传递一个空字符串。这对于从二进制备份还原以前运行的群集非常有用。如果所有复制创建方法都失败，Patroni将在下一个事件循环周期中按顺序重试所有方法。

A `basebackup` method is a special case: it will be used if `create_replica_method` is empty, although it is possible to list it explicitly among the `create_replica_method` methods. This method initializes a new replica with the `pg_basebackup`, the base backup is taken from the master unless there are replicas with `clonefrom` tag, in which case one of such replicas will be used as the origin for pg\_basebackup. 

It works without any configuration; however, it is possible to specify a `basebackup` configuration section. Same rules as with the other method configuration apply, namely, only long (with --) options should be specified there. Not all parameters make sense, if you override a connection string or provide an option to created tar-ed or compressed base backups, patroni won't be able to make a replica out of it. There is no validation performed on the names or values of the parameters passed to the `basebackup` section. You can specify

basebackup parameters as either a map (key-value pairs) or a list of elements, where each element could be either a key-value pair or a single key (for options that does not receive any values, for instance,
`--verbose`). Consider those 2 examples:

基本备份方法是一种特殊情况： 如果`create_replica_method`是空的，则将使用该方法，尽管可以在 `create_replica_method`方法中显式列出该方法。这个备份从master初始化数据，除非备库带有`clonefrom`标签，在那种情况下，将使用其中一个副本作为`basebackup`的源。 他在没有任何配置的情况下工作，然而，他也可以指定`basebackup` 中的一部分。与其他项的配置方法相同，即只应在其中指定长(带--)选项。 如果你想重写一个连接串或者提供一个创建的选项tar-ed或者压缩基础备份的选项，这些参数都是无效的，Patroni将无法从中生成副本。没有对传递给基本备份部分的参数名称或者值执行验证，您可以将基本备份参数指定为映射（键值对）或者元素列表，其中每个元素可以是键值对，也可以是单个键（例如，对于不不接收任何值的选项，添加-verbose），看如下两个例子：

``` {.sourceCode .YAML}
postgresql:
    basebackup:
        max-rate: '100M'
        checkpoint: 'fast'
```

and

``` {.sourceCode .YAML}
postgresql:
    basebackup:
        - verbose
        - max-rate: '100M'
```

If all replica creation methods fail, Patroni will try again all methods in order during the next event loop cycle.

如果所有复制创建的方法都失败，Patroni将在下一个事件循环周期中按照顺序将所有方法重试一遍。