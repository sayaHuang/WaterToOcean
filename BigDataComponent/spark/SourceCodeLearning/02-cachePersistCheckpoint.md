[TOC]
# cache() persist() checkpoint()
首先三者都是做 RDD 持久化的
**使用场景** 将共用的或者重复使用的RDD按照持久化的级别进行缓存

**cache()**
cache 底层调用的是 persist 方法，存储等级为: memory only

**persist()**
persist 的默认存储级别也是 memory only, persist 与 cache 的主要区别是 persist 可以自定义存储级别

```scala
object StorageLevel {
  val NONE = new StorageLevel(false, false, false, false)
  val DISK_ONLY = new StorageLevel(true, false, false, false)
  val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
  val DISK_ONLY_3 = new StorageLevel(true, false, false, false, 3)
  val MEMORY_ONLY = new StorageLevel(false, true, false, true)
  val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
  val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
  val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
  val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
  val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
  val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
  val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
  val OFF_HEAP = new StorageLevel(true, true, true, false, 1)
```

## checkpoint()
**使用场景** 将业务非常长的逻辑计算的中间结果缓存到HDFS上

**原理**
1. 在代码中，当使用SparkContext可以设置一个checkpointFile文件目录，比如HDFS文件目录。
2. 在代码中对需要checkpoint的RDD调用checkpoint方法。
3. RDDCheckpointData（spark内部的API），接管你的RDD，会标记为marked for checkpointing，准备进行checkpoint。
4. 你的job运行完之后，会调用一个finalRDD.doCheckpoint()方法，会顺着rdd lineage，回溯扫描，发现有标记为待checkpoint的rdd，就会进行二次标记，标记为checkpointing in progress，正在接受checkpoint操作。
5. job执行完之后，就会启动一个内部的新job，去将标记为checkpointing in progress的rdd的数据，都写入hdfs文件中。（如果rdd之前cache过，会直接从缓存中获取数据，写入hdfs中；如果没有cache过，那么就会重新计算一遍这个rdd，再checkpoint）
6. 将checkpoint过的rdd之前的依赖rdd，改成一个CheckpointRDD*，强制改变你的rdd的lineage。后面如果rdd的cache数据获取失败，直接会通过它的上游CheckpointRDD，去容错的文件系统，比如hdfs，中，获取checkpoint的数据。

## 对比
### cache VS checkpoint
cache()进行持久化操作，但是当某个节点或者executor挂掉之后，持久化的数据会丢失，因为我们的数据是保存在内存当中的，这时就会重新根据血线计算RDD

### persist VS checkpoint
persist和 checkpoint 之间的区别：persist()可以将 RDD 的 partition 持久化到磁盘，但该 partition 由 blockManager 管理。一旦 driver program 执行结束，也就是 executor 所在进程 CoarseGrainedExecutorBackend stop，blockManager 也会 stop，被 cache 到磁盘上的 RDD 也会被清空（整个 blockManager 使用的 local 文件夹被删除）。而 checkpoint 将 RDD 持久化到 HDFS 或本地文件夹，如果不被手动 remove 掉，是一直存在的，也就是说可以被下一个 driver program 使用，而 cached RDD 不能被其他 dirver program 使用。


## 读取存储的 RDD
当我们对某个RDD进行了缓存操作之后，首先会去CacheManager中去找，然后紧接着去BlockManager中去获取内存或者磁盘中缓存的数据，如果没有进行缓存或者缓存丢失，那么就会去checkpoint的容错文件系统中查找数据，如果最终没有找到，那就会按照RDD lineage重新计算。

