---
title: Abase基础SQL
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
## 创建表
### 1.create table
``` 
略 
```
### 2.select * into
```sql
--复制一张表,包括全部结构和数据
select * into db_test.t_aj_linych from db_test.t_aj;

--复制一张表,包括指定的字段和数据
select c_bh, n_ajzlb, n_jbfy into db_test.t_aj_linych from db_test.t_aj;

--复制一张表,按条件填充数据
select * into db_test.t_aj_linych from db_test.t_aj where n_ajzlb = 30010;

--复制一张表,只包括表结构
select * into db_test.t_aj_linych from db_test.t_aj where 1 <> 1;
```
### 3.create table as
```sql
--复制一张表和数据
create table db_test.t_aj_linych as select * from db_test.t_aj;

--复制一张表,按条件填充数据
create table db_test.t_aj_linych as select * from db_test.t_aj where n_ajzlb = 30010;
```
### 4.create table like
```sql

--including constraints:需要复制check约束
--including indexes:需要复制索引
--including comments:需要复制注释
--including defaults:需要复制默认值
--including storage:需要复制存储策略

--复制一张表,只复制表结构
create table db_test.t_aj_linych (like db_test.t_aj);

--复制一张表,复制表结构和索引
create table db_test.t_aj_linych (like db_test.t_aj including indexes);

--复制一张表,复制表结构、索引、注释
create table db_test.t_aj_linych (like db_test.t_aj including indexes including comments);
```
### 4.create temp table
```sql
--create temp tbl_name() on commit{preserve rows|delete rows|drop};
--preserve rows：默认值，事务提交后保留临时表和数据
--delete rows：事务提交后删除数据，保留临时表
--drop：事务提交后删除表

create temp table tbl_temp(id int);

create temp table tbl_temp(id int) on commit delete rows;

create temp table tbl_temp(id int) on commit drop;
```

### 5.create table with
```sql

--fillfactor,填充因子，一个表的填充因子(fillfactor)是一个介于 10 和 100 之间的百分数。100(完全填充)是默认值。
--如果指定了较小的填充因子，INSERT 操作仅按照填充因子指定的百分率填充表页。每个页上的剩余空间将用于在该页上更新行，
--这就使得 UPDATE 有机会在同一页上放置同一条记录的新版本，这比把新版本放置在其它页上更有效。
--对于一个从不更新的表将填充因子设为 100是最佳选择，但是对于频繁更新的表，较小的填充因子则更加有效

--oid,行对象标识符（对象ID），这个字段只有在创建表时使用了“with oids”或配置参数“default_with_oids”的值为真时才出现，
--这个字段的类型是oid（类型名与字段名同名）PostgreSQL在内部使用对象标识符（oid）作为系统表的主键。
--系统不会给用户创建的表增加一个oid字段。oid类型用一个四字节的无符号整数实现，不能提供大数据范围内的唯一性保证，甚至在单个大表中也不行。
--因此PostgreSQL官方不鼓励在用户创建的表中使用oid字段。

--指定fillfactor(填充因子)和oids
create table db_test.t_linych(id int) with (FILLFACTOR=20, oids=false);
```
## 字段操作
```sql
--创建一张表
create table  db_test.t_lyc (id int, age int, addr varchar);

--修改表名
alter table db_test.t_lyc rename to t_linych;

--新增一个字段
alter table db_test.t_linych add c_name varchar(300);

--删除一个字段
alter table db_test.t_linych drop column c_name;

--修改字段名称
alter table db_test.t_linych rename "id" to "n_id";

--修改字段数据类型
alter table db_test.t_linych alter column "c_name" type text;

--插入随机数据，为验证修改做准备
insert into db_test.t_linych SELECT generate_series(1,10) as key,(random()*(6^2))::integer, repeat( chr(int4(random()*26)+65),4),(random()*(10^4))::integer;

--特殊的修改为integer
alter table db_test.t_linych alter column c_name type int using to_number(c_name,'9999999999999999999');
```

