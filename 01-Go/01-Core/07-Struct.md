## Misc
- `struct`定义对象的状态和行为
- `struct`是值，赋值时将创建新对象并复制全部属性值
- `embed struct`结合`composition`设计模式，可实现继承

```
//声明struct并初始化，如省略字段名称则按顺序初始化赋值
s:= struct {
	x,y int
}{1,2}

//声明struct类型	
type xs struct {
	x,y int
}

xs1:=xs{x:1,y:2}

//内嵌数据类型，建议不用
s1:=struct {
    int
    bool
}{1,true}

//内嵌匿名struct
s2:= struct {
	child struct{ n int}
}{
	child:struct{ n int }{1},
}
```

## Embedding Struct
- 被嵌入的`struct`自动融合(`composite`)到宿主`struct`
- 宿主`struct`可直接调用嵌入`struct`的方法，即访问重定向(不是继承)
- 如果嵌入多个`struct`导致方法等命名冲突，需要显式调用
- 如果命名有冲突，但是并不需要使用这些方法和变量，则不会报错
- 创建宿主`struct`对象时，不会创建被嵌入`struct`的对象
```
type Runner struct {
	running bool
}

func (r *Runner) run()  {
	if !r.running{
		r.running=true
	}
}

//Runner嵌入Worker，字段名称默认为Runner
//嵌入后Worker即拥有Runner的属性和方法
type Worker struct {
	Runner
}

//创建Worker对象，不会创建Runner对象
w:=Worker{}
w.run()          //重定向到Runner.run()
w.Runner.run()   //显式调用Runner.run()避免命名冲突
```