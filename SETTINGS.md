YAML Configuration Settings
===========================

Global/Universal
----------------

# 全局变量

-   **name**: the name of the host. Must be unique for the cluster.
-   **namespace**: path within the configuration store where Patroni will keep information about the cluster. Default value: "/service"
-   **scope**: cluster name
-   **name**: 主机名，在集群内必须唯一
-   **namespace**:Patroni保留集群信息的存储路径配置，默认为“/service”
-   **scope**: 集群名

Bootstrap configuration
-----------------------

- **dcs**: This section will be written into /&lt;namespace&gt;/&lt;scope&gt;/config of a given configuration store after initializing of new cluster. This is the global configuration for the cluster. If you want to change some parameters for all cluster nodes - just do it in DCS (or via Patroni API) and all nodes will apply this configuration.

- 在初始化一个新的集群之后，这节将被写入到/&lt;namespace&gt;/&lt;scope&gt;/config 位置进行配置文件存储。着是集群的全局配置。如果你想要修改所有节点上的一些参数—可以通过DCS（或者通过Patroni API）进行修改，集群中所有节点都会应用此配置

    - **loop\_wait**: the number of seconds the loop will sleep.Default value: 10
    - **loop\_wait**: 循环休眠的秒数。 默认值：10

    - **ttl**: the TTL to acquire the leader lock. Think of it as the length of time before initiation of the automatic failover process. Default value: 30
    - **ttl**：TTL获得领导锁。将其视为启动自动故障转移过程之前的时间长度。默认值：30 

    - **retry\_timeout**: timeout for DCS and PostgreSQL operation retries. DCS or network issues shorter than this will not cause Patroni to demote the leader. Default value: 10
    - **retry\_timeout:**DCS和PostgreSQL操作重试超时时间。短于此的DCS或网络问题不会导致Patroni降级领导。默认值：10 

    - **maximum\_lag\_on\_failover**: the maximum bytes a follower may lag to be able to participate in leader election.
    - **maximum\_lag\_on\_failover：**备库可能滞后于能够参加领导选举要求的最大字节数。 

    - **master\_start\_timeout**: the amount of time a master is allowed to recover from failures before failover is triggered. Default is 300 seconds. When set to 0 failover is done immediately after a crash is detected if possible. When using asynchronous replication a failover can cause lost transactions. Best worst case failover time for master failure is: loop\_wait + master\_start\_timeout + loop\_wait, unless master\_start\_timeout is zero, in which case it's just loop\_wait. Set the value according to your durability/availability tradeoff.
    - **master\_start\_timeout**: 在触发故障转移之前，允许主机从故障中恢复的时间。默认为300秒。 当设置为0时，如果可能的话，在检测到崩溃后立即执行故障转移。使用异步复制时，故障转移可能导致事务丢失。主故障的最坏情况故障转移时间是： loop\_wait + master\_start\_timeout + loop\_wait，除非master\_start\_timeout设置为0，这样它只需要loop\_wait，根据您的耐久性/可用性权衡来设置此值 

    - **synchronous\_mode**: turns on synchronous replication mode. In this mode a replica will be chosen as synchronous and only the latest leader and synchronous replica are able to participate in leader election. Synchronous mode makes sure that successfully committed transactions will not be lost at failover, at the cost of losing availability for writes when Patroni cannot ensure transaction durability. See  replication modes documentation &lt;replication\_modes> for details.
    - **synchronous\_mode**: 打开同步流复制方式。在这种模式下，将选择一个副本作为同步备库，只有最新的领导者和同步备库才能参与领导选举。同步模式确保在故障转移时不会丢失已成功提交的事务，而当Patroni无法确保事务持久性时，会损失写操作的可用性。 

    - postgresql：
      - **use_pg_rewind**:whether or not to use pg_rewind
      - **use_pg_rewind**: 是否使用pg_rewind
      - **use_slots**: whether or not to use replication_slots. Must be False for PostgreSQL 9.3. You should comment out max_replication_slots before it becomes ineligible for leader status.
      - **use_slots**：是否使用复制槽。对于Postgres9.3及以下版本，必须设置为false。你应该在他失去领导地位之前注释掉这个`max_replication_slots `这个参数
      - **recovery_conf**: additional configuration settings written to recovery.conf when configuring follower.
      - **recovery_conf**: 在配置从库加点的时候额外的配置项写入到recovery.conf文件中
      - **parameters**: list of configuration settings for Postgres. Many of these are required for replication to work.
      - **parameters**: 列出Postgres数据库的配置，其中许多都是复制工作必须的

