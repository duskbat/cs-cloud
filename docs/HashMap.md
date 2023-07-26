# HashMap

```java
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
        // 以 key.equals() 为准, hash 在这里无关紧要
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
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
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
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
    if (++size > threshold)
        resize();
    // 什么都不做, LinkedHashMap 重写此方法
    afterNodeInsertion(evict);
    return null;
}
```
