Environment Configuration Settings
==================================

It is possible to override some of the configuration parameters defined in the Patroni configuration file using the system environment variables. This document lists all environment variables handled by Patroni. The values set via those variables always take precedence over the ones set in the Patroni configuration file.

# Environment Configuration Settings

可以使用环境变量来重写Patroni配置文件中定义的一些配置参数。本文档列出了Patroni操作的所有环境变量。通过这些变量设置的值总是优先于Patroni配置文件中设置的值。

Global/Universal
----------------

# 全局变量

-   **PATRONI\_CONFIGURATION**: it is possible to set the entire configuration for the Patroni via `PATRONI_CONFIGURATION` environment variable. In this case any other environment variables
    will not be considered!
-   **PATRONI\_NAME**: name of the node where the current instance of Patroni is running. Must be unique for the cluster.
-   **PATRONI\_NAMESPACE**: path within the configuration store where Patroni will keep information about the cluster. Default value:"/service"
-   **PATRONI\_SCOPE**: cluster name
-   **PATRONI\_LOGLEVEL**: sets the general logging level (see [the docs
    for Pythonlogging](https://docs.python.org/3.6/library/logging.html#levels))
-   **PATRONI\_REQUESTS\_LOGLEVEL**: sets the logging level for all HTTP requests e.g. Kubernetes API calls (see [the docs for Python logging](https://docs.python.org/3.6/library/logging.html#levels))
-   PATRONI_CONFIGURATION: 可以通过Patroni_Configuration环境变量为Patroni设置整个配置。在这种情况下，将不考虑任何其他环境变量
-   PATRONI_NAME：当前正在运行的Patroni示例节点名称，这个名称在集群中必须是唯一的
-   PATRONI_NAMESPACE：Patroni配置的保存有关群集信息的存储路径。默认值为“/service”
-   PATRONI_SCOPE：集群名称
-   PATRONI_LOGLEVEL：设置一般日志记录级别(请参见Python日志记录的文档：https://docs.python.org/3.6/library/logging.html#logging-levels)
-   PATRONI_REQUESTS_LOGLEVEL：设置所有HTTP请求的日志记录级别，例如Kubernetes API调用（请参见Python日志记录的文档：https://docs.python.org/3.6/library/logging.html#logging-levels）

Bootstrap configuration
-----------------------

It is possible to create new database users right after the successful
initialization of a new cluster. This process is defined by the
following variables:

## 引导配置

在成功初始化一个新的集群之后创建一个数据库用户是可以的。此过程由以下变量定义：

-   **PATRONI\_&lt;username&gt;\_PASSWORD='&lt;password&gt;'**
-   **PATRONI\_&lt;username&gt;\_OPTIONS='list,of,options'**

Example: defining `PATRONI_admin_PASSWORD=strongpasswd` and
`PATRONI_admin_OPTIONS='createrole,createdb'` will cause creation of the
user **admin** with the password **strongpasswd** that is allowed to
create other users and databases.

例如：定义 “PATRONI_admin_PASSWORD=strongpasswd” 和 “PATRONI_admin_OPTIONS='createrole,createdb'” 将会创建一个新用户“admin”，他的密码为 “strongpasswd”。

Consul
------

## 前导库

-   **PATRONI\_CONSUL\_HOST**: the host:port for the Consul endpoint.
-   **PATRONI\_CONSUL\_URL**: url for the Consul, in format:
    http(s)://host:port
-   **PATRONI\_CONSUL\_PORT**: (optional) Consul port
-   **PATRONI\_CONSUL\_SCHEME**: (optional) **http** or **https**,
    defaults to **http**
-   **PATRONI\_CONSUL\_TOKEN**: (optional) ACL token
-   **PATRONI\_CONSUL\_VERIFY**: (optional) whether to verify the SSL
    certificate for HTTPS requests
-   **PATRONI\_CONSUL\_CACERT**: (optional) The ca certificate. If
    present it will enable validation.
-   **PATRONI\_CONSUL\_CERT**: (optional) File with the client
    certificate
-   **PATRONI\_CONSUL\_KEY**: (optional) File with the client key. Can
    be empty if the key is part of certificate.
-   **PATRONI\_CONSUL\_DC**: (optional) Datacenter to communicate with.
    By default the datacenter of the host is used.
-   **PATRONI\_CONSUL\_CHECKS**: (optional) list of Consul health checks
    used for the session. If not specified Consul will use "serfHealth"
    in additional to the TTL based check created by Patroni. Additional
    checks, in particular the "serfHealth", may cause the leader lock to
    expire faster than in ttl seconds when the leader instance becomes
    unavailable.
-   PATRONI_CONSUL_HOST: host:port格式的前导库地址
-   PATRONI_CONSUL_URL: http(s)://host:port格式的前导端地址
-   PATRONI_CONSUL_PORT: （可选择的） 前导库端口
-   PATRONI_CONSUL_SCHEME: （可选择的） http or https, defaults to http
-   PATRONI_CONSUL_TOKEN: （可选择的） ACL token
-   PATRONI_CONSUL_VERIFY: （可选择的） 是否验证HTTPS请求的SSL证书
-   PATRONI_CONSUL_CACERT: （可选择的） CA证书。如果存在，它将启用验证
-   PATRONI_CONSUL_CERT: （可选择的） 使用客户端证书归档
-   PATRONI_CONSUL_KEY: （可选择的） 使用客户端密钥归档。如果密钥是证书的一部分，则可以为空
-   PATRONI_CONSUL_DC: （可选择的） 要与之通信的数据中心。默认情况下，使用主机的数据中心
-   PATRONI_CONSUL_CHECKS: （可选择的） 前导库健康检查清单。如果未指定，前导库将在Patroni创建的基于TTL的检查之外使用"serfHealth"。其他检查，尤其是"serfHealth"，可能导致领导锁在前导库实例不可用时比ttl秒更快过期。

Etcd
----

- **PATRONI\_ETCD\_HOST**: the host:port for the etcd endpoint.

- **PATRONI\_ETCD\_HOSTS**: list of etcd endpoints in format
    host1:port1,host2:port2,etc...

- **PATRONI\_ETCD\_URL**: url for the etcd, in format:
    http(s)://(username:password@)host:port

- **PATRONI\_ETCD\_PROXY**: proxy url for the etcd. If you are
    connecting to the etcd using proxy, use this parameter instead of
    **PATRONI\_ETCD\_URL**

- **PATRONI\_ETCD\_SRV**: Domain to search the SRV record(s) for
    cluster autodiscovery.

- **PATRONI\_ETCD\_CACERT**: The ca certificate. If present it will
    enable validation.

- **PATRONI\_ETCD\_CERT**: File with the client certificate

- **PATRONI\_ETCD\_KEY**: File with the client key. Can be empty if
    the key is part of certificate.

- PATRONI_ETCD_HOST: host:port格式的etcd地址

- PATRONI_ETCD_HOSTS:按着host1:port1,host2:port2的格式罗列etcd地址

- PATRONI_ETCD_URL: http(s)://(username:password@)host:port格式的etcd的url连接串

- PATRONI_ETCD_PROXY: etcd的proxy地址. 如果你使用proxy连接etcd, 要使用PATRONI_ETCD_PROXY参数而不是使用PATRONI_ETCD_URL参数

- PATRONI_ETCD_SRV: 集群自动发现服务记录的搜索域。

- PATRONI_ETCD_CACERT: CA证书。如果存在，它将启用验证

- PATRONI_ETCD_CERT: 客户端认证文件

- PATRONI_ETCD_KEY: 使用客户端秘钥的文件。如果密钥是证书的一部分，则可以为空

    

Exhibitor
---------

## 集群列表

-   **PATRONI\_EXHIBITOR\_HOSTS**: initial list of Exhibitor (ZooKeeper) nodes in format: 'host1,host2,etc...'. This list updates automatically whenever the Exhibitor (ZooKeeper) cluster topology changes.
-   **PATRONI\_EXHIBITOR\_PORT**: Exhibitor port.
-   PATRONI_EXHIBITOR_HOSTS: 按照如下格式罗列初始化节点(ZooKeeper) : ‘host1,host2,etc…’. 每当列表展示集群（zookeeper）拓扑结构发生更改时，此列表都会自动更新
-   PATRONI_EXHIBITOR_PORT: 展示端口

Kubernetes
----------

- **PATRONI\_KUBERNETES\_NAMESPACE**: (optional) Kubernetes namespace where the Patroni pod is running. Default value is default.

- **PATRONI\_KUBERNETES\_LABELS**: Labels in format `{label1: value1, label2: value2}`. These labels will be used to find existing objects (Pods and either Endpoints or ConfigMaps) associated with the current cluster. Also Patroni will set them on every object (Endpoint or ConfigMap) it creates.

- **PATRONI\_KUBERNETES\_SCOPE\_LABEL**: (optional) name of the label containing cluster name. Default value is cluster-name.

- **PATRONI\_KUBERNETES\_ROLE\_LABEL**: (optional) name of the label containing Postgres role (master or replica). Patroni will set this label on the pod it is running in. Default value is role.

- **PATRONI\_KUBERNETES\_USE\_ENDPOINTS**: (optional) if set to true, Patroni will use Endpoints instead of ConfigMaps to run leader elections and keep cluster state.

- **PATRONI\_KUBERNETES\_POD\_IP**: (optional) IP address of the pod Patroni is running in. This value is required when PATRONI\_KUBERNETES\_USE\_ENDPOINTS is enabled and is used to populate the leader endpoint subsets when the pod's PostgreSQL is promoted.

- **PATRONI\_KUBERNETES\_PORTS**: (optional) if the Service object has the name for the port, the same name must appear in the Endpoint object, otherwise service won't work. For example, if your service
    is defined as
    `{Kind: Service, spec: {ports: [{name: postgresql, port: 5432, targetPort: 5432}]}}`,
    then you have to set
    `PATRONI_KUBERNETES_PORTS='{[{"name": "postgresql", "port": 5432}]}'`
    and Patroni will use it for updating subsets of the leader Endpoint. This parameter is used only if  ATRONI\_KUBERNETES\_USE\_ENDPOINTS is set.

- 

- **PATRONI\_KUBERNETES\_NAMESPACE**: （可选择的）patroni pod运行的Kubernetes 命名空间。默认值是default。

- **PATRONI\_KUBERNETES\_LABELS**：按照格式`{label1: value1, label2: value2}`的标签。这些标签将用于查找与当前集群关联的现有对象(Pod和Endpoint或ConfigMap)。 此外，Patroni将在它创建的每个对象(Endpoint或ConfigMap)上设置它们。 

- **PATRONI\_KUBERNETES\_SCOPE\_LABEL**: （可选择的）包含群集名称的标签的名称。默认值为群集名 

- **PATRONI\_KUBERNETES\_ROLE\_LABEL**：（可选择的）包含Postgres角色(主或副本)标签的名称。Patroni会在它运行的pod上设置这个标签。默认值是角色。 

- **PATRONI\_KUBERNETES\_USE\_ENDPOINTS**: （可选择的）如果设置成true，那么patroni将使用Endpoints而不是ConfigMaps来进行领导锁选举并保持集群状态。

- **PATRONI\_KUBERNETES\_POD\_IP**:（可选择的）patroni pod的IP地址正在运行。启用Patroni_Kubernetes_USE_Endpoint时需要此值，并用于在提升pod的PostgreSQL时填充前导端点子集。 

- **PATRONI\_KUBERNETES\_PORTS**:（可选择的）如果服务对象有端口名称，那么在端点对象中也必须出现相同的名字，否则的话，服务将不能运行。例如，如果你的服务是这样的：

    `{Kind: Service, spec: {ports: [{name: postgresql, port: 5432, targetPort: 5432}]}}`,

    那么你就要去设置：

    `PATRONI_KUBERNETES_PORTS='{[{"name": "postgresql", "port": 5432}]}'`

    那样，patroni讲使用他来更新领导者端点的子集。只有在设置了 ATRONI\_KUBERNETES\_USE\_ENDPOINTS参数的时候才能使用此参数。

PostgreSQL
----------

-   **PATRONI\_POSTGRESQL\_LISTEN**: IP address + port that Postgres listens to. Multiple comma-separated  addresses are permitted, as long as the port component is appended after to the last one with a colon, i.e. `listen: 127.0.0.1,127.0.0.2:5432`. Patroni will use the first address from this list to establish local connections to the PostgreSQL node.
-   **PATRONI\_POSTGRESQL\_CONNECT\_ADDRESS**: IP address + port through which Postgres is accessible from other nodes and applications.
-   **PATRONI\_POSTGRESQL\_DATA\_DIR**: The location of the Postgres data directory, either existing or to be initialized by Patroni.
-   **PATRONI\_POSTGRESQL\_CONFIG\_DIR**: The location of the Postgres configuration directory,  defaults to the data directory. Must be writable by Patroni.
-   **PATRONI\_POSTGRESQL\_BIN\_DIR**: Path to PostgreSQL binaries. (pg\_ctl, pg\_rewind, pg\_basebackup, postgres) The default value is an empty string meaning that PATH environment variable will be used
    to find the executables.
-   **PATRONI\_POSTGRESQL\_PGPASS**: path to the [.pgpass(https://www.postgresql.org/docs/current/static/libpq-pgpass.html) password file. Patroni creates this file before executing pg\_basebackup and under some other circumstances. The location must be writable by Patroni.
-   **PATRONI\_REPLICATION\_USERNAME**: replication username; the user will be created during initialization. Replicas will use this user to access master via streaming replication
-   **PATRONI\_REPLICATION\_PASSWORD**: replication password; the user will be created during initialization.
-   **PATRONI\_SUPERUSER\_USERNAME**: name for the superuser, set during initialization (initdb) and later used by Patroni to connect to the postgres. Also this user is used by pg\_rewind.
-   **PATRONI\_SUPERUSER\_PASSWORD**: password for the superuser, set during initialization (initdb).
-   **PATRONI\_POSTGRESQL\_LISTEN**: IP地址+postgres监听端口。只要端口组件后面附加了一个冒号就允许使用多个逗号分隔的地址，如：`listen: 127.0.0.1,127.0.0.2:5432`。Patroni将使用此列表中的第一个地址来建立到PostgreSQL节点的本地连接
-   **PATRONI\_POSTGRESQL\_CONNECT\_ADDRESS：**IP地址+端口格式，通过它可以从其他节点和应用程序访问Postgres
-   **PATRONI\_POSTGRESQL\_DATA\_DIR**：已存在或由Patroni初始化的Postgres数据目录的位置
-   **PATRONI\_POSTGRESQL\_CONFIG\_DIR**：数据库配置文件目录位置，默认是数据目录。他必须是由Patroni写入的
-   **PATRONI\_POSTGRESQL\_BIN\_DIR**：Postgres数据库二进制文件目录。（pg_ctl, pg_rewind, pg_basebackup, postgres）默认值为空字符串，表示将使用PATH环境变量找到可执行文件 
-   **PATRONI\_POSTGRESQL\_PGPASS**：密码文件（[.pgpass](https://www.postgresql.org/docs/current/static/libpq-pgpass.html)）的文件路径。Patroni在执行 pg_basebackup及其他的一些情况下会创建此文件。该目录patroni必须有写入权限。

REST API
--------

-   **PATRONI\_RESTAPI\_CONNECT\_ADDRESS**: IP address and port to access the REST API.
-   **PATRONI\_RESTAPI\_LISTEN**: IP address and port that Patroni will listen to, to provide health-check information for HAProxy.
-   **PATRONI\_RESTAPI\_USERNAME**: Basic-auth username to protect unsafe REST API endpoints.
-   **PATRONI\_RESTAPI\_PASSWORD**: Basic-auth password to protect unsafe REST API endpoints.
-   **PATRONI\_RESTAPI\_CERTFILE**: Specifies the file with the certificate in the PEM format. If the certfile is not specified or is left empty, the API server will work without SSL.
-   **PATRONI\_RESTAPI\_KEYFILE**: Specifies the file with the secret key in the PEM format.
-   **PATRONI\_RESTAPI\_CONNECT\_ADDRESS**：访问RESTAPI的IP地址和端口
-   **PATRONI\_RESTAPI\_LISTEN**：Patroni将监听的IP地址和端口，为HAProxy提供健康检查信息
-   **PATRONI\_RESTAPI\_USERNAME**：经过认证的用户名可以保护不安全的REST API端点
-   **PATRONI\_RESTAPI\_PASSWORD**：经过认证的密码可以保护不安全的REST API端点
-   **PATRONI\_RESTAPI\_CERTFILE**：指定具有PEM格式证书的文件。如果未指定证书或将其保留为空，则API服务器将在没有SSL的情况下工作
-    **PATRONI\_RESTAPI\_KEYFILE**: 指定具有PEM格式密钥的文件。 

ZooKeeper
---------

-   **PATRONI\_ZOOKEEPER\_HOSTS**: comma separated list of ZooKeeper cluster members: "'host1:port1','host2:port2','etc...'". It is important to quote every single entity!
-   **PATRONI\_ZOOKEEPER\_HOSTS**：zookeeper集群成员逗号分隔列表：“'host1:port1','host2:port2','etc...'”引用每一个实体是很重要的 

