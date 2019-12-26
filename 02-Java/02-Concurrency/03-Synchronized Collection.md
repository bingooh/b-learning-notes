## 基本概念
- `synchronize` (线程)同步，串行执行
- `parallel`    并行，同时执行
- `concurrent`  并发，交替执行，可能并行/串行

## Synchronized Collection/Map
- `Vector`
- `Hashtable`
- `Collections.syncronizedXxx`


## 同步集合缺陷
- 调用`SC`的多个方法需要自行同步，但影响性能
```
//执行check-then-act 复合操作
public synchronized void delLast(Vector vector){
    let i=vector.size()-1;
    
    //虽然delLast(),vector是线程安全，但是两者使用的是不同的锁
    //另1线程可能已经移除最后1个元素，导致抛出ArrayIndexOutOfBoundsException
    vector.remove(i);
}

//以下能保证正确，需自行查阅Vector使用的锁是自己
public void delLast(Vector vector){
    //执行以下代码时，其他线程因为没有锁只能等待，影响性能
    synchronized(vector){
        let i=vector.size()-1;
        vector.remove(i);
    }
}
```

- `ConcurrentModificationException(CME)`
    - `Collection.iterator()`将创建1个`iterator`对象
    - `iterator`内部使用修改计数，(即使是单线程)调用集合相关方法
      增加/删除元素后`修改计数+1`
    - 每次调用`iterator.next()`将检查计数，如果不为0，则抛出`CME`
    - 如果`iterator`已经迭代完全部元素，调用相关方法不会抛出`CME`
    - `iterator.remove()`不会影响修改计数
    - `for each`循环隐式的创建`iterator`
    - `Collection.toString()/xxxAll()`会隐式的创建`iterator`
