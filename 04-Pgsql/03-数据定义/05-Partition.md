## Misc
- 分区使用基本原则：表的尺寸超过机器物理内存
- 分区优点
    - 提高查询性能：仅扫描符合约束条件的分区
    - 提高新增/删除性能：直接操作单独1个分区
    - 可对分区单独创建索引，索引尺寸小
    - 可对频繁访问的分区移到SSD等快速存储上
- PG分区可视为分区表的子表
- 分区限制
    - 分区表不支持约束：主键，外键，唯一键，排除
    - 注：需测试以下2种实现方式对外键的支持
- 分区实现类型
    - 通过声明
        - 仅支持`range/list`分区方法
        - 分区只能包含分区表拥有的字段
        - 分区表的`check/not null`约束总是被分区继承
        - `range`分区的区间为`[)`，不允许有重叠
        - 每个分区将创建1个隐式的约束(分区条件)
        - 不支持创建跨所有分区的主键，唯一键，排除等约束
        - 分区表不支持主键，外键(包括引用和被引用)，`on conflict`语句
        - 添加/删除分区需对分区表使用`ACCESS EXCLUSIVE`锁，范围较大
    - 通过继承
        - 支持自定义分区方法
        - 分区允许多继承，即分区可包含分区表以外的字段
        - 添加/删除分区只需对分区表使用`SHARE UPDATE EXCLUSIVE`锁
- 创建通过继承实现的分区流程(更多见参考文档)
    - 创建分区表(主表)，可以定义主键约束
    - 创建分区(子表)继承分区表，并指定分区条件(表约束)
    - 对每个分区的分区键创建索引，按需创建其他索引
    - 创建触发器函数并添加为主表触发器，根据分区条件插入记录到分区
    - 也可创建`rule`代替触发器，但规则消耗系统资源较高
- ` constraint_exclusion`(配置参数)
    - 执行查询时，是否先检查约束以排除不包含查询结果的子表
    - 默认为`partition`,仅对子表/分区/`union all`先检查约束
```
-- 确保启用constraint_exclusion
show constraint_exclusion;

-- 创建分区表，其包含`partition by`语句
create table person(
  id serial,   --分区表不能包含主键等约束
  sex int,age int
) partition by range(age);

-- 创建分区(子表)，分区范围不能重叠
create table person_age_min_30 partition of person
for values from (minvalue) to (30);

create table person_age_30_60 partition of person
for values from (30) to (60);

create table person_age_60_100 partition of person
for values from (60) to (100);

-- 可以进一步创建子分区,指定表空间等
-- create table person_age_min_30 partition of person(
--   id primary key
-- )
-- for values from (minvalue) to (30)
-- tablespace xx_fast_ts
-- partition by(sex);

-- 为每个分区创建分区键索引，可创建唯一键或其他索引
create index on person_age_min_30(age);
create index on person_age_30_60(age);
create index on person_age_60_100(age);

insert into person(age,sex) values
(0,20),(0,40),(1,80);

--报错，age不支持null。建议对age增加not null约束
insert into person values(10,1,null);

-- 假设创建分区时添加：id primary key
-- 以下也不会报错，主键可在不同分区重复
insert into person values
(10,1,20),
(10,1,40);

-- 删除分区
drop table person_age_min_30;

-- 删除分区关系，但保留分区
alter table person detach pertition person_age_min_30;

-- 新建表，并作为分区加入分区表
create table person_age_100_max
  (like person including defaults including constraints);

alter table person_age_100_max add constraint age_100_max
  check((age is not null) and (age >= 100));

-- 导入数据：\copy person_age_100_max from 'xx'

create index on person_age_100_max(age);

alter table person attach partition person_age_100_max
for values from (100) to (maxvalue);

```