## module
- 模块指包含python定义和语句的文件，模块名称默认为文件名称
- 特殊模块
    - 主模块  ：作为脚本文件直接执行的模块
    - 内建模块：`builtins`
- 模块有自己的`private symbol table`
    - 保存模块内部定义的变量，函数等 
    - 模块首次执行时创建，可视为模块对象
    - 对模块内部来说，可视为`global symbol table`
    - 内建模块对象在解析器启动时创建，其属性不能被删除
- 模块导入
    - 将模块对象(`symbol table`)赋值给导入模块的变量
    - 模块仅在首次被导入时执行(创建模块对象)
    - `importlib.reload(md)` 重新创建模块对象
- 模块导入搜索路径
    - 当前执行模块所在目录
    - `$PYTHONPATH`指定的目录
    - 默认安装的依赖目录
    - 注1：`sys.path`包含全部导入搜索路径(优先级从高到低)
    - 注2：当前目录导入优先级高，不要在当前目录定义与标准库名称相同的模块
- 模块可能被编译为`*.pyc`文件并缓存到`__pycache__`目录下，以提高加载速度
- 模块对象特殊属性
    - `__name__` 当前模块名称。如果直接执行模块，则值为`__main__` 
- `globals()` 获取当前模块的`global symbol table`
- `locals()`  获取当前范围的`local symbol table`，在模块内同`globals()`
- `dir()`     获取传入对象的属性名称，默认获取`locals().keys()`
    - `import buildins;dir(buildins)` 显示内建模块定义的名称 

## package
- 包本身为1个目录，包含：
    - 多个模块文件
    - 多个子目录(子包，可选)
    - `__init__.py` 包初始化文件，定义包对象
- 所有包目录名称组成包路径
- `__init__.py`特殊属性
    - `__all__`     定义包导出的模块，需自定义
    - `__package__` 包路径 
    - `__path__`    包的初始化文件所在目录路径，用于多目录包
```
# 以下使用pkg/md/fn分别表示包/模块/函数
import md1,md2      #导入模块
from md1 import fn1 #直接导入函数定义

#导入包下的模块
import pkg1.md1 
import pkg1.subpkg1.md1
from pkg1 import md1
from pkg1.md1 import fn1

#使用相对路径，主模块不支持
from .. import md1 

# 以下两种导入方式等价。默认执行__init__.py进行导入
# 如果包初始化文件未导入任何模块，则以下语句无任何作用
# __init__.py可编写代码导入包下的所有模块文件，具体参考网上代码
import pkg1
from pkg1 import *
```