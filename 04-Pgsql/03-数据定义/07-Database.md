## 参考文档
- [create database](http://www.postgres.cn/docs/10/sql-createdatabase.html)
- [createdb](http://www.postgres.cn/docs/10/app-createdb.html)

## Misc
- 数据库实例(以下简称数据库)是数据库对象的集合
- 1台数据库服务器上的全部数据库实例称为集簇(`cluster`)，可能理解有误
- 数据库实例是物理上隔离的，拥有独立的数据文件`$PGDATA/base`
- 初始化(`initdb`)默认创建的数据库
    -  `postgres` 默认数据库
    -  `template0/template1` 模板数据库

## 模板数据库
- 创建新数据库时，会复制指定的模板数据库
- 模板数据库在复制的时候，不接受任何数据库连接
- 创建新数据库时，可指定创建为1个模板数据库
- 默认不能删除模板数据库，需先设置为普通数据库
- 初始化创建的模板数据库
    - `template0`
        - 一般用于创建“纯净的”用户数据库的模板 
        - 只包含当前PG版本预定义的标准数据库对象
        - 未包含任何编码相关和区域相关的数据
        - 在初始化后不应被修改，可用于恢复`template1`数据库
    - `template1`
        - 默认的新建用户数据库的模板
        - 一般用于定制模板功能，如指定表空间，编码集，时区
        - `postgres`数据库简单复制此模板数据库
```
-- oid 数据库oid，对应数据库数据文件目录名称$PGDATA/base
-- datistemplate 是否为模板数据库，如需删除模板需先设置为false
-- datallowconn  是否允许连接数据库，一般template0不允许连接
select oid,datname,datistemplate,datallowconn from pg_database;

  oid  |  datname  | datistemplate | datallowconn 
-------+-----------+---------------+--------------
 13808 | postgres  | f             | t
     1 | template1 | t             | t
 13807 | template0 | t             | f

```