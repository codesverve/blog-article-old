# ArrayList、Vector、CopyOnWriteArrayList对比

ArrayList、Vector、CopyOnWriteArrayList均是实现了List接口的容器，他们之间究竟有何区别？为何JDK要同时提供这三种List？本文针对这个问题，进行深入的探究。

## ArrayList

### 1. 研究一个类源码，首先了解它有哪些成员变量

查看ArrayList源码，可以很清除的看到，ArrayList最主要的成员变量就两个，变量的作用很明显，存储对象数组和记录数组内存储的变量个数，ArrayList内元素是以Array方式存储的，所以叫做ArrayList

```
/**
* The array buffer into which the elements of the ArrayList are stored.
* The capacity of the ArrayList is the length of this array buffer. Any
* empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
* will be expanded to DEFAULT_CAPACITY when the first element is added.
*/
transient Object[] elementData; // non-private to simplify nested class access

/**
* The size of the ArrayList (the number of elements it contains).
*
* @serial
*/
private int size;
```

### 2. 针对特征功能的探究

ArrayList之所以是一个List，而不是Array，就在于ArrayList长度可以扩展，这里探究它在add元素时是如何进行扩容的。

查看add代码

```
/**
* Appends the specified element to the end of this list.
*
* @param e element to be appended to this list
* @return <tt>true</tt> (as specified by {@link Collection#add})
*/
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

可以看出ArrayList的扩容是在ensureCapacityInternal方法中完成的，按着ensureCapacityInternal方法顺藤摸瓜可以找到grow方法这里才是真正的扩容代码，可以看出ArrayList扩容是直接用新的更长的数组替换掉旧的elementData数组，扩容后新容量是扩容前的容量的1.5倍，并与实际需求容量取最大值。

```
/**
* Increases the capacity to ensure that it can hold at least the
* number of elements specified by the minimum capacity argument.
*
* @param minCapacity the desired minimum capacity
*/
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

## Vector

Vector类相比较ArrayList类，在成员变量上多了一个`protected int capacityIncrement`变量，主要用于扩容时的容量增长的计算，也就是说，Vector容量增长值的算法与ArrayList是有所不同的。

将关注点置于Vector与ArrayList类的不同点上，阅读源码可发现Vector类在`get`、`add`、`set`、`addAll`、`forEach`、`toArray`等方法上，均加了`synchronized`关键字，这里可以看出，Vector类与ArrayList相比增加了线程安全方面的考虑。

## CopyOnWriteArrayList

成员变量上，主要增加了一个ReentrantLock变量。

```
/** The lock protecting all mutators */
final transient ReentrantLock lock = new ReentrantLock();

/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
```

了解`ReentrantLock`类的都知道，这是一个可重入锁，也就是说CopyOnWriteArrayList和Vector一样都有线程安全方面的考虑。那CopyOnWriteArrayList和Vector的区别是什么呢？

阅读CopyOnWriteArrayList源码发现，CopyOnWriteArrayList类在`get`、`forEach`、`toArray`等读的方法上，并没有加锁。在`add`、`set`、`addAll`等写的方法上除了使用了ReentrantLock锁之外，还增加了Array Copy的操作，使用Copy出的新的Array替换了就的Array，也正是因为这个操作，使得CopyOnWriteArrayList类在写的操作上不需要加锁。

## 总结

ArrayList与 CopyOnWriteArrayList和Vector的区别在于，CopyOnWriteArrayList和Vector是线程安全的，ArrayList不是线程安全的。CopyOnWriteArrayList和Vector的区别在于，CopyOnWriteArrayList读时不加锁，Vector读时加锁，CopyOnWriteArrayList写时进行了Array的拷贝，而Vector不进行拷贝。从性能上看，不加锁的ArrayList读写速度肯定都是最快的，而Vector写的速度比CopyOnWriteArrayList快，CopyOnWriteArrayList读的速度比Vector块。

