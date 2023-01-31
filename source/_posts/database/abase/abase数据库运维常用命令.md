---
title: abase数据库运维常用命令
author: zohar
top: false
cover: false
toc: true
date: 2023-01-30 18:27:31
tags:
- abase
- postgre
categories:
- 数据库
---

# 前言

随着生产环境下abase数据库越来越多，使用过程中会面临下面几个问题，本文针对这些问题进行一些总结。

1. 数据库出现性能问题如何排查
2. 日常需要进行哪些数据库维护操作
3. 数据库备份恢复该使用哪种方式

[TOC]

# 一、性能问题排查

## 1.1 查询锁等待        

```sql
-- 查询锁等待的进程号
select * from pg_locks where granted='f';
-- 查询是哪个进程阻塞的
select pg_blocking_pids(进程号);
-- 多次查询上述命令，直到结果为{},确定最初是哪个进程导致的问题。
-- 查看进程正在执行的sql
select * from pg_stat_activity where pid=进程号;
```

## 1.2 查询耗cpu高的sql（实时）      

postgres 数据库是多进程数据库，每个数据库连接是单独的进程。可以通过linux的命令查看耗cpu高的进程。

如top命令，按c可以看进程的更多信息。

```shell
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND               
29992 appuser   20   0 16.661g 787440 768960 R  99.7  0.1   0:45.98 postgres: sa abase [local] SELECT  
```

或使用pidstat命令，获取占用cpu高的进程。

```
pidstat -u 5 1 
平均时间:   UID       PID    %usr %system  %guest    %CPU   CPU  Command
平均时间:  1007     29992   89.63    9.00    0.00   98.63     -  postgres
平均时间:     0     32765    0.20    0.20    0.00    0.39     -  java
平均时间:  1007     42047    0.39    1.37    0.00    1.76     -  pidstat
平均时间:  1007     42062   36.59    3.72    0.00   40.31     -  postgres
```

得到耗cpu的进程pid后，到数据库中查看具体执行的sql语句。 

```sql
abase=# select pid,usename,application_name,client_addr,state,query from pg_stat_activity where pid = 29992;
  pid  | usename | application_name | client_addr | state  |               query               
-------+---------+------------------+-------------+--------+-----------------------------------
 29992 | sa      | psql             |             | active | select * from ywst.t1,ywst.t1 t2;
```

## 1.3 查询耗io高的sql（实时）    

使用pidstat命令，获取占用io高的进程。  

```shell
pidstat -d 5 1 
10时22分38秒   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
10时22分43秒  1007      4196      0.00      1.56      0.00  postgres
10时22分43秒  1007      4209      0.00      1.56      0.00  postgres
10时22分43秒  1007      4211      0.00     23.44     15.62  postgres
10时22分43秒  1007     18947      0.00      1.56      0.00  postgres
10时22分43秒  1007     18948      0.00      3.12      0.00  postgres
10时22分43秒  1007     18949      0.00      3.12      0.00  postgres
10时22分43秒  1007     18950      0.00      3.12      0.00  postgres
10时22分43秒  1007     18964      0.00      1.56      0.00  postgres

平均时间:   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
平均时间:  1007      4196      0.00      1.56      0.00  postgres
平均时间:  1007      4209      0.00      1.56      0.00  postgres
平均时间:  1007      4211      0.00     23.44     15.62  postgres
平均时间:  1007     18947      0.00      1.56      0.00  postgres
平均时间:  1007     18948      0.00      3.12      0.00  postgres
平均时间:  1007     18949      0.00      3.12      0.00  postgres
平均时间:  1007     18950      0.00      3.12      0.00  postgres
平均时间:  1007     18964      0.00      1.56      0.00  postgres
```

得到进程号后，到数据库中查看具体执行的sql语句。与确定耗cpu的sql方法一致。

## 1.4 查询慢sql（历史） 

### 方法一 设置数据库参数

设置log_min_duration_statement参数，在sql执行时间超过参数值时记录到数据库日志中。

```sql
-- 记录所有类型执行时间超过3秒的sql
alter system set log_statement = 'all';
alter system set log_min_duration_statement='3000';
```
### 方法二 使用pg_stat_statements插件

插件会对数据库性能带来一定影响（影响不大），但记录的信息会很详细，包括sql执行次数，平均执行时间等信息。

```sql
-- 修改配置文件，注意shared_preload_libraries参数若已有值，则增加pg_stat_statements，用逗号分隔。
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.max = 10000
pg_stat_statements.track = all
-- 重启数据库后，创建插件。
create extension pg_stat_statements;
-- 查询慢sql
select query, calls, total_time, (total_time/calls) as average, rows, 100.0 * shared_blks_hit/nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent 
from pg_stat_statements 
order by average desc limit 10;
-- 清空历史慢sql记录
select pg_stat_statements_reset();
```

