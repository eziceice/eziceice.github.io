---
title: Kubernetes
date: 2020-05-18 12:32:45
tags: kubernetes
categories:
- service management
---
# 1. Introduction

## 1.1 基本概念

- Kubernetes内的所有资源都可以通过`kubectl`来执行增/删/查/改等操作并且把资源的状态保存在etcd中。
- Kubernetes通过跟踪**对比etcd里保存的资源期望状态与当前环境里的实际资源状态的差异**来实现自动控制和自动纠错等高级功能。

### 1.1.1 Master

- 集群控制节点，在每一个Kubernetes的集群里都需要有一个Master来负责整体集群的控制和管理，基本所有的Kubernetes的所有控制命令都发给它，负责具体的执行过程。
  Master上的关键进程:
    - kube-apiserver: 提供REST的关键服务进程，集群控制的唯一入口。
    - kube-controller-manager: 所有资源对象的自动化控制中心，可理解为大总管。
    - kube-scheduler: 负责资源调度的进程(POD调度)， 相当于调度室。
    - etcd: 所有资源对象的数据保存中心

### 1.1.2 Node

- 集群中的工作负载节点，每个Node都会被Master分配一些工作负载，当某个Node宕机时，其上的工作负载会被Master转移到其他节点上。
  Node上的关键进程:
    - kubelet: 负责pod对应的容器的创建，启动停止等任务，同时与Master密切协作，实现集群管理的基本功能。
    - kube-proxy：实现kubernetes service的通信与负载均衡机制的重要组件。
    - docker：Docker引擎，负责容器的创建和管理。
- kubelet会向Master进行注册，并且定期向Master汇报自身的情况，使得Master可以根据当前信息来对集群进行管理。

### 1.1.3 Pod

![Pod][1]
- Kubernetes中最小的资源单位，每个pod中除了多个业务容器之外，还有一个根容器Pause容器。
  为什么需要Pause容器？
    - 对整个Pod的状态进行判断，使用对于业务无关并且不易死亡的Pause容器更有效。
    - 其他业务容器共享Pause容器的网络和挂载的Volume，简化了容器间的通信和文件共享的问题。
- Kubernetes为每一个Pod都分配了唯一的IP地址，一个Pod里的多个容器共享Pod IP地址。
- **一个Pod里的容器与另外主机上的Pod容器能够直接通信**。
- 两种Pod类型:
    - 普通的Pod: 一旦被创建，状态就会被放入etcd中存储，然后调度到某个Node上进行binding，随后该Pod会被对应Node上的kubelet进程实例化成相关的docker容器并启动。
    - Static Pod: 并没有放在etcd中，而是被放在Node上的一个具体文件中，并且只能在Node上启动和运行。
- Pod中的资源限额有CPU和Memory两种，资源单位都是绝对值而非相对值。
    - Requests: 该资源的最小申请量，如果想使用autoscalar则必须定义requests。
    - Limits：该资源最大允许使用的量，不能被突破。

### 1.1.4 Label

- Label可以被附加到各种资源对象上。Label通常在资源对象定义时确定，也可以在对象创建后动态添加或者删除。
- Label Selector当前支持:
    - Equality-based，等式表达式匹配 = or !=
    - Set-based，集合表达式匹配 in or not in
- matchLabels用于定义一组Label，与直接写在Selector中的作用相同；matchExpressions用于定义一组基于集合的筛选条件，可用的条件运算符包括In/NotIn/Exists/DoesNotExists。
- matchLabels和matchExpressions为AND关系，即必须同时满足才能完成筛选。

### 1.1.5 ReplicationController

- RC是用来声明某种Pod的数量在任意时刻都符合某个预期值。
    - Pod期待的instance数量。
    - 用于筛选目标Pod的Label。
    - 当Pod的副本数量小于预期数量时，用于创建新Pod的Pod template。
- 删除RC并不会影响已经通过RC创建好的Pod。
- RollingUpdate: 旧版本的Pod每停止一个，新版本的Pod就创建一个，当所有Pod都是新版本时，升级完成。

### 1.1.6 Deployment

- Deployment更好的解决了Pod的编排的问题，内部使用Replica Set实现的。
- Deployment相对于RC的一个最大的升级就是可以随时知道当前Pod部署的进度。

### 1.1.7 Horizontal Pod Autoscaler

- 当前HPA有两种方式作为Pod负载的度量指标：
    - CPUUtilizationPercentage: 一个Pod自身的CPU利用率是该Pod当前CPU的使用量除以它的Pod Request的值。**如果目标Pod没有定义Pod Request，则无法使用CPUUtilizationPercentage进行扩容**。**CPU的使用值通常是1min内的平均值**。
    - 自定义的程序指标

### 1.1.8 StatefulSet

- 对于有状态的服务类型，需要使用StatefulSet，比如复杂的中间件或者数据库集群。
  StatefulSet的特点：
    - 每个节点都有固定的身份ID，通过这个ID，集群中的成员可以互相发现和通信。
    - 集群的规模通常是固定的，不能随意变动。
    - 集群中的每个节点都是有状态的，通常会持久化数据到永久存储中。
    - 如果磁盘损坏，某个节点无法正常运行，集群功能受损。
    - Pod的启停顺序是受控制的，操作第N个Pod的时候，前N-1个Pod一定是运行且准备好的状态。
