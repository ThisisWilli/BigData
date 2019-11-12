# Spark资源调度与任务调度

## 资源调度与任务调度示意图

![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark%E8%B5%84%E6%BA%90%E8%B0%83%E5%BA%A6%E5%92%8C%E4%BB%BB%E5%8A%A1%E8%B0%83%E5%BA%A6.png )

* 资源调度为1-6
* 任务调度为7之后

## 资源调度

### 参考文章

* https://zhuanlan.zhihu.com/p/28893155

### 名词解释

![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark%E4%B8%ADDriver.png )

* Application：表示你的应用程序
* Driver：**表示main()函数**，创建SparkContext。由SparkContext负责与ClusterManager通信，进行资源的申请，任务的分配和监控等。程序执行完毕后关闭SparkContext
* Executor：某个Application运行在Worker节点上的一个**进程**，该进程负责运行某些task，并且负责将数据存在内存或者磁盘上。在Spark on Yarn模式下，其进程名称为 CoarseGrainedExecutor Backend，一个CoarseGrainedExecutor Backend进程有且仅有一个executor对象，它负责将Task包装成taskRunner，并从线程池中抽取出一个空闲线程运行Task，这样，每个CoarseGrainedExecutorBackend能并行运行Task的数据就取决于分配给它的CPU的个数，每个executor的默认启动内存为1024
* Worker：集群中可以运行Application代码的节点。在Standalone模式中指的是通过slave文件配置的worker节点，在Spark on Yarn模式中指的就是NodeManager节点。
* Task：在Executor进程中执行任务的工作单元，多个Task组成一个Stage
* Job：包含多个Task组成的并行计算，是由Action行为触发的
* Stage：每个Job会被拆分很多组Task，作为一个TaskSet，其名称为Stage，**Job>Stage>Task**
* DAGScheduler：**根据Job构建基于Stage的DAG**，并提交Stage给TaskScheduler，其划分Stage的依据是RDD之间的依赖关系
* TaskScheduler：将TaskSet提交给Worker（集群）运行，**每个Executor运行什么Task就是在此处分配的**。
* dispatcher:NettyRpcEnv中包含Dispatcher，主要针对服务端，帮助路由到正确的RpcEndpoint，并且调用其业务逻辑。
* inbox:每个Endpoint都有一个Inbox，Inbox里面有一个InboxMessage的链表，InboxMessage有很多子类，可以是远程调用过来的RpcMessage，可以是远程调用过来的fire-and-forget的单向消息OneWayMessage，还可以是各种服务启动，链路建立断开等Message，这些Message都会在Inbox内部的方法内做模式匹配，调用相应的RpcEndpoint的函数（都是一一对应的）。
* outbox:和Inbox类似，Outbox内部包含一个OutboxMessage的链表，OutboxMessage有两个子类，OneWayOutboxMessage和RpcOutboxMessage，分别对应调用RpcEndpoint的receive和receiveAndReply方法

![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark%20%E8%B5%84%E6%BA%90%E8%B0%83%E5%BA%A6%E6%BA%90%E7%A0%81%E5%92%8C%E4%BB%BB%E5%8A%A1%E8%B0%83%E5%BA%A6%E6%BA%90%E7%A0%81.png )

### 创建RPC环境并进行通信

* 在Master类中

  ```scala
  def startRpcEnvAndEndpoint(
        host: String,
        port: Int,
        webUiPort: Int,
        conf: SparkConf): (RpcEnv, Int, Option[Int]) = {
      val securityMgr = new SecurityManager(conf)
      /**
       * 创建RPC(Remote Procedure Call）环境
       * 这里只是创建准备好Rpc的环境，后面会向RPCEnv中注册角色[Driver，Master，Worker，Executor]
       */
      val rpcEnv = RpcEnv.create(SYSTEM_NAME, host, port, conf, securityMgr)
      val masterEndpoint = rpcEnv.setupEndpoint(ENDPOINT_NAME,
        new Master(rpcEnv, rpcEnv.address, webUiPort, securityMgr, conf))
      val portsResponse = masterEndpoint.askSync[BoundPortsResponse](BoundPortsRequest)
      (rpcEnv, portsResponse.webUIPort, portsResponse.restPort)
  ```

  * 先创建master的endpoint,endpoint注册到RPCEnv中
  
    ```scala
    // 注册endpoint，必须指定名称，客户端路由就靠这个名称来找endpoint
    def setupEndpoint(name: String, endpoint: RpcEndpoint): RpcEndpointRef 
    
    // 拿到一个endpoint的引用
    def setupEndpointRef(address: RpcAddress, endpointName: String): RpcEndpointRef
    ```
  
    
  
  * NettyRpcEnv中的setupEndpoint方法注册一个
  
    ```scala
    override def setupEndpoint(name: String, endpoint: RpcEndpoint): RpcEndpointRef = {
        dispatcher.registerRpcEndpoint(name, endpoint)
      }
    ```
  
