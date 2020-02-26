# flink运行架构

## flink组件

flink运行时架构主要包含四个组件，他们在处理流应用程序时协同工作，为以下四个组件

* JobManager(作业管理器)
* ResourceManager(资源管理器)
* TaskManager(任务管理器)
* Dispatcher(分发器)

### JobManager(作业管理器)

​		控制一个应用程序执行的主进程(Process)，也就是说，每个应用程序都会被一个不同的 JobManager 所控制执行。JobManager 会先接收到要执行的应用程序，这个应用程序会包括: 作业图(JobGraph)、逻辑数据流图(logical dataflow graph)和打包了所有的类、库和其它 资源的 JAR 包。JobManager 会把 JobGraph 转换成一个物理层面的数据流图，这个图被叫做 “执行图”(ExecutionGraph)，包含了所有可以并发执行的任务。JobManager 会向资源管 理器(ResourceManager)请求执行任务必要的资源，也就是任务管理器(TaskManager)上 的插槽(slot)。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的 TaskManager 上。而在运行过程中，JobManager 会负责所有需要中央协调的操作，比如说检 查点(checkpoints)的协调。	

### ResourceManager(资源管理器)

​		**主要负责管理任务管理器(TaskManager)的插槽(slot)，TaskManger 插槽是 Flink 中 定义的处理资源单元**。Flink 为不同的环境和资源管理工具提供了不同资源管理器，比如 YARN、Mesos、K8s，以及 standalone 部署。当 JobManager 申请插槽资源时，ResourceManager 会将有空闲插槽的 TaskManager 分配给 JobManager。如果 ResourceManager 没有足够的插槽 来满足 JobManager 的请求，它还可以向资源提供平台发起会话，以提供启动 TaskManager 进程的容器。另外，ResourceManager 还负责终止空闲的 TaskManager，释放计算资源。

### TaskManager(任务管理器)

​		Flink 中的工作进程。通常在 Flink 中会有多个 TaskManager 运行，每一个 TaskManager，都包含了一定数量的插槽(slots)。插槽的数量限制了 TaskManager 能够执行的任务数量。 启动之后，TaskManager 会向资源管理器注册它的插槽;收到资源管理器的指令后， TaskManager 就会将一个或者多个插槽提供给 JobManager 调用。JobManager 就可以向插槽 分配任务(tasks)来执行了。在执行过程中，一个 TaskManager 可以跟其它运行同一应用程 序的 TaskManager 交换数据。

### Dispatcher(分发器)

​		可以跨作业运行，它为应用提交提供了 REST 接口。当一个应用被提交执行时，分发器 就会启动并将应用移交给一个 JobManager。由于是 REST 接口，所以 Dispatcher 可以作为集 群的一个 HTTP 接入点，这样就能够不受防火墙阻挡。Dispatcher 也会启动一个 Web UI，用 来方便地展示和监控作业执行的信息。Dispatcher 在架构中可能并不是必需的，这取决于应 用提交运行的方式。

## flink任务提交流程

### Standalone模式