- StatefulSet通常和Headless Service配合使用，为每个Pod实例都创建一个DNS域名。这些DNS可以直接在集群的配置文件中固定下来。

### 1.1.9 Service

- Kubernetes中的服务发现是通过DNS系统来实现的。
- Service一旦创建，Kubernetes就会自动分配一个可用的Cluster IP，而且在Service的生命周期内，它的Cluster IP不会发生变动，因此Cluster IP和Name组成一个DNS域名映射便可以完美解决服务发现的问题。
- Kubernetes有三种IP:
    - Node IP: Node的IP地址。
        - 集群中每个节点的物理网卡的IP地址，是一个真实存在的物理网络。所有属于这个网络的服务器都能够通过这个网络直接通信，不管其中是否有部分节点不属于这个集群。**这也表明在集群之外的节点访问集群之内的节点必须通过Node IP来通信**。
    - Pod IP: Pod的IP地址。
        - Docker Engine根据docker0网桥的IP地址段进行分配的，通常是一个虚拟的二层网络。
    - Service Cluster IP: 虚拟的Cluster IP地址。
        - 仅仅用于Service对象，由Kubernetes管理和分配IP地址。
        - Cluster IP无法被Ping，因为没有实体网络对象来响应。
        - Cluster IP只能结合Service Port组成一个具体的通信端口，单独的Cluster IP不具备TCP/IP通信的基础，如果想要从集群外访问需要做一些额外的工作。
- NodePort的实现方式是在Kubernetes集群里的每个Node上都为需要外部访问的Service开启一个对应的TCP监听端口，外部系统只要用任意一个Node的IP+具体的Port端口号就可以访问此服务。
- LoadBalancer可以对接到外部的LoadBalancer从而实现集群外对集群内的服务访问。

### 1.1.10 Job

- 并行或者川行启动多个计算进程去处理一项工作。
- Job所控制的Pod副本是短暂运行的并且无法自动重启的。

### 1.1.11 Volume

- Volume与Pod的生命周期相同，但与容器的生命周期不想关，因为Volume是mount在Pause容器上的。常见的几种Volume类型：
    - emptyDir: Pod分配到Node时创建的，和Pod的生命周期相关，常见用途：
        - 临时空间。
        - 长时间任务中的CheckPoint的临时保存目录。
        - 多容器目录共享。
    - hostPath: 挂载在主机上的文件或者目录。
        - 可永久存储在主机上。
    - gcePersistentDisk/awsElasticBlockStore: cloud provider提供的存储目录。
    - NFS： 网络文件系统

### 1.1.12 Persistent Volume

- 提前被定义的网络存储，不是定义在Pod上，而是挂载在虚拟机上。
- PV只能是网络储存，不属于任何Node，但可以被每个Node访问。
- PV独立于Pod之外。
- PV是有状态的对象：
    - Available：空闲可用。
    - Bound：绑定到某个PVC上。
    - Released：对应的PVC已经被删除，资源还没有被集群回收。
    - Failed：PV自动回收失败。

### 1.1.13 Namespace

- 用来实现多租户的资源隔离，通过将集群内部的资源对象“分配”到不同的Namespace中，形成逻辑上分组的不同项目，小组或者用户组，便于不同的分组在共享整个集群的资源的同时还能被分别管理。

### 1.1.14 Annotation

- 和Tag使用方法类似。

### 1.1.15 ConfigMap

- 通过Docker Volume将容器外的配置文件映射到容器内。

![ConfigMap][2]


[1]:https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/PausePod.png
[2]:https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/Kubernetes-ConfigMap.png

# 2. 常用命令

## 2.1 Container Runtime Interface(CRI)

![CRI][1]

- kubelet的职责在于通过RPC管理容器的生命周期，实现容器生命周期的钩子，存活和健康监测，以及执行Pod的重启策略，保证容器应用的实际状态和生命状态的一致性。
-


## 2.2 Kubectl

 ```bash
   # RUN Shell In POD
   kubectl exec POD [-c CONTAINER][-i][-t][flags][-- COMMAND [args...]]
   # Expose a service
   kubectl expose
   # Foward a local port traffic to pod
   kubectl port-foward POD [LOCAL_PORT]:[POD_PORT]
   # Output format
   kubectl -o yaml/json
   # More output
   kubectl -o wide
   # Logs
   kubectl logs <pod-name>
   kubectl logs -f <pod-name> -c <container-name>
   # Edit resource in cmd
   kubectl edit deploy <deployment-name>
   # Copy file to host
   kubectl cp <pod-name>:<pod-directory>:/tmp
   # Show all available resources
   kubectl api-resources
   ```

- 通过定义可执行的shell文件并放置在`usr/local/bin`下，可以使用自定义的kubectl命令。文件名必须与以“kubectl-”开头。
- Master kubectl - [kubectl cheatsheet][2]

[1]:https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/CRI.png
[2]:https://kubernetes.io/docs/reference/kubectl/cheatsheet/


# 3. 深入掌握Pod

## 3.1 Pod的重要属性
```
mountPath -> 存储卷在容器内Mount的绝对路径，少于512个字符
ports -> 容器需要暴露的端口号列表，指的是对外的端口
containerPort -> 容器需要监听的端口号，指的是容器内服务运行的端口
```