- **method**: custom script to use for bootstrapping this cluster. See custom bootstrap methods documentation &lt;custom\_bootstrap&gt; for details. When `initdb` is specified revert to the default `initdb`
    command. `initdb` is also triggered when no `method` parameter is present in the configuration file.

- **method**: 用于引导此集群的自定义脚本，有关详细信息参见文档《[custom\_bootstrap](./replica_bootstrap.md)》。如果指定了`initdb`，则恢复到默认的`initdb`命令。当配置文件中没有`metbod`参数时，也会触发initdb。 

- **initdb**: List options to be passed on to initdb.

- **initdb**:列出要传递给initdb的选项。 

    - **- data-checksums**: Must be enabled when pg\_rewind is needed on 9.3.
    - **- data-checksums**:在Postgres 9.3版本的数据库中，如果要pg_rewind可用，则必须启用data-checksums

    - **- encoding: UTF8**: default encoding for new databases.
    - **- encoding: UTF8**: 新数据库的默认编码
    - **- locale: UTF8**: default locale for new databases.
    - **- locale: UTF8**: 新数据库的默认字符集

- **pg\_hba**: list of lines that you should add to pg\_hba.conf.

- **pg\_hba**:  你将要添加到pg_hba.conf文件中的所有行

    - **- host all all 0.0.0.0/0 md5**.
    - **- host replication replicator 127.0.0.1/32 md5**: A line like this is required for replication.
    - **- host replication replicator 127.0.0.1/32 md5**: 这样的一行是被复制需要的

- **users**: Some additional users which need to be created after initializing new cluster

- **users**: 在初始化一个新的集群后需要被创建的额外用户

    - **admin**: the name of user 
      - **password: zalando**:
      - **options**: list of options for CREATE USER statement 
        - **- createrole**
        - **- createdb**

- **post\_bootstrap** or **post\_init**: An additional script that will be executed after initializing the cluster. The script receives a connection string URL (with the cluster superuser as a user name). The PGPASSFILE variable is set to the location of pgpass file.

- **post\_bootstrap** or **post\_init**: 初始化集群后将执行的附加脚本。这个脚本接受一个连接字符串URL（以集群超级用户作为用户名）。PGPASSFILE 变量被设置为pgpass文件的位置。

Consul
------

## 前导库

Most of the parameters are optional, but you have to specify one of the **host** or **url**

大多数参数是可选的，但是您必须指定一个主机或url 

-   **host**: the host:port for the Consul endpoint, in format: http(s)://host:port
-   **host：**host:port格式的前导库地址
-   **url**: url for the Consul endpoint
-   **url：**http(s)://host:port格式的前导库地址
-   **port**: (optional) Consul port
-   **port：**（可选择的） 前导库端口
-   **scheme**: (optional) **http** or **https**, defaults to **http**
-   **scheme：** （可选择的） http or https, defaults to http
-   **token**: (optional) ACL token
-   **token：**（可选择的） ACL token
-   **verify**: (optional) whether to verify the SSL certificate for HTTPS requests
-   **verify：**（可选择的） 是否验证HTTPS请求的SSL证书
-   **cacert**: (optional) The ca certificate. If present it will enable validation.
-   **cacert：**（可选择的） CA证书。如果存在，它将启用验证
-   **cert**: (optional) file with the client certificate
-   **cert：**（可选择的） 使用客户端证书归档
-   **key**: (optional) file with the client key. Can be empty if the key is part of **cert**.
-   **key：**（可选择的） 使用客户端密钥归档。如果密钥是证书的一部分，则可以为空
-   **dc**: (optional) Datacenter to communicate with. By default the datacenter of the host is used.
-   **dc：**（可选择的） 要与之通信的数据中心。默认情况下，使用主机的数据中心
-   **checks**: (optional) list of Consul health checks used for the session. If not specified Consul will use "serfHealth" in additional to the TTL based check created by Patroni. Additional checks, in particular the "serfHealth", may cause the leader lock to expire faster than in ttl seconds when the leader instance becomes unavailable
-   **checks：**（可选择的） 前导库健康检查清单。如果未指定，前导库将在Patroni创建的基于TTL的检查之外使用"serfHealth"。其他检查，尤其是"serfHealth"，可能导致领导锁在前导库实例不可用时比ttl秒更快过期。

