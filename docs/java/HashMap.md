# <center>HashMap<center/>

> 源码版本: 8u202

## <center>构造函数<center/>

构造函数初始化**负载因子**和**扩容阈值**, 并没有初始化 table, 也没有设定 capacity

-   4 个重载:

    ```java
    public HashMap()
    public HashMap(int initialCapacity)
    public HashMap(int initialCapacity, float loadFactor)
    public HashMap(Map<? extends K, ? extends V> m)
    ```

-   `HashMap()`  
    只初始化负载因子, 换句话说, thr=0

    ```java
    this.loadFactor = 0.75;
    ```

-   `HashMap(int initialCapacity)`  
    有参构造函数把 threshold 设为了 2 的整数倍

    > 如果入参 cap=0, 则 thr=1

    ```java
    this.loadFactor = 0.75;
    this.threshold = tableSizeFor(initialCapacity);

    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
    ```

<br/>

---

## <center>关键方法<center/>

### hash

HashMap 中的 `hash()` 会将 hashcode 高 16 位跟低 16 位异或在一起

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

### get

```java
final Node<K, V> getNode(int hash, Object key) {
    Node<K, V>[] tab;
    Node<K, V> first, e;
    int n;
    K k;
    if ((tab = table) != null &&
            (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
        // 检查第0个
        if (first.hash == hash && ((k = first.key) == key ||
                (key != null && key.equals(k))))
            return first;

        if ((e = first.next) != null) {
            // 树节点
            if (first instanceof TreeNode)
                return ((TreeNode<K, V>) first).getTreeNode(hash, key);
            // 遍历链表
            do {
                if (e.hash == hash && ((k = e.key) == key ||
                        (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### put

过程描述:

1. 首先判定是否空数组, 如果是空数组 `resize()`
2. 计算 key 在 table 中的 `idx = (n-1) & hash`
3. 放进桶中

    1. 如果是空桶, `new Node()`
    2. 如果是树节点, 塞进树里
    3. 从头到尾搜索链表节点

        1. 没找到, 尾插; 尾插后树形化判定
        2. 找到则标记

        如果找到, 判断是否替换, return;

4. 最后还有一次扩容判定

```java
static final int TREEIFY_THRESHOLD = 8;

