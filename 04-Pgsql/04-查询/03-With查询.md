## Misc
- `with`            子查询,可视为仅用于主语句的临时表
- `with recursive`  递归查询(实际应为迭代)
- 递归查询流程
    - 当前迭代前的全部查询结果保存在工作表 
    - 上次迭代的查询结果保存在中间表
    - 使用中间表作为当前迭代查询的目标表
    - 执行当前迭代查询获取结果
        - 如果使用`union`，则使用工作表对当前结果去重
        - 将当前结果放入中间表，用于下次迭代
        - 将当前结果放入工作表，用于输出最终结果
    - 如果当前迭代查询没有返回任何记录则终止迭代
        - 如果使用`union`，去重后无任何记录也会终止迭代
```
create table t1(
  id int,pid int,depth int,
  title varchar(10)
);

-- 以下打乱记录顺序
insert into t1 values
(1,null,1,'n1'),
(2,null,1,'n2'),
(3,1,2,'n3'),
(4,1,2,'n4'),
(7,3,3,'n7'),
(8,3,3,'n8'),
(9,3,3,'n9'),
(5,2,2,'n5'),
(6,2,2,'n6');

-- 子查询，同1个子查询可使用多次，但只会被计算1次
with cs1 as (
 select id from t1 where pid is not null
),cs2 as (
 select id from t1 where depth > 2
 and id in (select id from cs1)
)
select * from t1 where id in (select id from cs2);

-- 递归查询
with recursive t(n) as(
 select 1   -- 中间表初始记录，也可使用values(1)
 union all  -- 把每次迭代结果放入工作表
            -- 使用union可去除工作表的重复记录
 select n+1 from t where n<10 -- 当前迭代，t仅包含中间表(上次迭代)的记录
                              -- 这里每次迭代中间表仅包含1条记录   
)
select n from t; -- t包含工作表(全部迭代)记录              

-- 递归查询，类似树状结构
with recursive qs(id,pid,depth,title) as (
  select * from t1 where pid is null
  union
  select t1.* from qs,t1
  where t1.pid=qs.id
)
select * from qs;

-- 递归查询必须避免死循环，以下记录将导致死循环
insert into t1 values
(10,12,5,'n10'),
(11,10,6,'n11'),
(12,11,7,'n12');

-- 以下如果使用union，则去重后会终止迭代，不会死循环
with recursive qs(id,pid,depth,title) as (
  select * from t1 where pid=12
  union all
  select t1.* from qs,t1
  where t1.pid=qs.id
)
select * from qs;

-- union去重是比较记录全部字段值，而不是id值
-- 以下仍然会出现死循环，每行记录的rownum值不同
with recursive qs(id,pid,depth,title,rownum) as (
  select t1.*,1 from t1 where pid=12
  union
  select t1.*,qs.rownum+1 from qs,t1
  where t1.pid=qs.id
)
select * from qs;

-- 主语句使用limit可限制子查询迭代次数
-- 但仅适用于子查询没有表连接的情况，不建议用于解决死循环
-- 以下使用array保存已记录id用于去重,多字段使用array[row(c1,c2)]
with recursive qs(id,pid,depth,title,ids,cycle) as (
  select t1.*,array[t1.id],false from t1 where pid=12
  union
  select t1.*,
   qs.ids||t1.id,     --保存id到数组
   t1.id=ANY(qs.ids)  --终止迭代条件
  from qs,t1
  where t1.pid=qs.id and not cycle
)
select * from qs where not cycle;

-- 同上
with recursive qs(id,pid,depth,title,ids) as (
  select t1.*,array[t1.id] from t1 where pid=12
  union
  select t1.*,
   qs.ids||t1.id     --保存id到数组
  from qs,t1
  where t1.pid=qs.id and not t1.id = any(qs.ids)
)
select * from qs;
```