* 注册endpoint
  
  * Master继承ThreadSafeRpcEndpoint又继承RpcEndpoint，里面有onStart(),onStop()
  
  * 查看注册时的代码setupEndpoint
  
  * 往rpc中注册masterendpoint
  
  ```scala
  override def setupEndpoint(name: String, endpoint: RpcEndpoint): RpcEndpointRef = {
      dispatcher.registerRpcEndpoint(name, endpoint)
    }
  ```
  
  

master启动，deploy，object中才有main方法,首先创建RPC，收消息，处理消息,有了RPC环境，master启动时先向RPC注册

### Spark启动向master注册application

* Spark Submit中查看main方法，childmainclass

  ```scala
  override def main(args: Array[String]): Unit = {
      // Initialize logging if it hasn't been done yet. Keep track of whether logging needs to
      // be reset before the application starts.
      val uninitLog = initializeLogIfNecessary(true, silent = true)
  
      val appArgs = new SparkSubmitArguments(args)
      if (appArgs.verbose) {
        // scalastyle:off println
        printStream.println(appArgs)
        // scalastyle:on println
      }
      appArgs.action match {
        case SparkSubmitAction.SUBMIT => submit(appArgs, uninitLog)
        case SparkSubmitAction.KILL => kill(appArgs)
        case SparkSubmitAction.REQUEST_STATUS => requestStatus(appArgs)
      }
    }
  ```

* client中的main方法

* 先判断Master是否存活master中的receiveandreply，如果存活，createDriver(description)waitingDrivers

* Master中的scheduler

  ```scala
  private def createTaskScheduler(
        sc: SparkContext,
        master: String,
        deployMode: String): (SchedulerBackend, TaskScheduler) = {
      import SparkMasterRegex._
  
      // When running locally, don't try to re-execute tasks on failure.
      val MAX_LOCAL_TASK_FAILURES = 1
  
      master match {
        case "local" =>
          val scheduler = new TaskSchedulerImpl(sc, MAX_LOCAL_TASK_FAILURES, isLocal = true)
          val backend = new LocalSchedulerBackend(sc.getConf, scheduler, 1)
          scheduler.initialize(backend)
          (backend, scheduler)
  
        case LOCAL_N_REGEX(threads) =>
          def localCpuCount: Int = Runtime.getRuntime.availableProcessors()
          // local[*] estimates the number of cores on the machine; local[N] uses exactly N threads.
          val threadCount = if (threads == "*") localCpuCount else threads.toInt
          if (threadCount <= 0) {
            throw new SparkException(s"Asked to run locally with $threadCount threads")
          }
          val scheduler = new TaskSchedulerImpl(sc, MAX_LOCAL_TASK_FAILURES, isLocal = true)
          val backend = new LocalSchedulerBackend(sc.getConf, scheduler, threadCount)
          scheduler.initialize(backend)
          (backend, scheduler)
  
        case LOCAL_N_FAILURES_REGEX(threads, maxFailures) =>
          def localCpuCount: Int = Runtime.getRuntime.availableProcessors()
          // local[*, M] means the number of cores on the computer with M failures
          // local[N, M] means exactly N threads with M failures
          val threadCount = if (threads == "*") localCpuCount else threads.toInt
          val scheduler = new TaskSchedulerImpl(sc, maxFailures.toInt, isLocal = true)
          val backend = new LocalSchedulerBackend(sc.getConf, scheduler, threadCount)
          scheduler.initialize(backend)
          (backend, scheduler)
        // standalone 提交任务都是以spark://这种方式提交的
        case SPARK_REGEX(sparkUrl) =>
          // schedule创建Task SchedulerImpl对象
          val scheduler = new TaskSchedulerImpl(sc)
          val masterUrls = sparkUrl.split(",").map("spark://" + _)
          /**
           *  这里的backend是standaloneSchedulerBackend这个类型
           */
          val backend = new StandaloneSchedulerBackend(scheduler, sc, masterUrls)
          scheduler.initialize(backend)
          // 返回了StandaloneSchedulerBackend和TaskSchedulerImpl两个对象
          (backend, scheduler)
  ```

* backend函数点进来

