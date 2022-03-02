# ArrayList

ArrayList实现了**RandomAccess，Cloneable，Serializable**接口

ArrayList是List接口的一个实现类，底层是基于数组实现的存储结构，可以用于装载数据，数据都是存放到一个数组变量中。

```java
transient Object[] elementData; // non-private to simplify nested class access 非私有，方便嵌套类访问
```

**transient**是一个关键字，它的作用可以总结为一句话：将不需要序列化的属性前添加关键字**transient**，序列化对象的时候，这个属性就不会被序列化。你可能会觉得奇怪，ArrayList可以被序列化的啊，源码可是实现了**java.io.Serializable**接口啊，为什么数组变量还要用**transient**定义呢？

关于**transient**，文章末尾会进行解答。

当我们新建一个实例时，ArrayList会默认帮我们初始化数组的大小为**10**

```java
//Default initial capacity.
private static final int DEFAULT_CAPACITY = 10;
```

但请注意，这个只是数组的容量大小，并不是List真正的大小，List的大小应该由存储数据的数量决定，在源码中，获取真实的容量其实是用一个变量**size**来表示

```java
//The size of the ArrayList (the number of elements it contains).
private int size;
```

在源码中，数据默认是从数组的第一个索引开始存储的，当我们添加数据时，ArrayList会把数据填充到上一个索引的后面去，所以，ArrayList的数据都是有序排列的。而且，由于ArrayList本身是基于数组存储，所以查询的时候只需要根据索引下标就可以找到对于的元素，查询性能非常的高，这也是我们非常青睐ArrayList的最重要的原因。

但是，数组的容量是确定的啊，如果要存储的数据大小超过了数组大小，那不就有数组越界的问题？

关于这点，我们不用担心，ArrayList帮我们做了动态扩容的处理，如果发现新增数据后，List的大小已经超过数组的容量的话，就会新增一个为原来1.5倍容量的新数组，然后把原数组的数据原封不动的复制到新数组中，再把新数组赋值给原来的数组对象就完成了。

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
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

数组的最大容量为**Integer.MAX_VALUE（2,147,483,648）**，**而-8只是为了避免一些机器内存溢出**

```java
/**
  * The maximum size of array to allocate.
  * Some VMs reserve some header words in an array.
  * Attempts to allocate larger arrays may result in
  * OutOfMemoryError: Requested array size exceeds VM limit
  */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```

扩容之后，数组的容量足够了，就可以正常新增数据了。

除此之外，ArrayList提供支持指定index新增的方法，就是可以把数据插入到设定的索引下标，删除的时候也是一样，指定index，然后把后面的数据拷贝一份，并且向前移动，这样原来index位置的数据就删除了。

而通过index进行add和remove方法，底层都是用到**System.arraycopy**方法

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}

public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

而**System.arraycopy**方法为本地方法

```java
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

这里插入一个知识点：java的复制操作可分为**浅复制**和**深复制**，深度复制可以将对象的值和对象的内容复制，浅复制是针对对象引用的复制。

**System.arraycopy**在二维数组或一维数组中放的是对象的时候，复制结果是一维的引用变量传递给副本的一维数组，修改副本时，会影响原来的数组。

而对于简单的一维数组，这种复制属性值传递，修改副本不会影响原来的值。



到这里我们也不难发现，这种基于数组的查询虽然高效，但增删数据的时候却很耗性能，因为每增删一个元素就要移动对应index后面的所有元素，数据量少点还无所谓，但如果存储上千上万的数据就很吃力了，所以，如果是频繁增删的情况，不建议用ArrayList。

既然ArrayList不建议用的话，这种情况下有没有其他的集合可用呢？

当然有啊，这就是我们下面要说的**LinkedList**

# LinkedList

LinkedList 是基于双向链表实现的，不需要指定初始容量，链表中任何一个存储单元都可以通过向前或者向后的指针获取到前面或者后面的存储单元。在 LinkedList 的源码中，其存储单元用一个**内部类Node**表示：

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

因为有保存前后节点的地址，LinkedList增删数据的时候不需要像ArrayList那样移动整片的数据，只需要通过引用指定index位置前后的两个节点即可。

删除数据也是同样原理，只需要改变index位置前后两个节点的指向地址即可。

这样的链表结构使得LinkedList能非常高效的增删数据，在频繁增删的情景下能很好的使用，但不足之处也是有的。

虽然增删数据很快，但查询就不怎么样了，LinkedList是基于双向链表存储的，当查询对应index位置的数据时，会先计算链表总长度一半的值，判读index是在这个值的左边还是右边，然后决定从头结点还是从尾结点开始遍历。

```java
Node<E> node(int index) {
    // assert isElementIndex(index);

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

虽然已经二分法来做优化，但依然会有遍历一半链表长度的情况，如果是数据量非常多的话，这样的查询无疑是非常慢的。

这也是LinkedList最无奈的地方，鱼和熊掌不可兼得，我们既想查的快，又想增删快，这样的好事怎么可能都让我们遇到呢？所以，一般建议LinkedList使用于增删多，查询少的情景。

除此之外，LinkedList对内存的占用也是比较大的，毕竟每个Node都维护着前后指向地址的节点，数据量大的话会占用不少内存空间。

表面上看，LinkedList的Node存储结构似乎更占空间，但别忘了前面介绍ArrayList扩容的时候，它会默认把数组的容量扩大到原来的1.5倍的，如果你只添加一个元素的话，那么会有将近原来一半大小的数组空间被浪费了，如果原先数组很大的话，那么这部分空间的浪费也是不少的。

所以，如果数据量很大又在实时添加数据的情况下，ArrayList占用的空间不一定会比LinkedList空间小，这样的回答就显得谨慎些了，听上去也更加让人容易认同，但你以为这样回答就完美了吗？非也

还记得我前面说的那个transient变量吗？它的作用已经说了，不想序列化的对象就可以用它来修饰，用transient修饰elementData意味着我不希望elementData数组被序列化。为什么要这么做呢？

这是因为序列化ArrayList的时候，ArrayList里面的elementData，也就是数组未必是满的，比方说elementData有10的大小，但是我只用了其中的3个，那么是否有必要序列化整个elementData呢？显然没有这个必要，因此ArrayList中重写了writeObject方法：

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    // modCount 与 expectedModCount用来保证，在进行迭代遍历时，add或remove等操作引起的错误
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

每次序列化的时候调用这个方法，先调用defaultWriteObject()方法序列化ArrayList中的非transient元素，elementData这个数组对象不去序列化它，而是遍历elementData，只序列化数组里面有数据的元素，这样一来，就可以加快序列化的速度，还能够减少空间的开销。

一般情况下，LinkedList的占用空间更大，因为每个节点要维护指向前后地址的两个节点，但也不是绝对，如果刚好数据量超过ArrayList默认的临时值时，ArrayList占用的空间也是不小的，因为扩容的原因会浪费将近原来数组一半的容量，不过，因为ArrayList的数组变量是用transient关键字修饰的，如果集合本身需要做序列化操作的话，ArrayList这部分多余的空间不会被序列化。

