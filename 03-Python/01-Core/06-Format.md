## 参考文档
- [Common string operations](https://docs.python.org/3/library/string.html)
- [PyFormat](https://pyformat.info/)

## 常用格式
- 建议阅读参考文档
- 如果被格式化的值超过指定宽带，会被保留
- `3.2f`  3个字符(含小数点)，2个小数，浮点数
- `0>10b` 填充`0`，右对齐，10个字符，二进制
- `#x`    添加`0x`前缀，16进制小写
- `,%`    千分位，百分比(乘100,添加`%`)
- `%Y-%m-%d %H:%M:%S` 日期格式
- `xx!r`  转换字符串，等价于`repr(xx)` 

## 格式化函数
- `f"{var1}"` 格式化字符串字面量
- `format()`  格式化字符串
- `print()`   打印字符串，支持`%`格式
- `Template`  模板，占位符使用`$xx`
- `rjust()`   右对齐 
- `ljust()`   左对齐
- `center()`  居中
- `zfill()`   填充`0`
- `str()`     返回人类可读字符串，默认调用`repr()`
- `repr()`    返回解释器可读字符串，如`<class 'int'>`

## 示例
```
'{},{}'.format('a','b')
'{},{}'.format(*'ab')
'{0},{1}'.format('a','b')

point={'x':1,'y':2}
'{0[x]}:{0[y]}'.format(point)
'{x}:{y}'.format(**point)
'{x}:{y}'.format(x=1,y=2)

a,b=1,2
f'{a}{b}'
'{a}{b}'.format(**vars())

'12'.zfill(5) #'00012'

str('a')  #'a'
repr('a') #"'a'"

for i in range(3):
    print("{0:2d}{1:6d}".format(i,i*i))

# sprintf格式，一般不用
# %r等价于repr()
print('%.2f' % 1)  #1.00
```