* SparkContext中

  ```scala
  /**
       * 启动调度程序，这里(sched, ts)分别对应standaloneSchedulerbackend和taskschedulerImpl两个和对象
       * master是提交任务写的spark：//node01：7077
       */
      // Create and start the scheduler
      val (sched, ts): (SchedulerBackend, TaskScheduler) = SparkContext.createTaskScheduler(this, master, deployMode)
      _schedulerBackend = sched
      _taskScheduler = ts
      _dagScheduler = new DAGScheduler(this)
      _heartbeatReceiver.ask[Boolean](TaskSchedulerIsSet)
  
      // start TaskScheduler after taskScheduler sets DAGScheduler reference in DAGScheduler's
      // constructor
      _taskScheduler.start()
  ```

  

在worker上找到节点去启动driver，worker info，

launchDriver

转到Worker中的receive

prepareAndRunDriver

DriverWrapper

SparkContext中的createTaskScheduler

createTaskScheduler中的initialize

### Spark为当前application划分资源

* 在Worker上启动executor

  ```scala
  private def startExecutorsOnWorkers(): Unit = {
      // Right now this is a very simple FIFO scheduler. We keep trying to fit in the first app
      // in the queue, then the second app, etc.
      // 从waitingApps中获取提交的app
      for (app <- waitingApps) {
        // coresPerExecutor 在application中获取启动一个Executor使用几个core，参数--executor--core可以指定，下面指明不指定就是1
        val coresPerExecutor = app.desc.coresPerExecutor.getOrElse(1)
        // If the cores left is less than the coresPerExecutor,the cores left will not be allocated
        // 判断是否给application分配了够了core，因为后面每次给application分配core后，app.coreLeft都会相应的减去分配的core数
        if (app.coresLeft >= coresPerExecutor) {
          // Filter out workers that don't have enough resources to launch an executor
          // 过滤出可用的worker
          val usableWorkers = workers.toArray.filter(_.state == WorkerState.ALIVE)
            .filter(worker => worker.memoryFree >= app.desc.memoryPerExecutorMB &&
              worker.coresFree >= coresPerExecutor)
            .sortBy(_.coresFree).reverse
          val assignedCores = scheduleExecutorsOnWorkers(app, usableWorkers, spreadOutApps)
  
          /**
           * 下面就是取Worker中划分每个Worker提供多少core和启动多少executor，注意：spreadOutApp是true
           * 返回的assignedCores就是每个Worker节点中应该给当前的application分配多少core
           */
          // Now that we've decided how many cores to allocate on each worker, let's allocate them
          for (pos <- 0 until usableWorkers.length if assignedCores(pos) > 0) {
            allocateWorkerResourceToExecutors(
              app, assignedCores(pos), app.desc.coresPerExecutor, usableWorkers(pos))
          }
        }
      }
    }
  ```