## 3.2 Pod的基本用法

- 属于同一个Pod的多个容器应用之间相互访问时仅需要通过localhost就可以通信，使得这一组容器被绑定在了同一个环境中。

## 3.3 Static Pod

- 静态Pod是由kubelet进行管理的仅存在于特定Node上的Pod。它们不能通过API Server进行管理，无法与ReplicationController，Deployment或者DaemonSet进行关联，并且kubelet无法对它们进行健康检查。静态Pod总是有kubelet创建的，并且总在kubelet所在的Node上运行。
- 静态Pod有两种创建方式：配置文件方式和HTTP方式。
    - 配置文件：指定kubelet需要监控的配置文件所在的目录，kubelet会定期扫描该目录，并根据目录下的yaml或者json文件进行创建操作。
    - HTTP方式：kubelet会定期从指定的url地址下载Pod的定义文件，然后创建Pod。
    - 静态Pod无法通过API Server进行管理，所以在Master上尝试删除该Pod时，会使其变成pending状态，却不会被删除。完全删除只能是到其所在的Node上将定义文件删除或者在url地址删除。

## 3.4 Pod容器共享Volume

- 同一个Pod中的多个容器能够共享Pod级别的存储卷Volume。Volume可以被定义为各种类型，多个容器各自进行mount操作。

## 3.5 Pod配置管理

### 3.5.1 ConfigMap

- ConfigMap的典型用法如下：
    - 生成为容器内的环境变量
    - 设置容器启动命令的启动参数
    - 以Volume的形式挂载为容器内部的文件或者目录
- ConfigMap既可以用于表示一个变量的值，也可以用于表示一个完整的配置文件的内容。
- ConfigMap可以通过yaml文件创建，也可以直接使用kubectl进行创建。

### 3.5.2 使用ConfigMap的限制条件

- ConfigMap必须在Pod之前创建。
- ConfigMap受Namespace的限制，只有处于相同Namespace中的Pod才可以引用它。
- ConfigMap的配额管理还没能实现。
- 静态Pod无法使用ConfigMap。
- 在Pod对ConfigMap进行挂载时，在容器内部只能挂载为目录，无法挂载为文件。在挂载到容器后，在目录下将包含ConfigMap定义的每个item，如果在该目录下原来还有其他文件，则容器内的该目录将被挂载的ConfigMap覆盖。

## 3.6 Downward API

- Downward API可以通过以下两种方式将Pod信息注入容器的内部：
    - 环境变量
        - ``metadata.name``
        - ``metadata.namespace``
        - ``status.podIP``
        - ``requests.cpu``
    - Volume挂载
- Downward API的价值在于服务发现，集群中的每个节点都需要将自身的标识（ID）及进程的绑定IP地址等事先写入配置文件中，进程在启动时会读取这些信息，然后将这些信息发不到某个类似与服务注册的地方，以事先集群节点的服务发现功能。通过使用initContainer和downward API就可以实现这个功能。

## 3.7 Pod生命周期和重启策略

- Pod的状态如下：

| Status      | Description |
| ----------- | ----------- |
| Pending     | API Server已经创建了该Pod，但在Pod内还有一个或者多个容器的镜像没有创建 |
| Running     | Pod内的所有容器都已经创建，且至少有一个容器处于运行状态，正在启动或者重启状态 |
| Succeeded   | Pod内所有容器都成功执行后退出，并且不会再重启        |
| Failed      | Pod内所有容器均已退出，但至少有一个容器退出为失败状态        |
| Unknown     | 无法获取Pod的状态        |

- Pod的重启策略应用于Pod内的所有容器，并且仅在Pod所处的Node上由kubelet进行判断和重启操作，当某个容器异常退出或者健康检查失败时，kubelet将根据RestartPolicy的设置来进行对应的操作
    - Always（default）：当容器失效时，由kubelet自动重启该容器。
    - onFailure：当容器终止运行且退出码不为0时，有kubelet自动重启该容器。
    - Never：不论容器运行状态如何，kubelet都不会重启该容器。
- kubelet重启失效容器的时间间隔以sync-frequency乘以2n来计算，最长延时5min，并且在成功重启后的10min重制该时间。

## 3.8 Pod的健康检查和服务可用性检查

- Kubernetes对Pod的健康检查有两种类型的探针：
    - LivenessProbe：用于判断容器是否在Running状态，如果探测到容器不健康，kubelet会将其杀掉并且通过RestartPolicy来做对应的处理。如果一个容器没有该探针，则kubelet认为该容器的LivenessProbe探针返回的值永远为true
    - ReadinessProb：用于判断容器服务是否可用（Ready状态），达到Ready状态的Pod才可以接受请求。如果运行过程中Ready状态变为False，则系统自动将其从Service的后端Endpoint列表中隔离出去，后续再把恢复到Ready状态的Pod加回到后段Endpoint列表。这样就能保证客户端在访问Service时不会被转发到服务不可用的Pod实例上。
- LivenessProb和ReadinessProbe均可以配置以下三种实现方式：
    - ExecAction：在容器内部执行一个命令，如果该命令的返回码为0，则表明容器健康。
    - TcpSocketAction：通过容器的IP地址和端口号执行TCP检查，如果能够建立TCP连接，表明容器健康。
    - HttpGetAction：通过调用容器内暴露的健康检查服务endpoint来进行判断。[liveness-and-readiness-probes-with-spring-boot ][1]

