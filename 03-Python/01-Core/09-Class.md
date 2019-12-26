## 参考文档
-[global/nonlocal statement](https://docs.python.org/3/reference/simple_stmts.html#global)

## 命名空间
- `NS(命名空间)` 即前面提到的`symbol table`
    - 程序运行时创建，保存定义的变量名称和值
    - 程序执行时，根据变量名称查询`NS`获取变量值
    - 同1个值可拥有多个变量名称(别名)，赋值可理解为创建别名
    - `NS`可理解为1个dict对象，模块对象/类对象等价于`NS`
    - `del x`可删除`NS`里定义的变量(某些`NS`不支持删除操作)
- `NS`主要创建时刻
    - 调用函数
    - 导入模块
    - 执行主模块
    - 加载内建模块
- `scope` 变量的词法作用域(`literal scope`)
    - 编程语言静态定义了不同范围的`scope`
    - 运行时，每个`scope`会动态绑定1个`NS`
    - 变量->`scope`->`NS`->递归向上查找变量值
    - `scope`    定义了`NS`的查找顺序
    - `NS Chain` 定义程序执行上下文对象
- `scope`范围(从里到外):
    - `local`    函数内部，保存局部变量和函数参数
    - `nonlocal` 
        - 介于`local`/`global`之间，如外部函数的`scope`
        - 可包含多层，如嵌套多个外部函数
        - 可理解为闭包作用域，保存自由变量
    - `global`    模块作用域(模块对象)
    - `outermost` 解释器加载的作用域，如`builtins`
- 函数执行过程
    - 根据函数名称，从`NS`递归查找函数(对象)
    - 创建`local NS`，保存传入的函数参数值
    - 绑定`local NS`到函数`scope`，执行函数
- 闭包(`closure`)
    - 内部函数被返回给调用者，且内部函数引用外部函数变量，则形成闭包
    - 内部函数引用的外部函数的变量称为自由变量，即不受外部函数局部作用域约束的变量
    - 自由变量保存在内部函数的`local NS`，即`__code__.co_freevars`
    - 外部函数执行完毕后即销毁`local NS`，但不会销毁自由变量
    - 注：较早的编程语言使用`dynamic scope`
```
a='is_global'

def do_assign():
    a='is_nonlocal'      #对内部函数而言，可视为自由变量

    def do_local():
        a='do_local'

    def do_nonlocal():
        nonlocal a       #声明访问nonlocal变量
        a='do_nonlocal'

    def do_global():
        global a         #声明global变量
        a='do_global'

    do_local()
    print('after do_local(): ',a)       #is_nonlocal

    do_nonlocal()
    print('after do_nonlocal(): ', a)   #do_nonlocal

    do_global()
    print('after do_global(): ', a)     #do_nonlocal，访问的是do_assign()局部变量

do_assign()
print('after do_assign(): ', a)         #do_global
```

## class
- `class`运行时创建1个`NS`，即`class object`，可定义：
    - 静态方法(不引用类对象或实例对象) 
    - 类的属性和方法(第1个参数为类对象)
    - 实例属性和方法(第1个参数为实例对象)
    - 注1：类对象主要定义实例属性变量查找顺序(类名称引用类对象)
    - 注2：类的属性也可被实例对象查找，一般用作私有实例属性
- 类对象的特殊属性
    - `__new__()`  对象构造函数，可用于实现单例等
    - `__init__()` 对象初始化函数，如果`__new__()`未返回值，则不会调用此方法
    - `__mro__`    实例方法父类查找顺序(`method resolution order`)
- 实例对象的特殊属性
    - `__class__` 引用实例的类对象
- 实例方法的特殊属性
    - `__self__` 引用实例对象
    - `__func__` 引用实例方法对应的函数
- 实例属性查找过程
    - 查找实例本身：避免实例数据字段覆盖实例属性
    - 查找实例的类：避免实例属性覆盖私有实例属性(类属性)
- 实例方法执行过程
    - 根据`inst.__class__`获取类对象
    - 根据方法名称从类对象查找对应的方法
    - 如果未找到，则查找`inst.__class__.__mro__`
    - 执行方法，第1个参数传入实例对象
    - 注：实例方法可理解为第1个参数绑定为实例对象的函数
- 继承
    - 所有对象继承`object` 
    - 允许多继承，查找父类的实例方法可简单理解为从左到右，具体见文档
    - `isinstance(o,c1)`   如果`o.__class__`等于`c1`或是其子类，则返回`true`
    - `issubclass(c1,c2)`  如果`c1`是`c2`的子类则返回`true`
    - `super()`  仅用于从父类查找实例方法，支持多继承。不支持`super()[xx]`
- 私有属性
    - `_xx()`   仅为约定，外部不应调用此方法
    - `__xx()`  自动解析为`_XXClazz__xx()`，避免子类覆盖。必须类内部调用有效
```
# 数据对象类
class MyBean():pass

bean=MyBean()
bean.name='name1' #name为数据字段


# 自定义类
class Person():
    count = 0 # 类属性

    def __init__(self,name):
        Person.count+=1
        self.name=name

    #实例方法，传入实例对象
    def hi(self):
        print(f'hi,{self.name}')

    #类方法，传入类对象
    @classmethod
    def inst_count(cls):
        print(cls.count)

    #静态方法
    @staticmethod
    def title():
        print(Person.__name__)  #类名称

p1=Person('p1')
p2=Person('p2')
Person.inst_count() #2          

# 继承
class Student(Person):
    def hi(self):
        Person.hi(self)  #显示调用父类
        super().hi()     #用于多继承，等价于super(Student,self).hi()

# 多继承
class MyError(NameError,SyntaxError):pass

# 私有属性，实现方式有点类似ts
# 如果使用exec()，eval()执行方法，则__engine不会被解析为_Class__engine
class Runner():
    _name='runner'  #仅约定
    __engine='raw'  #等价于_Runner_engine

    #_name是类属性，但通过实例属性引用，此方式实现了私有实例属性
    #如果通过类引用，如Runner._name，则_name在所有实例共享，即类属性
    #在__init__()定义实例属性，需小心不要覆盖私有实例属性
    def run(self):
        print(self._name)
        print(self.__engine)

class Robot(Runner):
    _name='robot'     #覆盖父类属性
    __engine = 'fast' #等价于_Robot__engine，不会覆盖父类__engine

    def run(self):
        super().run()
        print(self.__engine)

r=Robot()
r.run()                 #robot raw fast
#print(r.__engine)      #出错,因为__engine解析为_Class__engine
print(r._Robot__engine) #fast

# 私有方法，使用私有属性引用实例方法
class Runner():
    def __init__(self):
        self.__run('run') #调用私有方法，实际解析为_Runner__run

    #私有方法，仅约定，此方法可以被子类覆盖
    def _run(self,a1):
        print(f'runner {a1}')

    #赋给私有变量，避免被子类覆盖
    __run=_run

class Robot(Runner):
    # 子类可实现自己的私有方法
    def _run(self, a1,*a2):
        print(f'robot {a1} {a2}')

Robot() #runner run

# 外部函数设置为实例方法
def run(self):
    print(f'run {self.name}')

class Runner:
    #外部函数只要第1个参数是self，就可作为实例方法赋给类属性
    #不能直接赋给实例属性，因为实例方法是在class scope定义的
    run=run
    
    def __init__(self,name):
        self.name=name
    
r1=Runner('r1')
r1.run() #run r1

f=r1.run #绑定实例r1
f() #run r1
print(f.__self__==r1)      #true
print(f.__func__==run)  #true, run函数

# 单例
class Singleton:
    _inst=None

    def __new__(cls, *args, **kwargs):
        if cls._inst is None:
            _inst=super().__new__(cls,*args,**kwargs)

        return cls._inst

print(Singleton()==Singleton())

# Final类
class FinalClazz:
    def __new__(cls, *args, **kwargs):
        if cls!=FinalClazz:
            raise Exception('final clazz')

        return super().__new__(cls,*args,**kwargs)

class MyClass(FinalClazz):pass

FinalClazz()
MyClass() #出错，final类不能被继承
```

## iterator/generator
- `iterator`指实现`__next__()`的对象
    - 每次调用`__next__()`将返回下1个元素
    - 如果没有更多元素，则抛出`StopIteration`
    - `iterator.__iter__()` 返回本身`self`
- `iter(o)` 
    - 调用`o.__iter__()`，获取1个`iterator`对象
    - `for..in..`默认调用此方法获取`iterator`
- `generator`
    - 指包含`yield`的函数，或`yield expression`
    - `generator`用于快速创建`iterator`
```
# 自定义Iterator，需自行记住迭代位置，较烦琐
class MyIterator:
    def __init__(self,max):
        self.index=-1
        self.max=max

    def __iter__(self):
        return self

    def __next__(self):
        if self.index==self.max:
            raise StopIteration
        self.index+=1
        return self.index

for i in MyIterator(2):print(i)

# 使用generator
def MyGen(max):
    for i in range(max):yield i

for i in MyGen(3):print(i)

# 使用yield expression
g=(i for i in range(3))
for i in g:print(i)

sum(i for i in range(10))
```