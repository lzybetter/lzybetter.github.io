---
title: Spark与TDengine连接总结
date: 2023-02-23 17:14:57
tags:
 - Spark
 - TDengine
---



TDengine是一款高性能的时序数据库，最近在研究如何在Spark中读取和写入数据，这里记录一下最近的成果。

> 版本信息：
>
> 1. Spark版本：2.4.0，yarn集群模式
> 2. Python版本：3.7.9
> 3. Scala版本：2.11.12
> 4. TDengine版本：2.6，商业版
> 5. TAOS-JDBC版本：2.0.42

## 1. 官方JDBC使用

TDengine官方提供了JDBC，Spark读取和写入均可以直接使用

### 1.1 依赖问题

1. 需要引用taos-jdbcdriver，版本上强烈推荐2.0.42，其他版本会有各种问题(针对2.x系列，3.x系列的没用过)；

```shell
<dependency>  
  <groupId>com.taosdata.jdbc</groupId>  
  <artifactId>taos-jdbcdriver</artifactId>  
  <version>2.0.42</version>  
</dependency>
```

2. taos-jdbcdriver的依赖与spark集群可能存在冲突(版本不同)，需要手动指定依赖，方法为：
   1. 下载正确的依赖，我用的是guava-30.1.1-jre.jar和failureaccess-1.0.1.jar 
   2. 将jar上传hadoop；
   3. 在提交spark任务的时候，手动制定依赖：


```shell
--conf spark.driver.extraClassPath=guava-30.1.1-jre.jar:failureaccess-1.0.1.jar \
--conf spark.executor.extraClassPath=guava-30.1.1-jre.jar:failureaccess-1.0.1.jar 
--jars hadoop-path-to-jar/guava-30.1.1-jre.jar,hadoop-path-to-jar/failureaccess-1.0.1.jar 
```

3. 对于pyspark程序，最好引用taos-jdbcdriver-2.0.42-dist.jar，补足依赖

```shell
--conf spark.driver.extraClassPath=guava-30.1.1-jre.jar:failureaccess-1.0.1.jar:taos-jdbcdriver-2.0.42-dist.jar 
--conf spark.executor.extraClassPath=guava-30.1.1-jre.jar:failureaccess-1.0.1.jar:taos-jdbcdriver-2.0.42-dist.jar 
--jars hadoop-path-to-jar/guava-30.1.1-jre.jar,hadoop-path-to-jar/failureaccess-1.0.1.jar,hadoop-path-to-jar/taos-jdbcdriver-2.0.42-dist.jar
```



### 1.2 读取

Spark通过TDBC读取TDengine数据和其他数据库并没有什么本质区别，可以通过以下方法来读取：

1. 读取整张表

```Python
info = spark.read\  
    .format("jdbc")\  
    .option("driver", "com.taosdata.jdbc.rs.RestfulDriver")\  
    .option("url", "jdbc:TAOS-RS://ip:port/db?user=user&password=password")\  
    .option("dbtable", "test.test")\  
    .load() 
```

2. 读取部分内容

 ```python
info = spark.read\  
    .format("jdbc")\  
    .option("driver", "com.taosdata.jdbc.rs.RestfulDriver")\  
    .option("url", "jdbc:TAOS-RS://ip:port/db?user=user&password=password")\  
    .option("query", "select * from test.test")\  
    .load() 
 ```

> 如果读取的数量较多，可能会遇到timeout错误，可以设置httpSocketTimeout，单位是ms，默认值为5000

### 1.3 存储

可以使用下面的方法向TDengine保存数据

```python
  toHbaseSave
    .write
    .format("jdbc")
    .option("url", "jdbc:TAOS-RS://ip:port/db?user=user&password=password")
    .option("driver", "com.taosdata.jdbc.rs.RestfulDriver")
    .mode(SaveMode.Append)
    .option("dbtable", "test.test")
    .save()
```



## 2. 使用自定义数据源写入数据

对于spark来说，直接使用官方的jdbc进行连接只能实现向普通表中写入数据，无法利用TDengine超级表的特性，为了能向超级表中写入数据，需要自定义数据源。

由于只需要写入特性，因此下面的内容中只实现了写入的逻辑，没有实现读取的逻辑，使用scala编写。