## 3.9 Pod调度

- 使用Deployment对象来控制Pod的副本。
- 通过给Node添加标签以及使用NodeSelector（硬条件）可以进行Pod的定向调度。
- 使用NodeAffinity（软条件）也可以进行Pod的定向调度。
- 通过NodeSelector/NodeAffinity/PodAffinity和Node Taint以及Pod Tolerance这些更加细致的调度策略，就可以完成对Pod的精准调度。

### 3.9.1 NodeSelector

- Pod调度是由Master上的Scheduler进程负责的，通过Node的Label和Pod上的nodeSelector属性相匹配，就可以将Pod调度到指定的Node上。
- 如果我们指定了Pod的nodeSelector条件，且在集群中不存在包含相应标签的Node，则即使在集群中还有其他可供使用的Node，这个Pod也无法被成功调度。
- NodeSelector通过标签的方式，简单实现了限制Pod所在节点的方法，亲和性调度机制则极大的扩展了Pod的调度能力：
    - 更具表达力
    - 可采用软限制，优先采用等限制方式，如果不存在满足优先需求的会退而求其次在其他Node上运行Pod。
    - 可以依据Node上正在运行的其他Pod的标签来进行限制，而非节点本身。

### 3.9.2 NodeAffinity

- NodeAffinity的两种属性：
    - RequiredDuringSchedulingIgnoredDuringExecution：相当于NodeSelector，硬限制。
    - PreferedDuringSchedulingIgnoredDuringExecution：软限制，强调与优先满足，可以设置权重，以定义先后顺序。
    - IgnoreDuringExecution：如果一个Pod所在的节点运行期间发生变化，则忽略该变化。
- NodeAffinity使用规则设置：
    - 如果同时定义了nodeSelector和nodeAffinity，那么必须两个条件都得到满足，Pod才能最终运行在指定的Node上。
    - NodeAffinity可以指定多个nodeSelectorTerms，只要有一个匹配就可以进行调度。
    - 如果一个nodeSelectorTerms有多个matchExpressions，则一个节点必须满足所有的matchExpressions才能运行该Pod。

### 3.9.3 PodAffinity

- 根据在节点上正在运行的Pod的标签而不是Node的标签进行判断和调度，要求对Node和Pod两个条件进行匹配。
- Node的标签被称为topologyKey，意为表达节点所属的topology的范围。topologyKey可以使用任何合法的标签Key，但是有如下规则：
    - 在Pod亲和性和RequiredDuringScheduling的Pod互斥性的定义中，不允许使用空的topologyKey。
    - 如果Admission Controller包含了LimitPodHardAntiAffinityTopology，那么topologyKey就被限制为kubernetes.io/hostname。
    - 在PreferredDuringScheduling类型的Pod互斥性定义中，空的topologyKey会被解释为kubernetes.io/hostname, failure-domain.beta.kubernetes.io/zone和failure-domain.beta.kubernetes.io/region。

#### 3.9.3.1 Taint and Tolerations

- Taint和Toleration配合使用，让Pod避开那些不合适的Node。在Node上设置一个或者多个Taint之后，除非Pod明确声明能够容忍这些污点，否则无法在这些Node上运行。Toleration是Pod的属性，让Pod能够运行在标注了Taint的Node上。
- Toleration特殊的规则：
    - 空的key配合Exists的操作符能够匹配所有的键和值。
    - 空的effect匹配所有的effect。
- Taint和Toleration的处理逻辑顺序：首先列出节点中所有的Taint，然后忽略Pod的Toleration能够匹配的部分，剩下的没有忽略的Taint就是对Pod的效果。
- 系统允许给具有NoExecute的Toleration加入一个可选的tolerationSeconds的字段，表明这个Pod在Taint添加到Node之后还可以在Node上运行多久。
- 定义Pod的驱逐行为，以应对故障节点：
    - 没有设置Toleration的Pod会立即驱逐。
    - 配置了对应Toleration的Pod，如果没有tolerationSeconds，则会一直留在节点中。
    - 配置了对应Toleration的Pod并且制定了tolerationSeconds，则会在指定的时间后被驱逐。

### 3.9.4 Pod Priority Preemption

- 当系统资源不足时，kubernetes可以通过配置来释放一些不重要的负载，保证重要的负载可以运行。主要有两个行为：
    - Eviction（驱逐）：kubelet执行操作，在Node上执行。
    - Preemption（抢占）：Scheduler执行操作，在Master上进行。
- **使用优先级抢占的调度策略可能会导致死锁，并且增加了系统的复杂性，还可能带来额外的不稳定因素。通常考虑的是集群扩容为最优解，如果无法扩容再考虑使用优先级**。

### 3.9.5 DaemonSet

- 用于管理在集群中每个Node上仅运行一份的Pod实例。
    - 存储Daemon进程
    - 日志采集程序
    - 性能监控
- 调度策略和RC类似。

### 3.9.6 Job

