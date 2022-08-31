#### docker
> 已运行 docker run -d -t --name demo ubuntu top 命令, docker exec -it demo kill -9 1 强行给容器内一号进程发KILL信号，容器是否会退出？ 否
> 
> 已运行 docker run -d -t --name demo ubuntu top 和 docker run --name demo-x --pid container:demo ubuntu ps 命令，是否可以在 demo-x 容器内部停止容器？ 是
> 
> 

#### 考察一个应用的架构是不是云原生的标准
> 应用实例能否快速水平扩展
> 
> 应用是否使用镜像机制打包来保证环境一致性
> 
> 应用数据是否都写在容器数据卷中

#### 关于pod的描述
> 一个pod里一个容器是最佳实践(不正确)、一个逻辑概念、多个容器的组合、kubernetes的原子调度单位

#### 两个容器之前的超亲密关系可能包括哪些情况
> 需要运行在同一个宿主机上(错)、直接发生文件减缓、低频率的rpc调用、共享某些linux namespace(错)

#### controller 中的workerqueue 中可以存放什么内容
> namespaceName + podName

#### 在controller 的event handler中，适合执行的操作是
> 根据资源的ownerreference 找到资源的创建者
> 
> 判断资源信息，对于不关心的对象，直接返回
> 
> 在workerqueue中加入资源

#### 当把应用迁移到Kubernetes之后，要如何去保障应用的健康与稳定呢？
其实很简单，可以从两个方面进行增强：
> 首先是提高应用的可观测性；
>
> 第二是提高应用的可恢复能力。

####pod的生命周期
刚开始它处在一个pending的状态，那接下来可能会转换到类似像running,也可能转换到unknown，甚至可以转换到failed，然后，当running执行了一段时间之后，
它可以转换到类似像successded或者是failed,然后当出现在unknown这个状态时,可能有于一些状态的恢复，它会重新恢复到running或者succeeded或者failed。

#### 为什么说statefulset 能较好地满足一些有状态应用特有的需求
> 每个pod有order序号，会按序号创建、删除、更新pod
> 
> 通过配置headless service，使每个pod 有一个唯一的网络标识(hostname)
> 
> 通过配置pvc template, 每个pod 有一块独享的pv存储盘
> 
> 支持一定数量的灰度发布

#### statefulset 扩缩容管理策略
.spec中，有一个字段名为podManagementPolicy,可选策略为
> OrderedReady: 阔所容按照order 顺序执行。扩容时，必须前面序号的pod都ready了,才能扩下一个；缩容时，按照倒叙删除
> 
> parallel：并行扩缩容，不需要等前面pod都ready或删除后再处理下一个

#### statefulset sts.spec.updateStrategy.rollingUpdate 总结
> partition 滚动升级时，保留旧版本pod的数量
> 假设replicas = N， partition  = M, 则最终旧版本pod为[0,M)，新版本pod为[M,n)

#### 可观测性上来讲，可以在三个方面来去做增强
>首先是应用的健康状态上面，可以实时地进行观测；
>
> 第二个是可以获取应用的资源使用情况；
>
> 第三个是可以拿到应用的实时日志，进行问题的诊断与分析。

#### etcd 满足了CAP原理中的哪些特性？
在理论计算机科学中，CAP定理（CAP theorem），又被称作布鲁尔定理（Brewer's theorem），它指出对于一个分布式计算系统来说，不可能同时满足以下三点：

一致性（Consistency） （等同于所有节点访问同一份最新的数据副本）

可用性（Availability）（每次请求都能获取到非错的响应——但是不保证获取的数据为最新数据）

分区容错性（Partition tolerance）（以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。）

根据定理，分布式系统只能满足三项中的两项而不可能满足全部三项。理解CAP理论的最简单方式是想象两个节点分处分区两侧。允许至少一个节点更新状态会导致数据不一致，即丧失了C性质。如果为了保证数据一致性，将分区一侧的节点设置为不可用，那么又丧失了A性质。除非两个节点可以互相通信，才能既保证C又保证A，这又会导致丧失P性质。

>  etcd 满足了 CP

#### client-go 结构

关于object: https://godleon.github.io/blog/Kubernetes/k8s-CoreConcept-ResourceObject-Overview/

#### 挑选一个合适节点作为抢占节点的策略当中，哪个因素是排在第一位的考虑因素？
> 破乖PDB最少的节点


#### devops与sre关系
https://zhuanlan.zhihu.com/p/87598465


#### kubernetes 对扩展资源的硬性限制
> Kubernetes 有一个要求，即扩展资源必须是整数的，所以我们没法申请到 0.5 的 GPU 这样的资源，只能申请 1 个 GPU 或者 2 个 GPU


