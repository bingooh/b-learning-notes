## 参考文档
- [命令行参数](https://docs.python.org/3/using/cmdline.html)
- [特殊方法](https://docs.python.org/3/reference/datamodel.html#special-method-names)

## Misc
- 变量可动态指定数据类型
- 所有变量都是对象，包括`int`等
- 赋值传递对象引用（类似Golang传指针值）
- `class`用于声明类，即定义对象模板，如`class int`
- `class`通过声明特殊方法定义对象行为，如`__init__()`
- `class`本身可声明方法，即定义类方法(静态方法)

## 编程规范
- 类名称：`XxxXxx`
- 变量/属性/函数/方法: `xx_xx`
- 私有属性/方法      : `_xx_xx`
- 使用4个空格缩进
- 每行不超过79字符
- 使用空格分隔运算符和变量/值

## Py解释器
- Windows系统命令
    - `py`   启动Python3解释器
    - `pyw`  同上，但打开1个新命令行窗口
- Linux系统，一般安装后使用`python3`运行解释器
- `Ctrl+D/Ctrl+Z/quit()/exit()` 退出解释器
- `py -c command [arg]` 执行表达式命令
- `py -m module  [arg]` 执行模块包含的命令
- `py *.py` 执行脚本文件
    - `#!/usr/bin/env python3` 脚本类型   
    - `# -*- coding: utf8 -*-` 脚本编码，需放在上1行下面
- `py -c help(xx)` 查看帮助
- 解释器会将最近1次输出保存到变量`_`，不建议使用此功能

## Py命令/语句
- `;`: 多条语句分隔符
- `\`: 命令换行连接符,类似Linux
```
# ";"将1行物理语句分隔为多条逻辑语句,不建议使用
print(1);print(2)

# 如果1行语句过长，必须使用"\"做换行连接符
'a'+ \ #如果此处不添加连接符将报错
'b'
```

## 安装pip
```
# Windows安装python3会自动设置环境变量py.exe
# 安装pip需自行设置环境变量%PYTHON3%\Scripts

# 安装包已自带pip，以下为升级
# %PYTHON3%\Scripts\pip intall -U pip
py -m pip install -U pip

# 重新安装,需设置环境变量执行python3
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
py get-pip.py

# 建议使用pipx，默认创建venv允许应用程序
py -m pip install --user pipx
py -m pipx ensurepath
```