## 一、背景
自从上次开始使用基于Hadoop的大数据体现方案之后，业务平稳发展，但是随着时间的推移，新的问题开始出现，主要出现的问题为两个：

1. 数据的变更越来越频繁，基于之前SparkSQL任务的方式，只要需要对表结构进行变更，就需要重新修改Scala代码，然后重新进行任务的打包，这对于一些不熟悉代码的人来说，不太友好，而且成本也很高。
2. 虽然使用了Presto对HIVE的数据查询进行了加速，但是所在数据量越来越大，分析要求越来越复杂，即席查询越来越多，由于集群本身资源有限，查询能力出现了显著瓶颈。



## 二、数据同步架构
随着技术的发展已经对大数据的认识，接触到了更多的大数据相关的知识与组件，基于此，通过认真分析与思考之后，对数据的同步方案进行了如下的重新设计。

1. 数据存储与查询放弃了HDFS+HIVE+Presto的组合，转而采用现代化的MPP数据库StarRocks，StarRocks在数据查询的效率层面非常优秀，在相同资源的情况下，可以解决目前遇到的数据查询瓶颈。
2. 数据同步放弃了SparkSQL，转而采用更加轻量级的DATAX来进行，其只需要通过简单的配置，即可完成数据的同步，同时其也支持StarRocks Writer，开发人员只需要具备简单的SQL知识，就可以完成整个数据同步任务的配置，难度大大降低，效率大大提升，友好度大大提升。
3. 定时任务调度放弃Azkaban，采用现代化的任务调度工作Apache DolphinScheduler，通过可视化的页面进行调度任务工作流的配置，更加友好。

![](https://cdn.nlark.com/yuque/0/2024/png/167378/1733124174997-72513213-01a4-4409-93e1-fe65d93f8bb2.png)

## 三、数据同步的详细流程
数据同步在这种方式下变动非常简单，只需要可视化的配置DataX任务，即可自动调度。下面的一个任务的配置示例

```json
{
  "job": {
    "setting": {
      "speed": {
        "channel":1
      }
    },
    "content": [
      {
        "reader": {
          "name": "mysqlreader",
          "parameter": {
            "username": "",
            "password": "",
            "connection": [
              {
                "querySql": [
                  "SELECT CustomerId AS customer_id FROM base_info.base_customer where date(UpdateTime) > '${sdt}' and date(UpdateTime) < '${edt}'"
                ],
                "jdbcUrl": [
                  "jdbc:mysql://IP:3306/base_info?characterEncoding=utf-8&useSSL=false&tinyInt1isBit=false"
                ]
              }
            ]
          }
        },
        "writer": {
          "name": "starrockswriter",
          "parameter": {
            "username": "xxx",
            "password": "xxx",
            "database": "ods_cjm_test",
            "table": "ods_base_customer",
            "column": ["id"],
            "preSql": [],
            "postSql": [], 
            "jdbcUrl": "jdbc:mysql://IP:9030/",
            "loadUrl": ["IP:8050", "IP:8050", "IP:8050"],
            "loadProps": {
              "format": "json",
              "strip_outer_array": true
            }
          }
        }
      }
        ]
    }
}
```



数据同步过程中，遇到了另外一个问题，即业务存在大量的分库分表的，这些分库分表的逻辑五花八门，60张左右的逻辑板，经过分库分表之后达到了惊人的5000多张，为每张表配置任务很显然不太正常，这就需要能够在进行数据同步的时候动态生成需要的表列表，把表列表配置到DataX的配置文件中去。

经过技术的调用，Apache DolphinScheduler的Python任务类型很适合做这个事情，由于公司本身使用了Apache DolphinScheduler3.0的版本，其Python任务还不支持返回数据到下游节点，但是社区最新版本已经支持该能力，因为按照已实现版本对其进行改造。

改造之后，Python节点能够将数据传递给他的下游节点，因此使用Python脚本查询获取需要进行同步的表列表，将其传递给DataX节点，完成动态表的数据同步

```python
import pymysql
import datetime


def select_all_table(date: str):
    result_list = []
    sql = """
    SELECT concat('"', table_name, '"') 
    FROM information_schema.`TABLES` 
    WHERE table_schema='hydra_production_flow' 
        and table_name like 't_package_flow_log_%'
        and table_name like '%_{}'
    """.format(date)
    conn = pymysql.connect(host='', port=3306, user='', passwd='',
                           db='information_schema')
    cur = conn.cursor()
    cur.execute(query=sql)
    while 1:
        res = cur.fetchone()
        if res is None:
            break
        result_list.append(res[0])
    cur.close()
    conn.close()
    return result_list


if __name__ == '__main__':
    # 获取当前年月
    # 获取当前日期
    today = datetime.date.today()
    # 计算前一天的日期
    yesterday = today - datetime.timedelta(days=1)
    current_date = yesterday.strftime("%Y_%m")
    table_list = select_all_table(current_date)
    table_str = ",".join(table_list)
    # 设置变量,传递给下游节点
    print('${setValue(table_list=%s)}' % table_str)
```

```json
{
  "job": {
    "setting": {
      "speed": {
        "channel":1
      }
    },
    "content": [
      {
        "reader": {
          "name": "mysqlreader",
          "parameter": {
            "username": "xxx",
            "password": "xxxx",
            "column": [
              "id",
              "concat('t_package_flow_log_',DATE_FORMAT(create_time,'%Y_%m'))",
              "operation_type"
            ],
            "where": "date(create_time) ${operator_symbol} '${dt}'",
            "connection": [
              {
                "table": [
                  ${table_list}
                ],
                "jdbcUrl": [
                  "jdbc:mysql://xx:3306/hydra_production_flow?characterEncoding=utf-8&useSSL=false&tinyInt1isBit=false"
                ]
              }
            ]
          }
        },
        "writer": {
                    "name": "starrockswriter",
                    "parameter": {
                        "username": "xxxxxx",
                        "password": "xxxxxxx",
                        "database": "ods_cjm",
                        "table": "ods_t_package_flow_log",
                        "column": ["id", "table_name","operation_type"],
                        "preSql": [],
                        "postSql": [], 
                        "jdbcUrl": "jdbc:mysql://IP:9030/",
                        "loadUrl": ["IP:8050", "IP:8050", "IP:8050"],
                        "loadProps": {
                            "format": "json",
                            "strip_outer_array": true
                        }
                    }
                }
            }
        ]
    }
}
```



## 四、踩坑记录
1. DATAX只支持python2.x

下载支持python3.x的相关文件，替换DataX中的相同文件，即可支持python3.x使用



## 五、备注
1. [StarRocks](https://www.starrocks.io/)    高性能的MPP数据库
2. [DataX](https://github.com/alibaba/DataX)  离线数据同步
3. [Apache DolphinScheduler](https://github.com/apache/dolphinscheduler)  任务调度工具

