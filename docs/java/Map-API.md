# Map API

学习一下这些函数的使用, 算法中用起来比较方便

```java
interface java.util.Map<K, V> {

    // func: (k,v) -> {}
    default void forEach(BiConsumer action);

    // func: (k,v) -> new_v
    default void replaceAll(BiFunction func);

    // 替换
    default V replace(K key, V value);

    // 替换, k,v 严格一致
    default boolean replace(K key, V oldValue, V newValue);

    // k,v 严格一致
    default boolean remove(Object key, Object value);

    default V getOrDefault(Object key, V defaultValue);

    // 如果没有则 put
    default V putIfAbsent(K key, V value);

    // func: (k,v) -> new_v
    default V compute(K key, BiFunction func);

    // func: k -> new_v
    default V computeIfAbsent(K key, Function func);

    // func: (k,v) -> new_v
    default V computeIfPresent(K key, BiFunction func);

    // func: (old_v, value) -> new_v
    // 例如: map.merge("key", 1, Integer::sum); 常用于计数
    default V merge(K key, V value, BiFunction func);

}
```