## 索引操作
```sql
--查看表的索引信息
select * from pg_indexes where tablename='t_aj';
select * from pg_statio_all_indexes where relname='t_aj';

--创建一个索引
--默认btree索引
create index i_t_linych_age on db_test.t_linych (age);
--指定btree索引，适合排序，比大小，和绝大部分场景
create index i_t_linych_age on db_test.t_linych using btree (age);
--指定hash索引，适合精确匹配，例如长字符串查询
create index i_t_linych_age on db_test.t_linych using hash (c_name);
--指定gin索引，适合like检索，尤其是三字符以上的检索，采用的是倒排索引的方式
create index i_t_linych_age on db_test.t_linych using gin (addr gin_trgm_ops);

--删除一个索引
drop index db_name.index_name;
```
想深究gin索引的同学请移步：http://artery.thunisoft.com/posts/detail/ce222e210dd0485fbc788fd0f190ed3a  
非常详尽

## 约束操作
```sql
--查询表的约束信息
--约束的类型包括 UNIQUE,PRIMARY KEY, CHECK, FOREIGN KEY, 注意这里的类型必须是大写
select
     tc.constraint_name, tc.table_name, kcu.column_name, 
     ccu.table_name as foreign_table_name,
     ccu.column_name as foreign_column_name,
     tc.is_deferrable,tc.initially_deferred
 from 
     information_schema.table_constraints as tc 
     join information_schema.key_column_usage as kcu on tc.constraint_name = kcu.constraint_name
     join information_schema.constraint_column_usage as ccu on ccu.constraint_name = tc.constraint_name
 where constraint_type = 'PRIMARY KEY' and tc.table_name = 't_aj';

--添加一个主键约束
alter table db_test.t_linych add constraint pk_id primary key(n_id);

--添加一个唯一性约束
alter table db_test.t_linych add constraint unique_id unique(n_id);

--添加非空约束
alter table db_test.t_linych alter column age set not null;

--删除非空约束
alter table db_test.t_linych alter column age drop not null;

--删除一个约束
alter table db_test.t_linych drop constraint unique_id;
```
### 表权限
```sql
--修改表的拥有者
alter table db_test.t_linych owner to sa;

--查询表权限
select relname,relacl from pg_class where relname='t_aj_sssq';

--为schema下的表添加权限
grant select on all tables in schema db_test to FY2000Login;

--对未来新建表赋予相关权限(给予默认权限)
alter default privileges in schema db_test grant select on tables to FY2000Login;
```
Access privileges 具体含义：       
a: insert  
r: select  
w: update  
d: delete  
x: references  
t: trigger  
D: truncate  