## 1.5 断开数据库会话进程        

```sql
-- 方法1：数据库中执行，推荐方式。下面2个命令区别是cancle取消查询语句，不会断开会话，但有时会不成功。terminate是中断会话，回滚未提交事务。
select pg_cancle_backend(进程号);
select pg_terminate_backend(进程号);  
-- 方法2：操作系统下执行，严禁执行Kill -9 进程号，否则会导致数据库postmaster主进程重启。仅当数据库无法连接时使用，仅可针对数据库会话进程执行。
kill 进程号
```

## 1.6 批量断开某类数据库会话进程        

```sql
-- 显示正在执行查询语句（sql以select开头）的进程信息 ，并断开进程。
select pg_terminate_backend(pid),* from pg_stat_activity where state='active' and lower(query) like 'select%' and pid <> pg_backend_pid();
```

或操作系统下执行，仅当数据库无法正常连接时使用。

```shell
-- 如下列命令会杀掉全部6543端口数据库下的select进程
ps -ef|grep `head -1 /tmp/.s.PGSQL.6543.lock`|grep SELECT|awk '{print $2}'|xargs kill
```

## 1.7 查看sql执行计划    

```sql
-- 查看查询语句执行计划
explain select * from t;
-- 查看查询语句执行计划，并显示实际执行情况。增加analyze选项，sql语句会被实际执行。
explain analyze select * from t;
-- 查看更新/删除/插入语句执行计划，并显示实际执行情况。
begin;
explain analyze update/delete/insert;
rollback;		
```

## 1.8 查询表上有哪些索引 

查询pg_indexes获取表上有哪些索引

```sql
abase=# select * from pg_indexes where schemaname='msaj' and tablename='t_ms_aj_jc';
 schemaname | tablename  |    indexname     | tablespace |                               indexdef
------------+------------+------------------+------------+-----------------------------------------------------------------------
 msaj       | t_ms_aj_jc | i_ms_aj_jc_jbfy4 |            | CREATE INDEX i_ms_aj_jc_jbfy4 ON msaj.t_ms_aj_jc USING btree (c_jbfy)
 msaj       | t_ms_aj_jc | i_ms_aj_jc_jbfy2 |            | CREATE INDEX i_ms_aj_jc_jbfy2 ON msaj.t_ms_aj_jc USING btree (c_jbfy)
 msaj       | t_ms_aj_jc | i_ms_aj_jc_jbfy1 |            | CREATE INDEX i_ms_aj_jc_jbfy1 ON msaj.t_ms_aj_jc USING btree (c_jbfy)
 msaj       | t_ms_aj_jc | i_ms_aj_jc_jbfy  |            | CREATE INDEX i_ms_aj_jc_jbfy ON msaj.t_ms_aj_jc USING btree (c_jbfy)
```

查询pg_index获取表上有哪些索引，以及索引状态等信息。修改WHERE c.oid='msaj.t_ms_aj_jc'::regclass 部分的模式名表名。

```sql
SELECT c2.relname, i.indisprimary, i.indisunique, i.indisclustered, i.indisvalid, pg_catalog.pg_get_indexdef(i.indexrelid, 0, true),
  pg_catalog.pg_get_constraintdef(con.oid, true), contype, condeferrable, condeferred, i.indisreplident, c2.reltablespace
FROM pg_catalog.pg_class c, pg_catalog.pg_class c2, pg_catalog.pg_index i
  LEFT JOIN pg_catalog.pg_constraint con ON (conrelid = i.indrelid AND conindid = i.indexrelid AND contype IN ('p','u','x'))
WHERE c.oid='msaj.t_ms_aj_jc'::regclass AND c.oid = i.indrelid AND i.indexrelid = c2.oid
ORDER BY i.indisprimary DESC, i.indisunique DESC, c2.relname;
     relname      | indisprimary | indisunique | indisclustered | indisvalid |                             pg_get_indexdef                             | pg_get_constraintdef | contype | condeferrable | condeferred | indisreplident | reltablespace
------------------+--------------+-------------+----------------+------------+-------------------------------------------------------------------------+----------------------+---------+---------------+-------------+----------------+---------------
 t_ms_aj_jc_pkey  | t            | t           | f              | t          | CREATE UNIQUE INDEX t_ms_aj_jc_pkey ON msaj.t_ms_aj_jc USING btree (id) | PRIMARY KEY (id)     | p       | f             | f           | f              |             0
 i_ms_aj_jc_jbfy  | f            | f           | f              | f          | CREATE INDEX i_ms_aj_jc_jbfy ON msaj.t_ms_aj_jc USING btree (c_jbfy)    |                      |         |               |             | f              |             0
 i_ms_aj_jc_jbfy1 | f            | f           | f              | f          | CREATE INDEX i_ms_aj_jc_jbfy1 ON msaj.t_ms_aj_jc USING btree (c_jbfy)   |                      |         |               |             | f              |             0
 i_ms_aj_jc_jbfy2 | f            | f           | f              | f          | CREATE INDEX i_ms_aj_jc_jbfy2 ON msaj.t_ms_aj_jc USING btree (c_jbfy)   |                      |         |               |             | f              |             0
 i_ms_aj_jc_jbfy4 | f            | f           | f              | t          | CREATE INDEX i_ms_aj_jc_jbfy4 ON msaj.t_ms_aj_jc USING btree (c_jbfy)   |                      |         |               |             | f              |             0
```

