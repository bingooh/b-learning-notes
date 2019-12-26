## `null`缺陷
- 导致`NullPointerException(NPE)`
- 导致代码膨胀，如深度嵌套`null`检查
- 自身无任何业务语义，仅能表示`缺失`值
- 破坏Java哲学，Java里唯一的`null`指针
- null不属于任何类型，多次传递后难以知道最初赋值是什么类型

## 基本概念
- `Optional`可视为1个容器，仅包含1个值
- `Optional`语义：包含的值可能为`空`
    - `Car car` 仅看声明不知道`car`可能为空
    - `Optional<Car> car` 仅从声明可看出`car`(包含的值)可能为空
- `Optional.empty()`表示`空`值，但引用它不会触发`NPE`
- `Optional`强迫用户处理值为空的情况

## 基本方法
`Optional`的方法包括：
- `empty()`       获取唯一的空`Optional`
- `of(v)`         创建`Optional`，如果`v`为空则抛出`NPE`
- `ofNullable(v)` 创建`Optional`，如果`v`为空则返回`Optional.empty()`
- `isPresent()`           值是否存在
- `ifPersent(consumer)`   值如果存在则执行`consumer`
- `get()`                 获取值，如果值为空则抛出`NPE`
- `orElse(t)`             如果值存在则返回，否则返回默认值`t`
- `orElseGet(supplier)`   如果值存在则返回，否则执行`supplier`获取默认值并返回
- `orElseThrow(supplier)` 如果值存在则返回，否则执行`supplier`获取错误并抛出

## 高级方法
- `Optional`可视为仅包含1个`Optional`对象的`stream`
- 使用高级方法可避免使用`isPresent()`等进行空值检查(同`null`检查)
- 以下方法将先判断包含的值是否为空，如果为空则返回`Optional.empty()`，否则执行相应的逻辑：
    - `filter()`  不满足条件返回`Optional.empty()`
    - `map()`     见举例
    - `flatMap()` 见举例

## 序列化
`Optional`不能序列化，以下属性`car`支持序列化
```
public class Person{
    private Car car;
    
    public Optional<Car> getCarAsOptional(){
        return Optional.ofNullable(this.car);
    }
}
```

## 举例
- 使用`map()`等避免进行空值检查
```
//getCar()返回Optional<Car>：person不一定有car
//getEngine()返回字符串    ：car一定有engine
person
    .flatMap(Person::getCar)   //如果此次使用map，则将返回Optional<Optional<Car>>
    .map(Car::getEngine)       //返回Optional<Engine>
    .orElse('Unknown')         //如果person没有car，此处实际调用Optional.empty().orElse()
```

- 封装异常
```
public static Optional<Integer> stringToInt(String s){
    try{
        return Optional.of(Integer.parseInt(s));
    }catch(NumberFormatException e){
        return Optional.empty();
    }
}

```