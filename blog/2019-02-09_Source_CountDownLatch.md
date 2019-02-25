# CountDownLatch源码阅读
## await方法如何实现线程等待
`await`方法，由`CountDownLatch.Sync.acquireSharedInterruptibly`代理完成，实际上由Sync的父类`AbstractQueuedSynchronizer`实现了该方法
```
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```
由于arg参数入参固定为1, 而Sync实现的`tryAcquireShared`方法对于入参数1返回-1，因此实际相当于调用了如下代码，`Thread.interrupted`确保了当前线程不是interrupted的状态
```
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        doAcquireSharedInterruptibly(arg);
    }
```
`AbstractQueuedSynchronizer`类的`doAcquireSharedInterruptibly`方法，其中`addWaiter`方法是往Node链表最后增加一个Node节点并返回该节点，Node对象中存储了当前线程t
```
    private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
实际上由于arg值固定为1，上面方法相当于如下
```
    private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
`shouldParkAfterFailedAcquire`方法判断当前新增的节点（node）的前置节点（p）是否持有信号，并在没有持有信号的情况下，使其变更为持有信号状态（使state值设为SIGNAL）。`doAcquireSharedInterruptibly`中for循环直到`shouldParkAfterFailedAcquire`方法判断node的前置节点持有信号时，才会调用`parkAndCheckInterrupt`方法。

`parkAndCheckInterrupt`代码便是阻塞代码真正所在的位置，该功方法代码很简单，直接委托`LockSupport.park(this)`完成，`LockSupport.park(this)`代码如下
```
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }
```
`setBlocker`时native的方法，它的作用是将第二个参数存储在第一个参数（线程）的堆内存中（即这是一个直接操作堆内存存储的实现），上面`park`方法中`UNSAFE.park`为阻塞功能的实现，点进去看发现也是一个native修饰的方法，即真正阻塞功能的还是由底层实现的，没法看到具体的代码。

## countDown方法如何实现计数递减并取消阻塞
与`await`方法一样，`countDown`方法也是将工作委托给了Sync类的方法完成。`CountDownLatch.Sync.releaseShared`代理完成该功能，`CountDownLatch.Sync.releaseShared`实际上也是已经由Sync的父类`AbstractQueuedSynchronizer`实现了，该方法如下
```
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```
`tryReleaseShared`方法尝试进行计数（即`state`）递减，`tryReleaseShared`方法中与前面一些方法一样，使用了原子性的方法`compareAndSetState`完成递减操作，该原子性操作保证了不会出现两次countDown之后`state`只递减一次的情况。当本次递减后计数达到0返回true，原先计数已经为0或者本次递减后计数不为0，返回false。`tryReleaseShared`返回true后，才能调用`doReleaseShared`完成Node恢复阻塞的线程。
```
    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
```
如下所示为`doReleaseShared`方法代码，当`ws == Node.SIGNAL`（持有信号）时，调用`compareAndSetWaitStatus`尝试使Node不再持有信号（state值从SIGNAL变为0，`compareAndSetWaitStatus`调用的`unsafe.compareAndSwapInt`方法是原子性的方法），释放信号成功则进行`unparkSuccessor`的调用，不成功则继续循环（一般另外一条线程同时操作这个node的时候才会导致不成功），当`ws == 0`时，将该Node标识转为PROPAGATE后不再作处理，这种情况基本与`CountDownLatch`的业务无关，只是`AbstractQueuedSynchronizer`对Node链表结构的维护工作
```
    private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```
最后`unparkSuccessor`方法，找到head Node的下一个Node s，找到阻塞的线程`s.thread`， 调用`LockSupport.unpark(s.thread)`解除线程阻塞（核心代码`UNSAFE.unpark(thread)`方法，是native方法，发现一点有意思的情况：该方法运行完会改变head的指向，虽然与这里研究的关系不大，但传入参数是thread却能改变Sync的head Node指针确是挺有意思的）
```
private void unparkSuccessor(Node node) {
        
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

## 总结
CountDownLatch基本由`CountDownLatch.Sync`代理，Sync大量调用了`sun.misc.Unsafe`的代码。使用Unsafe的用于改变变量值的原子性方法，减少一些锁的使用；另外一点是使用park和`sun.misc.Unsaft.park`和`sun.misc.Unsaft.unpark`方法来实现指定线程的阻塞与解除阻塞。可惜的是`sun.misc.Unsafe`类oracle并不开放给外部代码使用。