1. 继承DataSourceV2和WriteSupport 并重写createWriter方法，创建自定义的数据源；
```scala
class TDSourceV2 extends DataSourceV2  with WriteSupport with Serializable {  
  
  override def createWriter(
                             jobId: String,  
                             structType: StructType,  
                             saveMode: SaveMode,  
                             dataSourceOptions: DataSourceOptions): Optional[DataSourceWriter] = {  
    Optional.of(new TDSourceWriter(  
	// url
      dataSourceOptions.get("url").get(),
    // 用户名  
      dataSourceOptions.get("user").get(),  
	// 密码
      dataSourceOptions.get("password").get(),  
	// 数据库名称
      dataSourceOptions.get("db").get(),  
	// 超级表名称
      dataSourceOptions.get("stable").get()))  
  }
```
2. 继承 DataSourceWriter 重写 createWriterFactory 方法并返回自定义的 DataWriterFactory，重写 commit 方法，用来提交整个事务, 重写 abort 方法，用来做事务回滚；
```scala
class TDSourceWriter(  
                      url: String,  
                      user: String,  
                      password: String,  
                      db: String,  
                      stable: String) extends DataSourceWriter with Serializable{  
                      
  override def createWriterFactory(): DataWriterFactory[InternalRow] = {  
    new TDWriterFactory(url, user, password, db, stable)  
  }  
  // 2.6版本的TDengine不支持事务
  override def commit(writerCommitMessages: Array[WriterCommitMessage]): Unit = Unit  
  // 2.6版本的TDengine不支持事务
  override def abort(writerCommitMessages: Array[WriterCommitMessage]): Unit = Unit  
}
```
3. 继承 DataWriterFactory, 重写 createDataWriter方法返回自定义的 DataWriter；
```scala
class TDWriterFactory(  
                       url: String,  
                       user: String,  
                       password: String,  
                       db: String,  
                       stable: String) extends DataWriterFactory[InternalRow] with Serializable {  
  override def createDataWriter(  
                                 partitionId: Int,  
                                 taskId: Long,  
                                 epochId: Long): DataWriter[InternalRow] = {  
    new TDDataWriter(url, user, password, db, stable)  
  }  
}
```
4. 继承 DataWriter 重写 write 方法实现具体的写入数据库逻辑，重写 commit 方法用来提交事务，重写 abort 方法用来做事务回滚 ；
```scala
class TDDataWriter(  
                    url: String,  
                    user: String,  
                    password: String,  
                    db: String,  
                    stable: String) extends DataWriter[InternalRow] with Serializable {  
  
  private val logger = LoggerFactory.getLogger(this.getClass)  
  private var conn: Connection = null  
  private var stmt: Statement = null  
  
  override def write(record: InternalRow): Unit = {  
  
	// 在这里编写将数据插入数据库的逻辑
	
    Class.forName("com.taosdata.jdbc.rs.RestfulDriver")  
    val jdbcUrl = s"jdbc:TAOS-RS://$url/$db?user=$user&password=$password"  
    
	// 可以通过record.getxxx获取对应列的内容
	// 如table_name = record.getString(0) 
	
    val variable1 = record.getString(0)  
    val variable2 = record.getInt(1)
    val variable3 = record.getFloat(2)
        
	// 构建插入数据库的sql语句
	// 请注意，只有原生连接方式才支持参数绑定的插入方式，
	// 由于要在spark集群上跑，这里用的是Rest的连接方式，
	// 所以只能手动构建插入数据库的sql语句，
	// 关于原生连接、Rest连接和参数绑定插入，请见官方文档
	
    val sql = "insert to ..."
  
    logger.info(sql)  
  
    conn = DriverManager.getConnection(jdbcUrl)  
    stmt = conn.createStatement()  
    stmt.execute(s"use $db")  
  
    try{  
      stmt.execute(sql)  
      conn.commit()  
    }catch {  
      case e: Exception => e.printStackTrace()  
    }finally {  
      conn.close()  
    }  
  }  

  // Tdengine2.6版本不支持事务，因此此处并无实际需要做的事情
  // 创建WriterCommitMessage类，绝对不能传null，网上有些代码是错误的
  object WriteSucceeded extends WriterCommitMessage  
  override def commit(): WriterCommitMessage = WriteSucceeded 
   
  // Tdengine2.6版本不支持事务，因此此处并无实际需要做的事情
  override def abort(): Unit = Unit  
}
```

5. 调用，在程序中指定自定义的datasource并传入参数
```scala
toTdSave  
  .write  
  .format("org.example.TDSourceV2")  
  .mode(SaveMode.Append)  
  // url
  .option("url", url)  
  // 数据库名
  .option("db", db)  
  // 用户名
  .option("user", user)  
  // 超级表名
  .option("stable", stable)  
  // 密码
  .option("password", password)  
  .save()
```


参考文献：
1. [暑期2021项目经验分享：实现Spark对接openGauss](https://www.modb.pro/db/132365)
2. [Spark DataSource V1 & V2 API 一文理解](https://blog.csdn.net/penriver/article/details/115672072)
3. [Spark SQL DataSource V2 学习入门 + 代码模板](http://www.jsledd.cn/2019/04/05/datasourcev2/)
4. [Tdengine v2.6 官方文档](https://docs.taosdata.com/2.6/reference/connector/java/)