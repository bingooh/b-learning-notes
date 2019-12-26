## 参考文档
- [create role](http://www.postgres.cn/docs/10/sql-createrole.html)
- [drop role](http://www.postgres.cn/docs/10/sql-droprole.html)
- [alt role](http://www.postgres.cn/docs/10/sql-alterrole.html)
- [grant](http://www.postgres.cn/docs/10/sql-grant.html)
- [revoke](http://www.postgres.cn/docs/10/sql-revoke.html)
- [alter default privileges](http://www.postgres.cn/docs/10/sql-alterdefaultprivileges.html)

## Misc
- 角色表示1个权限集，权限表示对数据库对象的操作
- 角色可继承，也可赋给另1个角色
- 角色的属性(`Attr`)定义角色的行为，常见属性：
    - `SUPERUSER`           超级用户，跳过所有权限校验
    - `LOGIN/NOLOGIN`       是否允许登录，默认否
    - `PASSWORD`            设置登录密码，仅适用于可登录的角色
    - `CREATEDB/CREATEROLE` 创建数据库/角色
    - `INHERIT/NOINHERIT`   是否继承父角色的权限,默认继承
    - `IN ROLE` 列出的角色将作为新角色的父角色
    - `ROLE`    列出的角色作为新角色的子角色，即新角色可视为1个用户组
    - `ADMIN`   同上，但子角色可将新角色赋给其他角色，等价于子角色使用`WITH ADMIN OPTINO`加入
- `SUPERUSER`属性
    - 初始化数据库系统时，默认会创建1个具有此属性的角色,
      角色名称与当前系统用户名称相同，默认为`postgres` 
    - 此属性不能被继承
- PG的用户/用户组实际不存在，其底层都是角色
    - `create user`  :`create role xx LOGIN`
    - `create group` :`create role xx ROLE|ADMIN xx,xx`
- `PUBLIC`角色
    - 可理解为1个用户组，所有角色都属于此用户组
    - `PUBLIC`是关键词，不能删除此角色，也不能回收/新增其子角色
    - `revoke all on schema public from PUBLIC`
    - `revoke all on database xx from PUBLIC`
- PG内建角色
    - `select rolname from pg_roles`
    - `pg_read_all_settings` 阅读所有配置变量
    - `pg_read_all_stats`    阅读所有`pg_stat_*`视图并使用相关扩展
    - `pg_stat_scan_tables`  执行可能会长时间锁表的监视功能
    - `pg_monitor`           读取/执行各种监视视图和函数，是以上的子角色
    - `pg_signal_backend`    给其他后端发送信号
- 数据库对象的拥有者
    - 每个数据库对象都有1个拥有者，默认为创建此对象的角色
    - 拥有者对此对象有全部权限，默认其他角色需要授权才能访问
- 删除角色的流程
    - 修改重要的数据库对象的为其他拥有者
    - 删除属于当前角色的不重要的对象
    - 删除当前角色
```
-- 修改拥有者
alter table xx owner to xx_role;

-- 删除角色，以下命令需在每个数据库实例执行
reassign owned by role1 to other_role;
drop owned by role1;
drop role role1;  -- 如果还有依赖此角色的对象，会报错

-- 设置角色相关的配置参数
alter role role1 set xx_param to xx_value;

-- 创建角色
create role role1 NOINHERIT;
create role role2 NOINHERIT;
create role role3 INHERIT;

-- role2不会继承role1的权限
-- role3继承role2的权限，不能继承role1的权限
grant role1 to role2;
grant role2 to role3;

psql -U role3       --使用role3登录，拥有本身和role2权限
set role to role2;  --仅拥有role2权限
set role to role1;  --仅拥有role1权限
reset role;         --恢复本身权限

create user role1;
create user role2 role role1;

-- 等价于：create user role3; 
-- grant role3 to role1 with admin option;
create user role3 admin role1;

create user role4;
\c postgres role1;
grant role2 to role4; --报错，must have admin option
grant role3 to role4;

-- 授权：grant xx on xx in schema xx to xx;
grant select on table xx to xx_role;
grant select on all tables to xx_role;
grant select on all tables in schema xx to xx_role;
grand all privileges on database xx to xx_role;

-- 回收全部公共权限
revoke all on database xx from PUBLIC;
revoke all on schema public from PUBLIC;
revoke all on all tables in schema public from PUBLIC; 

-- 创建只读用户组
create user readonly;
grant connect on database xx to readonly;
grant usage on schema public to readonly;
grant select on all tables in schema public to readonly;
grant select on all sequences in schema public to readonly;
grant execute on all functions in schema public to readonly;

-- 以下group1角色每次创建表，其select权限将自动赋给readonly组
-- sequences、functions的授权省略
alter default privileges 
for group1 in schema public
grant select on tables to readonly;
```