### 表的元数据
```sql

--查询库中的schema
select schema_name,*
from information_schema.schemata 
where catalog_name = 'db_ywst_ms' 
  and schema_name like '%test%';

-- 查询schema中的表
SELECT * FROM information_schema.tables WHERE table_name IN ('t_aj');

--查询数据库占用空间
 select d.datname as name,  pg_catalog.pg_get_userbyid(d.datdba) as owner,
    case when pg_catalog.has_database_privilege(d.datname, 'connect')
        then pg_catalog.pg_size_pretty(pg_catalog.pg_database_size(d.datname))
        else 'no access'
    end as size
from pg_catalog.pg_database d
    order by
    case when pg_catalog.has_database_privilege(d.datname, 'connect')
        then pg_catalog.pg_database_size(d.datname)
        else null
    end desc -- nulls first
    limit 20;

--查询表的存储空间
select
    table_schema || '.' || table_name as table_full_name,
    pg_size_pretty(pg_total_relation_size('"' || table_schema || '"."' || table_name || '"')) as size
from information_schema.tables
 where table_schema = 'db_ywst'
   and table_type = 'base table'
   and table_name = 't_aj' 
order by pg_total_relation_size('"' || table_schema || '"."' || table_name || '"') desc;

--查询列信息
 select column_name,
        udt_name,
        is_nullable,
        character_maximum_length,
        numeric_precision,
        numeric_precision_radix,
        numeric_scale,
        datetime_precision
   from information_schema.columns
  where table_name = 't_aj'
    AND table_schema = 'db_test';

--查询字段名、类型、注释
select
    a.attname as name, 
    replace(replace(replace(format_type (a.atttypid, a.atttypmod), 'character varying' ,'varchar'),'integer','int4'),' without time zone','') as type,
    col_description(a.attrelid, a.attnum) as comment
from pg_class as c,
     pg_attribute as a
where c.relname = 't_aj'
  and a .attrelid = c.oid
  and a .attnum > 0
order by attnum;

--查询主键列
select
    pg_attribute.attname as colname,
    pg_type.typname as typename,
    pg_constraint.conname as pk_name
from
    pg_constraint
inner join pg_class on pg_constraint.conrelid = pg_class.oid
inner join pg_attribute on pg_attribute.attrelid = pg_class.oid and pg_attribute.attnum = pg_constraint.conkey [ 1 ]
inner join pg_type on pg_type.oid = pg_attribute.atttypid
where pg_class.relname = 't_aj'
  and pg_constraint.contype = 'p';

--查询视图创建语句
select pg_get_viewdef('db_ywst.v_aj',true);
  
--查询依赖于某个表的视图
set search_path to db_ywst;
select 
rulename,
ev_class::regclass,
ev_class 
from pg_rewrite 
where oid in (select distinct objid from pg_depend where refobjid='db_ywst.t_ms_aj'::regclass);

--查询视图用到的表
select distinct c.view_name,c.table_name 
  from (
        select  a.ev_class::regclass as view_name, 
                b.refobjid::regclass as table_name,
                a.ev_class 
          from pg_rewrite a, pg_depend b
         where a.oid = b.objid
           and b.deptype <> 'i' 
           and b.refobjsubid <> 0
       ) c 
 where c.ev_class = 'db_ywst.v_aj'::regclass;
```

### 向表中插入数据
```sql
--insert into
insert into "db_test"."t_linych" ("n_id", "age", "addr", "c_name") values ('1', '31', 'bbbb', '7313');

--批量insert into
insert into "db_test"."t_linych" ("n_id", "age", "addr", "c_name") values 
('12', '0', 'oooo', '9207'),
('13', '9', 'hhhh', '8575'),
('14', '32', 'qqqq', '3843'),
('15', '14', 'jjjj', '7886'),
('16', '17', 'wwww', '6787');

--insert into from
insert into db_test.t_linych select to_number(c_bh,'9999999999999999999'), n_ajzlb, c_ah, n_ajjzjd from db_test.t_aj limit 100;

--copy,copy to 的文件存储在数据库所在的服务器上
copy (select * from db_test.t_linych where age < 50) to '/home/gpadmin/db_test.t_linych.dat';

copy db_test.t_linych from '/home/gpadmin/db_test.t_linych.dat';

--dump 
--dump指令需要在数据库服务器上psql中执行
--dump出的文件存储在数据库所在的服务器上,恢复时可以用psql指令执行备份文件(前提是以sql形式备份，如果是以流的方式备份，则需要用其他方式恢复)
pg_dump -h172.23.33.4 -p 6543 -U sa -d abase -t db_dat_wb.dim* -a -f ~/db_dat_wb_20190215.dat;
```

### VACUUM
```sql
--只将磁盘空间标记为可用(删除的数据如果在末端，会直接释放磁盘空间)
vacuum db_test.t_linych

--该命令会为指定的表或索引重新生成一个数据文件，并将原有文件中可用的数据导入到新文件中，之后再删除原来的数据文件。
--因此在导入过程中，要求当前磁盘有更多的空间可用于此操作。由此可见，该命令的执行效率相对较低。
vacuum full db_test.t_linych
```