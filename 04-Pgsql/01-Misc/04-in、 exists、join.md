## Misc
- 以下结论
    - 不同数据库，不同表统计数据可能导致使用不同的SQL执行计划
    - 仅用于SQL优化分析，具体以实际的执行计划为准
- 连接的2个表可分为
    - 外表：驱动表，循环遍历其记录或索引以执行匹配
    - 内表：提供匹配条件值，如执行子查询获取匹配值

## join查询
- `left semi join` 
    - 类似`inner join`，但连接结果不会有重复记录(类似`in/exists`查询)
    - 外表的1条记录一旦匹配内表某条记录后，会立刻中止匹配下条外表记录
- `anti join`
    - 与`inner join`相反 
- 下图连接结果应为`bag`,而不是`set`,即可能包含重复记录
    - 举例：如内表的连接字段允许重复值，则连接结果可包含重复记录 
![sql joins](https://i.stack.imgur.com/VQ5XP.png)

## join算法
- SQL `select t1.* from t1,t2 where t1.id=t2.id`
- `hash join`
    - 扫描小表建立hashtable
    - 遍历大表匹配小表的hashtable
    - 适用于小表记录数少，两个表记录数差异大的情况
    - 小表的hashtable如果太大，可分片部分存储在磁盘
- `sort merge join`
    - 分别扫描2个表的连接字段值并排序
    - 遍历1个表已排序的连接字段值，匹配另1个表已排序的连接字段值
    - 此算法需要排序较消耗资源，适用于连接字段已有排序索引的情况
- `nested loop`
    - 遍历外表，对每条外表记录再遍历内表进行匹配
    - 适用于表的记录数较少的情况
- `index lookup join`
    - 对外表记录进行分块，每次处理1块
    - 当前外表记录块记为`B`,取其全部连接字段值记为`JF`
    - 扫描内表，获取与`JF`匹配的内表记录，并构建hashtable记为`HT`
    - 对`B`和`HT`进行`hash join`匹配

## 主要区别
- `in`     执行子查询，对每个值遍历主表记录进行匹配
- `exists` 遍历主表记录，对每条记录执行子查询进行匹配 
- `join`   遍历主表记录，对每条主记录再遍历连接表进行匹配
- 注：以上说法很不准确，实际数据库一般不会使用以上执行计划
```
-- 以下表t1(大表),t2(小表)，字段为id,age
-- 为方便理解，可将索引视为1个hashtable，key为被索引列值
-- 进而可将t1,t2视为以id为key的hashtable
--
-- 为提高查询速度，应使用小数据集匹配大数据集
-- 如t1有1万条，t2有10条记录。如果用t1匹配t2，
-- 则需遍历t1，需匹配1万次，反之仅需匹配10次
--
-- 以下SQL大部分数据库都将使用相同执行计划，即t2匹配t1
select * from t1 where t1.id in (select id from t2)

select * from t1 where exists 
  (select 1 from t2 where t1.id=t2.id)

select t1.* from t1 inner join t2 on t1.id=t2.id

-- 如果匹配字段包含重复值或Null或没有索引，则使用join会出现重复记录
-- 即inner join on子查询会保留重复记录，而in的子查询不会
select t1.* from t1 inner join t2 on t1.age=t2.age

-- not in一般为范围查询，可能不支持使用索引，建议谨慎使用
select * from t1 where t1.id not in (select id from t2)

-- 以下查询因查询条件需要计算，无法使用索引
select * from t1 where exists 
  (select 1 from t2 where t1.id-t2.id=0)
```

