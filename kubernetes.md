Using Patroni with Kubernetes
=============================

# 将Patroni和Kubernate结合使用

Patroni can use Kubernetes objects in order to store the state of the cluster and manage the leader key. That makes it capable of operating Postgres in Kubernetes environment without any consistency store,namely, one doesn't need to run an extra Etcd deployment. There are two different type of Kubernetes objects Patroni can use to store the leader and the configuration keys, they are configured with the kubernetes.use\_endpoints or PATRONI\_KUBERNETES\_USE\_ENDPOINTS environment variable.

Patroni可以使用Kubernate对象来存储集群的状态并管理领导键。这使得它能够在Kubernetes环境中运行Postgres而无需任何一致性存储，即不需要运行额外的etcd部署。这使得它能够在Kubernetes环境中运行Postgres而无需任何一致性存储，即不需要运行额外的etcd部署。atroni可以使用两种不同类型的Kubernetes对象来存储前导符和配置键，它们是用kubernetes.use_endpoints或PATRONI_KUBERNETES_USE_ENDPOINTS环境变量配置的。

Use Endpoints
-------------

# 使用端点

Despite the fact that this is the recommended mode, it is turned off by default for compatibility reasons. When it is on, Patroni stores the cluster configuration and the leader key in the metadata: annotations
fields of the respective Endpoints it creates. Changing the leader is safer than when using ConfigMaps, since both the annotations, containing the leader information, and the actual addresses pointing to the running leader pod are updated simultaneously in one go.

尽管这种模式是被推荐使用的，但是出于兼容性的原因，他在默认情况下是关闭的。当我们把他打开时，Patroni将集群配置和领导键存储在元数据：它创建的各个端点的注释字段中。修改集群的leader比使用配置图（Use ConfigMaps）更加安全，因为包含leader信息的注释和指向正在运行的leader程序的实际地址同时更新。

Use ConfigMaps
--------------

# 使用配置图

In this mode, Patroni will create ConfigMaps instead of Endpoints and store keys inside meta-data of those ConfigMaps. Changing the leader takes at least two updates, one to the leader ConfigMap and another to
the respective Endpoint.

There are two ways to direct the traffic to the Postgres master:

-   use the [callback script](https://github.com/zalando/patroni/blob/master/kubernetes/callback.py) provided by Patroni
-   configure the Kubernetes Postgres service to use the label selector with the role\_label (configured in patroni configuration).

Note that in some cases, for instance, when running on OpenShift, there is no alternative to using ConfigMaps.

在这种模式下，Patroni将创建配置图而不是端点（Endpoints），并将键值存储在配置图的元数据中。修改leader至少需要两次更新，一次是到leader的配置图，另一次是到各自的端点。

这有两种方式到Postgres主节点：

- 使用Patroni提供的callback脚本
- 将Kubernete Postgres服务配置为使用带有role_label标签的选择器（在Patroni的配置文件配置）

请注意，在某些情况下，例如，在OpenShift上运行时，除了使用ConfigMap之外，没有其他选择。

Configuration
-------------

Patroni Kubernetes settings &lt;kubernetes\_settings&gt; and environment variables &lt;kubernetes\_environment&gt; are described in the general chapters of the documentation.

Patroni Kubernetes的设置和环境变量在文档的章节中进行了描述。

Examples
--------

-   The[kubernetes](https://github.com/zalando/patroni/tree/master/kubernetes) folder of the Patroni repository contains examples of the Docker image, the Kubernetes manifest and the callback script in order to test Patroni Kubernetes setup. Note that in the current state it will not be able to use PersistentVolumes because of permission issues.
-   You can find the full-featured Docker image that can use Persistent Volumes in the [Spilo Project](https://github.com/zalando/spilo).
-   There is also a [Helm chart](https://github.com/kubernetes/charts/tree/master/incubator/patroni) to deploy the Spilo image configured with Patroni running using Kubernetes.
-   In order to run your database clusters at scale using Patroni and Spilo, take a look at the [postgres-operator](https://github.com/zalando-incubator/postgres-operator)
    project. It implements the operator pattern to manage Spilo clusters.
-   Patroni存储库的[Kubernetes](https://github.com/zalando/patroni/tree/master/kubernetes)文件夹下包含Docker映像、Kubernetes清单和回调脚本的示例，以便测试Patroni Kubernetes设置。请注意，在当前状态下，由于权限问题，它将无法使用PersistentVolume。 
-   您可以在[Spilo项目](https://github.com/zalando/spilo)中找到可以使用持久卷的全功能Docker映像。 
-   还有一个配置了Patroni部署在Spilo镜像使用Kubernetes运行的[Helm图表](https://github.com/kubernetes/charts/tree/master/incubator/patroni) 
-   为了使用Patroni和Spilo大规模运行数据库集群，请查看[Postgres-Operator](https://github.com/zalando-incubator/postgres-operator)项目。它实现了管理Spilo集群的操作符模式。 