## 源码解读
### 业务代码调用
```scala
scala> val rdd = sc.parallelize(List("test1","test2","test3"))
rdd: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[0] at parallelize at <console>:24

scala> sc.setCheckpointDir("/Users/saya/Downloads/")
//最好在此处增加 rdd.cache
scala> rdd.checkpoint
scala> rdd.count
res4: Long = 3

scala> rdd.persist
res6: rdd.type = ParallelCollectionRDD[0] at parallelize at <console>:24

scala> rdd.cache
res7: rdd.type = ParallelCollectionRDD[0] at parallelize at <console>:24
```
1. `sc.setCheckpointDir("/Users/saya/Downloads/")` 执行完之后download 目录下出现文件夹`ca501833-452b-42f9-8ad5-2b1372a92b18`
2. `rdd.count` 执行之后,  /Users/saya/Downloads/ 路径下出现备份数据

<div align="center">
<img src=./image/Snipaste_2022-04-14_22-29-07.jpg  width=60%  height=60%/>
</div>
###  解读
#### checkpoint
```scala
//SparkContext类
class SparkContext(config: SparkConf) extends Logging {
//1. local 模式, 可以接收本地 path
//2. 非 local 模式, 如果是本地地址抛出 logwarning, 如果是 hdfs 地址则不进入 if 语句,继续往下执行
  /**
   * Set the directory under which RDDs are going to be checkpointed.
   * @param directory path to the directory where checkpoint files will be stored
   * (must be HDFS path if running in cluster)
   */
  def setCheckpointDir(directory: String): Unit = {

    // If we are running on a cluster, log a warning if the directory is local.
    // Otherwise, the driver may attempt to reconstruct the checkpointed RDD from
    // its own local file system, which is incorrect because the checkpoint files
    // are actually on the executor machines.
    if (!isLocal && Utils.nonLocalPaths(directory).isEmpty) {
      logWarning("Spark is not running in local mode, therefore the checkpoint directory " +
        s"must not be on the local filesystem. Directory '$directory' " +
        "appears to be on the local filesystem.")
    }

    checkpointDir = Option(directory).map { dir =>
      val path = new Path(dir, UUID.randomUUID().toString)
      val fs = path.getFileSystem(hadoopConfiguration)
      //创建用于缓存的文件夹
      fs.mkdirs(path)
      fs.getFileStatus(path).getPath.toString
    }
  }
}

//RDD 类
//This function must be called before any job has been executed on this RDD.
//这个方法要在 action 方法之前调用
// It is strongly recommended that this RDD is persisted in memory, otherwise saving it on a file will require recomputation.
//特别建议这个 rdd 已经被缓存在内存中了, 如果没有的话需要重新计算

abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging {
    private[spark] var checkpointData: Option[RDDCheckpointData[T]] = None
  
  /**
   * Mark this RDD for checkpointing. It will be saved to a file inside the checkpoint
   * directory set with `SparkContext#setCheckpointDir` and all references to its parent
   * RDDs will be removed. This function must be called before any job has been
   * executed on this RDD. It is strongly recommended that this RDD is persisted in
   * memory, otherwise saving it on a file will require recomputation.
   */
  def checkpoint(): Unit = RDDCheckpointData.synchronized {
    // NOTE: we use a global lock here due to complexities downstream with ensuring
    // children RDD partitions point to the correct parent partitions. In the future
    // we should revisit this consideration.
    if (context.checkpointDir.isEmpty) {
      throw new SparkException("Checkpoint directory has not been set in the SparkContext")
    } else if (checkpointData.isEmpty) {
      checkpointData = Some(new ReliableRDDCheckpointData(this))
    }
  }
}
```


方法主要代码 `checkpointData = Some(new ReliableRDDCheckpointData(this))`
checkpointData的类型为`Option[RDDCheckpointData[T]] `

ReliableRDDCheckpointData 是 RDDCheckpointData的子类
**ReliableRDDCheckpointData的作用**
ReliableRDDCheckpointData包含与 RDD 检查点相关的所有信息。 此类的每个实例都与一个 RDD 相关联。
ReliableRDDCheckpointData 将 RDD 数据写入可靠存储的检查点。
这允许以先前计算的状态在失败时重新启动驱动程序。

**这段代码的作用**
对RDD调用`checkpoint`函数，其实就是初始化了`checkpointData`，并不立即执行checkpint操作，你可以理解成这里只是对RDD进行checkpint标记操作。  


```scala
//RDD 类, 调用 count() 方法
abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging {
  
  def count(): Long = sc.runJob(this, Utils.getIteratorSize _).sum
  //最后会调用到下面这个runJob 方法

  /**
   * Run a function on a given set of partitions in an RDD and pass the results to the given
   * handler function. This is the main entry point for all actions in Spark.
   *
   * @param rdd target RDD to run tasks on
   * @param func a function to run on each partition of the RDD
   * @param partitions set of partitions to run on; some jobs may not want to compute on all
   * partitions of the target RDD, e.g. for operations like `first()`
   * @param resultHandler callback to pass each result to
   */
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
    rdd.doCheckpoint()//继续追下去
  }
  
  //doCheckpoint() 方法会在这个 RDD 的父 RDD 中递归调用
    /**
   * Performs the checkpointing of this RDD by saving this. It is called after a job using this RDD
   * has completed (therefore the RDD has been materialized and potentially stored in memory).
   * doCheckpoint() is called recursively on the parent RDDs.
   */