// put方法会返回旧值
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K, V>[] tab;
    Node<K, V> p;
    int n, i;
    // 空 table
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 空桶
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

    else {
        Node<K, V> e;
        K k;
        // 与桶中链表第0个相等
        // hash 很重要
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // TreeNode
        else if (p instanceof TreeNode)
            e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
        // 遍历桶中链表
        else {
            // binCount 计数
            for (int binCount = 0; ; ++binCount) {
                // 最后一个元素
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 链表数量 >= 8, 树形化
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 找到相同的key
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                // next
                p = e;
            }
        }

        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // put() 传入的 onlyIfAbsent=false, 只替换等于null的Val
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 什么都不做, LinkedHashMap 重写此方法
            afterNodeAccess(e);
            return oldValue;
        }
    }

    ++modCount;
    if (++size > threshold) resize();
    // 什么都不做, LinkedHashMap 重写此方法
    afterNodeInsertion(evict);
    return null;
}
```

### remove

过程描述:

1. 首先判断表和桶不为空
2. 如果首位就是目标值, 标记
3. 如果 next!=null
    1. 是树节点, 树查找, 标记
    2. 是链表节点, 链表查找, 标记
4. 如果找到
    1. 树节点, 删除
    2. 头节点, 删除
    3. 链表节点, 删除

```java
final Node<K, V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
    Node<K, V>[] tab;
    Node<K, V> p; // pos, 桶位第0个Node
    int n, index;
    // 桶位不为空
    if ((tab = table) != null &&
        (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        // node 指向找到的Node
        Node<K, V> node = null, e;
        K k;
        V v;
        // 第0个
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        // 往下找
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K, V>) p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }

        if (node != null &&
            (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K, V>) node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;

            ++modCount;
            --size;
            // 什么都不做, LinkedHashMap 实现该方法
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

### treeifyBin

过程描述:

1. 首先扩容判定, cap<64
2. 构建 TreeNode 类型的双向链表
3. 桶的 head 重置为刚创建的 head
4. 双向链表树形化 treeify()

```java
static final int MIN_TREEIFY_CAPACITY = 64;

final void treeifyBin(Node<K, V>[] tab, int hash) {
    int n, index;
    Node<K, V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K, V> hd = null, tl = null;
        do {
            TreeNode<K, V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);

        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

### resize

1. 计算 新阈值和新容量

    1. **无参初始化** `(cap==0 && thr==0)`  
       新阈值 = 16\*0.75 = 12  
       新容量 = 16
    2. **含参初始化** `(cap==0 && thr>0)`  
       新容量 = 老阈值

        > 老阈值在构造方法中已设为 2 的幂

    3. **非初始化** `(cap!=0)`

        1. 如果容量已经最大, 阈值拉满, return;
        2. 否则: 容量 2 倍扩容; 如果 2 倍还没最大, 并且原容量>=16:  
           阈值 \*= 2

    如果新阈值没设定

    > 此时有 2 种情况, (含参初始化 or 旧容量<16) 都是含参初始化造成的;

    - 新阈值 = 新容量 \* factor

        > 在容量小于 16 的情况下, 每次都计算新阈值 `newThr=newCap*factor`, 以精确快速扩容(thr 直接\*2 不精确)

2. 扩容
   按照新容量扩容

3. 重新散列
   遍历每个桶

    - 单节点
    - 树节点: 判定是否需要链表化
    - 链表节点: 拆成两条链表

        > (n-1) & hash  
        > n-1: 00001111 -> 00011111  
        > hash:00010101 -> 00010101  
        > 要么在原位置, 要么在原位置+原容量的位置

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 16

static final int MAXIMUM_CAPACITY = 1 << 30; // 1073741824

final Node<K, V>[] resize() {
    Node<K, V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 先计算cap, thr
    // cap>0
    if (oldCap > 0) {
        // 已经最大容量, 把下次触发的阈值拉满
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // cap 扩容
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 如果 cap>=dft_cap, thr也扩
            newThr = oldThr << 1;
    }
    // cap=0 && thr>0
    else if (oldThr > 0) newCap = oldThr;
    // cap=0 && thr=0
    else {
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }

    if (newThr == 0) {
        float ft = (float) newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
            (int) ft :
            Integer.MAX_VALUE);
    }
    threshold = newThr;

    @SuppressWarnings({"rawtypes", "unchecked"})
    Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
    table = newTab;
    // old_tab ==> new_tab
    if (oldTab != null) {
        // 遍历每个桶
        for (int j = 0; j < oldCap; ++j) {
            Node<K, V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 单个元素
                if (e.next == null)
                    // 要么new_idx==old_idx, 要么new_idx=old_idx+old_cap
                    newTab[e.hash & (newCap - 1)] = e;
                // TreeNode
                else if (e instanceof TreeNode)
                    ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                // ListNode
                else {
                    Node<K, V> loHead = null, loTail = null;
                    Node<K, V> hiHead = null, hiTail = null;
                    Node<K, V> next;
                    do {
                        next = e.next;
                        // 取最高位, 如果0就是原位
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null) loHead = e;
                            else loTail.next = e;
                            loTail = e;
                        } else {
                            if (hiTail == null) hiHead = e;
                            else hiTail.next = e;
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

<br/>

---

## <center>细节提问<center/>

### 如何判定 key 相等

```java
// hash && (== || equal)
if (p.hash == hash &&
    ((k = p.key) == key || (key != null && key.equals(k))))
```

---

### 什么时候链表化

1. `resize() -> TreeNode.split() -> untreeify()`
2. `removeNode() -> TreeNode.removeTreeNode() -> untreeify()`

条件: 桶中元素数<=6

---

### 什么时候树形化

-   `put() -> treeifyBin()`

条件: 桶中元素数>=8

---

### table 什么时候扩容

-   `put() -> resize()`

调用 `resize()` 的场景:

1. 空数组
2. `put()` 函数最后
3. `treeifyBin()` 的时候 `if tab.length < MIN_TREEIFY_CAPACITY`, 则 `resize()`

---

## <center>tips<center/>

可以试着 debug 一下, 关注一下 `resize()` 的细节

```java
HashMap<Integer, Integer> map = new HashMap<>(0);
map.put(1, 1);
map.put(2, 2);
```