Etcd
----

Most of the parameters are optional, but you have to specify one of the**host**, **hosts**, **url**, **proxy** or **srv**

大多数参数是可选的，但是您必须指定**host**, **hosts**, **url**, **proxy** or **srv**中的一个

-   **host**: the host:port for the etcd endpoint.
-   **hosts**: list of etcd endpoint in format host1:port1,host2:port2,etc... Could be a comma separated string or an actual yaml list.
-   **url**: url for the etcd
-   **proxy**: proxy url for the etcd. If you are connecting to the etcd using proxy, use this parameter instead of **url**
-   **srv**: Domain to search the SRV record(s) for cluster autodiscovery.
-   **protocol**: (optional) http or https, if not specified http is used. If the **url** or **proxy** is specified - will take protocol from them.
-   **username**: (optional) username for etcd authentication
-   **password**: (optional) password for etcd authentication.
-   **cacert**: (optional) The ca certificate. If present it will enable validation.
-   **cert**: (optional) file with the client certificate
-   **key**: (optional) file with the client key. Can be empty if the key is part of **cert**.

Exhibitor
---------

## 集群列表

-   **hosts**: initial list of Exhibitor (ZooKeeper) nodes in format: 'host1,host2,etc...'. This list updates automatically whenever the Exhibitor (ZooKeeper) cluster topology changes.
-   **hosts**: 按照如下格式罗列初始化节点(ZooKeeper) : ‘host1,host2,etc…’. 每当列表展示集群（zookeeper）拓扑结构发生更改时，此列表都会自动更新
-   **poll\_interval**: how often the list of ZooKeeper and Exhibitor nodes should be updated from Exhibitor
-   **poll\_interval**: ZooKeeper 和集群节点应该多久从Exhibitor被更新一次
-   **port**: Exhibitor port.
-   **port**: 展示端口

Kubernetes
----------

-   **namespace**: (optional) Kubernetes namespace where Patroni pod is running. Default value is default.
-   **namespace：**（可选择的）patroni pod运行的Kubernetes 命名空间。默认值是default。
-   **labels**: Labels in format `{label1: value1, label2: value2}`.These labels will be used to find existing objects (Pods and either Endpoints or ConfigMaps) associated with the current cluster. Also Patroni will set them on every object (Endpoint or ConfigMap) it creates.
-   **labels：** 按照格式`{label1: value1, label2: value2}`的标签。这些标签将用于查找与当前集群关联的现有对象(Pod和Endpoint或ConfigMap)。 此外，Patroni将在它创建的每个对象(Endpoint或ConfigMap)上设置它们。 
-   **scope\_label**: (optional) name of the label containing cluster name. Default value is cluster-name.
-   **scope\_label：**（可选择的）包含群集名称的标签的名称。默认值为群集名
-   **role\_label**: (optional) name of the label containing role (master or replica). Patroni will set this label on the pod it runs in. Default value is `role`.
-   **role\_label：**（可选择的）包含Postgres角色(主或副本)标签的名称。Patroni会在它运行的pod上设置这个标签。默认值是角色
-   **use\_endpoints**: (optional) if set to true, Patroni will use Endpoints instead of ConfigMaps to run leader elections and keep cluster state.
-   **use\_endpoints：**（可选择的）如果设置成true，那么patroni将使用Endpoints而不是ConfigMaps来进行领导锁选举并保持集群状态
-   **pod\_ip**: (optional) IP address of the pod Patroni is running in.This value is required when use\_endpoints is enabled and is used to populate the leader endpoint subsets when the pod's PostgreSQL is promoted.
-   **pod\_ip：**（可选择的）patroni pod的IP地址正在运行。启用Patroni_Kubernetes_USE_Endpoint时需要此值，并用于在提升pod的PostgreSQL时填充前导端点子集