private[spark] def doCheckpoint(): Unit = {
  RDDOperationScope.withScope(sc, "checkpoint", allowNesting = false, ignoreParent = true) {
    if (!doCheckpointCalled) {
      doCheckpointCalled = true
      if (checkpointData.isDefined) {
        checkpointData.get.checkpoint()//继续追下去
      } else {
        dependencies.foreach(_.rdd.doCheckpoint())
      }
    }
  }
}


private[spark] abstract class RDDCheckpointData[T: ClassTag](@transient private val rdd: RDD[T])
  /**
   * Materialize this RDD and persist its content.
   * This is called immediately after the first action invoked on this RDD has completed.
   */
  final def checkpoint(): Unit = {
    // Guard against multiple threads checkpointing the same RDD by
    // atomically flipping the state of this RDDCheckpointData
    //为了防止多个线程对同一个RDD进行checkpint操作，首先是把checkpint的状态由Initialized变成CheckpointingInProgress，所以如果另一个线程发现checkpint的状态不是Initialized就直接return了。
    RDDCheckpointData.synchronized {
      if (cpState == Initialized) {
        cpState = CheckpointingInProgress
      } else {
        return
      }
    }

    val newRDD = doCheckpoint() //继续追下去

    // Update our state and truncate the RDD lineage
    //切断了血线
    RDDCheckpointData.synchronized {
      cpRDD = Some(newRDD)
      cpState = Checkpointed
      rdd.markCheckpointed()
    }
  }
}

private[spark] class ReliableRDDCheckpointData[T: ClassTag](@transient private val rdd: RDD[T])
  extends RDDCheckpointData[T](rdd) with Logging {
  /**
   * Materialize this RDD and write its content to a reliable DFS.
   * This is called immediately after the first action invoked on this RDD has completed.
   */
  protected override def doCheckpoint(): CheckpointRDD[T] = {
    //在这个方法中将数据写入本地目录, 继续追下去
    val newRDD = ReliableCheckpointRDD.writeRDDToCheckpointDirectory(rdd, cpDir)

    // Optionally clean our checkpoint files if the reference is out of scope
    if (rdd.conf.get(CLEANER_REFERENCE_TRACKING_CLEAN_CHECKPOINTS)) {
      rdd.context.cleaner.foreach { cleaner =>
        cleaner.registerRDDCheckpointDataForCleanup(newRDD, rdd.id)
      }
    }

    logInfo(s"Done checkpointing RDD ${rdd.id} to $cpDir, new parent is RDD ${newRDD.id}")
    newRDD
  }
}


/**
 * An RDD that reads from checkpoint files previously written to reliable storage.
 */