#### kubernetes对qos（quality of service）的分类
> 服务质量灯了分三类
> 
> 第一类是Guaranteed 它是一类高的qos class，一般用guaranteed来为一些需要资源保证能力的pod进行配置  cpu/memory必须requst=limit，其余资源可不等
> 
> 第二类是Burstable 它其实是中等的一个qos label ，一般会为一些希望有弹性能力的pod来配置burstable  cpu/memory request 和limit 不相等
>  
> 第三类是besteffort， 它是一种尽力而为式的服务质量  所有资源的request/limit必须都不填
>
> 非整数的 Guaranteed/Burstable/BestEffort，它们的 CPU 会放在一块，组成一个 CPU share pool，它会被非整数的 Guaranteed/Burstable/BestEffort 共享，然后它们会根据不同的权重划分时间片来使用 CPU. 需要开启KUBELET的一个特性，cpu-manager-policy=static
> 
> 在 memory 上会按照不同的 Qos 进行划分：OOMScore。比如说 Guaranteed，它会配置默认的 -998 的 OOMScore；Burstable 的话，它会根据内存设计的大小和节点的关系来分配 2-999 的 OOMScore。BestEffort 会固定分配 1000 的 OOMScore，OOMScore 得分越高的话，在物理机出现 OOM 的时候会优先被 kill 掉


#### pod与pod关系
> podAffinity/podAntiAffinity
```yaml
apiVersion: v1
kind: pod
metadata:
  namespace: demo-ns
  name: demo-pod
spec:
  containers:
  - image: nginx:latest
    name: demo-container
  affinitry:
    podAffinity:  # podAntiAffinity
      requiredDuringSchedulingIgnoredDuringExecution:   # 强制   preferredDuringSchedulingIgnoredDuringExecution 优先
      - labelSelector:
        matchExpressions:
        - key: v1
          operator: In    # IN/NotIn/Exists/DoesNotExit
          - v1
      topologyKey: "kubernetes.io/hostname"
```
#### pod与node关系
> nodeSelector / NodeAffinity
````yaml
apiVersion: v1
kind: pod
metadata:
  namespace: demo-node
  name: demo-pod
spec:
  containers:
  - image: nginx:latest
    name: demo-container
  affinitry:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:   # 强制   preferredDuringSchedulingIgnoredDuringExecution 优先
        nodeSelectorTerms:
        - matchExpressions:
          - key: v1
            operator: In  # IN/NotIn/Exists/DoesNotExit/Gt/Lt
            values:
              - v1
              - v2
  nodeSelector:
    k1: v1
````
> node taints / pod tolerations
```yaml
apiVersion: v1
kind: Node
metadata:
  namespace: demo-node
spec:
  taints:
    - key: "k1"
      value: "v1"
      effect: "NoSchedule"
--- 
apiVersion: v1
kind: pod
metadata:
  namespace: demo-ns
  name: demo-pod
spec:
  containers:
  - image: nginx:latest
    name: demo-container
  tolerations:
    - key: "k1"
      operator: "Equal"     # exists / equal
      value: "v1"           # 当op = exists 可为空
      effect: "NoSchedule"  # 可以为空 匹配所有
```
> 目前kubernets 立庙有三个taints 行为:
> 
> NoSchedule 禁止新的pod调度上来；
> 
> PreferNoSchedul 尽量不调度到这台
> 
> NoExecute 会evict 没有对应toleration的pods,并且也不会调度新的上来，这个策略是非常严格的。

#### kubernete高级调度能力
当集群的资源不够，如何做到集群的合理利用？通常的策略有两类：
> 先到先得策略（fifo） 简单，相对公平，上手快
> 
> 优先级策略（priority） 符合日常业务特点，核心资源：PriorityClass pod字段：priorityClassName


#### 调度算法流程
> 通过informer去watch 到需要等待的pod数据，放到队列里面，通过调度算法流程里面，会一直循环从队列里面拿数据，然后经过调度流水线(1、调度器的调度流程 2、wait流程 3、bind流程)。

#### EvenPodsSpread
描述符合条件的一组pod在指定topologyKey上的打散要求
```yaml
spec:
 # 多个之间是and的关系
  topologySpreadConstraints:
  # maxSkew：最大允许不均衡的数量
  - maxSkew: 1
  topologyKey: k8s.io/zone
  # ScheduleAnyway | DoNotSchedule
  whenUnsatisfiable: DoNotSchedule  
  # 描述符合的一组pod
  selector:
    matchLabeles:
      app: foo
    matchExpressions:
      - key: app
        operator: In
        values: ['foo','foo2']
```