## 1.9 创建/重建索引   

创建索引和重建 的语法。若生产环境执行，需要增加CONCURRENTLY关键字，可以避免对表的独占锁锁定。reindex暂时不支持CONCURRENTLY并行方式（postgres12版本新增功能，abase6.0对应postgres11.1不支持），可以并行方式新建一个索引，然后删除旧索引。

```sql
-- 语法
CREATE [ UNIQUE ] INDEX [ CONCURRENTLY ] [ [ IF NOT EXISTS ] name ] ON table_name [ USING method ]
    ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass ] [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] )
    [ WITH ( storage_parameter = value [, ... ] ) ]
    [ TABLESPACE tablespace_name ]
    [ WHERE predicate ]

REINDEX [ ( VERBOSE ) ] { INDEX | TABLE | SCHEMA | DATABASE | SYSTEM } name

-- 例子
create index concurrently i_ms_aj_jc_jbfy on msaj.t_ms_aj_jc using btree (c_jbfy);
-- 创建索引前可以临时设置较大的maintenance_work_mem加快索引创建速度，仅当前会话生效。
set maintenance_work_mem='2GB';
create index concurrently i_ms_aj_jc_jbfy on msaj.t_ms_aj_jc using btree (c_jbfy);
```

## 1.10 临时禁用某索引     

```sql
-- 修改索引是否有效字段为false，禁用索引。禁用时新数据依然会维护这个索引，一般用来调试SQL执行计划。
update pg_index set indisvalid=false where indexrelid='模式名.索引名'::regclass;

-- 修改索引是否有效字段为true，恢复索引。
update pg_index set indisvalid=true where indexrelid='模式名.索引名'::regclass;
```

# 二、其他日常维护操作

## 2.1 更新统计信息

更新统计信息``analyze``命令，会在表级别增加``ShareUpdateExclusiveLock``锁。不影响正常的dml操作，若有必要可以在工作时间段执行。建议每周非业务时间执行一次。

```sql
-- 更新单张表统计信息
analyze 表名;
-- 更新数据库下所有表统计信息
analyze;
-- 更新统计信息前可以临时设置较大的maintenance_work_mem加快分析速度。
set maintenance_work_mem='2GB';
set max_parallel_maintenance_workers = 8;
analyze;
```

## 2.2 整理碎片

整理碎片``vacuum ful``l命令，会在表级别增加``AccessExclusiveLock``锁。影响表的其他访问，必须在非工作时段执行。整理碎片会自动重建表上的索引，建议一个月执行一次。需要观察执行耗时，若耗时较长可分多个会话并行执行（每个客户端整理部分表）。

```sql
-- 整理单表碎片，并更新统计信息。
vacuum full analyze 表名;
-- 对数据库下所有表进行碎片整理，并更新统计信息
vacuum full analyze;
-- 整理碎片前可以临时设置较大的maintenance_work_mem加快整理速度。
set maintenance_work_mem='2GB';
set max_parallel_maintenance_workers = 8;
vacuum full analyze;
```

## 2.3 清理表内历史版本数据
与``vacuum full``不同，vacuum仅对表内历史版本数据进行清理，并不回收空间。执行时会在表级别增加``ShareUpdateExclusiveLock``锁。不影响正常的dml操作，若有必要可以在工作时间段执行。建议每周非业务时间执行一次。

```sql
-- 清理单张表
vacuum 表名;
-- 对数据库下所有表进行清理
vacuum;
-- 清理前可以临时设置较大的maintenance_work_mem加快清理速度。
set maintenance_work_mem='2GB';
set max_parallel_maintenance_workers = 8;
vacuum analyze;
```

## 2.4 冻结表内事物id

