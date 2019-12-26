## 重复注解
java8以前实现重复注解
```
@interface Author { String name(); }
@interface Authors {
    Author[] value();
}

@Authors(
    { @Author(name="Raoul"), @Author(name="Mario")}
)
class Book{}
```

java8的实现
```
@Repeatable(Authors.class)
@interface Author { String name(); }
@interface Authors {
    Author[] value();
}

@Author(name="Raoul") 
@Author(name="Mario")
class Book{ }
```

java8里注释可用在任何地方，如
```
@NonNull String name=...;
List<@NonNull Car> cars=...;
```

## 类库变化
- `Map`新增方法
    - `getOrDefault()`
    - `computeIfAbsent()`
    - `putIfAbsent()`
    - `replace()`
    - `merge()`
    
- `Collection`新增方法
    - `removeIf()`
    - `stream()`
    - `parallelStream()`

- `List`新增方法
    - `replaceAll()`
    - `sort()`

-  `Collections`新增方法
    - `checkedQueue()`
    - `checkedNavigableSet()`
    - `checkedNavigableMap()`

- `atomic`包新增方法
    - `getAndUpdate()`
    - `updateAndGet()`
    - `getAndAccumulate()`
    - `accumulateAndGet()`
    - `XxxAdder()`       用于多线程环境下频繁更新操作
    - `XxxAccumulator()` 同上

- `ConcurrentHashMap`新增方法
    - `forEach(threshold)` 如果元素个数超过阈值则并行执行，否则顺序执行
    - `reduce(threshold)`
    - `search(threshold)`
    - `mappingCount()`  返回`long`，避免使用`int size()`

- `Arrays`新增方法    
    - `setAll()`
    - `parallelSetAll()`
    - `parallelPrefix()`
    
- `Number`新增方法    
    - `sum()`
    - `min()`
    - `max()`
    - `xxxValueExact()`

- `Math`新增方法    
    - `xxxExact()` 精确计算，如果计算结果导致数据类型溢出则报错

- `String`新增方法    
    - `join()`

- `reflect`包新增类    
    - `Parameter`  用于反射参数类型
    - `Executable` 用于反射通用函数和构造函数的父类

- `Files`    
    - `list()`
    - `walk()`
    - `find()`
    - `lines()` 读取行内容