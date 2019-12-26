## 参考文档
- [create tablespace](http://www.postgres.cn/docs/10/sql-createtablespace.html)

## Misc
- 表空间指定数据库对象的存储文件位置
- 表空间可用于`create database/table/index/add constraint`
- 表空间非拥有者必须授权(`create`)后才能使用
- PG使用符号连接管理表空间，自定义表空间符号链接见`$PGDATA/pg_tblspc`
- 表空间系统配置参数
    - `default_tablespace` 新建数据库对象默认使用的表空间
    - `temp_tablespaces`   临时表和索引使用的表空间，可指定多个
- 初始化数据库自动创建的表空间
    - `pg_global`  存储共享系统目录
    - `pg_default` template0/template1默认表空间   
- 在同1个存储设备上设置多个表空间没有太多意义
```
-- 查看所有表空间，也可使用\db
select spcname from pg_tablespace;
```