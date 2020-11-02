# SparkIssues

Data Skewness:-
Let's say some key is getting repeated across the dataset i.e key is not distributed uniformly 

Data Skewness impact:-
1)Performance of Spark Job is impacted
2)Jobs runs for longer time than usual
3)Resouces allocated are not utilised
4) Most of the resoucres will be idle without performing any task
5)Although spark is used in distributed processing ,we will not get advantage of distribution processing

Data Skewness can be resolved:
If one the table is smalll then can do broadcast join


1)Repartition:
Increasing the number of partition for data distribution based on the key column

If on of the table is mid size table then while joining we cannot use broacast join,so will use salting
2)Salting:
Let's say some key is getting repeated across the dataset i.e key is not distributed uniformly .So in this case we add some prefix or some suffix to original key to make it distributed across other partitions


2)Below is the issue coming while spark job:
For heavy workloads it is recommended to increase spark.network.timeout to 800 seconds:

--conf spark.network.timeout=800

Kindly try Increasing spark.rpc.askTimeout from default 120 seconds to a higher value in Ambari UI -> Spark Configs -> spark2-defaults. Recommendation is to increase it to at least 480 seconds and restart the necessary services.possibly the Driver and Executer are not able to  get Heartbeat response in configured timeout. If you don’t want to do any cluster level change then you may try overriding this value in the job level.

 

For example:  spark-submit by adding --conf spark.rpc.askTimeout=600s while submitting the job
org.apache.spark.rpc.RpcTimeoutException: Cannot receive any reply in 120 seconds. This timeout is controlled by spark.rpc.askTimeout
	at org.apache.spark.rpc.RpcTimeout.org$apache$spark$rpc$RpcTimeout$$createRpcTimeoutException(RpcTimeout.scala:48)
	at org.apache.spark.rpc.RpcTimeout$$anonfun$addMessageIfTimeout$1.applyOrElse(RpcTimeout.scala:63)
	at org.apache.spark.rpc.RpcTimeout$$anonfun$addMessageIfTimeout$1.applyOrElse(RpcTimeout.scala:59)
	at scala.runtime.AbstractPartialFunction.apply(AbstractPartialFunction.scala:36)
	at scala.util.Failure$$anonfun$recover$1.apply(Try.scala:216)
	at scala.util.Try$.apply(Try.scala:192)
	at scala.util.Failure.recover(Try.scala:216)
	at scala.concurrent.Future$$anonfun$recover$1.apply(Future.scala:326)
	at scala.concurrent.Future$$anonfun$recover$1.apply(Future.scala:326)
	at scala.concurrent.impl.CallbackRunnable.run(Promise.scala:32)
	at org.spark_project.guava.util.concurrent.MoreExecutors$SameThreadExecutorService.execute(MoreExecutors.java:293)
	at scala.concurrent.impl.ExecutionContextImpl$$anon$1.execute(ExecutionContextImpl.scala:136)
	at scala.concurrent.impl.CallbackRunnable.executeWithValue(Promise.scala:40)
	at scala.concurrent.impl.Promise$DefaultPromise.tryComplete(Promise.scala:248)
	at scala.concurrent.Promise$class.complete(Promise.scala:55)
	at scala.concurrent.impl.Promise$DefaultPromise.complete(Promise.scala:153)
	at scala.concurrent.Future$$anonfun$map$1.apply(Future.scala:237)
	at scala.concurrent.Future$$anonfun$map$1.apply(Future.scala:237)
	at scala.concurrent.impl.CallbackRunnable.run(Promise.scala:32)
	at scala.concurrent.BatchingExecutor$Batch$$anonfun$run$1.processBatch$1(BatchingExecutor.scala:63)
	at scala.concurrent.BatchingExecutor$Batch$$anonfun$run$1.apply$mcV$sp(BatchingExecutor.scala:78)
	at scala.concurrent.BatchingExecutor$Batch$$anonfun$run$1.apply(BatchingExecutor.scala:55)
	at scala.concurrent.BatchingExecutor$Batch$$anonfun$run$1.apply(BatchingExecutor.scala:55)
	at scala.concurrent.BlockContext$.withBlockContext(BlockContext.scala:72)
	at scala.concurrent.BatchingExecutor$Batch.run(BatchingExecutor.scala:54)
	at scala.concurrent.Future$InternalCallbackExecutor$.unbatchedExecute(Future.scala:601)
	at scala.concurrent.BatchingExecutor$class.execute(BatchingExecutor.scala:106)
	at scala.concurrent.Future$InternalCallbackExecutor$.execute(Future.scala:599)
	at scala.concurrent.impl.CallbackRunnable.executeWithValue(Promise.scala:40)
	at scala.concurrent.impl.Promise$DefaultPromise.tryComplete(Promise.scala:248)
	at scala.concurrent.Promise$class.tryFailure(Promise.scala:112)
	at scala.concurrent.impl.Promise$DefaultPromise.tryFailure(Promise.scala:153)
	at org.apache.spark.rpc.netty.NettyRpcEnv.org$apache$spark$rpc$netty$NettyRpcEnv$$onFailure$1(NettyRpcEnv.scala:205)
	at org.apache.spark.rpc.netty.NettyRpcEnv$$anon$1.run(NettyRpcEnv.scala:239)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: java.util.concurrent.TimeoutException: Cannot receive any reply in 120 seconds
	... 8 more
