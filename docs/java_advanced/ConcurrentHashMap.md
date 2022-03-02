## Constants

```java
// The default initial table capacity. Must be a power of 2 (i.e., at least 1) and at most MAXIMUM_CAPACITY.
private static final int DEFAULT_CAPACITY = 16;
```

### DEFAULT_CAPACITY

默认Node[] 数组大小， 16



## Fields

```java
// The array of bins. Lazily initialized upon first insertion. Size is always a power of two. Accessed directly by iterators.
transient volatile Node<K,V>[] table;

// Base counter value, used mainly when there is no contention, but also as a fallback during table initialization races. Updated via CAS.
private transient volatile long baseCount;

// Table initialization and resizing control. When negative, the table is being initialized or resized: -1 for initialization, else -(1 + the number of active resizing threads). Otherwise, when table is null, holds the initial table size to use upon creation, or 0 for default. After initialization, holds the next element count value upon which to resize the table.
private transient volatile int sizeCtl;

// Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
private transient volatile int cellsBusy;

// Table of counter cells. When non-null, size is a power of 2.
private transient volatile CounterCell[] counterCells;


```

### table

懒加载，线程可见的Node数组，用来存放Node

### sizeCtl

这里面有一个非常重要的变量 `sizeCtl`，这个变量对理解整个 `ConcurrentHashMap `的原理非常重要。

`sizeCtl` 有四个含义：

- `sizeCtl < -1` 表示有 `N-1` 个线程正在执行扩容操作，如 `-2` 就表示有 `2-1` 个线程正在扩容。
- `sizeCtl = -1` 占位符，表示当前正在初始化数组。
- `sizeCtl = 0` 默认状态，表示数组还没有被初始化。
- `sizeCtl > 0` 记录可用数组大小。



### counterCells

用来计算容器实际存放的大小，由于并发下不能通过简单的size++，若通过cas，则在多线程操作的情况下，会有不断的自旋，从而影响concurrentHashMap的性能，因此作者采用了数组的方式计算容器大小。



### baseCount

基计数器值，在没有线程竞争（无并发）情况下使用，但也可以在容器初始化竞争时用作回退，通过CAS更新





### cellsBusy

默认为0，

- 0：表示当前没有线程在初始化或者扩容
- 1：表示当前自己在初始化了



## Counter support

### CounterCell

```java
@sun.misc.Contended static final class CounterCell {
    volatile long value;
    CounterCell(long x) { value = x; }
}
```









## Methonds