#### scheduler extender 支持的扩展点
> filter prioritize  bind

#### 关于打分器(prioritize)的作用描述
> 用来尽量满足pod和node亲和部署
> 
> 支持pod在节点上尽量打散
> 
> 可以用来支持pod尽量调度到已有此镜像的节点

#### kubernetes 通过哪些内部机制支持GPU管理
> device plugin  /  extended plugin

#### kubernetes 挂载volume 过程
![img.png](img.png)

#### pvc 状态迁移
> 创建好一个 PV 以后，我们就处于一个 Available 的状态，当一个 PVC 和一个 PV 绑定的时候，这个 PV 就进入了 Bound 的状态，此时如果我们把 PVC 删掉，Bound 状态的 PV 就会进入 Released 的状态。
>
> 一个 Released 状态的 PV 会根据自己定义的 ReclaimPolicy 字段来决定自己是进入一个 Available 的状态还是进入一个 Deleted 的状态。如果 ReclaimPolicy 定义的是 "recycle" 类型，它会进入一个 Available 状态，如果转变失败，就会进入 Failed 的状态。

#### pvc的状态迁移
> 一个创建好的 PVC 会处于 Pending 状态，当一个 PVC 与 PV 绑定之后，PVC 就会进入 Bound 的状态，当一个 Bound 状态的 PVC 的 PV 被删掉之后，该 PVC 就会进入一个 Lost 的状态。对于一个 Lost 状态的 PVC，它的 PV 如果又被重新创建，并且重新与该 PVC 绑定之后，该 PVC 就会重新回到 Bound 状态。

