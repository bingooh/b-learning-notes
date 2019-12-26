## 参考文档
- [排他约束](http://www.postgres.cn/docs/10/sql-createtable.html#SQL-CREATETABLE-EXCLUDE)
- [match full](https://dba.stackexchange.com/questions/58894/differences-between-match-full-match-simple-and-match-partial)

## Misc
- 约束分为：列级约束和表级约束
- 1个列可以有多个约束
- NULL约束仅为兼容，表示此字段值可以为NULL
- `default`可指定列的默认值，应尽可能避免使用NULL列
- 以下约束将自动创建索引：主键，唯一键，排他约束

## 检查约束
```
create table t1(
 c1 int check(c1>0),   --列级约束
 c2 int,c3 int,
 check(c2>0 and c3<0), --表级约束
 constraint ck_c2_c3 check(c2>c3) --指定约束名
);
```

## 非空约束
```
create table t1(
  c1 int not null default 0,   --非空约束，设置默认值
  c2 int not null check(c2>0), --多个约束
  c3 null
);
```

## 主键/唯一键
```
create table t1(
  id serial primary key,     --1个表只能有1个主键
  code int unique not null,  --等价于主键，但1个表可有多个唯一键
);

create table t2(
  c1 int not null,
  c2 int not null,
  constrain pk_c1_c2 primary key(c1,c2)
)
```

## 外键
- 主表的外键列必须有主键或唯一键约束
- 删除主表外键值时，可对引用此外键的子表记录设置如下操作
    - `NO ACTION` 默认值，在事务的最后抛出错误禁止删除
    - `RESTRICT`  执行删除语句时立刻抛出错误禁止删除
    - `CASCADE`   删除所有子记录
    - `SET NULL/SET DEFAULT` 设置子记录外键字段值为NULL或默认值
    - 注：修改主表外键字段值视为删除外键值
```
create table t1(
 id int primary key,
 c1 int unique,
 c2 int unique
)

create table t2(
 id int references t1, -- 默认引用t1主键
 c1 int,c2 int,
 
 -- 外键列的数量和类型必须一一对应
 -- 如果表记录t1(1,1,1)
 -- match full  ：t2(1,1,1)受约束,t2(1,1,null)将报错
 -- match simple：t2(1,1,1)受约束,t2(1,1,null)不受外键约束
 foreign key(c1,c2) references t1(c1,c2) match full
)

```