abase数据库根据行内隐藏列xmin、xmax记录的事物id，以及事物的提交记录判断记录的可见性。而事物id是32位的整数，40亿+循环使用。为了避免事物无法判断新旧（即事物回卷），数据库需要讲过老的事物id进行冻结。表的年龄可以认为是表上最早事物id距离当前事物id的距离。默认情况当表年龄达到2亿，自动运行autovacuum进程时，会触发``vacuum freeze``操作。

``vacuum freeze``会在表级别增加``ShareUpdateExclusiveLock``，行级别增加``RowExclusiveLock``，对系统影响较大。因此我们需要定期监控表年龄增长情况，及时手工运行``vacuum freeze;``命令。

```sql
-- 冻结单张表
vacuum freeze 表名;
-- 对数据库下所有表进行清理
vacuum freeze;
-- 清理前可以临时设置较大的maintenance_work_mem加快清理速度。
set maintenance_work_mem='2GB';
set max_parallel_maintenance_workers = 8;
vacuum freeze;
```

 <font color=#ff0000 face="黑体">**注：若未定期检查，导致生产数据库表年龄达到2亿，触发了autovacuum的freeze操作，影响数据库使用时。可临时修改autovacuum_freeze_max_age为更大的值，如4亿，修改后重启数据库生效。这样数据库暂时就不会触发freeze操作。待周末非工作时间，手工执行``vacuum freeze``冻结事物id。冻结后，需要将autovacuum_freeze_max_age改回2亿。** </font> 

## 2.5 查看年龄过大的表

当表年龄超过1亿时，在非工作时间执行``vacuum freeze;``冻结表上事物id。可以运行下列语句，查看年龄大于1亿的表。

```sql
select current_database(), 
    nspname, 
    case relkind when $$r$$ then $$ordinary table$$ when $$t$$ then $$toast table$$ end as relkind,
    relname,
    age(relfrozenxid), 
    case when (substring(reloptions::text,$$autovacuum_freeze_max_age=(\d+)$$)::int8) is not null then (substring(reloptions::text,$$autovacuum_freeze_max_age=(\d+)$$)::int8)-age(relfrozenxid) 
         else (select setting from pg_settings where name=$$autovacuum_freeze_max_age$$)::int8 - age(relfrozenxid) 
    end as age_remain,
	$$vacuum freeze $$||nspname||$$.$$||relname||$$;$$ as cmd
from pg_class t2 join pg_namespace t3 on t2.relnamespace=t3.oid 
where t2.relkind in ($$t$$, $$r$$) and t2.relname LIKE $$t_%$$ and age(relfrozenxid) > 100000000
order by age(relfrozenxid) desc;
```


# 三、备份恢复

## 3.1 逻辑导出导入         

针对一些数据安全性要求低，故障时可以接受丢失当天数据的应用，可以采用pg_dump进行数据备份（更推荐3.2的备份方式）。

```shell
-- 备份
-- 备份abase数据库
pg_dump -Usa -dabase > abase.dmp
-- 备份abase数据库下的ywst模式、db_aty模式
pg_dump -Usa -dabase -n ywst -n db_aty > ywst_db_aty.dmp
-- 备份abase数据库下的ywst.t1表、ywst.t2表
pg_dump -Usa -dabase -t ywst.t1 -t ywst.t2 > t1_t2.dmp
-- 备份用户、角色信息
pg_dumpall -r > role.dmp
-- 备份abase数据库建库建表语句（不包括索引约束）
pg_dump -Usa -dabase --section=pre-data
-- 备份abase数据库的数据
pg_dump -Usa -dabase --section=data
-- 备份abase数据库的索引及约束
pg_dump -Usa -dabase --section=post-data
-- 恢复
-- 是否需要提前创建库或模式，可以看一下备份文件开头是否有创建库或模式的命令。
psql -Usa -dabase < 备份文件
```

## 3.2 备份恢复                             

abase数据库备份推荐使用安装包带的arterybase-wal-backup-linux-6.0.0工具，工具会开启数据库日志归档，配合每天晚上的基础备份，支持做基于时间点的恢复，可以恢复到任意故障时刻之前。适合数据安全级别高的应用，对磁盘空间的要求较大，需要存储大量归档日志。

```shell
-- 工具基于abase的pg_basebackup命令实现，主要命令如下。
pg_basebackup -D $bakdir -Ft -z -U ${PGUSER} -p ${PGPORT}
```

具体工具的使用及恢复方法见《ArteryBase基于事务日志的增量备份工具使用说明.pdf》。

**注：归档日志无法配合pg_dump进行数据恢复，需要使用pg_basebackup进行基础备份。**