private[spark] class ReliableCheckpointRDD[T: ClassTag](
    sc: SparkContext,
    val checkpointPath: String,
    _partitioner: Option[Partitioner] = 
  /**
   * Write RDD to checkpoint files and return a ReliableCheckpointRDD representing the RDD.
   */
  def writeRDDToCheckpointDirectory[T: ClassTag](
      originalRDD: RDD[T],
      checkpointDir: String,
      blockSize: Int = -1): ReliableCheckpointRDD[T] = {
    .....
    
      // Save to file, and reload it as an RDD
      val broadcastedConf = sc.broadcast(
        new SerializableConfiguration(sc.hadoopConfiguration))
      // TODO: This is expensive because it computes the RDD again unnecessarily (SPARK-8582)
    //又跑了一遍 job, 很昂贵, 也标记了 TODO, 继续追下去
      sc.runJob(originalRDD,
        writePartitionToCheckpointFile[T](checkpointDirPath.toString, broadcastedConf) _)

      if (originalRDD.partitioner.nonEmpty) {
        writePartitionerToCheckpointDir(sc, originalRDD.partitioner.get, checkpointDirPath)
      }
    .....
  }
  
    /**
   * Write an RDD partition's data to a checkpoint file.
   */
  def writePartitionToCheckpointFile[T: ClassTag](
          path: String,
      broadcastedConf: Broadcast[SerializableConfiguration],
      blockSize: Int = -1)(ctx: TaskContext, iterator: Iterator[T]): Unit = {
    ....
    Utils.tryWithSafeFinallyAndFailureCallbacks {
      //将数据写入本地
      serializeStream.writeAll(iterator)
    } (catchBlock = {
      val deleted = fs.delete(tempOutputPath, false)
      if (!deleted) {
        logInfo(s"Failed to delete tempOutputPath $tempOutputPath.")
      }
    }, finallyBlock = {
      serializeStream.close()
    })
    ....
  }
}
```

#### persist
```scala
abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging {

....

/**
   * Mark this RDD for persisting using the specified level.
   *
   * @param newLevel the target storage level
   * @param allowOverride whether to override any existing level with the new one
   */
  private def persist(newLevel: StorageLevel, allowOverride: Boolean): this.type = {
    // TODO: Handle changes of StorageLevel
    if (storageLevel != StorageLevel.NONE && newLevel != storageLevel && !allowOverride) {
      throw new UnsupportedOperationException(
        "Cannot change storage level of an RDD after it was already assigned a level")
    }
    // If this is the first time this RDD is marked for persisting, register it
    // with the SparkContext for cleanups and accounting. Do this only once.
    if (storageLevel == StorageLevel.NONE) {
      sc.cleaner.foreach(_.registerRDDForCleanup(this))
      sc.persistRDD(this)
    }
    storageLevel = newLevel
    this
  }
  

  /**
   * Set this RDD's storage level to persist its values across operations after the first time
   * it is computed. This can only be used to assign a new storage level if the RDD does not
   * have a storage level set yet. Local checkpointing is an exception.
   */
  def persist(newLevel: StorageLevel): this.type = {
    if (isLocallyCheckpointed) {
      // This means the user previously called localCheckpoint(), which should have already
      // marked this RDD for persisting. Here we should override the old storage level with
      // one that is explicitly requested by the user (after adapting it to use disk).
      persist(LocalRDDCheckpointData.transformStorageLevel(newLevel), allowOverride = true)
    } else {
      persist(newLevel, allowOverride = false)
    }
  }

  /**
   * Persist this RDD with the default storage level (`MEMORY_ONLY`).
   */
  def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)

  /**
   * Persist this RDD with the default storage level (`MEMORY_ONLY`).
   */
  def cache(): this.type = persist()


  /**
   * Mark the RDD as non-persistent, and remove all blocks for it from memory and disk.
   *
   * @param blocking Whether to block until all blocks are deleted (default: false)
   * @return This RDD.
   */
  def unpersist(blocking: Boolean = false): this.type = {
    logInfo(s"Removing RDD $id from persistence list")
    sc.unpersistRDD(id, blocking)
    storageLevel = StorageLevel.NONE
    this
  }
}
```