- 批处理任务通常并行或者串行启动多个计算进程去处理一批work item，处理完成后，任务结束。可以分为三种模式：
    - Job template expansion模式：一个job处理一个work item，有几个work item就产生几个独立的Job，通常适合work item数量少，每个work item要处理的数据量比较大的场景。
    - Queue with pod per work item模式：采用一个任务队列存work item，一个job对象作为消费者去完成这些work item，job会启动N个Pod，每个Pod对应一个work item。（推荐）
    - Queue with variable pod count模式：和上面类似，区别在于Job启动的pod数量是可变的。（推荐）
- Job的三种类型：
    - Non-parallel jobs：一个job启动一个pod，完成后job结束。
    - Parallel jobs with a fixed completion count：并行的job会启动多个pod，此时需要设定job的.spec.completions参数为一个正数，当正常结束的pod数量达到参数设定的值后，job结束。此外.spec.parallelism参数来控制并行度，即同时启动几个Job来处理work item。
    - Parallel jobs with a work queue：任何队列方式的并行job需要一个独立的queue，work item都在一个queue中存放，不能设置job的.spec.completions参数，此时：
        - 每个Pod都能独立判断和决定还有任务项需要处理。
        - 如果某个Pod正常结束，则Job不会再启动新的Pod。
        - 如果一个Pod成功结束，则此时应该不存在其他Pod还在工作的情况，它们应该都处于即将结束和退出的状态。
        - 如果所有Pod都结束了，且至少有一个Pod成功结束，则整个Job成功结束。
- Kubernetes同时也支持cronjob。

### 3.9.7 自定义调度器

- 可以在Pod中提供自定义调度器的名字，则该Pod将会有自定义的调度器进行调度，而非默认的调度器。

## 3.10 Init Container

- 初始化容器操作有以下应用场景：
    - 等待其他关联组件正确运行。
    - 基于环境变量或配置模版生成配置文件。
    - 从远程数据库获取本地所需配置，或者将自身注册到某个中央数据库中。
    - 下载相关依赖包，或者对系统进行一些预配置。
- Init container的本质与应用容器都是一样的，但它们是仅运行一次就结束的任务，并且必须在成功执行完成后，系统才能继续执行下一个容器。
- Init container与应用容器的区别：
    - init container的运行方式和应用容器不同，它们必须先于应用容器执行完成，多个init container按顺序执行，并且只有前一个成功了后一个才能启动。当所有init container都成功运行后，才开始创建和运行应用容器。
    - 在init container的定义中也可以设置资源限制，volume的使用和安全策略，但资源限制的设置和应用容器略微不同：
        - 多个init container的资源limit/request取其中的最大值。
        - Pod的有效资源limit/request取二者中的较大值：
            - 所有应用容器的资源limit/request值之和。
            - init container的有效资源limit/request限制值。
- init container不能设置readinessProb探针。

## 3.11 Pod的升级和回滚

### 3.11.1 Deployment的升级

- Deployment的升级方式有两种：
    - `kubectl set image`
    - `kubectl edit deployment的配置`
- 默认情况下，Deployment确保可用的Pod总数至少为Desired减1，也就是最多一个不可用(maxUnavailable=1)。Deployment还需要确保在整个更新过程中Pod的总数量不会超过所需要的副本数量太多，在默认情况下，Deployment确保Pod的总数最多比所需要的Pod数多1，也就是最多一个(maxSurge=1)。1.6之后上面两个值变为25%。
- Deployment指定Pod更新有两种策略：
    - Recreate：更新时杀掉所有正在运行的Pod，然后创建新的Pod
    - RollingUpdate：以滚动更新的方式来逐个更新Pod
- Rollover（多重更新）：
    - Deployment的上一次更新正在进行，此时用户再次发起Deployment的更新操作，那么Deployment会为每一次更新都创建一个ReplicaSet，而每次在新的ReplicaSet创建成功后，会逐个增加Pod的副本数量，同时将之前正在扩容的ReplicaSet停止扩容，并将其加入旧版本的ReplicaSet列表中，然后开始缩容到0。
- 尽量不要更新Deployment的标签选择器，如果一定需要更新，则必须谨慎使用。
    - 添加selector label时，必须同步修改Deployment配置的Pod的标签。
    - 更新selector label时和添加效果相同，必须同步修改Pod的标签。
    - 删除selector label时需要注意，被删除的标签仍然会存在在已经有的Pod上。

### 3.11.2 Deployment的升级

- `kubectl rollout`

### 3.11.3 暂停和恢复Deployment的部署操作，以完成复杂的修改

- 对于一次复杂的Deployment配置修改，为了避免频繁出发Deployment的更新操作，可以先暂停Deployment的更新操作，然后进行配置修改，在恢复Deployment，一次性触发完整的更新操作，就可以避免不必要的Deployment更新操作了。
    - `kubectl rollout pause deployment`
    - `kubectl rollout resume deployment`
- 在恢复暂停的Deployment之前，无法回滚该Deployment。

### 3.11.4 使用kubectl rolling-update完成对RC的滚动升级

- 优先使用Deployment完成对Pod的部署和升级操作。

### 3.11.5 其他管理对象的更新策略

- DaemonSet
    - OnDelete：默认升级策略，创建好新的DaemonSet后，新的Pod并不会被自动创建，直到用户手动删除旧版本的Pod，才触发新建操作。
    - RollingUpdate：和Deployment类似，但是目前不支持查看和管理DaemonSet的更新历史记录，并且不能通过`kubectl rollback`来实现回滚，必须再次提交旧版本。

