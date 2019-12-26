## 参考文档
- [Built-in Exceptions](https://docs.python.org/3/library/exceptions.html)

## 内建异常类
- 异常：可被捕捉并被恢复的错误
- 内建异常类是`identifier`,不是`keyword`
- `BaseException`
    - 所有异常基类
    - `args`             设置错误信息，通过构造函数传入
    - `with_traceback()` 设置错误堆栈
- `Exception`
    - 所有不会导致进程退出的异常基类
    - 所有用户自定义异常类应继承此类
```
# 自定义异常类
class MyException(Exception):pass

# 设置异常信息
try:
    raise MyException(1,2)
except MyException as err:
    print(err.args[0]) #1
    
```

## 处理异常
```
try:
    print(1/0)
except ZeroDivisionError:
    print('catch an error')
except MyException as err:
    print(err.args[0])
else:
    print('no exception')
finally:
    #任何情况下都将执行，如果仅为释放资源，可使用with
    print('finally done')

# 从上到下匹配异常
try:
    pass
except (NameError,):  #匹配多个异常
    pass
except Exception:     #匹配Exception及其子类
    pass
except :              #匹配任何异常
    raise Exception   #重新抛出异常，等价于raise Exception()

```
