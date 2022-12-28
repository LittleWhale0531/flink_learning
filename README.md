# flink_learning
Apache Flink 文档学习笔记 https://nightlies.apache.org/flink/flink-docs-release-1.16/zh/

## [1_概念](https://github.com/LittleWhale0531/flink_learning/blob/main/1_%E6%A6%82%E5%BF%B5.md)
1）在catlog中创建表，

2）查询表，

3）输出表，

4）解释表
## [2_流式概念](https://github.com/LittleWhale0531/flink_learning/blob/main/2_%E6%B5%81%E5%BC%8F%E6%A6%82%E5%BF%B5.md)
1）状态管理

2）动态表

3）时间属性

4）时态表
## [3_流式聚合和数据类型](https://github.com/LittleWhale0531/flink_learning/blob/main/3_%E6%B5%81%E5%BC%8F%E8%81%9A%E5%90%88%E5%92%8C%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.md)
1)聚合调优方式（mini-batch、Local-Global、拆分distinct)，

2)构造数据类型（array、map、multiset、row)
## [4_flinkSQL](https://github.com/LittleWhale0531/flink_learning/blob/main/4_flinkSQL.md)
1)入门（source表、连续查询、sink表），

2）join方式（普通join，间隔器连接，时间连接）

## [5_自定义函数](https://github.com/LittleWhale0531/flink_learning/blob/main/5_%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0.md)
主要看 标量函数 与 表值函数 的开发方法；注意自动类型推导@DataTypeHint 和 @FunctionHint；

1）标量函数需要扩展 org.apache.flink.table.functions 里面的 ScalarFunction，参数和返回值类型可以是数据类型里列出的任何数据类型

2）表值函数需要扩展 org.apache.flink.table.functions 里面的 TableFunction，表值函数的求值方法本身不包含返回类型，而是通过 collect(T) 方法来发送要输出的行
