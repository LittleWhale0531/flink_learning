# [一、入门](https://nightlies.apache.org/flink/flink-docs-release-1.16/zh/docs/dev/table/sql/gettingstarted/)

#### 1、source表
Flink的查询操作， 不基于本地管理静态数据；相反，它的查询在外部表上连续运行。Flink 数据处理流水线开始于 source 表，这些表可能是 Kafka 的 topics，数据库，文件系统，或者任何其它 Flink 知道如何消费的系统。
Flink 支持不同的**连接器**和**格式**相结合以定义表。下面是一个示例，定义一个以 CSV 文件作为存储格式的 source 表，其中 emp_id，name，dept_id 作为 CREATE 表语句中的列。
```
CREATE TABLE employee_information (
    emp_id INT,
    name VARCHAR,
    dept_id INT
) WITH ( 
    'connector' = 'filesystem',
    'path' = '/path/to/something.csv',
    'format' = 'csv'
);
```
可以从该表中定义一个连续查询
```
SELECT * from employee_information WHERE dept_id = 1;
```
#### 2、连续查询
Flink SQL 与传统数据库查询的不同之处在于，Flink SQL 持续消费到达的行并对其结果进行更新。

一个连续查询永远不会终止，并会产生一个动态表作为结果。动态表是 Flink 中 Table API 和 SQL 对流数据支持的核心概念。
```
SELECT 
   dept_id,
   COUNT(*) as emp_count 
FROM employee_information 
GROUP BY dept_id;
```

#### 3、Sink表
当运行此查询时，SQL 客户端实时但是以只读方式提供输出。存储结果，作为报表或仪表板的数据来源，需要写到另一个表。这可以使用 INSERT INTO 语句来实现。本节中引用的表称为 sink 表。INSERT INTO 语句将作为一个独立查询被提交到 Flink 集群
```
INSERT INTO department_counts
SELECT 
   dept_id,
   COUNT(*) as emp_count 
FROM employee_information;
```

# [二、join](https://nightlies.apache.org/flink/flink-docs-release-1.16/zh/docs/dev/table/sql/queries/joins/)

Flink SQL 支持动态表上的复杂和灵活的连接操作。有几种不同类型的联接可以满足各种各样的语义查询的需要。

#### 1、Regular Joins 普通连接
返回一个受连接条件限制的结果
```
SELECT * FROM Orders
INNER JOIN Product -- 还支持LEFT JOIN、RIGHT JOIN、FULL OUTER JOIN
ON Orders.productId = Product.id
```
ps：对于流计算，它要求双方永远在 Flink 保持联合输入。因此，计算查询结果所需的状态可能会无限增长，这取决于所有输入表和中间连接结果的不同输入行的数量。您可以提供具有适当状态生存时间(TTL)的查询配置，以防止状态大小过大。

#### 2、Interval Joins 间隔连接器
返回一个受连接条件和时间约束限制的结果
```
SELECT *
FROM Orders o, Shipments s
WHERE o.id = s.order_id
AND o.order_time BETWEEN s.ship_time - INTERVAL '4' HOUR AND s.ship_time
```
以下是有效的连接条件示例
```
ltime = rtime
ltime >= rtime AND ltime < rtime + INTERVAL '10' MINUTE
ltime BETWEEN rtime - INTERVAL '10' SECOND AND rtime + INTERVAL '5' SECOND
```
与常规连接相比，间隔连接只支持带有 time attributes的表。由于时间属性是单调递增的，Flink 可以在不影响结果正确性的前提下去除状态中的旧值。

#### 3、Temporal Joins 时间连接
Temporal table在 Flink 中称为动态表。时态表中的行与一个或多个时态周期相关联，并且所有 Flink 表都是时态的(动态的)
*1）Event Time Temporal Join 事件时间时态联接*
时态联接接受任意一个表(左侧输入) ，并将每一行与版本化表(右侧输入)中相应行的相关版本关联起来。Flink 使用 FOR SYSTEM _ TIME AS OF 的 SQL 语法完成这个操作。
时态连接的语法如下;
```
SELECT [column_list]
FROM table1 [AS <alias1>]
[LEFT] JOIN table2 FOR SYSTEM_TIME AS OF table1.{ proctime | rowtime } [AS <alias2>]
ON table1.column-name1 = table2.column-name1
```
*2）Processing Time Temporal Join 处理时间时态连接*
根据定义，使用处理时间属性，join 将始终返回给定键的最新值。

```
SELECT
  o_amount, r_rate
FROM
  Orders,
  LATERAL TABLE (Rates(o_proctime))
WHERE
  r_currency = o_currency
```
Orders是一个表,Rates也是一个表

*3）Temporal Table Function Join 时态表函数连接*
使用时态表函数连接表的语法与使用表函数连接表的语法相同。
假设 Rate 是一个时态表函数，连接可以用 SQL 表示如下:
```
SELECT
  o_amount, r_rate
FROM
  Orders,
  LATERAL TABLE (Rates(o_proctime))
WHERE
  r_currency = o_currency
```

其余语法与SQL相似，不赘述
