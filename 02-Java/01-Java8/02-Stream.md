## 基本概念
- `Stream`可用于计算的数据源产生的元素序列
- `Stream`用于计算，`Collection`用于存储数据
- `Stream`按需计算，只能遍历(消费)1次
- `Stream`组成：
    - 数据源
    - 中间操作(链)，如`fliter,map,limit,sorted,distinc`
    - 终端操作，如`forEach,count,collect`
- 操作符分为：
    - `中间/终端`：操作符是否结束流计算，如`filter`是中间，`reduce`是终端
    - `有界/无界`：操作符是否处理有限的元素，如`limit`是有界，`sorted`是无界（需要全部元素放入内存里排序）
    - `无状态/有状态`：操作符当前处理是否依赖前面处理过的元素，如`filter`是无状态，`reduce`是有状态

## 数值流
- 对象流做数值计算时将使用自动封装，性能较低
```
list.stream()
    .map(Person::getAge)      //getAge()返回int，但是map()返回Stream<Integer>
    .reduce(0,Integer::sum)   //这里先转换为int，再求和
```

- `IntStream,LongStream,DoubleStream`为数值流，不存在自动封装
```
//转换为数值流再计算
list.stream()
    .mapToInt(Person::getAge)
    .sum();

//求最大值，返回结果可选
OptionalInt rs=list.stream()
    .mapToInt(Person::getAge)
    .max();
```

- 数值流转换为对象流
```
//转换为对象流，以调用对象流方法
Stream<Integer> s=list.stream()
    .mapToInt(Person::getAge)
    .boxed();
    
Stream<Integer> s=IntStream
    .range(1,10)
    .mapToObj(Integer::valueOf)
```

- `range()/rangeClosed()`
    -  `range(1,10)`        `[1,10)`
    -  `rangeClosed(1,10)`  `[1,10]`
```
IntStream.range(1,10).forEach(System.out::println);
```

## 生成流
- `Stream.empty()`
- `Stream.of()/IntStream.of()`
- `Arrays.stream()`
- `Files.lines()/Files.list()` 仅NIO支持，包括多个方法
- `Stream.iterate(0,n->n+1)`  无限流，提供1个初始值和`UnaryOperator`函数
- `Stream.generate(()->1)`    无限流，提供1个`Supplier`函数

## 收集流的数据
- `Stream.collect(c:Collector)` 使用`collector`收集流的数据
- `Collector`执行过程可理解为`reduce`，但是前者更容易支持并行
- `Collectors`提供多个`Collector`实现
    - `mapping(mapper,downstream)` 返回`map collector`
    - `reducing(id,mapper,op)`     返回`reduce ollector`
    - `groupingBy(classifier,downstream)` 返回`Map<K,V>`
    - `partitioningBy(classifier,downstream)` 返回`Map<Boolean,V>`
    - `toSet()` 可实现`distinct`功能
    - `toCollection(HashSet::new)` 指定使用`HashSet`作为容器

## 并行流
- `Stream.parallel()`   转换为并行流
- `Stream.sequential()` 转换为串行流
- `parallelStream()`    直接获取并行流
- `stream()`    直接获取串行流
- 并行流底层使用`ForkJoinPool`(默认线程数等于CPU核心数)执行计算
- `Spliterator(拆分迭代器)`接口定义如何拆分数据源以便执行并行计算
    - `tryAdvance()` 定义对元素的操作 
    - `trySplit()` 定义如何拆分，返回`null`表示不能拆分
    - `Collection`实现了此接口(默认方法)
- 并行流并不总是比串行流快，需要考虑
    - 严格测试并行执行性能，以串行执行为基准
    - 注意自动封装，使用数值流避免
    - 考虑计算成本，如果计算链较短，或流包含的元素很少，使用串行流
    - 某些操作不适合使用并行流，比如`findFirst(),limit()`等要求顺序的操作
    - 考虑数据结构是否易分解，如`iterate(),LinkedList`元素之间互相依赖的数据结构不易分解，`range()`等易分解
    - 考虑流自身特点，使用`filter()`过滤元素导致流的大小不定，进而难以在执行过滤前对流进行分解
    - 考虑合并子流的计算结果成本(使用`Collector.combiner`合并结果),在多个CPU核之间移动数据可能成本较高

## 举例
- `flatMap`
```
list.stream()
    .map(w->w.split(""))
    .flatMap(Arrays::stream)
    .distinct()
    .collect(toList());

//{[1,2],[1,2],[1,2],[1,2]}
list1=Arrays.asList(1,1);
list2=Arrays.asList(2,2);
list1.stream()
    .flatMap(i->
        list2.stream()
        .map(j->new int[]{i,j})
    )
    .collect(toList());
```

- `match`
```
//其他还有allMatch(), noneMatch()
boolean ok=list.stream().allMatch(i->i>100);

//findAny()支持并行处理，findFirst()较难支持
Optional<String> rs=list.stream()
    .filter(s=>s.length>10)
    .findAny();
    
rs.ifPresent(System.out::Println);
```

- `reduce`
```
//reduce(identity) 参数identify表示如果流为空则返回此值
//reduce()没有设置identify参数值，返回Optional
int sum=list.stream().reduce(0,Integer::sum);
Optional<Integer> sum=list.stream().reduce(Integer::sum);//无初始值
```

- `iterate`
```
//斐波纳契序列
//不包含内部状态，支持并行
IntStream.iterate(
    new Int(){0,1},
    t->new Int(){t[1],t[0]+t[1])
    )
    .limit(10)  //一定要节流
    .forEach(t->println(Arrays::toString)
```

- `collect`
```
import static java.util.stream.Collectors.*;

stream().count();
stream().collect(counting());

stream().mapToInt(Person::getAge).sum();
stream().collect(summingInt(Person::getAge));

//同时计算count,sum,average,min,max
stream().collect(summarizingInt(Person::getAge));

//拼接字符串
stream().collect(joining());        //内部使用StringBuffer
stream().reduce("",(s1,s2)->s1+s2); //never do this


stream().collect(counting());
stream().collect(reducing(0,e->1,Integer::sum));
```

- `groupingBy`
```
//根据用户等级分组。getLevel()返回String
//如果某个等级下没有用户，则此等级不会作为key放入rs里
Map<String,List<Person>> rs=persons.stream()
    .collect(groupingBy(Person::getLevel); 

//根据用户等级分组，然后统计每组用户数
Map<String,Integer> rs=persons.stream()
    .collect(groupingBy(
        Person::getLevel,
        counting()
    )); 

//根据用户等级分组，然后根据用户性别分组
Map<String,Map<String,List<Person>>> rs=persons.stream()
    .collect(groupingBy(
        Person::getLevel,
        groupingBy(Person::getGender)
    )); 

//根据用户等级分组，然后找出最大年龄的用户
//maxBy()返回Optional：
//如果不使用collectingAndThen()，返回值为Map<String,Optional<Person>>
//实际上如果某等级下没有用户，不会放1个Optional.empty()到Map里
//所以使用collectingAndThen()转换最终结果
Map<String,Person> rs=persons.stream()
    .collect(groupingBy(
        Person::getLevel,
        collectingAndThen(
            maxBy(Person::getAge),
            Optional::get //不会抛出异常，不可能为Optional.empty()
        )
    ));

```