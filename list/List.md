# 链表

## 接口
```java
public interface List<E> extends Collection<E> {
    // 是否包含某个元素
    boolean contains(Object o);

    // 添加元素
    boolean add(E e);

    // 删除元素
    boolean remove(Object o);

    // 修改元素
    E set(int index, E element);
}

```