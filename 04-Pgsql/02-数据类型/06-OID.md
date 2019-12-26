## OID
- 标识符数据类型(以下系统列为隐藏列)
    - `oid`  对象标识符，`oid`系统列的数据类型
    - `xid`  事务标识符，`xmin,xmax`系统列的数据类型(事务ID)
    - `cid`  命令标识符，`cmin,cmax`系统列的数据类型(事务里SQL语句ID)
    - `ctid` 元组标识符，`ctid`系统列的数据类型(行存储位置ID)
- `oid` 无符号4字节整数
    - 即是标识符数据类型，也可作为系统列`oid`的名称
    - 作为系统列时，默认不会添加到用户自定义表，除非：
        - 启用`default_with_oids`
        - 建表时使用`with oids`
    - `oid`受大小限制，不建议作为用户自定义表主键
- `oid`一般用作内部系统表的主键，包含多个别名

名字          |引用         |描述         |值示例 
---           |---          |---          |---
oid           |任意         |整数         |123
regproc       |pg_proc      |函数名字     |sum
regprocedure  |pg_proc      |带参数函数   |sum(int4)
regoper       |pg_operator  |操作符名字   |+
regoperator   |pg_operator  |带参数操作符 |*(integer,integer) or -(NONE,integer
regclass      |pg_class     |关系名字     |pg_type
regtype       |pg_type      |数据类型名字 |integer
regrole       |pg_authid    |角色名       |oper
regnamespace  |pg_namespace |名字空间名称 |pg_catalog
regconfig     |pg_ts_config |文本搜索配置 |english
regdictionary |pg_ts_dict   |文本搜索字典 |simple

```
SELECT * FROM pg_attribute WHERE attrelid = 'mytable'::regclass;

-- ::regclass转换"oid"/"name"，即regclass->pg_class->"oid"/"name"
-- pg_class.relname      数据类型为'name'
-- pg_class.oid          数据类型为'oid'
-- 以上SQL等价于：
SELECT * FROM pg_attribute
  WHERE attrelid = (SELECT oid FROM pg_class WHERE relname = 'mytable');
  
-- 也可以将oid转换为表名称
-- 每行记录包含1个tableoid系统列，显示记录所属表的oid
SELECT oid::regclass FROM pg_class WHERE relname = 'mytable'

```