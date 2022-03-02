# HashMap

对于HashMap，直接抛出几个问题，再讲讲HashMap里重要的方法

- 为什么HashMap的大小一定要2的n次幂？如何保证这个大小？
- 为什么链表树化的阈值为8？
- 为什么需要负载因子？



## HashMap的大小

默认大小为16

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```







## hash函数 （扰动算法

最主要是因为 下标计算是由：hashcode & (length - 1)

即取hashcode低n位作为下标

而这样hashcode在低n位的碰撞肯定不少，因此通过扰动函数，可以将低n位变得更加不确定



网上说的hashcode碰撞根本就是错的，不然hashcode碰撞都一样了，高低位混合结果不也是一样的 

