# Clickhouse

Clickhouse是一个用于联机分析处理（OLAP）的列式数据库管理系统（columnar DBMS）。

## 引擎

### TinyLog

最简单的一种引擎，每一列保存为一个文件，里面的内容是压缩过的，不支持索引
这种引擎没有并发控制，所以，当你需要在读，又在写时，读会出错。并发写，内容都会坏掉。

`应用场景:`
a. 基本上就是那种只写一次
b. 然后就是只读的场景。
c. 不适用于处理量大的数据，官方推荐，使用这种引擎的表最多 100 万行的数据

```sql
drop table if exists test.tinylog;
create table test.tinylog (a UInt16, b UInt16) ENGINE = TinyLog;
insert into test.tinylog(a,b) values (7,13);
```

### Log

这种引擎跟 TinyLog 基本一致
它的改进点，是加了一个 __marks.mrk 文件，里面记录了每个数据块的偏移
这样做的一个用处，就是可以准确地切分读的范围，从而使用并发读取成为可能

但是，它是不能支持并发写的，一个写操作会阻塞其它读写操作
Log 不支持索引，同时因为有一个 __marks.mrk 的冗余数据，所以在写入数据时，一旦出现问题，这个表就废了`应用场景:`
同 TinyLog 差不多，它适用的场景也是那种写一次之后，后面就是只读的场景，临时数据用它保存也可以

### Memory

内存引擎，数据以未压缩的原始形式直接保存在内存当中，服务器重启数据就会消失
 可以并行读，读写互斥锁的时间也非常短
 不支持索引，简单查询下有非常非常高的性能表现

`应用场景:`
 a. 进行测试
 b. 在需要非常高的性能，同时数据量又不太大（上限大概 1 亿行）的场景



### MergeTree

这个引擎是 ClickHouse 的`重头戏`，它支持`一个日期和一组主键的两层式索引`，还可以`实时更新数据`。同时，索引的粒度可以自定义，外加直接支持采样功能

```sql
drop table if exists test.mergetree;
create table test.mergetree (sdt  Date, id UInt16, name String, cnt UInt16) ENGINE=MergeTree(sdt, (id, name), 10);

-- 日期的格式，好像必须是 yyyy-mm-dd
insert into test.mergetree1(sdt, id, name, cnt) values ('2018-06-01', 1, 'aaa', 10);
insert into test.mergetree1(sdt, id, name, cnt) values ('2018-06-02', 4, 'bbb', 10);
insert into test.mergetree1(sdt, id, name, cnt) values ('2018-06-03', 5, 'ccc', 11);
```

目录结构

```css
├── 20180601_20180601_1_1_0
│   ├── checksums.txt
│   ├── columns.txt
│   ├── id.bin
│   ├── id.mrk
│   ├── name.bin
│   ├── name.mrk
│   ├── cnt.bin
│   ├── cnt.mrk 
│   ├── cnt.idx
│   ├── primary.idx
│   ├── sdt.bin
│   └── sdt.mrk -- 保存一下块偏移量
├── 20180602_20180602_2_2_0
│   └── ...
├── 20180603_20180603_3_3_0
│   └── ...
├── format_version.txt
└── detached
```

### SummingMergeTree

SummingMergeTree 就是在 merge 阶段把数据sum求和

可加列不能是主键中的列，并且如果某行数据可加列都是 null ，则这行会被删除

```sql
create table test.summingmergetree (sdt Date, name String, a UInt16, b UInt16) ENGINE=SummingMergeTree(sdt, (sdt, name), 8192, (a));

insert into test.summingmergetree (sdt, name, a, b) values ('2018-06-10', 'a', 1, 20);
insert into test.summingmergetree (sdt, name, a, b) values ('2018-06-10', 'b', 2, 11);
insert into test.summingmergetree (sdt, name, a, b) values ('2018-06-11', 'b', 3, 18);
insert into test.summingmergetree (sdt, name, a, b) values ('2018-06-11', 'b', 3, 82);
insert into test.summingmergetree (sdt, name, a, b) values ('2018-06-11', 'a', 3, 11);
insert into test.summingmergetree (sdt, name, a, b) values ('2018-06-12', 'c', 1, 35);

-- 手动触发一下 merge 行为
optimize table test.summingmergetree;

select * from test.summingmergetree;
```

### AggregatingMergeTree

```sql
drop table if exists test.aggregatingmergetree;
create table test.aggregatingmergetree(
sdt Date
, dim1 String
, dim2 String
, dim3 String
, measure1 UInt64
) ENGINE=MergeTree(sdt, (sdt, dim1, dim2, dim3), 8192);

-- 创建一个物化视图，使用 AggregatingMergeTree
drop table if exists test.aggregatingmergetree_view;
create materialized view test.aggregatingmergetree_view
ENGINE = AggregatingMergeTree(sdt,(dim2, dim3), 8192)
as
select sdt,dim2, dim3, uniqState(dim1) as uv
from test.aggregatingmergetree
group by sdt,dim2, dim3;

insert into test.aggregatingmergetree (sdt, dim1, dim2, dim3, measure1) values ('2018-06-10', 'aaaa', 'a', '10', 1);
insert into test.aggregatingmergetree (sdt, dim1, dim2, dim3, measure1) values ('2018-06-10', 'aaaa', 'a', '10', 1);
insert into test.aggregatingmergetree (sdt, dim1, dim2, dim3, measure1) values ('2018-06-10', 'aaaa', 'b', '20', 1);
insert into test.aggregatingmergetree (sdt, dim1, dim2, dim3, measure1) values ('2018-06-10', 'bbbb', 'b', '30', 1);
insert into test.aggregatingmergetree (sdt, dim1, dim2, dim3, measure1) values ('2018-06-10', 'cccc', 'b', '20', 1);
insert into test.aggregatingmergetree (sdt, dim1, dim2, dim3, measure1) values ('2018-06-10', 'cccc', 'c', '10', 1);
insert into test.aggregatingmergetree (sdt, dim1, dim2, dim3, measure1) values ('2018-06-10', 'dddd', 'c', '20', 1);
insert into test.aggregatingmergetree (sdt, dim1, dim2, dim3, measure1) values ('2018-06-10', 'dddd', 'a', '10', 1);

-- 按 dim2 和 dim3 聚合 count(measure1)
select dim2, dim3, count(measure1) from test.aggregatingmergetree group by dim2, dim3;

-- 按 dim2 聚合 UV
select dim2, uniq(dim1) from test.aggregatingmergetree group by dim2;

-- 手动触发merge
OPTIMIZE TABLE test.aggregatingmergetree_view;
select * from test.aggregatingmergetree_view;

-- 查 dim2 的 uv
select dim2, uniqMerge(uv) from test.aggregatingmergetree_view group by dim2 order by dim2;
```

## 函数

```sql
SELECT
    sum(charged_fees) AS charged_fees,
    sum(conversion_count) AS conversion_count,
    (charged_fees / conversion_count) / 100000 AS conversion_cost
FROM t
WHERE dt = '2019-07-01'

┌──charged_fees─┬─conversion_count─┬────conversion_cost─┐
│ 3143142724482 │           250537 │ 150.37090954370022 │
└───────────────┴──────────────────┴────────────────────┘
(虚假数据，可能不准确)

1 rows in set. Elapsed: 0.155 sec. Processed 32.56 million rows, 1.20 GB (210.00 million rows/s., 7.77 GB/s.)
```