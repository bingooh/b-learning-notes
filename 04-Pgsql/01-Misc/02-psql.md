## 参考文档
- [copy](https://www.postgresql.org/docs/10/sql-copy.html)

## 常用参数
- `psql xxdb xxuser`
- `-h/-p` 主机/端口号
- `-W`    强制显示密码输入提示
- `-E`    显示命令对应的SQL
- `-c`    指定查询语句
- `-f`    指定sql文件
- `-l`    列出数据库
- `-A`    返回结果不需对齐
- `-t`    返回结果不包含字段名称
- `-q`    不显示输出信息
- `-1`    在单事务里执行sql

## 常用命令
- `help` 帮助
- `\h`   SQL帮助
- `\?`   命令帮助
- `\q`   退出，或Ctrl+D
- `\g`   执行SQL，或`;`
- `\l`   显示数据库
- `\dn`  显示模式
- `\du`  显示用户
- `\dp`  显示用户权限
- `\db`  显示表空间
- `\d`       查看所有表
- `\d xx`    查看表的结构
- `\dt+ xx`  查看表大小
- `\di+ xx`  查看索引大小
- `\sf  xx`  查看函数定义
- `\c xx xx`    查看或改变当前连接数据库和用户,`-`表示当前         
- `\conninfo`   查看当前连接数据库，用户，端口等信息
- `\x`       打开或关闭行列转置
- `\set`     设置变量值
- `\echo`    显示变量值
- `\timing`  显示SQL执行时间
- `\watch n` 不断执行缓冲区最近1条SQL直到失败或强行中断,
             `n`指开始下次执行时等待的秒数
- 注：psql支持Tab提示和UP/DOWN查询历史命令

## 导入/导出数据
- `copy`,`\copy`区别：
    - 前者是SQL命令，执行需要`SUPERUSER`权限
    - 前者操作数据库服务端文件，后者操作客户端文件
    - 前者性能较高，适用于大数据量(因为数据都在服务端
- `\copy`
    - `\copy xx_table from xx_file_path`
    - `\copy xx_table to xx_file_path`
- `copy`
    - `copy xx_table from file|cmd|stdin with xx`
    - `copy xx_table|(query) to file|cmd|stdout with xx`

## 传递变量
```
# 设置变量,取消设置：\set k1
\set k1 1

# 执行sql
select * from t1 where id=:k1

# 直接通过命令行传递
psql -v k1=1 -c 'xxxx id=:k1'
```

## 定制维护脚本
```
# psql连接后会执行~/.psqlrc，可在此文件里定制脚本，如：
# 注意：SQL语句需写成1行
\set test_q 'select * from t1 limit 1'

# 登录控制台，执行以下命令可获取查询结果
:test_q
```

## 命令提示符
- 默认命令提示符变量
    - `\echo :PROMPTn` 查看当前变量值
    - `PROMPT1` 常规提示符
    - `PROMPT2` 等待更多输入
    - `PROMPT3` 等待终端输入，如执行`copy from stdin`会显示
- 命令提示符格式
    - `%M` 数据库服务器别名
    - `%>` 端口号  
    - `%n` 当前会话用户名
    - `%/` 数据库名称
    - `%#` 超级用户显示`#`,其他`>`
    - `%p` 当前连接的后台进程号
    - `%R` 在`PROMPT1`一般显示`=`,如果进程端口显示`!`