- **ports**: (optional) if the Service object has the name for the port, the same name must appear in the Endpoint object, otherwise ervice won't work. For example, if your service is defined as
    `{Kind: Service, spec: {ports: [{name: postgresql, port: 5432, targetPort: 5432}]}}`,
    then you have to set
    `kubernetes.ports: {[{"name": "postgresql", "port": 5432}]}` and
    Patroni will use it for updating subsets of the leader Endpoint. This parameter is used only if kubernetes.use\_endpoints is set.

- **ports：**（可选择的）如果服务对象有端口名称，那么在端点对象中也必须出现相同的名字，否则的话，服务将不能运行。例如，如果你的服务是这样的：

    `{Kind: Service, spec: {ports: [{name: postgresql, port: 5432, targetPort: 5432}]}}`,

    那么你就要去设置：

    `PATRONI_KUBERNETES_PORTS='{[{"name": "postgresql", "port": 5432}]}'`

    那样，patroni讲使用他来更新领导者端点的子集。只有在设置了 ATRONI\_KUBERNETES\_USE\_ENDPOINTS参数的时候才能使用此参数。

PostgreSQL
----------

- **authentication**:

    - **superuser**: 
      - **username**: name for the superuser, set during initialization (initdb) and later used by Patroni to connect to the postgres.
      - **password**: password for the superuser, set during initialization (initdb).
      - **username：**数据库超级用户名称，在初始化数据库集群时或稍后通过Patroni连接到postgres数据库登陆
      - **password：**超级用户的密码，在初始化集群时设置
    - **replication**: 
      - **username**: replication username; the user will be created during initialization. Replicas will use this user to access master via streaming replication
      - **password**: replication password; the user will be created during initialization.
      - **username：**复制用户名；用户名在初始化时被创建。复制端将使用这个用户经过流复制到主节点
      - **password：**复制用户密码；在初始化过程创建
- **callbacks**: callback scripts to run on certain actions. Patroni will pass the action, role and cluster name. (See scripts/aws.py as an example of how to write them.)
- **callbacks：**在某些操作上运行的回调脚本。 Patroni将传递动作、角色和集群名称（请参阅script/aws.py作为如何编写脚本的示例 ）
    - **on\_reload**: run this script when configuration reload is triggered.
    - **on\_reload：**在配置重新加载时运行此脚本 
    - **on\_restart**: run this script when the cluster restarts.
    - **on\_restart：**在重启集群时运行此脚本。
    - **on\_role\_change**: run this script when the cluster is being promoted or demoted.
    - **on\_role\_change：**在集群角色被提升或降级时候使用
    - **on\_start**: run this script when the cluster starts.
    - **on\_start：**在集群启动时运行这个脚本
    - **on\_stop**: run this script when the cluster stops.
    - **on\_stop：**在集群停止时运行这个脚本
- **connect\_address**: IP address + port through which Postgres is accessible from other nodes and applications.
- **connect\_address：**IP地址+端口格式，通过它可以从其他节点和应用程序访问Postgres
- **create\_replica\_method**: an ordered list of the create methods for turning a Patroni node into a new replica. "basebackup" is the default method; other methods are assumed to refer to scripts, each
    of which is configured as its own config item. See custom replica creation methods documentation &lt;custom\_replica\_creation&gt; for further explanation.
