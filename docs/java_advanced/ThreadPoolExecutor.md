# ThreadPoolExecutor

> 

## 成员变量

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

### AtomicInteger ctl

**ctl**代表线程池的状态，借助高低位**复合包装**了两个概念：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

- workCount：线程池中当前活动的数量，占据ctl的**低29位**；
- runState：线程池的运行状态，占据ctl的**高3位**，有RUNNING、SHUTDOWN、STOP、TIDYING、TERMINATED五种状态。

### COUNT_BITS

**COUNT_BITS**代表workcount占据的位数：29

```java
private static final int COUNT_BITS = Integer.SIZE - 3;
```

### workerCount

既然workerCount代表了线程池中当前活动的线程数量，那么它肯定有个上下限阈值，下限很明显就是0，上限为CAPACITY，ThreadPoolExecutor中理论上的最大活跃线程数，其定义如下：

```java
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
```

为1 << 29 - 1 = 2^29 - 1 ≈ 5亿

### ctl如何包装两个概念

ctl是通过以下方法包装线程池状态和工作线程数的：

```java
 // Packing and unpacking ctl
 private static int runStateOf(int c)     { return c & ~CAPACITY; }
 private static int workerCountOf(int c)  { return c & CAPACITY; }
 private static int ctlOf(int rs, int wc) { return rs | wc; }
```

**Packing and unpacking ctl**的意思是包装和拆包，即传入的c代表的是ctl的值，也就是高3位为线程池状态，低29位为线程池中当前活动的线程数量workerCount，将其余CAPACITY进行与操作，也就是与**000 11111 11111111 11111111 11111111**进行与操作。

- 若与CAPACITY进行与运算，前三位不管是什么都会为0，因此，而c的低29位于29个1进行与操作，c的低29位还是会保持原来的大小，因此就可以通过**AtomicInteger ctl**解析出当前活跃线程数workerCount。
- 若与CAPACITY的取反进行与运算，得到的结果则会保存高3位，同理可以通过**AtomicInteger ctl**解析出当前线程池的状态runState。

## 线程池状态

runState（int 32位）有五种状态：

- RUNNING：接受新任务，并处理队列任务

  ```java
  private static final int RUNNING    = -1 << COUNT_BITS;
  ```

  -1在使用时为补码，即：32个1，因此左移29位后为**111** 00000 00000000 00000000 00000000，也就是低29位全部为0，高3位全部为1的话，表示RUNNING状态，即-536870912；

- SHUTDOWN：不接受新任务，但会处理队列任务

  ```java
  private static final int SHUTDOWN   =  0 << COUNT_BITS;
  ```

   0的二进制表示为32个0，无论左移多少位，还是32个0，即**000** 00000 00000000 00000000 00000000，也就是低29位全部为0，高3位全部为0的话，表示SHUTDOWN状态，即0

- STOP：不接受新任务，不会处理队列任务，而且会中断正在处理过程中的任务

  ```java
  private static final int STOP       =  1 << COUNT_BITS;
  ```

   1的二进制表示为前31位为0，最后一位为1，左移29位为，**001** 00000 00000000 00000000 00000000，也就是低29位全部为0，高3位为001的话，表示STOP状态，即536870912；

- TIDYING：所有的任务已结束，workerCount为0，线程过渡到TIDYING状态，将会执行terminated()钩子方法

  ```java
  private static final int TIDYING    =  2 << COUNT_BITS;
  ```

  2的二进制表示为前30个0和1个10组成的，左移29位为，**010** 00000 00000000 00000000 00000000，也就是低29位全部为0，高3位为010的话，表示TIDYING状态，即1073741824；

- TERMINATED：terminated()方法已经完成

  ```java
  private static final int TERMINATED =  3 << COUNT_BITS;
  ```

  3的二进制表示为前30个0和1个11组成的，左移29位为，**011** 00000 00000000 00000000 00000000，也就是低29位全部为0，高3位为011的话，表示TERMINATED状态，即1610612736

## 初始状态

原子变量ctl的初始化方法ctlOf()，代码如下：

```java
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

传入的rs表示线程池运行状态runState，其是高3位有值，低29位全部为0的int，而wc则代表线程池中有效线程的数量workerCount，其为高3位全部为0，而低29位有值得int，将runState和workerCount做或操作|处理，即用runState的高3位，workerCount的低29位填充的数字。因此runState的初始值为**RUNNING**、workerCount为**0**。







