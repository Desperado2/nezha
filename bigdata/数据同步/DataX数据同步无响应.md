**出现原因**：由于Starrocks设定了查询超时时间，DataX数据同步使用流式数据读取，导致数据读取超过了数据库指定的查询超时时间，数据读取被中断，DataX没有报错，出现了Speed一直为0的情况。


**处理方法**：

1. 可以暂时将数据库的query_timout参数调大，保证数据同步时间不会超过该值。

```sql
set global query_timeout=3000;
```

2. 在当前SQL语句中设置query_timeout的值，详见：[https://docs.starrocks.com/zh-cn/latest/reference/System_variable](https://docs.starrocks.com/zh-cn/latest/reference/System_variable)

```sql
SELECT /*+ SET_VAR(query_timeout = 1) */ name FROM people ORDER BY name;
```



**具体说明:**

1. DataX的数据同步，采用的是使用java.sql.Statement从数据库拉取数据，并且将fetchSize设置成了Integer.MIN_VALUE, 该方式使用流数据接受方式，每次只从服务器接受部分数据，直到数据处理完毕。

源码如下：

```java
/**
* 任务初始化
*/

public void init() {
    this.originalConfig = super.getPluginJobConf();

    Integer userConfigedFetchSize = this.originalConfig.getInt(Constant.FETCH_SIZE);
    if (userConfigedFetchSize != null) {
        LOG.warn("对 mysqlreader 不需要配置 fetchSize, mysqlreader 将会忽略这项配置. 如果您不想再看到此警告,请去除fetchSize 配置.");
    }
	// 默认被设置为Integer.MIN_VALUE
    this.originalConfig.set(Constant.FETCH_SIZE, Integer.MIN_VALUE);

    this.commonRdbmsReaderJob = new CommonRdbmsReader.Job(DATABASE_TYPE);
    this.commonRdbmsReaderJob.init(this.originalConfig);
}


/**
* 任务调用
**/
 public void startRead(RecordSender recordSender) {
    int fetchSize = this.readerSliceConfig.getInt(Constant.FETCH_SIZE);

    this.commonRdbmsReaderTask.startRead(this.readerSliceConfig, recordSender,
            super.getTaskPluginCollector(), fetchSize);
}


/**
* a wrapped method to execute select-like sql statement .
*
* @param conn         Database connection .
* @param sql          sql statement to be executed
* @param fetchSize
* @param queryTimeout unit:second
* @return
* @throws SQLException
*/
public static ResultSet query(Connection conn, String sql, int fetchSize, int queryTimeout)
    throws SQLException {
    // make sure autocommit is off
    conn.setAutoCommit(false);
    // ResultSet.RTYPE_FORWORD_ONLY,只可向前滚动；
    // ResultSet.CONCUR_READ_ONLY,指定不可以更新 ResultSet 
    Statement stmt = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY,
                                          ResultSet.CONCUR_READ_ONLY);

    // 指定了fetchSize为Integer.MIN_VALUE
    stmt.setFetchSize(fetchSize);
    stmt.setQueryTimeout(queryTimeout);
    return query(stmt, sql);
}
```



2. 数据库中配置了数据的查询超时时间，Starrocks中该配置名称为query_timeout。默认值为300s。如果一个查询持续时间超过了该参数的值，数据库就会返回查询超时错误。

```sql
show variables like '%timeout%';


# 结果如下：
interactive_timeout	3600
net_read_timeout	60
net_write_timeout	60
new_planner_optimize_timeout	3000
query_delivery_timeout	300
query_timeout	300
tx_visible_wait_timeout	10
wait_timeout	28800
```



3. DataX未将该异常抛出，导致程序没有中止，实际数据库的查询已经结束，所有出现了Speed为0的现象。

```plain
2022-08-26 13:58:27.724 [job-0] INFO  StandAloneJobContainerCommunicator - Total 778208 records, 497121061 bytes | Speed 0B/s, 0 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 0.046s |  All Task WaitReaderTime 291.284s | Percentage 0.00%
2022-08-26 13:58:37.731 [job-0] INFO  StandAloneJobContainerCommunicator - Total 778208 records, 497121061 bytes | Speed 0B/s, 0 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 0.046s |  All Task WaitReaderTime 291.284s | Percentage 0.00%
```



4. 代码调试，当超过query_timeout时，抛出如下错误。

```plain
经DataX智能分析,该任务最可能的错误原因是:
com.alibaba.datax.common.exception.DataXException: Code:[DBUtilErrorCode-07], Description:[读取数据库数据失败. 请检查您的配置的 column/table/where/querySql或者向 DBA 寻求帮助.].  - 执行的SQL为: select id,coordinate,latitude,domain_name,uri,url,user_name,city_name,city,district,province,postcode,country,first_name,first_romanized_name,last_name,name_female,ssn,phone_number,email,date,year_data,month_data,day_of_week,pystr,random_element,random_letter,company,company_suffix,company_prefix,company_email,sentence,text,word from test_v7  具体错误信息为：com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Query exceeded time limit of 30 seconds
at com.alibaba.datax.common.exception.DataXException.asDataXException(DataXException.java:26)
at com.alibaba.datax.plugin.rdbms.util.RdbmsException.asQueryException(RdbmsException.java:81)
at com.alibaba.datax.plugin.rdbms.reader.CommonRdbmsReader$Task.startRead(CommonRdbmsReader.java:220)
at com.alibaba.datax.plugin.reader.mysqlreader.MysqlReader$Task.startRead(MysqlReader.java:81)
at com.alibaba.datax.core.taskgroup.runner.ReaderRunner.run(ReaderRunner.java:57)
at java.lang.Thread.run(Thread.java:748)
```



5. 测试用例如下：

测试表信息：

+ 字段数：34
+ 表数据量：12965900条
+ 表大小：9.074 GB

不同query_timeout测试结果如下：

| query_timeout值 | 成功导出数据量 |
| --- | --- |
| 30s | 77792 |
| 300s | 778208 |
| 3000s | 7821522 |


<font style="color:rgb(23, 43, 77);">job_json如下:

```json
{
  "core":{
    "transport": {
      "channel":{
        "speed":{
          "byte": 5242880
        }
      }
    }
  },
  "job": {
    "setting": {
      "speed": {
        "channel":1,
        "batchSize": 2048,
      },
      "errorLimit": {
        "record": 0,
        "percentage": 0.02
      }
    },
    "content": [
      {
        "reader": {
          "name": "mysqlreader",
          "parameter": {
            "username": "root",
            "password": "root",
            "column" : [
              "id","coordinate","latitude","domain_name","uri","url","user_name","city_name","city",
              "district","province","postcode","country","first_name","first_romanized_name","last_name",
              "name_female","ssn","phone_number","email","date","year_data","month_data","day_of_week","pystr",
              "random_element","random_letter","company","company_suffix","company_prefix","company_email","sentence",
              "text","word"
            ],
            "splitPk":"id",
            "connection": [
              {
                "table": ["test_v7"],
                "jdbcUrl": ["jdbc:mysql://192.168.20.213:9030/TEST?connectTimeout=60000000&socketTimeout=60000000"]
              }
            ]
          }
        },
        "writer": {
          "name": "txtfilewriter",
          "parameter": {
            "path": "E:\\opt",
            "fileName": "text_datax_export",
            "writeMode": "truncate",
            "dateFormat": "yyyy-MM-dd"
          }
        }
      }
    ]
  }
}
```

使用SQL设置变量的job如下

```json
{
  "core":{
    "transport": {
      "channel":{
        "speed":{
          "byte": 5242880
        }
      }
    }
  },
  "job": {
    "setting": {
      "speed": {
        "channel":1,
        "batchSize": 2048,
      },
      "errorLimit": {
        "record": 0,
        "percentage": 0.02
      }
    },
    "content": [
      {
        "reader": {
          "name": "mysqlreader",
          "parameter": {
            "username": "root",
            "password": "root",
            "connection": [
              {
                "querySql": [
                  "select /*+ SET_VAR(query_timeout = 100) */ id,coordinate,latitude,domain_name,uri,url,user_name,city_name,city,district,province,postcode,country,first_name,first_romanized_name,last_name,name_female,ssn,phone_number,email,date,year_data,month_data,day_of_week,pystr,random_element,random_letter,company,company_suffix,company_prefix,company_email,sentence,text,word from test_v7"
                ],
                "jdbcUrl": ["jdbc:mysql://192.168.20.213:9030/TEST"]
              }
            ]
          }
        },
        "writer": {
          "name": "txtfilewriter",
          "parameter": {
            "path": "E:\\opt",
            "fileName": "text_datax_export",
            "writeMode": "truncate",
            "dateFormat": "yyyy-MM-dd"
          }
        }
      }
    ]
  }
}

```