## 3.12 Pod的扩容

- 有手动扩容和自动扩容两种模式，手动通过kubetl或REST API来实现，自动则根据性能指标来调整Pod的数量。

### 3.12.1 自动扩缩容机制

- HPA的工作原理
    - HPA控制器通过Metrics Server的API获取到Pod的数据，然后向Pod的副本控制器发起scale操作。
- 指标的类型
    - Pod的资源使用率：Pod级别的性能指标，通常是一个ratio。
    - Pod自定义指标：Pod级别的性能指标，通常是一个数值。
    - Object自定义指标或外部自定义指标：通常是一个数值，需要容器应用以某种方式提供。
- 以下Pod异常情况不会被扩缩容算法计入平均值：
    - Pod正在被删除 - 不会计入目标Pod的副本数量
    - Pod的当前指标值无法获得 - 不会计入目标Pod的副本数量
    - 如果指标是CPU使用率，则对正在启动但是还未达到Ready状态的Pod，也暂时不会纳入目标副本的数量范围。
- 当存在缺失指标的Pod时，系统将更保守地重新计算平均值。系统会假设这些Pod在scale down的时候消耗了期望指标值的100%，在需要scale up的时候消耗了期望指标值的0%，这样可以抑制潜在的扩缩容操作。

[1]: https://spring.io/blog/2020/03/25/liveness-and-readiness-probes-with-spring-boot


# 4. 深入掌握Service

## 4.1 Service的基本用法

- 对外提供服务的应用程序需要通过某种机制来实现，对于容器应用最简便的方式就是通过TCP/IP机制以及监听IP和端口号来实现。
- 直接通过Pod的IP地址和端口号可以访问到容器应用内的服务，但是Pod的IP地址是不可靠的，例如当Pod所在的Node发生故障时，Pod将被Kubernetes重新调度到另一个Node，Pod的IP地址将发生变化，更重要的是，如果容器应用本身是分布式的部署方式，通过多个实例共同提供服务，就需要在这些实例的前端设置一个负载均衡器来实现请求的分发。
- 目前基础的Service提供两种负载分发策略：
    - RoundRobin：轮询模式，即轮询将请求转发到后端各个Pod上。（默认）
    - SessionAffinity：基于客户端IP地址进行会话保持的模式，即第一次将某个客户端发起的请求转发到后端的某个Pod上，之后从相同的客户端发起的请求都将被转发到后端相同的Pod上。

### 4.1.1 多端口Service

- 可以对一个Service打开不同的端口，也可以打开同一个端口上的不同的protocol

### 4.1.2 外部服务Service

- 在某些环境中，应用系统需要将一个外部数据库作为后端服务进行连接，或将另外一个集群或Namespace中的服务作为服务的后端，这时可以创建一个无label selector的service来实现。
- 通过创建无label selector的service，系统不会自动创建endpoint，因此需要手动创建一个和该service同名的endpoint，用于指向实际的后端访问地址。

## 4.2 Headless Service

- 如果开发人员希望自己控制负载均衡策略，可以使用headless service，即不为service设置clusterIP，仅通过label selector来将后端的Pod列表返回给调用的客户端。
- 对于去中心话的应用集群，headless service将非常有用。

## 4.3 从集群外部访问Pod或者Service

### 4.3.1 将Pod容器应用的端口号映射到物理机

- 通过使用hostPort
- 通过设置Pod级别的`hostNetwork=true`，该Pod中所有容器的端口号都将被直接映射到物理机上。默认hostPort等于containerPort，如果指定了hostPort，则hostPort必须等于containerPort的值。

### 4.3.1 将Service的端口号映射到物理机

- NodePort
- Cloud LoadBalancer
- Ingress

## 4.4 CoreDNS

- 可以在kubernetes内部搭建DNS服务，来实现对于域名的查找。
- CoreDNS实现了一种链式插件结构，将DNS的逻辑抽象成一个个插件，能够灵活组合使用。
- etcd和hosts插件都可以用于用户自定义域名记录。
- forward和proxy插件都可以用于配置上游DNS服务器或者其他的DNS服务器，当在CoreDNS中查询不到域名时，会到其他DNS服务器上进行查询。在实际环境中，可以将Kubernetes集群外部的DNS纳入CoreDNS，进行统一的DNS管理。

### 4.4.1 Pod级别的DNS配置

- spec.dnsPolicy：
    - Default：继承Pod所在宿主机的DNS设置
    - ClusterFirst：优先使用Kubernetes环境内的DNS服务，无法解析的转发到宿主机。
    - ClusterFirstWithHostNet：与ClusterFirst相同，对于以hostNetwork模式运行的Pod，应明确指定使用该策略。
    - None：忽略Kubernetes环境的DNS配置。通过spec.dnsConfig自定义DNS配置。
- spec.dnsConfig:
    - nameservers: 一组DNS服务器的列表，最多可以设置3个。
    - searches：一组用于域名搜索的DNS域名后缀，最多6个。
    - options：其他配置可选参数。

## 4.5 Ingress