* 判断是否可以启动，开始要每个节点的核，最终变成20

  ![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E5%BB%BA%E7%AB%8B%E4%B8%A4%E4%B8%AA%E5%BE%88%E9%87%8D%E8%A6%81%E7%9A%84%E5%AF%B9%E8%B1%A1.png )

  ```scala
  private def scheduleExecutorsOnWorkers(
        app: ApplicationInfo,
        usableWorkers: Array[WorkerInfo],
        spreadOutApps: Boolean): Array[Int] = {
      // 启动一个Executor使用多少core。这里如果提交任务没有指定 executor-core这个值就是None
      val coresPerExecutor = app.desc.coresPerExecutor
      // 这里指定如果提交任务没有指定启动一个Executor使用几个core，默认就是1
      val minCoresPerExecutor = coresPerExecutor.getOrElse(1)
      // oneExecutorPerWorker当前为true
      val oneExecutorPerWorker = coresPerExecutor.isEmpty
      // 默认启动一个Executor使用的内存就是1024M，这个设置在SparkContext中
      // 若提交命令中有--executor-memory 5*1024 就是指定的参数
      val memoryPerExecutor = app.desc.memoryPerExecutorMB
      // 可用的Worker的个数
      val numUsable = usableWorkers.length
      // 创建两个重要对象
      val assignedCores = new Array[Int](numUsable) // Number of cores to give to each worker
      val assignedExecutors = new Array[Int](numUsable) // Number of new executors on each worker
      /**
       * coresToAssign指的是当前药给application分配的core是多少，app.coresleft与集群所有worker剩余的全部core取最小值
       * 这里如果提交了application时指定了--total-executor-core，那么app.coresleft就是指定的值
       */
      var coresToAssign = math.min(app.coresLeft, usableWorkers.map(_.coresFree).sum)
  
      /** Return whether the specified worker can launch an executor for this app. */
        // 判断某台worker节点是否还可以启动executor
      def canLaunchExecutor(pos: Int): Boolean = {
          // 可分配的core是否大于启动一个Executor使用的1个core
        val keepScheduling = coresToAssign >= minCoresPerExecutor
          // 是否有足够的core
        val enoughCores = usableWorkers(pos).coresFree - assignedCores(pos) >= minCoresPerExecutor
  
        // If we allow multiple executors per worker, then we can always launch new executors.
        // Otherwise, if there is already an executor on this worker, just give it more cores.
        val launchingNewExecutor = !oneExecutorPerWorker || assignedExecutors(pos) == 0
          // 启动新的executor
        if (launchingNewExecutor) {
          val assignedMemory = assignedExecutors(pos) * memoryPerExecutor
          // 是否有足够的内存
          val enoughMemory = usableWorkers(pos).memoryFree - assignedMemory >= memoryPerExecutor
          // 这里做安全判断，说的是要分配启动的executor和当前的application启动的使用的executor总数是否在application总的executor
          val underLimit = assignedExecutors.sum + app.executors.size < app.executorLimit
          keepScheduling && enoughCores && enoughMemory && underLimit
        } else {
          // We're adding cores to an existing executor, so no need
          // to check memory and executor limits
          keepScheduling && enoughCores
        }
      }
  
      // Keep launching executors until no more workers can accommodate any
      // more executors, or if we have reached this application's limits
      var freeWorkers = (0 until numUsable).filter(canLaunchExecutor)
      while (freeWorkers.nonEmpty) {
        freeWorkers.foreach { pos =>
          var keepScheduling = true
          while (keepScheduling && canLaunchExecutor(pos)) {
            coresToAssign -= minCoresPerExecutor
            assignedCores(pos) += minCoresPerExecutor
  
            // If we are launching one executor per worker, then every iteration assigns 1 core
            // to the executor. Otherwise, every iteration assigns cores to a new executor.
            if (oneExecutorPerWorker) {
              assignedExecutors(pos) = 1
            } else {
              assignedExecutors(pos) += 1
            }
  
            // Spreading out an application means spreading out its executors across as
            // many workers as possible. If we are not spreading out, then we should keep
            // scheduling executors on this worker until we use all of its resources.
            // Otherwise, just move on to the next worker.
            if (spreadOutApps) {
              keepScheduling = false
            }
          }
        }
        freeWorkers = freeWorkers.filter(canLaunchExecutor)
      }
      assignedCores
    }
  ```

## 任务调度

以count算子为例，在RDD.scala中找到count算子，runJob中一直寻找下去，留意以下checkpoint

```scala
def runJob[T, U: ClassTag](
      rdd: RDD[T],
      func: (TaskContext, Iterator[T]) => U,
      partitions: Seq[Int],
      resultHandler: (Int, U) => Unit): Unit = {
    if (stopped.get()) {
      throw new IllegalStateException("SparkContext has been shutdown")
    }
    val callSite = getCallSite
    val cleanedFunc = clean(func)
    logInfo("Starting job: " + callSite.shortForm)
    if (conf.getBoolean("spark.logLineage", false)) {
      logInfo("RDD's recursive dependencies:\n" + rdd.toDebugString)
    }
    dagScheduler.runJob(rdd, cleanedFunc, partitions, callSite, resultHandler, localProperties.get)
    progressBar.foreach(_.finishAll())
    rdd.doCheckpoint()
  }
```

在runJob向下寻找，关注submitJob中的post方法

```scala
def submitJob[T, U](
      rdd: RDD[T],
      func: (TaskContext, Iterator[T]) => U,
      partitions: Seq[Int],
      callSite: CallSite,
      resultHandler: (Int, U) => Unit,
      properties: Properties): JobWaiter[U] = {
    // Check to make sure we are not launching a task on a partition that does not exist.
    val maxPartitions = rdd.partitions.length
    partitions.find(p => p >= maxPartitions || p < 0).foreach { p =>
      throw new IllegalArgumentException(
        "Attempting to access a non-existent partition: " + p + ". " +
          "Total number of partitions: " + maxPartitions)
    }

    val jobId = nextJobId.getAndIncrement()
    if (partitions.size == 0) {
      // Return immediately if the job is running 0 tasks
      return new JobWaiter[U](this, jobId, 0, resultHandler)
    }

    assert(partitions.size > 0)
    val func2 = func.asInstanceOf[(TaskContext, Iterator[_]) => _]
    val waiter = new JobWaiter(this, jobId, partitions.size, resultHandler)
    eventProcessLoop.post(JobSubmitted(
      jobId, rdd, func2, partitions.toArray, callSite, waiter,
      SerializationUtils.clone(properties)))
    waiter
  }
```

```scala
/**
   * Put the event into the event queue. The event thread will process it later.
   */
  def post(event: E): Unit = {
    eventQueue.put(event)
  }
```



