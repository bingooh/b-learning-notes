## 基本概念
- Lambda语法
    - `(params)->expression`
    - `(params)->{statements;}`
- Lambda与匿名类的区别
    - Lambda的`this`指向包含类，匿名类指向自己
    - Lambda不能覆盖包含类的变量，匿名类可以
    - Lambda的类型根据上下文推断确定，匿名类的类型在初始化时确定
    - Lambda使用`InvokeDynamic`执行字节码，而不是编译为匿名类

## 函数式接口
`函数式接口`指仅定义1个抽象方法的接口,推荐使用`@FunctionalInterface`标注
- 旧函数式接口(jdk8 以前)：
    - `Runnable:        ()->void`
    - `Callable<V>:     ()->V`
    - `Comparable<T>:   T->int`
    - `Comparator<T,T>: T->int` 
- 标准(jdk8自带)函数式接口：
    - `Predicate<T>:      T->boolean`
    - `Consumer<T>:       T->void`
    - `Function<T,R>:     T->R`
    - `Supplier<T>:       ()->T`
    - `UnaryOperator<T>:  T->T`
    - `BinaryOperator<T>: (T,T)->T`
    - `BiPredicate<L,R>:  (T,R)->boolean`
    - `BiConsumer<T,U>:   (T,U)->void`
    - `BiFunction<T,U,R>: (T,U)->R`
- 标准函数接口不能抛出`checked exception`，可封装为`RuntimeException`抛出
- 如果输入参数是基本数据类型，应使用基本数据类型函数接口，避免自动封装
```
//never do this。参数i将先转换为Integer再比较，影响性能
Predicate<Integer> isGtZero=(int i)->i>0;

//基本数据类型函数式接口
IntPredicate isGtZero=(int i)->i>0;
```

## 类型推断
- 自动推断参数类型
```
//i类型为Interger
Consumer<Interger> fn=i->println(i);
```

- 如果Lambda仅包含1个表达式，其返回值兼容`void`
```
//list.add(s)返回boolean，可兼容void
Consumer<String> fn=s->list.add(s);
```

## 捕获变量
- Lambda可视为局部变量，可捕获外部的`自由变量`包括
    - 静态变量
    - 实例变量
    - 事实上的`final`局部变量：使用`final`修饰，或在其作用域里仅被赋值1次
- 静态变量和实例变量存储在`堆`里，在所有线程间共享
- 局部变量存储在当前线程的`栈`里，默认仅限当前线程访问
    - 外部线程的局部变量与执行Lambda表达式的线程可能不同
    - 仅有外部线程的局部变量是事实上的`final`，Lambda线程才可得到1个副本值
    - Lambda线程获得副本值时，外部线程的局部变量可能已经回收

## 方法引用
- `::` 声明方法引用，如：`Person::sayHi`
- 引用静态方法： `String::format`
- 引用实例方法
    - 引用参数的方法：`(String s)->String::length`
    - 引用变量的方法：`()->s::length`
- 引用构造函数: `String::new`


举例：
- 引用实例方法
```
//List.sort(c),参数c的方法签名：(T,T)->int
//String.compareToIgnoreCase()的方法签名：T->int
//String::compareToIgnoreCase的方法签名(第1个T为函数的receiver)： //(T,T)->int或(this,T)->int或T.(T)->int
//以下代码正确
list.sort(String::compareToIgnoreCase);
```

- 引用构造函数
```
//构造函数Apple:(String name)->Apple
Function<String,Apple> f=Apple:new;
Apple a=f.apply('a1');
```

## 复合函数
- `Comparator`
```
Comparator a,b;
a.reversed().thenComparing(b);
```

- `Predicate`
```
//从左到右执行
Predicate a,b,c
a.and(b).or(c);//a&&b||c
a.or(b).and(c);//(a||b)&&c
```

- `Function`
```
Function a,b;
a.andThen(b);//从左到右执行，同pipe
a.compose(b);//从右到左执行
```
