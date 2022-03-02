## HashTable

承接Dictionary类，里面说到**getEnumeration(int type)**是不支持迭代器的，而HashTable中实现Map接口后，实现keySet()、entrySet()、values()中就是通过迭代器来获取，下面是源码实现：

值得注意的是，hashTable为线程安全的，在没处不为原子操作的地方都加了synchronized，因此在keySet()、entrySet()、values()中也是通过**Collections.synchronizedSet**实现

keySet:

```java
public Set<K> keySet() {
    if (keySet == null)
        keySet = Collections.synchronizedSet(new KeySet(), this);
    return keySet;
}

private class KeySet extends AbstractSet<K> {
    public Iterator<K> iterator() {
        // 使用getIlterator获取带有迭代器的Enumerator
        return getIterator(KEYS);
    }
    public int size() {
        return count;
    }
    public boolean contains(Object o) {
        return containsKey(o);
    }
    public boolean remove(Object o) {
        return Hashtable.this.remove(o) != null;
    }
    public void clear() {
        Hashtable.this.clear();
    }
}
```

values:

```java
public Collection<V> values() {
    if (values==null)
        values = Collections.synchronizedCollection(new ValueCollection(), this);
    return values;
}

private class ValueCollection extends AbstractCollection<V> {
    public Iterator<V> iterator() {
        return getIterator(VALUES);
    }
    public int size() {
        return count;
    }
    public boolean contains(Object o) {
        return containsValue(o);
    }
    public void clear() {
        Hashtable.this.clear();
    }
}
```

entrySet:

```java
public Set<Map.Entry<K,V>> entrySet() {
    if (entrySet==null)
        entrySet = Collections.synchronizedSet(new EntrySet(), this);
    return entrySet;
}

private class EntrySet extends AbstractSet<Map.Entry<K,V>> {
    public Iterator<Map.Entry<K,V>> iterator() {
        return getIterator(ENTRIES);
    }

    public boolean add(Map.Entry<K,V> o) {
        return super.add(o);
    }

    public boolean contains(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> entry = (Map.Entry<?,?>)o;
        Object key = entry.getKey();
        Entry<?,?>[] tab = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;

        for (Entry<?,?> e = tab[index]; e != null; e = e.next)
            if (e.hash==hash && e.equals(entry))
                return true;
        return false;
    }

    public boolean remove(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> entry = (Map.Entry<?,?>) o;
        Object key = entry.getKey();
        Entry<?,?>[] tab = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;

        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for(Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
            if (e.hash==hash && e.equals(entry)) {
                modCount++;
                if (prev != null)
                    prev.next = e.next;
                else
                    tab[index] = e.next;

                count--;
                e.value = null;
                return true;
            }
        }
        return false;
    }

    public int size() {
        return count;
    }

    public void clear() {
        Hashtable.this.clear();
    }
}
```