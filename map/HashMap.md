## PUT

```java
   public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

步骤：

1. 计算key的hash值

2. 添加值

计算hash

```java
   /**
     * Computes key.hashCode() and spreads (XORs) higher bits of hash
     * to lower.  Because the table uses power-of-two masking, sets of
     * hashes that vary only in bits above the current mask will
     * always collide. (Among known examples are sets of Float keys
     * holding consecutive whole numbers in small tables.)  So we
     * apply a transform that spreads the impact of higher bits
     * downward. There is a tradeoff between speed, utility, and
     * quality of bit-spreading. Because many common sets of hashes
     * are already reasonably distributed (so don't benefit from
     * spreading), and because we use trees to handle large sets of
     * collisions in bins, we just XOR some shifted bits in the
     * cheapest possible way to reduce systematic lossage, as well as
     * to incorporate impact of the highest bits that would otherwise
     * never be used in index calculations because of table bounds.
     */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
如果Key是空，那么hash则为0，否则将当前的key的hash的高位与地位 xor 操作。原因是table使用的是power-of-two masking，也就是在映射到table里的时候是使用的 (n - 1) & hash，而n是2的倍数，因此，只有log2n的bit被用于映射到table里，所以产生碰撞的可能性非常大(下文有相关代码，看不懂可以往下看)。因此要做权衡。目前hash函数已经设计的非常好了，作者采用了一个很中庸的做法就是将高位16比特与低位16比特异或来产生新的hash值，一次来重新映射到table里。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
            // 隐射的时候是通过 (n - 1) & hash 来计算bucket下标的
        if ((p = tab[i = (n - 1) & hash]) == null)
            // 如果当前没有节点，那么要new一个新的节点
            // 当产生hash碰撞时，jdk会以链表的形式存放隐射到同一个bucket的值
            // 可以考虑到的是，如果足够极端，最后存放的值就是一个链表，以导致查找时间变为线性时间O(n)
            // 所以在java 8之后，如果冲突足够多，会以树的形式存放冲突的键值
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 如果bucket的第一个节点就是当前要的，那么直接结束
                e = p;
            else if (p instanceof TreeNode)
                // 如果是一个树，那么遍历树就行了
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 顺序查找
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        // 如果是最后一个节点，那么直接填入
                        p.next = newNode(hash, key, value, null);
                        // 太大了，就需要2的倍数增长
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 如果找到了节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }

            // 找到了就给它赋值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

看到这里有两个疑问：

1. 当binCount大到TREEIFY_THRESHOLD时，如何treeify bin的，即将线性数据结构变为树结构。

2. 树结构是如何添加节点的，即下面这段代码运行的机制

```java
 e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```


### 将线性结构转化为树结构

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        // 如果空间不够 就要resize
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            // 否则开始转成数结构，读取对应bucket的第一个Node
            TreeNode<K,V> hd = null, tl = null;
            do {
                // 替换节点
                // 内部代码就一行
                // return new TreeNode<>(p.hash, p.key, p.value, next);
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    // 保存头部
                    hd = p;
                else {
                    // 新节点 与之前的节点互相连接
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            // 将根重新设置回原来的bucket内
            if ((tab[index] = hd) != null)
                // 调用 root的treeify方法
                hd.treeify(tab);
        }
    }
```

treeify就是一个转红黑树的过程

HashMap相对于比较复杂，一个put操作就包含了查找。所以后面的函数分析都十分相似。

额外的就是resize，再重新分配大小后，会重新映射bucket。所以，可以带到如下代码
```java
newTab[e.hash & (newCap - 1)] = e;
```
因为都是2的倍数，所以依旧沿用之前的操作，理论上之前隐射到同一个bucket的，如果高位不同，那么就会分割到两个不同的bucket中了，bucket之间的距离相差之前的size

```java
 final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```