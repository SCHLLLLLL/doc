## Dictionary

Dictionary是JDK1.0就出现的，目的是为了实现键值映射

Map是JDK1.2出现的

字典的作用跟Map是一样的，就是为了实现key-value映射

Dictionary是一个抽象类，具体实现为hashtable等，Dictionary已经过时了，作者推荐使用Map去实现字典

Enumeration类的目的在于，实现对Key，Value的存储，有点像链表，结构如下

```java
public interface Enumeration<E> {
    
    boolean hasMoreElements();

    E nextElement();
}
```

Dictionary中用Enumeration<K> keys()和Enumeration<V> elements()分别存放key集合和value集合

Dictionary中获取Enumeration<K> keys()或Enumeration<V> elements()通过`getEnumeration(int type)`方法，type类型为**KEY**或**VALUE**，值得注意的是，这个方法是不支持迭代器的，而hashtable中的**KeySet或EntrySet**是支持迭代器的，使用的是`getIterator(int type)`方法

```java
public synchronized Enumeration<K> keys() {
    return this.<K>getEnumeration(KEYS);
}
public synchronized Enumeration<V> elements() {
    return this.<V>getEnumeration(VALUES);
}
private <T> Enumeration<T> getEnumeration(int type) {
    if (count == 0) {
        return Collections.emptyEnumeration();
    } else {
        return new Enumerator<>(type, false);
    }
}
private <T> Iterator<T> getIterator(int type) {
    if (count == 0) {
        return Collections.emptyIterator();
    } else {
        return new Enumerator<>(type, true);
    }
}
```

HashTable中的**Enumerator**实现了**Enumeration和Iterator**接口，具体部分代码如下：

```java
private class Enumerator<T> implements Enumeration<T>, Iterator<T> {
    Entry<?,?>[] table = Hashtable.this.table;
    int index = table.length;
    Entry<?,?> entry;
    Entry<?,?> lastReturned;
    int type;

    boolean iterator;
    protected int expectedModCount = modCount;
}
```

hasMoreElements和nextElement方法在Hashtable中的具体实现为：

nextElement在

```java
public boolean hasMoreElements() {
    Entry<?,?> e = entry;
    int i = index;
    Entry<?,?>[] t = table;
    /* Use locals for faster loop iteration */
    while (e == null && i > 0) {
        e = t[--i];
    }
    // 这里会记录迭代的元素
    entry = e;
    index = i;
    return e != null;
}

@SuppressWarnings("unchecked")
public T nextElement() {
    Entry<?,?> et = entry;
    int i = index;
    Entry<?,?>[] t = table;
    /* Use locals for faster loop iteration */
    while (et == null && i > 0) {
        et = t[--i];
    }
    // 这里会记录迭代的元素
    entry = et;
    index = i;
    if (et != null) {
        Entry<?,?> e = lastReturned = entry;
        entry = e.next;
        return type == KEYS ? (T)e.key : (type == VALUES ? (T)e.value : (T)e);
    }
    throw new NoSuchElementException("Hashtable Enumerator");
}

// Iterator methods，使用迭代器的实现
public boolean hasNext() {
    return hasMoreElements();
}

public T next() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    return nextElement();
}
```