- **create\_replica\_method：**用于将Patroni节点转换为新副本的创建方法的有序列表。“basebackup”是默认的方法。假定其他方法引用脚本，每个脚本都被配置为自己的配置项。更多详细说明参见自定义副本创建方法文档<custom\_replica\_creation&gt;
- **data\_dir**: The location of the Postgres data directory, either existing or to be initialized by Patroni.
- **data\_dir：**已经存在或者由patroni初始化的Postgres数据库数据目录位置
- **config\_dir**: The location of the Postgres configuration directory, defaults to the data directory. Must be writable by Patroni.
- **config\_dir：**Postgres数据库的配置文件存放目录，默认是数据目录。这一定是由Patroni写入的
- **bin\_dir**: Path to PostgreSQL binaries (pg\_ctl, pg\_rewind, pg\_basebackup, postgres). The default value is an empty stringmeaning that PATH environment variable will be used to find the executables.
- **bin\_dir：**Postgres数据库二进制文件（pg\_ctl, pg\_rewind, pg\_basebackup, postgres）路径。默认值是一个空字符串，这意味着环境变量PATH将被用来查找可执行文件 
- **listen**: IP address + port that Postgres listens to; must be accessible from other nodes in the cluster, if you're using streaming replication. Multiple comma-separated addresses are permitted, as long as the port component is appended after to the last one with a colon, i.e. `listen: 127.0.0.1,127.0.0.2:5432`.
    Patroni will use the first address from this list to establish local connections to the PostgreSQL node.
