## add

```java
  public boolean add(E e) {
        linkLast(e);
        return true;
  }
```

```java
    void linkLast(E e) {
        // 保存末尾的Node
        final Node<E> l = last;
        // 新建node
        final Node<E> newNode = new Node<>(l, e, null);
        // 新的last节点变成了新建的Node
        last = newNode;
        if (l == null)
            // 如果之前没有元素，那么标头指向新node
            first = newNode;
        else
            // 否则将之前node的next指针指向新的node
            l.next = newNode;
        size++;
        modCount++;
    }
```

## query

```java
    public E get(int index) {
        // 先检查下表 然后遍历列表
        checkElementIndex(index);
        return node(index).item;
    }
```

```java
    Node<E> node(int index) {
        // assert isElementIndex(index);

        // 如果靠前 那么从前面找，否则从后面找
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

## remove

```java
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

```java
E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        // 前面的Node的next指向当前node的next
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        // 后面的node的Preview指向当前的prev
        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

## update

```java
    public E set(int index, E element) {
        // 检查下表
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        // 更新元素
        x.item = element;
        return oldVal;
    }
```