### initTable

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        // 当sizeCtl >=0 时，sizeCtl置为-1，表示正在初始化数组
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                // 当Node数组为空时
                if ((tab = table) == null || tab.length == 0) {
                    // 如果sizeCtl > 0, 则表示需要扩容的大小，若sizeCtl为空，则用默认大小
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    // 扩容 / 新的 Node数组
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    // * 0.75
                    sc = n - (n >>> 2);
                }
            } finally {
                // 可用数组大小
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```





### putVal

`ConcurrentHashMap ` 中存储数据采用的 `Node` 数组是采用了 `volatile` 来修饰的，但是这只能保证数组的引用在不同线程之间是可用的，并不能保证数组内部的元素在各个线程之间也是可见的，所以这里我们判定某一个下标是否有元素，并不能直接通过下标来访问

如果发现当前节点元素为空，也是通过 `CAS` 操作（`casTabAt`）来存储当前元素。

如果当前节点元素不为空，则会使用 `synchronized` 关键字锁住当前节点，并进行对应的设值操作

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    // 哈希扰动
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        // f：新增/已经存在的Node
        // fh：新增/已经存在的Node的hash
        // n：Node数组大小
        // i：put Key的下标
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        // cas判断数组对应下标下是否有元素
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 没有元素，cas进行设置
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 有元素
        else {
            // 旧值
            V oldVal = null;
            // 同步锁，锁住单个Node，锁的粒度更细
            synchronized (f) {
                // 再次判断，防止并发
                if (tabAt(tab, i) == f) {
                    // 判断是否>0，如果当前线程已经被转移了，hash值会改为负数
                    if (fh >= 0) {
                        // 计算桶的大小
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // hash值和equals都相等，则替换旧值
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                // 旧值 = 新值
                                oldVal = e.val;
                                // 存在的情况下则覆盖
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            // hash值和equals不相等，则遍历
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                // 将新Node存放进桶
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 如果是树节点
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        // 放进红黑树
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                // 如果桶长度大于 转化成红黑树的阈值
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    // 
    addCount(1L, binCount);
    return null;
}
```

可以看到，这里是通过 `tabAt` 方法来获取元素，而 `tableAt` 方法实际上就是一个 `CAS` 操作：

### tabAt

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
```



### size

调用sumCount获取值

```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}
```



### sumCount

通过counterCells计算size值，size为counterCells中的总和

```java
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```







### addCount

在 `HashMap` 中，调用 `put` 方法之后会通过 `++size` 的方式来存储当前集合中元素的个数，但是在并发模式下，这种操作是不安全的，所以不能通过这种方式

直接通过 `CAS` 操作来修改 `size` 是可行的，但是假如同时有非常多的线程要修改 `size` 操作，那么只会有一个线程能够替换成功，其他线程只能不断的尝试 `CAS`，这会影响到 `ConcurrentHashMap ` 集合的性能，所以作者就想到了一个分而治之的思想来完成计数。

作者定义了一个数组来计数，而且这个用来计数的数组也能扩容，每次线程需要计数的时候，都通过随机的方式获取一个数组下标的位置进行操作，这样就可以尽可能的降低了锁的粒度，最后获取 `size` 时，则通过遍历数组来实现计数



首先会判断 `CounterCell` 数组是不是为空，需要这里的是，这里的 `CAS` 操作是将 `BASECOUNT` 和 `baseCount` 进行比较，如果相等，则说明当前没有其他线程过来修改 `baseCount`（即 `CAS` 操作成功），此时则不需要使用 `CounterCell` 数组，而直接采用 `baseCount` 来计数。

假如 `CounterCell` 为空且 `CAS` 失败，那么就会通过调用 `fullAddCount` 方法来对 `CounterCell` 数组进行初始化。

```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    // 判断CounterCell数组不为空，或者BASECOUNT与baseCount不相等，则有竞争
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        // m：counterCells的长度
        // a：counterCells的随机元素
        // v：a的值
        CounterCell a; long v; int m;
        // 是否竞争，true表示没有线程竞争
        boolean uncontended = true;
        // counterCell计数数组为空，则直接调用fullAddCount
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            // 通过cas操作去修改当前数组下标中对应的计数，如果修改失败则表示有线程竞争
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            // 内部包括CounterCell的扩容，初始化等操作
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```





### fullAddCount

整个方法是：fore(;;)

这个方法分为两部分

- 初始化counterCells

  这里面有一个比较重要的变量 `cellsBusy`，默认是 `0`，表示当前没有线程在初始化或者扩容，所以这里判断如果 `cellsBusy==0`，而 `as` 其实在前面就是把全局变量 `CounterCell` 数组的赋值，这里之所以再判断一次就是再确认有没有其他线程修改过全局数组 `CounterCell`，所以条件满足的话就会通过 `CAS` 操作修改 `cellsBusy` 为 `1`，表示当前自己在初始化了，其他线程就不能同时进来初始化操作了。

  最后可以看到，默认是一个长度为 `2` 的数组，也就是采用了 `2` 个数组位置进行存储当前 `ConcurrentHashMap` 的元素数量。

  ```java
  // 初始化counterCells，并将cellsBuys改为1
  else if (cellsBusy == 0 && counterCells == as &&
           U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
      boolean init = false;
      try {                           // Initialize table
          if (counterCells == as) {
              CounterCell[] rs = new CounterCell[2];
              rs[h & 1] = new CounterCell(x);
              counterCells = rs;
              // 初始化完成标志
              init = true;
          }
      } finally {
          // 初始化结束后将cellsBuys值改回0
          cellsBusy = 0;
      }
      if (init)
          break;
  }
  ```

- 初始化完成后，调用fullAddCount，会进入另一个分支：

  ```java
  // as: counterCells
  // a: CounterCell
  // n: counterCells.length
  CounterCell[] as; CounterCell a; int n; long v;
  // counterCells不为空
  if ((as = counterCells) != null && (n = as.length) > 0) {
      // 如果元素为空，则需要初始化一个CounterCell
      if ((a = as[(n - 1) & h]) == null) {
          // cellsBuy为0表示counterCells没有初始化
          if (cellsBusy == 0) {            // Try to attach new Cell
              // CounterCell(1)
              CounterCell r = new CounterCell(x); // Optimistic create
              // 尝试将cellsBusy由0改为1，表示有线程在操作counterCells
              if (cellsBusy == 0 &&
                  U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                  boolean created = false;
                  try {               // Recheck under lock
                      // 计算下标，并将CounterCell对象放到counterCells数组对应下标
                      CounterCell[] rs; int m, j;
                      if ((rs = counterCells) != null &&
                          (m = rs.length) > 0 &&
                          rs[j = (m - 1) & h] == null) {
                          rs[j] = r;
                          // 是否创建成功标记设置为true
                          created = true;
                      }
                  } finally {
                      // 初始化CounterCell对象结束后将cellsBuys值改回0
                      cellsBusy = 0;
                  }
                  if (created)
                      break;
                  continue;           // Slot is now non-empty
              }
          }
          collide = false;
      }
      else if (!wasUncontended)       // CAS already known to fail
          wasUncontended = true;      // Continue after rehash
      // 对a中的值进行累加
      else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
          break;
      else if (counterCells != as || n >= NCPU)
          collide = false;            // At max size or stale
      else if (!collide)
          collide = true;
      else if (cellsBusy == 0 &&
               U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
          // 扩容
          try {
              if (counterCells == as) {// Expand table unless stale
                  CounterCell[] rs = new CounterCell[n << 1];
                  for (int i = 0; i < n; ++i)
                      rs[i] = as[i];
                  counterCells = rs;
              }
          } finally {
              cellsBusy = 0;
          }
          collide = false;
          continue;                   // Retry with expanded table
      }
      h = ThreadLocalRandom.advanceProbe(h);
  }
  ```

- 扩容分支：

  一旦会进入这个分支，就说明前面所有分支都不满足，即：

  - 当前 `CounterCell` 数组已经初始化完成。
  - 当前通过 `hash` 计算出来的 `CounterCell` 数组下标中的元素不为 `null`。
  - 直接通过 `CAS` 操作修改 `CounterCell` 数组中指定下标位置中对象的数量失败，说明有其他线程在竞争修改同一个数组下标中的元素。
  - 当前操作不满足不允许扩容的条件。
  - 当前没有其他线程创建了新的 `CounterCell` 数组，且当前 `CounterCell` 数组的大小仍然小于 `CPU` 数量。

  ```java
  else if (cellsBusy == 0 &&
           U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
      // 扩容
      try {
          if (counterCells == as) {// Expand table unless stale
              CounterCell[] rs = new CounterCell[n << 1];
              for (int i = 0; i < n; ++i)
                  rs[i] = as[i];
              counterCells = rs;
          }
      } finally {
          cellsBusy = 0;
      }
      collide = false;
      continue;                   // Retry with expanded table
  }
  ```

  