- 对于Http业务层的路由机制，kubernetes使用一个Ingress策略定义和一个具体的Ingress Controller，两者结合并实现了一个完整的Ingress负载均衡器。
- 使用Ingress进行负载分发时，Ingress Controller基于Ingress规则将客户端请求直接转发到Service对应的后端的endpoint上，这样会跳过kube-proxy的转发功能。如果Ingress Controller提供的是对外服务，则实际上实现的是边缘路由器的功能。
  ![Ingress][1]
- 可以将云服务商的ALB设置成Ingress Controller
- Ingress Controller将以Pod的形式运行
- Ingress Controller需要配置一个默认的backend，用于在客户端访问的URL地址不存在时返回的404应答。
- 使用Ingress可以将流量转发到
    - 单个后端服务上
    - 同一域名的不同的URL路径转发到不同的服务上
    - 不同的域名被转发到不同的服务上

[1]: https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/Ingress.png


# 5. 核心组件运行机制

## 5.1 Kubernetes API Server原理解析

- Kubernetes API Server的核心功能是提供Kubernetes各类资源对象的增删改查和Watch等HTTP Rest接口，成为集群内各个功能模块之间数据交互和通信的中心枢纽，是整个系统的数据总线和数据中心。
- 是集群管理的API入口
- 是资源配合控制的入口
- 提供了完备的集群安全机制

### 5.1.1 Kubernetes API Server概述

- Kubernetes API Server通过一个名为kube-apiserver的进程提供服务，该进程运行在Master上。在默认情况下，kube-apiserver进程在本机的8080端口提供REST服务，也可以启动443端口。
- **Kubernetes API Server本身也是一个Service,  它的名称就是kubernetes**，并且它的Cluster IP地址就是Cluster IP地址池里的第一个地址。
- 由于API Server是Kubernetes集群数据的唯一访问入口，因此安全性与高性能就成为API Server设计和实现的两大核心目标。
    - 安全：通过HTTPS安全传输通道与CA签名数字证书强制双向认证的方式，API Server的安全性得以保障，此外还有RBAC访问控制策略。
    - 性能：
        - API Server拥有大量的高性能底层代码。在API Server源码中使用了协程和队列这种轻量级的高性能并发代码，使得单进程的API Server具备了超强的多核处理能力，从而以很快的速度并发处理大量的请求。
        - 普通的List接口结合异步Watch接口
        - 采用高性能etcd数据库而非传统的关系型数据库，不仅解决了数据的可靠性问题，也极大的提升了API Server数据访问层的性能。

### 5.1.2 API Server架构解析

- API Server架构
  ![API Server][1]
    - API层：以REST方式提供各种API接口。
    - 访问控制层
    - 注册表层：Kubernetes把所有资源对象都保存在Registry中。
    - etcd数据库：用于持久化存储的Kubernetes资源对象的KV数据库。etcd的watch API接口对于API Server来说至关重要，API Server创新性地设计了List-Watch这种高性能的资源对象实时同步机制，使Kubernetes可以管理超大规模的集群，及时响应和快速处理集群中的各种事件。
      ![ListWatch][2]

- Kubernetes的List-Watch用于实现数据同步代码的逻辑。客户端首先调用API Server的List接口获取相关资源对象的全量数据并将其缓存到内存中，然后启动对应资源对象的Watch协程，在接收到Watch事件后，再根据时间的类型，对内存中的全量资源列表作出相应的同步修改，从实现上来看，这是一种全量结合增量的，高性能的，近乎实时的数据同步方式。
- API Server对每种资源对象都引入了一个相应不变的internal版本，每个版本只要支持转换为internal版本，就能够与其他版本进行间接转换。

### 5.1.3 Kubernetes Proxy API接口

- API Server还提供Kubernetes Proxy API接口，这个接口类的作用是代理REST请求，即Kubernetes API Server把收到的REST请求转发到某个Node上的kubelet守护进程的REST端口，由kubelet进程负责响应。
- Proxy API接口的作用：多做管理目的，比如逐一排查Service的Pod副本，检查哪些Pod的服务存在异常。

### 5.1.4 集群功能模块之间的通信

![Structure][3]

- 集群内的各个功能模块通过API Server将信息存入etcd，当需要获取和操作这些数据时，则通过API Server提供的REST接口来实现，从而实现各模块之间的信息交互。
- 最常见的交互场景时kubelet进程和API Server的交互。每个Node上的kubelet每隔一个时间周期，就会调用一次API Server的REST接口报告自身状态，API Server在接收到这些信息后，会将节点状态信息更新到etcd中。此外，kubelet也通过API Server的Watch接口监听Pod信息，如果监听到新的Pod副本被调度绑定到本节点上，则执行Pod对应的容器的创建和启动逻辑；如果监听到Pod对象被删除，则删除本节点上对应的Pod容器；如果监听到修改Pod的信息，kubelet就会相应地修改本节点的Pod容器。
- 另一个交互场景就是kube-controller-manager进程和API Server的交互。监控Node信息并做相应处理。
- 还有一个交互场景就是kube-scheduler与API Server的交互。Scheduler通过API Server的Watch接口监听到新建Pod副本的信息后，会检索所有符合该Pod要求的Node列表，开始执行Pod调度逻辑，在调度成功后将Pod绑定到目标节点上。
- 各功能模块都采用缓存机制来缓存数据，在某些情况下各功能模块并不直接访问API Server，而是通过访问缓存数据来间接访问API Server。

