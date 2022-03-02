# 锁



锁的目的：



总结一句话：





## Sychronized









## Lock







## AQS







```java
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

