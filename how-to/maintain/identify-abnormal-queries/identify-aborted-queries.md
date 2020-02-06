---
title: 定位消耗系统资源多的查询
category: how-to
---

# expensive query 日志

TiDB 会将执行时间超过 tidb_expensive_query_time_threshold（默认值为 60s），或使用内存超过 [mem-quota-query](https://pingcap.com/docs-cn/stable/reference/configuration/tidb-server/configuration-file/#mem-quota-query)（默认值为 32 GB）的语句输出到 [tidb-server 日志文件](https://pingcap.com/docs-cn/stable/reference/configuration/tidb-server/configuration-file/#logfile)（默认文件为：“tidb.log”）中，用于帮助用户在语句执行结束前定位由于执行时间过长或使用内存过多的语句，分析和解决语句执行的性能问题。

注意，expensive query 日志和[慢查询日志](https://pingcap.com/docs-cn/dev/how-to/maintain/identify-slow-queries/)的区别是，慢查询日志是在语句执行完后才打印，expensive query 日志可以将正在执行的语句其相关信息打印出来。当一条语句在执行过程中达到资源使用阈值时（执行时间/使用内存量），TiDB 会将即时的语句相关信息写入日志。

## 日志示例

```sql
[2020/02/05 15:32:25.096 +08:00] [WARN] [expensivequery.go:167] [expensive_query] [cost_time=60.008338935s] [wait_time=0s] [request_count=1] [total_keys=70] [process_keys=65] [num_cop_tasks=1] [process_avg_time=0s] [process_p90_time=0s] [process_max_time=0s] [process_max_addr=10.0.1.9:20160] [wait_avg_time=0.002s] [wait_p90_time=0.002s] [wait_max_time=0.002s] [wait_max_addr=10.0.1.9:20160] [stats=t:pseudo] [conn_id=60026] [user=root] [database=test] [table_ids="[122]"] [txn_start_ts=414420273735139329] [mem_max="1035 Bytes (1.0107421875 KB)"] [sql="insert into t select sleep(1) from t"]
```

## 字段含义说明

Query 基础信息：

* `cost_time`：日志打印时语句已经花费的执行时间。
* `stats`：语句涉及到的表或索引使用的统计信息版本。值为 pesudo 时表示无可用统计信息，需要对表或索引进行 analyze。
* `table_ids`：语句涉及到的表的 ID。
* `txn_start_ts`：事务的开始时间戳，也是事务的唯一 ID，可以用这个值在 TiDB 日志中查找事务相关的其他日志。
* `sql`：SQL 语句。

和内存使用相关的字段：

* `mem_max`：日志打印时语句已经使用的内存空间。该项使用两种单位标识内存使用量，分别为 Bytes 以及易于阅读的自适应单位。

和 SQL 执行的用户相关的字段：

* `user`：执行语句的用户名。
* `conn_id`：用户的链接 ID，可以用类似 `con:60026` 的关键字在 TiDB 日志中查找该链接相关的其他日志。
* `database`：执行语句时使用的 database。