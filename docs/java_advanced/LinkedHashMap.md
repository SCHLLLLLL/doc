## LinkHashMap



实现了自己的Node结构

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```





我们知道，hashMap在putVal的时候，会调用**newNode**，而LinkedHashMap重写了这个方法，其新创建的Node为LinkedHashMap中的Entry，并在新建完Node之后，将Node添加到链表尾部。

```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

// link at the end of list
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
```



下面是HashMap预留给LinkedHashMap的回调方法

- afterNodeInsertion
- afterNodeAccess
- afterNodeRemoval

在实现为LinkedHashMap，进行put操作时，会调用**afterNodeInsertion**方法。而此方法的作用：**根据evict参数决定是否删除最老的插入的元素（第一个节点）**

```jade
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

在实现为LinkedHashMap，进行putVal，replace，computeIfAbsent，computeIfPresent，compute，merge操作时，或者在设置了**accessOrder（排序模式）的get，getOrDefault**时，会调用**afterNodeAccess**方法。而此方法的作用为：**在遍历到此Node时，将Node移动至链表尾部**，值得注意的是，**如果在accessOrder模式下遍历LinkedHashMap，会修改modCount值，如果同时访问数据，会导致fast-fail**

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```







### containsValue

LinkedHashMap重写了**containsValue**方法，具体实现为：

```java
public boolean containsValue(Object value) {
    for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
        V v = e.value;
        if (v == value || (value != null && value.equals(v)))
            return true;
    }
    return false;
}
```

HashMap中**containsValue**方法：

```java
public boolean containsValue(Object value) {
    Node<K,V>[] tab; V v;
    if ((tab = table) != null && size > 0) {
        for (int i = 0; i < tab.length; ++i) {
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                if ((v = e.value) == value ||
                    (value != null && value.equals(v)))
                    return true;
            }
        }
    }
    return false;
}
```

对比HashMap，LinkedHashMap遍历一边链表，更加高效，时间复杂度为O(n)，而hashMap为0(n^2)，

