## add

新增元素

```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

在添加元素之前会先确保容量足够，所以会先调用ensureCapacityInternal

```java
    private void ensureCapacityInternal(int minCapacity) {
        // 内部做的优化 忽略
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        // 确保容量足够
        ensureExplicitCapacity(minCapacity);
    }
```

```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        // 如果容量不够就要增加
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

```java
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        // 拷贝之前的元素
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

## remove

```java
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                // 找到相同的元素，然后移出元素
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```

拷贝复制元素
```java
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

## query

```java
    public E get(int index) {
        // 检查下表
        rangeCheck(index);

        // 获取元素
        return elementData(index);
    }
```

```java
   private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

```java
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
```

## update

```java
    public E set(int index, E element) {
        // 检查下表
        rangeCheck(index);

        // 设置内容
        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```