![](https://img-blog.csdnimg.cn/20191023101544636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMzMzc0MjU=,size_16,color_FFFFFF,t_70)

* App程序通过REST接口提交给Dispatcher，REST是跨平台的，无需考虑防火墙拦截
* Dispatcher把JobManager进程启动，并把任务交给JobManager
* JobManager拿到任务之后，向ResourceManager申请资源(也就是slots)，ResourceManager会启动TaskManager进程，TaskManager中空闲的slot会向ResourceManager进行注册
* ResourceManager根据JobManager申请的资源，将TaskManager发送指令
* 然后，JobManager就可以和TaskManager进行通信了，TaskManager提供slots，JobManger给TaskManager分配任务

### Yarn模式

![](https://img-blog.csdnimg.cn/20191023101840463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMzMzc0MjU=,size_16,color_FFFFFF,t_70)

* 提交App之前，先上传Flink的Jar包和配置到HDFS，以便JobManager和TaskManager共享HDFS的数据。
* 客户端向ResourceManager提交Job，ResouceManager接到请求后，先分配container资源，然后通知NodeManager启动ApplicationMaster。
* ApplicationMaster会加载HDFS的配置，启动对应的JobManager，然后JobManager会分析当前的作业图，将它转化成执行图（包含了所有可以并发执行的任务），从而知道当前需要的具体资源。
* 接着，JobManager会向ResourceManager申请资源，ResouceManager接到请求后，继续分配container资源，然后通知ApplictaionMaster启动更多的TaskManager（先分配好container资源，再启动TaskManager）。container在启动TaskManager时也会从HDFS加载数据。
* 最后，TaskManager启动后，会向JobManager发送心跳包。JobManager向TaskManager分配任务。

## 任务调度原理

![](https://img-blog.csdnimg.cn/2020013022125221.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phY2tzb25fbXZw,size_16,color_FFFFFF,t_70)

* Program code：程序运行代码
* Client：Client可以运行在任何机器上，只要该机器根JobManager相连，负责将数据流图传给JobManager，传送完数据流图之后，客户端可以断开连接，也可以不断开连接等待结果
* JobManager：主要负责调度 Job 并协调 Task 做 checkpoint，职责上很像 Storm 的 Nimbus。从 Client 处接收到 Job 和 JAR 包等资源后，会生成优化后的 执行计划，并以 Task 的单元调度到各个 TaskManager 去执行
* TaskManager：在启动的时候就设置好了槽位数(Slot)，每个 slot 能启动一个 Task，Task 为线程。从 JobManager 处接收需要部署的 Task，部署启动后，与自 己的上游建立 Netty 连接，接收数据并处理。

### TaskManager和slot

​		Flink 中每一个 worker(TaskManager)都是一个 **JVM 进程**，它可能会在独立的线程上执行一个或多个 subtask。为了控制一个 worker 能接收多少个 task，worker 通过控制task slot 来进行控制(一个worker至少有一个task slot)。
​		每个 task slot 表示 TaskManager 拥有资源的一个固定大小的子集。假如一个 TaskManager 有三个 slot，那么它会将其管理的内存分成三份给各个 slot。资源 slot 化意味着一个 subtask 将不需要跟来自其他 job 的 subtask 竞争被管理的内存，取而 代之的是它将拥有一定数量的内存储备。需要注意的是，这里不会涉及到 CPU 的隔 离，slot 目前仅仅用来隔离 task 的受管理的内存。
​		通过调整 task slot 的数量，允许用户定义 subtask 之间如何互相隔离。如果一个 TaskManager 一个 slot，那将意味着**每个 task group 运行在独立的 JVM 中**(该 JVM 可能是通过一个特定的容器启动的)，而一个 TaskManager 多个 slot 意味着更多的 subtask 可以共享同一个 JVM。而在同一个 JVM 进程中的 task 将共享 TCP 连接(基 于多路复用)和心跳消息。它们也可能共享数据集和数据结构，因此这减少了每个 task 的负载。

![](https://ci.apache.org/projects/flink/flink-docs-release-1.10/fig/tasks_slots.svg)

默认情况下，Flink 允许 subtasks 共享 slots，即使它们是不同 tasks 的 subtasks，只要它们来自同一个 job。因此，一个 slot 可能会负责这个 job 的整个管道（pipeline）。允许 slot sharing 有两个好处：

* Flink 集群需要与 job 中使用的最高并行度一样多的 slots。这样不需要计算作业总共包含多少个 tasks（具有不同并行度）。
* 更好的资源利用率。在没有 slot sharing 的情况下，简单的 subtasks（source/map()）将会占用和复杂的 subtasks （window）一样多的资源。通过 slot sharing，将示例中的并行度从 2 增加到 6 (图二中增加了并行率)可以充分利用 slot 的资源，同时确保繁重的 subtask 在 TaskManagers 之间公平地获取资源。

![](https://ci.apache.org/projects/flink/flink-docs-release-1.10/fig/slot_sharing.svg)

​		Task Slot 是静􏱙态􏰖概念，是指 TaskManager 具有的􏰖并发执行能􏰙􏰃力，可以调整􏱚􏱛 参数 taskmanager.numberOfTaskSlots 􏰘􏰙􏱜􏱝;􏰸并􏰙度 parallelism 是动态概念， 即TaskManager运行程序时的实际使用的并行能力 􏱎􏰙􏱃，可以􏱚􏱛参数 parallelism.default 􏰘􏰙􏱜􏱝

​		也就是􏰼，假设共有 3 个 TaskManager，每一个 TaskManager 中􏰖分􏱜 3 个 TaskSlot，也就是每个 TaskManager 可以接收 3 个 task，一共 9 个 TaskSlot，如果我 们􏱉􏱝 parallelism.default=1，即􏱎程􏰙􏱃序􏱑􏱒􏰖并行􏰙度为 1，9 个 TaskSlot 只􏰯了 1 个，有 8 个􏱟􏱠，因此，􏱉􏱝设置合适并行度才能提高效率

### 程序和数据流

所有flink程序由三部分组成

* Source：读取数据
* Transformation：算子进行加工
* Sink：输出数据

![](https://ci.apache.org/projects/flink/flink-docs-release-1.10/fig/program_dataflow.svg)

​		在运行时，Flink 上􏱎􏰙􏰖􏱃运行的程序会被􏰝映射成“􏱪􏱫数据􏱂流”(dataflows)，它包 含了􏰢三􏱤分。每一个 dataflow 以一个或多个 sources 开始以一个或多个 sinks结束。dataflow 类似于任意􏰖有向无􏱍图(DAG)。在大部分情况下，􏱃程序中􏰖􏱭转换运算 􏱎􏱈(transformations)􏰁和 dataflow 中􏰖的算􏱈子(operator)是一一对应􏰖关系，但有 时候，一个 transformation 可􏰃对应多个 operator

### 执行图

􏰔 flink中的执行图分为四层StreamGraph->JobGraph->ExecutionGraph->物理执行图

* StreamGraph：在client上生成，根据用户通过StreamAPI编写的代码生成的最初的图，用来表示程序的拓扑结构。
* JobGraph：在client上生成，StreamGraph通过优化后生成了JobGraph，**提交给**JobManager的数据结构，主要优化为，将多个符合条件的节点chain在一起作为一个节点，这样可以减少数据在节点之间流动所需要的序列化和反序列化的传输消耗
* ExecutionGraph：在JobManager上生成，JobManager根据JobGraph生成ExecutionGraph，ExecutionGraph是JobGraph的并行化技术，**是调度层最核心的数据结构**
* 物理执行图：在task上运行，JobManager根据ExecutionGraph对Job进行调度后，在每个TaskManager上部署Task后形成的图，并不是一个具体的数据结构

### 并行度

​		Flink中的程序本质上是并行的和分布式的。在执行期间，一个流具有一个或多个流分区，并且每个算子operator具有一个或多个运算符子任务operator subtask。这些子任务相互独立，并在不同的线程中执行，并且可能在不同的机器或容器上执行。

​		一个特定算子的子任务的个数(subtask)称为并行度。一个流程序的并行度可以认为是其所有算子中的最大并行度，一个程序中，不同的算子可能具有不同的并行度

​		Stream在算子中传送数据可以有两种模式，one-to-one（Spark中的窄依赖) ，redistributing(Spark中的宽依赖)