## 5.2 Controller Manager原理解析

- Kubernetes中，每个Controller都是一个操作系统，通过API Server提供的List-Watch接口实时监控集群中特定资源的状态变化，当发生各种故障导致某资源对象的状态发生变化时，Controller会尝试将其状态调整为期望的状态。Controller Manager是Kubernetes中各种操作系统的管理者，是集群内部的管理控制中心，也是Kubernetes自动化功能的核心。
- Replication Controller：控制预期副本数量，伸缩，滚动更新。
- Node Controller：Node信息和健康状态。
- ResourceQuota Controller：对系统的资源进行配额管理，防止某些业务进程的设计缺陷导致超量占用系统物理资源。通过Admission Control来控制，提供两种方式的配额约束。
    - LimitRanger：作用与Pod和Container。
    - ResourceQuota：作用与Namespace，限定一个Namespace里的各类资源的使用总额。
- Service Controller & Endpoints Controller：负责维护一个服务中后端所有的Pod endpoints的controller， kube-proxy进程通过使用endpoints对象来实现Service的负载均衡。

## 5.3 Scheduler原理解析

- Kubernetes Schedular在整个系统中承担了“承上启下”的重要功能，“承上”是指它负责接受Controller Manager创建的新Pod，为其安排一个目标Node；“启下”是指安置工作完成后，目标Node上的kubelet服务进程接管后继工作，负责Pod生命周期中的下半生。
- Schedular的作用是将待调度的Pod按照特定的调度算法和调度策略binding到集群中某个合适的Node上，并将绑定信息写入etcd。整个调度过程涉及三个对象，分别是待调度Pod列表，可用Node列表，以及调度算法和策略。随后目标Node上的kubelet通过API Server监听到schedular产生的Pod绑定事件，获取对应的Pod清单，下载Image镜像并启动容器。
- 在调度过程中，先遍历所有Node，筛选出符合要求的候选节点。然后通过算法确定出最优节点。

## 5.4 Kubelet运行机制分析

- 每个Node上都会有一个kubelet服务进程。该进程用于处理Master下发到本节点的任务，管理Pod以及Pod中的容器。每个kubelet进程都会在API Server上注册节点自身的信息，定期向Master汇报节点资源的使用情况。

### 5.4.1 节点管理

- 每个kubelet都被手续创建和修改任何Node的权限，但在实践中，它仅仅用来创建和修改自己。
- kubelet在启动时向API Server进行注册，并定时（默认10秒）向API Server发送节点的新消息。

### 5.4.2 Pod管理

- Kubelet通过以下几种方式获取自身Node上要运行的Pod清单
    - Node上的配置文件
    - HTTP端点
    - API Server：kubelet通过API Server监听etcd目录（有本地缓存），同步Pod列表。
- 所有以非API Server方式创建的Pod都叫做Static Pod，kubelet将Static Pod的状态汇报给API Server，API Server为该Static Pod创建一个Mirror Pod和其相匹配。Mirror Pod的状态将真实反应Static Pod的状态。当Static Pod被删除时，与之对应的Mirror Pod也会被删除。

## 5.5 Kube-proxy 运行机制解析

- Userspace: 该模式下kube-proxy会为每一个Service创建一个监听端口。发向Cluster IP的请求被Iptables规则重定向到Kube-proxy监听的端口上，Kube-proxy根据LB算法选择一个提供服务的Pod并和其建立链接，以将请求转发到Pod上。 该模式下，Kube-proxy充当了一个四层Load balancer的角色。由于kube-proxy运行在userspace中，在进行转发处理时会增加两次内核和用户空间之间的数据拷贝，效率较另外两种模式低一些；好处是当后端的Pod不可用时，kube-proxy可以重试其他Pod。
  ![userspace][4]

- Iptables: 为了避免增加内核和用户空间的数据拷贝操作，提高转发效率，Kube-proxy提供了iptables模式。在该模式下，Kube-proxy为service后端的每个Pod创建对应的iptables规则，直接将发向Cluster IP的请求重定向到一个Pod IP。 该模式下Kube-proxy不承担四层代理的角色，只负责创建iptables规则。该模式的优点是较userspace模式效率更高，但不能提供灵活的LB策略，当后端Pod不可用时也无法进行重试。
  ![iptables][5]

- IPVS: 该模式和iptables类似，kube-proxy监控Pod的变化并创建相应的ipvs rules。ipvs也是在kernel模式下通过netfilter实现的，但采用了hash table来存储规则，因此在规则较多的情况下，Ipvs相对iptables转发效率更高。除此以外，ipvs支持更多的LB算法。如果要设置kube-proxy为ipvs模式，必须在操作系统中安装IPVS内核模块。
    - 为大型集群提供了更好的可扩展性和性能
    - 支持比iptables更复杂的均衡算法
    - 支持服务器健康检查和连接重试
    - 可以动态修改ipset的集合，即使iptables的规则正在使用这个集合
      ![ipvs][6]

[1]: https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/API%20Server.png
[2]: https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/ListWatch.jpg
[3]: https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/KubernetesStructure.png
[4]: https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/userspace.png
[5]: https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/iptables.png
[6]: https://raw.githubusercontent.com/eziceice/blog/master/kubernetes/ipvs.png
