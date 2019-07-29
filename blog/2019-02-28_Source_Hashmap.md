# HashMap源码阅读笔记

## tableSizeFor方法
```
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
由公式：`(a|b)>>>k = (a>>>k)|(b>>>k)` 及`a|b|b = a|b`及交换律得到一个推论： 
```
若a = n>>>0|n>>>1|n>>>2|n>>>3|....|n>>>(2的k次方-1) ，那么a | a >>> (2的k次方) = n >>> (2的k+1次方 - 1)
```
所以上面的`n>>>1`到`n>>>16`部分的代码计算相当于求 `n|n>>>1|n>>>2|n>>>3|...|n>>>31`(n取原始值），即得到的数为二进制包含最高位以下的位全部置1，例如：若n=15=0b1111，最终得到0b1111=15；若n=12=0b1100，最终得到0b1111=15；若n=0b0101，最终得到0b0111=7


## resize方法
```
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {  // 实际数量超过Integer.MAX_VALUE的一半，数组的容量设为Integer.MAX_VALUE
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 数组容量翻倍，并且旧的容量不小于于初始容量时，同时也将阈值翻倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // 新建类时有设置阈值但没设置数组容量，则以阈值作为容量
            newCap = oldThr;
        else {               // 新建时什么都没有设置，给默认值：16 和 16 * 0.75
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) { // 检查阈值设置，未设置时由 容量阈值比和容量乘积计算
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 扩容后重新放置节点
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null) // 数组位置的树只有一个节点，扩容后，新位置当然也只有一个节点
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode) // 数组+树结构时，执行树的分割
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // 数组+链表时，链表分割为两份，直接比较扩容后(capacity - 1)的值新增加的那一位即可知道分割到哪一部分
                        Node<K,V> loHead = null, loTail = null;    // 分割到低位上的节点的头与尾
                        Node<K,V> hiHead = null, hiTail = null;    // 分割到高位上的节点的头和尾
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) { // hash值对应于新扩容后的位（oldCap刚好是新扩容的长度的新增位，实际上应写为(newCap - 1 - (oldCap - 1))的值，由于刚好等于oldCap所以就减少了计算）的值是0，分割到0值的那一部分
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {   // hash值对应于(newCap - 1)的新的位的值为1，分割到1值的那一部分
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

## TreeNode结构
HashMap在数据量较小时，使用的数组+链表的结构，当数据量较大时使用数组+树的结构。由数组+链表到数组+树的结构的转化，在于`treeifyBin`方法。当链表的长度大于7时，会触发`treeifyBin`方法判断是否转变存储结构，当数组的长度也大于63时，即会使该hash值的链表结构变更为树结构。