**#HiveIssues**

**Connection timeout issue from hive beeline:**

1)Checked the hive server2 log and came to know there was error saying failed to connect hive metastore 

2)then check hive metastore log and found that there is deadlock error at mysql 

exception is : txn.TxnHandler deadlock detected in acquireLock(Initiator)

Refernce:
https://community.cloudera.com/t5/Support-Questions/Hive-Metastore-Lock-wait-timeout-exceeded/td-p/161774

**Getting Null pointer issue while running query from hive beeline**
org.apache.hive.service.cli.HiveSQLException: Error while compiling statement: FAILED: NullPointerException null
	at org.apache.hive.jdbc.Utils.verifySuccess(Utils.java:300)
	at org.apache.hive.jdbc.Utils.verifySuccessWithInfo(Utils.java:286)
	at org.apache.hive.jdbc.HiveStatement.runAsyncOnServer(HiveStatement.java:324)
	at org.apache.hive.jdbc.HiveStatement.execute(HiveStatement.java:265)
	at org.apache.commons.dbcp2.DelegatingStatement.execute(DelegatingStatement.java:291)
	at org.apache.commons.dbcp2.DelegatingStatement.execute(DelegatingStatement.java:291)
	at org.apache.zeppelin.jdbc.JDBCInterpreter.executeSql(JDBCInterpreter.java:718)
	at org.apache.zeppelin.jdbc.JDBCInterpreter.interpret(JDBCInterpreter.java:801)
	at org.apache.zeppelin.interpreter.LazyOpenInterpreter.interpret(LazyOpenInterpreter.java:103)
	at org.apache.zeppelin.interpreter.remote.RemoteInterpreterServer$InterpretJob.jobRun(RemoteInterpreterServer.java:633)
	at org.apache.zeppelin.scheduler.Job.run(Job.java:188)
	at org.apache.zeppelin.scheduler.ParallelScheduler$JobRunner.run(ParallelScheduler.java:162)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.hive.service.cli.HiveSQLException: Error while compiling statement: FAILED: NullPointerException null
	at org.apache.hive.service.cli.operation.Operation.toSQLException(Operation.java:348)
	at org.apache.hive.service.cli.operation.SQLOperation.prepare(SQLOperation.java:199)
	at org.apache.hive.service.cli.operation.SQLOperation.runInternal(SQLOperation.java:262)
	at org.apache.hive.service.cli.operation.Operation.run(Operation.java:260)
	at org.apache.hive.service.cli.session.HiveSessionImpl.executeStatementInternal(HiveSessionImpl.java:575)
	at org.apache.hive.service.cli.session.HiveSessionImpl.executeStatementAsync(HiveSessionImpl.java:561)
	at org.apache.hive.service.cli.CLIService.executeStatementAsync(CLIService.java:315)
	at org.apache.hive.service.cli.thrift.ThriftCLIService.ExecuteStatement(ThriftCLIService.java:566)
	
1)After searching what we found that hive maintains its cache and found that hive cache is corrupted.

2)Resolution is to set the hive property as 

hive.query.results.cache.enabled=false

The above issue is also bug.Below is the url:

http://mail-archives.apache.org/mod_mbox/hive-issues/201910.mbox/%3CJIRA.13263860.1571769590000.20563.1571769960025@Atlassian.JIRA%3E

b) In metastore log,it is showing Host is blocked because of many connections error
Resolution:Run -p flush-hosts command in mysql and increase no of max connections.
# SparkIssues

spark.sql.shuffle.partition=5
Importance of above property
By default Spark creates 200 partitions over a table which might  not even be required. By setting shuffle partitions configuration to, say 5, you are asking it to set the number of table partitions to that value. 

Now this can obviously be explicitly changed by coalition or repartitioning. What AQE provides is an advantage here.

When AQE is enabled, spark decides the post-shuffle partition size and number automatically based on the tables involved in the shuffle and broadcast exchange.
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

 '=Few fundamentals about Hive Part 1:-
1. you can check the hive default configs under the file hive-site.xml.The location is /etc/hive/conf , you can see this file here.

2. when you are in hive command line you can type : set;
with this we will get a bunch of default properties to see on the screen, if we want we can overwrite these properties using the command
set propertyname = value;

3. The above properties that we set using the set command are valid only for that session. if you want to specify the properties permanently then we need to create a file .hiverc under the home directory.
cd ~
gedit .hiverc
in this file now mention the above properties.
now once we login to give we can see these properties are applied.

4. logging level for hive can be changed by editing the file hive-log4j.properties.This is found under the path /etc/hive/conf.

5. Also the log files can be found under /tmp/<username>
As we are logging in with the user cloudera,so the log files are present in /tmp/cloudera.we can see log files under this.

6. In hive type: set fs.defaultFS;
this will tell to which hdfs your hive will talk to
for cloudera it will show:
fs.defaultFS=hdfs://quickstart.cloudera:8020



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



Checkpointing ,State Store and Watermark in Structured Streaming
============================================

CHECKPOINTING

Checkpoint dir is a physical location which serves 2 purposes:-
1)Stores the current state of application (eg :-lets say in file streaming application 3 of 5 files have been processed)
2)Aggregated values (eg- the state of running total in streaming application)

Checkpointing is purposed to recover the state in case of streaming application failure hence provides fault tolerance.


STATE STORE

=> In case of Spark streaming application the state of aggregation is stored in state store(in executors).
=>The same state is also maintained by checkpoint dir.
=>For the fast look up application refers to the State store (which in is memory) but not check point dir(which stores in disk)
=>If the state store in not cleaned on regular basis we may experience a out of memory exception.

WATERMARKS

=>Watermarks comes to our rescue in order to clean the state store
=>It is way to treat the late arriving records in spark streaming application by imposing some expiry date.
=> we can define a specific time after which the late arriving records will not be considered to be stored and window prior to that
will be removed from state store.
