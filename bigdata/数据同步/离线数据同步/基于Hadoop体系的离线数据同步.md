## 一、背景
随着业务的发展，系统进行了微服务的差分，导致数据越来越分散，很难进行一个完整的生命周期的数据查询，对于某些业务的需求支持变得越来越难，越来越复杂，也越来越难以进行职责划分。对着业务的发展，数据量越来越大之后，为了良好的业务支持，进行了分库分表，分库分表规则五花八门，一旦脱离了业务逻辑，很难确定某一条数据在哪个库哪个表。

基于这样的问题和情况，为了满足业务需求，很自然的就想到了使用大数据服务，将业务数据归集到一起，建立完整的数据仓库，便于数据的查询。



## 二、数据同步架构
为了追求简单和通用，由于自身的认识现在，选择了最标准的大数据架构，即基于Hadoop的大数据体现。整个集群采用三节点，通过CDH进行集群的部署和维护。

![](https://cdn.nlark.com/yuque/0/2024/png/167378/1733120214951-1c316ad8-ac10-4933-b796-ebef498b3d8a.png)

整个数据链路为：

通过Azkaban调用Spark应用，将数据从RDS同步到Hive，运营平台和报表系统采用Presto加速访问Hive的数据。



## 三、数据同步详细过程
数据同步采用Spark任务来进行，将任务打包之后，上传到Azkaban调度平台，使用Azkaban进行定时调度，完成T+1级别的数据同步工作。



数据同步代码示例:

```scala
object MarketMysqlToHiveEtl extends SparkHivePartitionOverwriteApplication{


  /**
   * 删除已存在的分区
   *
   * @param spark SparkSessions实例
   * @param date 日期
   * @param properties 数据库配置
   */
  def delete_partition(spark: SparkSession, properties:Properties, date: String):Unit={
    val odsDatabaseName = properties.getProperty("hive.datasource.ods")
    DropPartitionTools.dropPartitionIfExists(spark,odsDatabaseName,"ods_t_money_record","ds",date)
    DropPartitionTools.dropPartitionIfExists(spark,odsDatabaseName,"ods_t_account","ds",date)
  }



  /**
   * 抽取数据
   * @param spark SparkSession实例
   * @param properties 数据库配置
   * @param date 日期
   */
  def loadData(spark: SparkSession, properties:Properties, date: String): Unit ={
    // 删除历史数据，解决重复同步问题
    delete_partition(spark,properties,date)

    // 获取数据源配置
    val odsDatabaseName = properties.get("hive.datasource.ods")
    val dataSource = DataSourceUtils.getDataSourceProperties(FinalCode.MARKET_MYSQL_FILENAME,properties)

    var sql = s"select id,account_id,type,original_id,original_code,money,reason,user_type,user_id,organization_id," +
    s"create_time,update_time,detail,deleted,parent_id,counts,'${date}' AS ds from TABLENAME where date(update_time) ='${date}'"

    // 同步数据
    MysqlToHiveTools.readFromMysqlIncrement(spark,dataSource,sql.replace("TABLENAME","t_money_record"),
                                            s"${odsDatabaseName}.ods_t_money_record",SaveMode.Append,"ds")


    sql = s"select id,code,customer_code,name,mobile,type,organization_id,organization_name,create_time,update_time,deleted,status,customer_name," +
    s"customer_id,channel_type,nike_name,version,register_Time,'${date}' AS ds from TABLENAME where date(update_time) ='${date}'"
    MysqlToHiveTools.readFromMysqlIncrement(spark,dataSource,sql.replace("TABLENAME","t_account"),
                                            s"${odsDatabaseName}.ods_t_account",SaveMode.Append,"ds")
  }



  /**
   * 数据etl
   * @param spark SparkSession实例
   * @param SparkSession 数据库配置
   */
  def etl(spark: SparkSession, properties:Properties): Unit = {
    val sparkConf = spark.sparkContext.getConf
    // 获取同步的日期
    var lastDate = sparkConf.get("spark.etl.last.day", DateUtils.getLastDayString)
    val dateList = new  ListBuffer[String]()
    if(lastDate.isEmpty){
      // 未配置，设置为前一天
      lastDate = DateUtils.getLastDayString
    }
    if(lastDate.contains("~")){
      // 如果是时间段，获取时间段中的每一天，解析为时间list
      val dateArray = lastDate.split("~")
      DateUtils.findBetweenDates(dateArray(0), dateArray(1)).foreach(it => dateList.append(it))
    }else if(lastDate.contains(",")){
      // 如果是使用,分隔的多个日期，解析为时间list
      lastDate.split(",").foreach(it => dateList.append(it))
    }else{
      // 添加进时间列表
      dateList.append(lastDate)
    }
    // 循环同步每天的数据
    dateList.foreach(it =>  loadData(spark, properties, it))
  }


  def main(args: Array[String]): Unit = {
    job() {
      val sparkAndProperties = SparkUtils.get()
      val spark = sparkAndProperties.spark
      val properties = sparkAndProperties.properties
      // 调度任务
      etl(spark,properties)
    }
  }
}

```

删除Partition的代码示例：

```scala
object DropPartitionTools {


  /**
   * 删除指定的Partition
   * @param SparkSession实例
   * @param database数据库名称
   * @param table表名称
   * @param partitionKey 分区字段的名称
   * @param partitionValue 具体的分区值
   */
  def dropPartitionIfExists(spark: SparkSession, database: String, table: String, partitionKey: String, partitionValue:String): Unit ={

     val df = spark.sql(
       s"""
         | show tables in ${database} like '${table}'
         |""".stripMargin)

    if(df.count() > 0 ){
      // 表存在，删除分区
      spark.sql(
        s"""
           |ALTER TABLE  ${database}.${table} DROP  IF EXISTS  PARTITION (${partitionKey}='${partitionValue}')
           |""".stripMargin)
    }
  }


  /**
   * 删除Partition
   * @param SparkSession实例
   * @param database数据库名称
   * @param table表名称
   * @param partitionKey 分区字段的名称
   */
  def dropHistoryPartitionIfExists(spark: SparkSession, database: String, table: String, partitionKey: String): Unit ={

    val df = spark.sql(
      s"""
         | show tables in ${database} like '${table}'
         |""".stripMargin)

    if(df.count() > 0 ){
      // 表存在，删除历史分区，获取8天前的日期
      val sevenDay = DateUtils.getSomeLastDayString(8);
      spark.sql(
        s"""
           |ALTER TABLE  ${database}.${table} DROP  IF EXISTS  PARTITION (${partitionKey} ='${sevenDay}')
           |""".stripMargin)
    }
  }

}

```

从RDS到HIVE的代码示例：

```scala
object MysqlToHiveTools {


  /**
   * 从mysql抽取数据到hive -- 全量
   * @param spark spark实例
   * @param dataSource 数据库配置信息
   * @param tableName 抽取的数据库表名
   * @param destTableName 目标表名
   * @param mode 抽取的模式
   */
  def mysqlToHiveTotal(spark: SparkSession, dataSource: JSONObject,tableName: String, destTableName:String,mode: SaveMode, partition: String): Unit = {
     val sql = "(select * from " + tableName + ") as t"
     mysqlToHive(spark, dataSource, sql, destTableName, mode, partition)
  }


  /**
   * 从mysql抽取数据到hive -- 增量量
   * @param spark spark实例
   * @param dataSource 数据库配置信息
   * @param sql 抽取数据的SQL
   * @param destTableName 目标表名
   * @param mode 抽取的模式
   */
  def readFromMysqlIncrement(spark: SparkSession, dataSource: JSONObject,sql: String, destTableName:String,mode: SaveMode, partition: String): Unit = {
    mysqlToHive(spark, dataSource, sql, destTableName, mode, partition)
  }


  /**
   * 真正的抽取数据
   * @param spark spark实例
   * @param properties 数据库配置信息
   * @param sql 抽取数据的SQL
   * @param destTableName 目标表名
   * @param mode 抽取的模式
   */
  def mysqlToHive(spark: SparkSession, dataSource: JSONObject,sql: String, destTableName:String, mode: SaveMode, partition: String):Unit={
    val df = spark.read.format("jdbc")
      .option("url",dataSource.getString("url"))
      .option("driver",dataSource.getString("driver"))
      .option("fetchSize", 10000)
      .option("numPartitions",2)
      .option("dbtable",s"(${sql}) AS t")
      .option("user",dataSource.getString("user"))
      .option("password",dataSource.getString("password"))
      .load()
    if(partition == null || partition.isEmpty){
      df.write.format("parquet").mode(mode).saveAsTable(destTableName)
    }else{
      df.write.format("parquet").mode(mode).partitionBy("ds").saveAsTable(destTableName)
    }
  }
}

```

Spark Application代码示例

```scala
trait SparkHivePartitionOverwriteApplication extends Logging{


  def getProperties(): Properties ={
    val prop:Properties = new Properties()
    val inputStream = this.getClass.getClassLoader.getResourceAsStream("config.properties")
    prop.load(inputStream);
    prop
  }

  def job(appName: String = null,
          master: String = null)(biz: => Unit): Unit = {
    var spark: SparkSession = null
    System.setProperty("HADOOP_USER_NAME", "mapred")
    val prop:Properties = getProperties()
    if (null == appName) {
      spark = SparkSession.builder
        .config("spark.sql.parquet.writeLegacyFormat", true)
        .config("spark.sql.sources.partitionOverwriteMode","dynamic")
        .config("hive.exec.dynamic.partition.mode","nonstrict")
        .config("spark.sql.hive.convertMetastoreParquet",false)
        .enableHiveSupport
        .getOrCreate
      var sparkAndProperties = SparkAndProperties(spark, prop)
      SparkUtils.set(sparkAndProperties)
    } else {
      spark = SparkSession.builder.master(master).appName(appName)
        .config("spark.sql.parquet.writeLegacyFormat", true)
        .config("spark.sql.sources.partitionOverwriteMode","dynamic")
        .config("hive.exec.dynamic.partition.mode","nonstrict")
        .config("spark.sql.hive.convertMetastoreParquet",false)
        .config("spark.testing.memory","2147480000")
        .config("spark.driver.memory","2147480000")
        .enableHiveSupport.getOrCreate
      var sparkAndProperties = SparkAndProperties(spark, prop)
      SparkUtils.set(sparkAndProperties)
      SparkUtils.set(sparkAndProperties)
    }
    biz
    spark.stop()
    SparkUtils.remove()
  }

}

case class SparkAndProperties(spark: SparkSession,
                              properties: Properties)
```



## 四、配套生态
1. 自定义UDF函数

在使用的过程中，需要将表中的IP地址，解析为所在地的名称，这需要调用第三方的一个服务接口来完成，为了完成这个任务，定义了一个自定义UDF函数，进行解析。

a. 自定义UDF函数

```scala
object ParseIp  {
    def evaluate(ip: String):String= {
      // 具体的IP解析服务
      SplitAddress.getPlaceFromIp(ip)
   }
}

```

b. 使用自定义UDF函数

```scala
object TraceTmpEtl extends SparkHivePartitionOverwriteApplication{

  /**
   * 数据同步任务
   * @param spark sparkSession实例
   * @param properties 数据库配置
   * @param date 日期
   */
  def tmp_t_trace_user_visit_real_time_statistic(spark: SparkSession,properties:Properties,date: String):Unit ={
    // 获取数据库配置的数据库名称
    val odsDatabaseName = properties.get("hive.datasource.ods")
    val tmpDatabaseName = properties.get("hive.datasource.tmp")

    // 注册自定义的UDF函数
    spark.udf.register("parseIP", (ip: String) => SplitAddress.getPlaceFromIp(ip))
    // 在Spark SQL中使用UDF函数
    spark.sql(
      s"""
         |INSERT OVERWRITE TABLE ${tmpDatabaseName}.tmp_t_statistic partition(ds='${date}')
         |select
         |	  `id` ,
         |	  `create_time` ,
         |	  `update_time` ,
         |	  `ip` ,
         |      replace( replace( replace(replace( case when parseIP(ip) rlike '^中国' then replace(parseIP(ip),'中国','')
         |          when parseIP(ip) rlike '^内蒙古' then replace(parseIP(ip),'内蒙古','内蒙古自治区')
         |          when parseIP(ip) rlike '^广西' then replace(parseIP(ip),'广西','广西壮族自治区')
         |          when parseIP(ip) rlike '^西藏' then replace(parseIP(ip),'西藏','西藏自治区')
         |          when parseIP(ip) rlike '^宁夏' then replace(parseIP(ip),'宁夏','宁夏回族自治区')
         |          when parseIP(ip) rlike '^新疆' then replace(parseIP(ip),'新疆','新疆维吾尔自治区')
         |          when parseIP(ip) rlike '^香港' then replace(parseIP(ip),'香港','香港特别行政区')
         |          when parseIP(ip) rlike '^澳门' then replace(parseIP(ip),'澳门','澳门特别行政区')
         |     else parseIP(ip) end, "省", "省."),"市", "市."),"县", "县."),"区", "区.") as ip_place,
         |	  `page_view` 
         |from ${odsDatabaseName}.ods_t_statistic where ds ='${date}'
         |""".stripMargin)
  }

  /**
   * 数据etl
   * @param spark SparkSession实例
   * @param properties 数据库配置
   */
  def etl(spark: SparkSession, properties:Properties): Unit = {
    val lastDate = DateUtils.getLastDayString
    tmp_t_trace_user_visit_real_time_statistic(spark,properties, lastDate)
  }


  
  def main(args: Array[String]): Unit = {
    job() {
      val sparkAndProperties = SparkUtils.get()
      val spark = sparkAndProperties.spark
      val properties = sparkAndProperties.properties
      etl(spark,properties)
    }
  }
}

```

2. 数据库的配置安全性问题

刚开始数据库配置同步配置文件直接写死，但是后续发现这样存在一些安全性的问题，后来采用将数据库相关的配置组合为一个JSON字符串，将其加密之后保存到MongoDB中，在使用时进行查询解密。

```scala
public class DataSourceUtils {

    private  static Logger logger = LoggerFactory.getLogger(DataSourceUtils.class);

    public static JSONObject getDataSourceProperties(String dataSourceKey,Properties properties){
        List<ServerAddress> adds = new ArrayList<>();
        try {
            String filePath = properties.getProperty("spark.mongo.properties.file.url");
            properties = new Properties();
            File file = new File(filePath);
            FileInputStream inputStream = null;
             inputStream = new FileInputStream(file);
            properties.load(inputStream);
        }catch (Exception e){
            logger.info("not load file, reason：" + e.getMessage());
            e.printStackTrace();
        }
        String mongoUrl = properties.getProperty("mongo_url");
        String mongoPort = properties.getProperty("mongo_port");
        String mongoDbName = properties.getProperty("mongo_dbName");
        String mongoCollect = properties.getProperty("mongo_collect");
        String mongoUser = properties.getProperty("mongo_user");
        String mongoPassword = properties.getProperty("mongo_password");
        String desKey = properties.getProperty("data_des_key");
        ServerAddress serverAddress = new ServerAddress(mongoUrl, Integer.parseInt(mongoPort));
        adds.add(serverAddress);
        List<MongoCredential> credentials = new ArrayList<>();
        MongoCredential mongoCredential = MongoCredential.createScramSha1Credential(mongoUser, mongoDbName, mongoPassword.toCharArray());
        credentials.add(mongoCredential);
        MongoClient mongoClient = new MongoClient(adds, credentials);
        MongoDatabase mongoDatabase = mongoClient.getDatabase(mongoDbName);
        MongoCollection<Document> collection = mongoDatabase.getCollection(mongoCollect);
        //指定查询过滤器
        Bson filter = Filters.eq("key", dataSourceKey);
        //指定查询过滤器查询
        FindIterable findIterable = collection.find(filter);
        //取出查询到的第一个文档
        Document document = (Document) findIterable.first();
        //打印输出
        String content = DESUtil.decrypt(desKey, document.getString("content"));
        return JSON.parseObject(content);
    }


    public static  Properties json2Properties(JSONObject jsonObject){
        String tmpKey = "";
        String tmpKeyPre = "";
        Properties properties = new Properties();
        j2p(jsonObject, tmpKey, tmpKeyPre, properties);
        return properties;
    }



    private static void j2p(JSONObject jsonObject, String tmpKey, String tmpKeyPre, Properties properties){
        for (String key : jsonObject.keySet()) {
            // 获得key
            String value = jsonObject.getString(key);
            try {
                JSONObject jsonStr = JSONObject.parseObject(value);
                tmpKeyPre = tmpKey;
                tmpKey += key + ".";
                j2p(jsonStr, tmpKey, tmpKeyPre, properties);
                tmpKey = tmpKeyPre;
            } catch (Exception e) {
                properties.put(tmpKey + key, value);
                System.out.println(tmpKey + key + "=" + value);
            }
        }
    }
    public static void main(String[] args) {

    }
}

```

3. Spark任务脚本示例

```shell
#!/bin/sh

##### env ###########
export JAVA_HOME=/usr/java/jdk1.8.0_151
export SPARK_HOME=/opt/cloudera/parcels/CDH/lib/spark
export PATH=${JAVA_HOME}/bin:${SPARK_HOME}/bin:${PATH}
export SPARK_USER=hadoop
export HADOOP_USER_NAME=hadoop
LAST_DAY="$1"
echo LAST_DAY

spark-submit \
--class net.app315.bigdata.operatereport.ods.MarketMysqlToHiveEtl \
--conf spark.sql.hive.metastore.version=2.1.1 \
--conf spark.sql.hive.metastore.jars=/opt/cloudera/parcels/CDH/lib/hive/lib/* \
--jars /opt/cloudera/parcels/CDH/lib/spark/jars/mysql-connector-java-5.1.48.jar,/opt/cloudera/parcels/CDH/lib/spark/jars/druid-1.1.10.jar \
--master yarn \
--deploy-mode cluster \
--executor-memory 4G \
--driver-memory 2G \
--num-executors 4 \
--executor-cores 2 \
--conf spark.dynamicAllocation.minExecutors=1 \
--conf spark.dynamicAllocation.maxExecutors=8 \
--conf spark.yarn.am.attemptFailuresValidityInterval=1h \
--conf spark.yarn.max.executor.failures=128 \
--conf spark.yarn.executor.failuresValidityInterval=1h \
--conf spark.task.maxFailures=4 \
--conf spark.yarn.maxAppAttempts=2 \
--conf spark.scheduler.mode=FIFO \
--conf spark.network.timeout=420000 \
--conf spark.dynamicAllocation.enabled=true \
--conf spark.executor.heartbeatInterval=360000 \
--conf spark.sql.crossJoin.enabled=true \
--conf spark.mongo.properties.file.url=/opt/conf/mongo.properties \
--conf spark.etl.last.day="${LAST_DAY}" \
./target/spark-operate-report-project-1.0.jar
```

4. Job任务脚本实例

```plain
nodes:

  - name: bigdata_market_ods_etl
    type: command
    config:
      command: sh -x ./script/bigdata_market_ods_etl.sh "${spark.etl.last.day}"
      failure.emails: mxx@xxx.com

  - name: bigdata_market_dim_etl
    type: command
    config:
      command: sh -x ./script/bigdata_market_dim_etl.sh "${spark.etl.last.day}"
      failure.emails: mxx@xxx.com
    dependsOn:
          - bigdata_market_ods_etl
          
  - name: bigdata_market_dw_etl
    type: command
    config:
      command: sh -x ./script/bigdata_market_dw_etl.sh "${spark.etl.last.day}"
      failure.emails: mxx@xxx.com
    dependsOn:
          - bigdata_market_dim_etl
          - bigdata_user_dw_etl
```



## 五、备注
1. [Davinci报表](https://github.com/edp963/davinci)   一个开源的报表平台