- **listen：**IP地址+postgres监听端口； 如果你使用流复制集群的话，集群内的其他节点必须可达。只要端口组件后面附加了一个冒号就允许使用多个逗号分隔的地址，如：`listen: 127.0.0.1,127.0.0.2:5432`。Patroni将使用此列表中的第一个地址来建立到PostgreSQL节点的本地连接
- **use\_unix\_socket**: specifies that Patroni should prefer to use unix sockets to connect to the cluster. Default value is `false`. If `unix_socket_directories` is definded, Patroni will use first suitable value from it to connect to the cluster and fallback to tcp if nothing is suitable. If `unix_socket_directories` is not specified in `postgresql.parameters`, Patroni will assume that default value should be used and omit `host` from connection parameters.
- **use\_unix\_socket：**指定Patroni应该使用套接字文件连接到集群。默认值是false。如果`unix_socket_directories`被定义，Patroni将使用第一个合适的值来连接到集群，如果不能连接到集群，将回退使用tcp的方式连接。如果`unix_socket_directories`没有定义`postgresql.parameters`，patroni将假定默认值应该被使用并且在连接参数中省略主机
- **pgpass**: path to the[.pgpass](https://www.postgresql.org/docs/current/static/libpq-pgpass.html) password file. Patroni creates this file before executing pg\_basebackup, the post\_init script and under some other circumstances. The location must be writable by Patroni.
- **pgpass：**密码文件.pgpass的路径。Patroni 在执行pg\_basebackup、初始化以及一些其他情况前创建这个文件。这个文件是由Patroni写入的。
- **recovery\_conf**: additional configuration settings written to recovery.conf when configuring follower.
- **recovery\_conf：**当配置从库时一些额外的配置将写入到recovery.conf
- **custom\_conf** : path to an optional custom `postgresql.conf` file, that will be used in place of `postgresql.base.conf`. The file must exist on all cluster nodes, be readable by PostgreSQL and will
    be included from its location on the real `postgresql.conf`. Note that Patroni will not monitor this file for changes, nor backup it. However, its settings can still be overridden by Patroni's own configuration facilities - see dynamic configuration &lt;dynamic\_configuration&gt; for details.
- **custom\_conf：**指定可选文件`postgresql.conf` 的路径，这个文件将被用来替换`postgresql.base.conf`。这个文件必须在集群中所有节点存在，由PostgreSQL读取，并在postgresql.conf包含他的位置。请注意，Patroni不会监视该文件的更改，也不会备份它。 然而，他的配置信息能够被Patroni自己的配置工具重写。通过动态配置 <dynamic\_configuration>章节了解更多详情
- **parameters**: list of configuration settings for Postgres. Many of these are required for replication to work.
- **parameters：**列出Postgres数据库的配置项。其中有许多是复制所必需的。
- **pg\_hba**: list of lines that Patroni will use to generate `pg_hba.conf`. This parameter has higher priority than `bootstrap.pg_hba`. Together with dynamic configuration &lt;dynamic\_configuration&gt; it simplifies management of `pg_hba.conf`.

    - **- host all all 0.0.0.0/0 md5**.
    - **- host replication replicator 127.0.0.1/32 md5**: A line like this is required for replication.
- **pg\_hba：**Patroni将用来生成pg_hba.conf文件的行的列表。 这个参数比`bootstrap.pg_hba`文件的优先级更高。加上动态配置 &lt;dynamic\_configuration&gt; 更方便管理`pg_hba.conf`文件
    - **- host all all 0.0.0.0/0 md5**.
    - **- host replication replicator 127.0.0.1/32 md5**: A line like this is required for replication.
- **pg\_ctl\_timeout**: How long should pg\_ctl wait when doing `start`, `stop` or `restart`. Default value is 60 seconds.
- **pg\_ctl\_timeout：**在做启动、停止、重启操作时pg_ctl应该等待的时间。默认值是60s
- **use\_pg\_rewind**: try to use pg\_rewind on the former leader when it joins cluster as a replica.
- **use\_pg\_rewind：**在旧的领导者作为备库重新加入集群时尝试使用pg_rewind命令。
- **remove\_data\_directory\_on\_rewind\_failure**: If this option is enabled, Patroni will remove postgres data directory and recreate replica. Otherwise it will try to follow the new leader. Default value is **false**.
- **remove\_data\_directory\_on\_rewind\_failure：** 如果启用此选项，Patroni将删除Postgres数据目录并重新创建副本。否则，它将尝试追随新的领导者。默认值为假。 
- **replica\_method**: for each create\_replica\_method other than basebackup, you would add a  configuration section of the same name. At a minimum, this should include "command" with a full path to the actual script to be executed. Other configuration parameters will be passed along to the script in the form "parameter=value".
- **replica\_method：**对于每个不是basebackup的创建副本的方法，你应该添加一个同名的配置节。至少，这应该包含一个有全路径脚本的”命令“被执行。其他的配置参数将以“"parameter=value”的形式传递给脚本

REST API
--------

-   **connect\_address**: IP address and port to access the REST API.
-   **connect\_address：**访问RESTAPI的IP地址和端口
-   **listen**: IP address and port that Patroni will listen to, to provide health-check information for HAProxy.
-   **listen：**Patroni将监听的IP地址和端口，为HAProxy提供健康检查信息
-   **Optional**：

    - **authentication**: 
      - **username**: Basic-auth username to protect unsafe REST API endpoints.
      - **password**: Basic-auth password to protect unsafe REST API endpoints.
    - **certfile**: Specifies the file with the certificate in the PEM format. If the certfile is not specified or is left empty, the API server will work without SSL.
    - **keyfile**: Specifies the file with the secret key in the PEM format.

- **Optional：** 
  - **authentication：**
    - **username：**经过认证的用户名可以保护不安全的REST API端点
    - **password：**经过认证的密码可以保护不安全的REST API端点
  - **certfile：**指定具有PEM格式证书的文件。如果未指定证书或将其保留为空，则API服务器将在没有SSL的情况下工作
  - **keyfile：**指定具有PEM格式密钥的文件。

ZooKeeper
---------

-   **hosts**: list of ZooKeeper cluster members in format:\['host1:port1', 'host2:port2', 'etc...'\].
-   **hosts：**zookeeper集群成员逗号分隔列表：“'host1:port1','host2:port2','etc...'”

Watchdog
--------

-   **mode**: `off`, `automatic` or `required`. When `off` watchdog is disabled. When `automatic` watchdog will be used if available, but ignored if it is not. When `required` the node will not become a leader unless watchdog can be successfully enabled.
-   **mode：**`off`, `automatic` or `required`。配置为`oFf`f时禁用Watchdog，配置为`automatic`时，如果Watchdog可用就启动，不可用就忽略启动。当配置为`required`模式时，除非Watchdog能够成功启用，否则节点将将永远不会成为主库节点
-   **device**: Path to watchdog device. Defaults to `/dev/watchdog`.
-   **device：**Watchdog设备路径。默认是`/dev/watchdog`
-   **safety\_margin**: Number of seconds of safety margin between watchdog triggering and leader key expiration.
-   **safety\_margin：**watchdog触发器和领导锁过期时间的安全阈值时间秒数
