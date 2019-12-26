## Misc
- `json`
    - 输入不加处理，直接存储为文本格式
    - 每次读取需重新解析，性能较低
    - 一般仅支持取值操作，不支持大部分的比较符如`<,>`等
    - 底层存储为text，所以不允许`零值字符`
- `jsonb`
    - 输入经过二进制编码后存储
    - 输入时性能较低，查询时性能高
    - 输入的额外空格被丢弃，key字段重新排序，重复键被删除
- `jsonb`解析时会将json数据类型映射到Pgsql数据类型
    - `string` :`text`     不允许`\u0000`，只接受ASCII Unicode转义，除非数据库使用UTF8字符集
    - `number` :`numeric`  不允许`NaN`,`infinity`
    - `boolean`:`boolean`  只允许`true/false`
    - `null`   : 无
- json数字使用IEEE754双精度浮点数，转换为`numeric`可能丢失精度
```
select '1.20e-1'::jsonb;     --0.120 转换为numeric

-- {"a":1,"b":3} 重新排序，重复键被删除
select '{"b":2,"a":1,"b":3}'::jsonb;
```

## 操作符
- [JSON函数和操作符](http://www.postgres.cn/docs/10/functions-json.html)
- `->`  获取指定key/索引对应的json对象
- `->>` 同上，但返回text类型
- `#>`  获取指定路径的json对象，路径类型text[]
- `#>>` 同上，但返回text类型
- `@>`  包含
- `<@`  被包含
- `?`   指定字符串是否为顶层key
- `?|`  指定字符串数组任意元素是否为顶层key
- `?&`  指定字符串数组全部元素是否为顶层key
- `||`  连接2个json对象
- `-`   删除指定(多个)key/索引对应的json对象
- `#-`  删除指定路径对应的json对象
```
-- 以下结果为true，jsonb比较时顺序不重要
-- 测试包含关系时，应将数组等理解为集合
select '"foo"'::jsonb @> '"foo"'::jsonb;
select '[1, 2, 3]'::jsonb @> '1'::jsonb;
select '[1, 2, 3]'::jsonb @> '[1, 3]'::jsonb;
select '[1, 2, 3]'::jsonb @> '[1, 2, 2]'::jsonb;
select '{"a": "1", "b": 2}'::jsonb @> '{"b": 2}'::jsonb;
select '{"a": {"b":1}}'::jsonb @> '{"a": {}}'::jsonb;

-- 以下结果返回false
select '[1, 2, [1, 3]]'::jsonb @> '[1, 3]'::jsonb;
select '{"a": {"b":1}}'::jsonb @> '{"b": 1}'::jsonb;

-- 测试指定字符串是否为顶层key
select '[1, 2, 3]'::jsonb ? '1';
select '"a"'::jsonb ? 'a'
select '{"a":1,"b":2}'::jsonb ? 'b';
select '{"a":1,"b":2}'::jsonb ?| array['b','c'];
select '{"a":1,"b":2}'::jsonb ?& array['b','c'];  --f

-- 取值操作
select '[1, 2, 3]'::jsonb ->2;       --3
select '{"a":1,"b":2}'::json->>'b';  --2
select '{"a": {"b":{"c": 1}}}'::jsonb #> '{a,b}'; --{"c":1}

-- 删除操作
select '{"a":"1","b":2}'::jsonb - array['a','b'];   --{}
select '{"a": {"b":{"c": 1}}}'::jsonb #- '{a,b}';  --{"a":{}}
```

## 索引类型
- 以下索引仅适用于jsonb
- `gin: jsonb_ops`
    - 支持`@>, ?, ?|, ?&`
    - GIN索引默认类型，对每个key，value(包括子项)建立单独索引
- `gin: jsonb_path_ops`
    - 支持`@>`
    - 仅对顶层value建立索引，使用key-value的hash值作为索引项值
    - 如果顶层value为数组，则每个元素值会单独建立索引
    - 索引占据空间下，查询速度快，但不支持精确查找，如查子项
```
create table t1(
 id serial2,
 jdoc jsonb
);

-- 可使用\watch插入多条
insert into t1(jdoc) values('{
  "name":"n1",
  "tags":["a","b","c"]
}');


create index idx1 on t1 using gin(jdoc); --默认使用jsonb_ops
create index idx2 on t1 using gin(jdoc jsonb_path_ops);
create index idx3 on t1 using gin((jdoc -> 'tags'));

-- 执行以下语句，应删除idx1，idx2其中1个
-- 测试结果：前2条语句使用idx2更快，最后1条语句仅能使用idx3
-- 如果仅用tags作为过滤条件，建议仅创建idx3
-- 如果仅用@>作为过滤条件，建议仅创建idx2
explain analyze select id from t1 where jdoc @> '{"name":"n1"}';
explain analyze select id from t1 where jdoc @> '{"tags":["a"]}';
explain analyze select id from t1 where jdoc -> 'tags' ? 'a';
```