#### PV与PVC之间的关系
> PV是集群中的资源。
>
> Pvc是对这些资源的请求，也是对资源的索引检查。
> 
> PV和Pvc之间的相互作用遵循这个生命周期：Provisioning(配置)—> Binding(绑定)—>Using(使用)—>Releasing(释放)—>Recycling(回收)
>
> Provisioning，即 PV的创建，可以直接创建PV(静态方式)，也可以使用StorageClass动态创建
>
> Binding，将PV 分配给pvc
>
> using，Pod通过 Pvc使用该Volume，并可以通过准入控制StorageProtection (1.9及以前版本为PVCProtection〉阻止删除正在使用的 Pvc
>
> Releasing，Pod 释放volume并删除pvc
>
> Reclaiming，回收 PV，可以保留PV以便下次使用，也可以直接从云存储中删除


#### 有了Flexvolume， 为何还要csi
> flexvolume只是给kubernetes这一个编排系统来使用的，而csi可以满足不同编排系统的需求，比如mesos,swarm
> 
> 其实csi是容器化部署，可以减少环境一来，增强安全性，丰富插件的功能。
> 
> flexvolume是在host空间的一个二进制,执行 Flexvolum 时相当于执行了本地的一个 shell 命令，这使得我们在安装 Flexvolume 的时候需要同时安装某些依赖，而这些依赖可能会对客户的应用产生一些影响。因此在安全性上、环境依赖上，就会有一个不好的影响。

#### kubernetes 存储架构及插件使用
> 这部分需要反复回味：https://edu.aliyun.com/lesson_1651_18380#_18380 

#### kubernetes crd 在 1.7 版本被引入

#### kubernetes webhook解释
> 本质是一种http回调，会注册到apiserver上，在apiserver特定事件发生时，会查询已注册的webhook，并把相应的信息转发过去。

#### controller 入队逻辑针对可能丢失事件的正确处理方法是什么
> 给相关对象增加finalizer

#### 在controller设计中不可取的方案
> controller 主循环函数不幂等 
>
> 开发的多个mutating webhook 有顺序依赖
> 
> controller 实时更新crd status信息(对)
> 
> validating webhook 依赖 mutating webhook 先执行(对)

#### operator 的书写逻辑
> 观察：通过kubernetes 资源对象变化的事件来获取当前对象状态，只需要注入EventHandler让client-go将变化的事件对象信息放入WorkerQueue中
> 
> 分析：确定当前状态和期望状态的不同，由Worker完成.
> 
> 执行：执行能够驱动对象当前状态变化的操作，由worker完成
> 
> 更新：更新对象的当前状态，由worker完成.

#### 一个容器的包所要解决的问题(网络拓扑)：
> 如何从容器的空间 (c) 跳到宿主机的空间 (infra)
> 
> 如何从宿主机空间到达远端。

#### CNI插件通常有三种实现模式:
> Overlay 模式的典型特征是容器独立于主机的 IP 段，这个 IP 段进行跨主机网络通信时是通过在主机之间创建隧道的方式，将整个容器网段的包全都封装成底层的物理网络中主机之间的包。该方式的好处在于它不依赖于底层网络；
> 
> 路由模式中主机和容器也分属不同的网段，它与 Overlay 模式的主要区别在于它的跨主机通信是通过路由打通，无需在不同主机之间做一个隧道封包。但路由打通就需要部分依赖于底层网络，比如说要求底层网络有二层可达的一个能力；
> 
> Underlay 模式中容器和宿主机位于同一层网络，两者拥有相同的地位。容器之间网络的打通主要依靠于底层网络。因此该模式是强依赖于底层能力的。

> 关于overlay与underlay 可参考的较详细文档：https://www.cnblogs.com/fengdejiyixx/p/15567609.html

#### 在虚拟化的环境中一般选择``Overlay``类型的网络插件

#### underlay 网络模式的特点
> 性能好 
> 
> 可以和集群外资源互联互通

#### flannel-hostGW 方案精髓，是选哪个网卡上的IP，做非本地节点网段的GW？
> Remote-Node-Nic (对端望断选对端node网卡ip做gw)

#### iptables类型的network policy可以通杀一切带Bridge的容器网络数据方案
> 错误 
> 原因：bridge不使用br-call-iptables 功能，则不能起作用。

#### 理解kubernetes api 请求访问控制 
> 谁在何种条件下可以对什么资源做什么操作
> 
> human user/ pod --> authentication(请求用户是否能够访问集群的合法用户) / 认证--> authorization(用户是否有优先进行请求中的操作) / 授权  --> admissioncontrol(请求是否安全合规) / 准入控制--> kubernetes objects

#### service account 是 kubernetes 中唯一能够通过api 方式管理的apiservice 访问凭证

#### 角色中的verbs 设置
> list get create watch delete patch 


#### 创建子账号的kubeconfig
demo: 授予zisefeizhu用户对zisefeizhu命名空间下的pod的查询和删除操作的权限
```shell
// 创建zisefeizhu ns
ctl create ns zisefeizhu

// 创建sa 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: zisefeizhu-sa
  namespace: zisefeizhu
  
// 创建role
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: zisefeizhu-role
  namespace: zisefeizhu
rules:
- apiGroups: [""]
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
  - delete
  
// 创建rolebinding
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: zisefeizhu-binding
  namespace: zisefeizhu
subjects:
- kind: ServiceAccount
  name: zisefeizhu-sa
roleRef:
  kind: Role
  name: zisefeizhu-role
  apiGroup: rbac.authorization.k8s.io

// kubeconfig 模版
apiVersion: v1
kind: Config
users:
- name: $USER
  user:
    token: $TOKEN_DECODE
clusters:
- cluster:
    certificate-authority-data: $CLUSTER_AUTH
    server: $KUBE_APISERVER
  name: $USER-cluster
contexts:
- context:
    cluster: $USER-cluster
    namespace: $USER
    user: $USER
  name: $USER-cluster
current-context: $USER-cluster

// 生成kubeconfig
KUBE_APISERVER=`kubectl config view --minify -o=jsonpath="{.clusters[*].cluster.server}"`
TOKEN_KEY=`kubectl get sa zisefeizhu-sa -n zisefeizhu -o=jsonpath="{.secrets[0].name}"`
TOKEN=`kubectl get secrets $TOKEN_KEY -n zisefeizhu -o=jsonpath="{.data.token}"`
CLUSTER_AUTH=`kubectl config view --flatten --minify -o=jsonpath="{.clusters[0].cluster.certificate-authority-data}"`
TOKEN_DECODE=`echo $TOKEN | base64 --decode`  # 注意点
```

#### pauseContainer 不是一个cri 接口，因为kubernetes 不支持pause 语义

#### runtimeclass 
> runtimeclass 对象代表了一个容器运行时，其结构体主要包含handler,overhead,scheduling三个字段
> overhead 1.16引入 表示pod中的业务运行所需要资源以外的额外开销
> overhead 主要使用场景：pod调度/resourceQuota/kubelet pod 驱逐

#### Pod Overhead 不支持手动配置或更改。

#### 疑问：创建statefulset的headless service 时，是否明确serivce 要先与statefulset创建???

#### 较有意思的cka/ckad 真题 
21年考这俩证的时候 其实也是刷题的，毕竟这和八股文查不了多少 ～～
> https://www.cnblogs.com/fengdejiyixx/p/15602074.html

> https://www.cnblogs.com/peteremperor/p/12785335.html