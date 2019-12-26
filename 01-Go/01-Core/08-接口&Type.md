## Type
- `type` 可声明新的数据类型或已有数据类型的别名
- `v.(string)` 断言`v`的类型为`string`
- `v.(type)`   获取`v`的类型
```
type hi func(id int) string  //声明函数类型

type x1 string      //声明新的类型
type x2=string      //声明类型别名
	
var s1 x1="a"
var s2 x2="b"
var s3 string="c"

fmt.Println(s1==s3) //编译报错，x1与string类型不同
fmt.Println(s2==s3) //不会报错，x2是string的别名

var v interface{}

s:=v.(string)     //断言v为string并返回，如果v不是string则抛出错误
s,ok:=v.(string)  //断言v为string并返回，如果v不是string则ok==false，不会抛出错误

switch s:=v.(type){....}  //类型判断,具体见switch
```

## 接口实现
- `interface`用于定义行为，不能包含变量值
- 函数支持`receiver`,此时函数视为`receiver`对象的方法
- 如果对象的方法与接口的方法签名一致，则认为对象实现此接口（`duck-type`）
- `interface{}`表示通用数据类型:
    - `[]int`等不能直接转换为`[]interface{}`

```
var v interface{}
var p *int
fmt.Printf("%#v",v)  //输出<nil>

v = p
fmt.Printf("%#v",v)  //输出(*int)(nil)

//声明类型，receiver不能为标准数据类型
type x string

//声明接口
type greeter interface {
	greet() string
}

//实现接口
func (r x)greet() string  {
	return string(r)
}

x("abc").greet();
```
