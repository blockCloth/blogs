> TIPS
> 本文只是对 `put()` 做出基本解释，如需学习更多知识：[https://www.javabetter.cn/collection/hashmap.html](https://www.javabetter.cn/collection/hashmap.html)

## HashMap 是如何添加数据的

1. 执行`put()` 方法将数据存储到 HashMap 中
2. 调用`hash()` 方法计算数组下标
3. 执行`putVal()`存储数据
```java
/**
* 将指定值与此映射中的指定键相关联。
* 如果映射之前包含键的映射，则旧的值被替换。
*
* @param key 与指定值关联的键
* @param value 与指定键关联的值
* @return 与 key 关联的先前值
*/
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```
### `hash(key)`详解
```java
/**
* 对 key 的哈希值进行处理，得到最终的hash值
*/
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

1. `(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)`：如果键值为null，则哈希码为0，也就是说如果键为null，则将其存放到第一个位置；否则调用`hashCode()`获取键的哈希码，并将其与右移16位的哈希码进行异或运算
2. `^` 运算符：异或运算符是Java中的一种位运算符，它用于将两个数的二进制位进行比较，如果相同则为0，不同则为1
3. `h >>> 16`：将哈希码向右移动16位，相当于将原来的哈希码分成了两个16位的部分
### `putVal()`详解

1. `**if (binCount >= TREEIFY_THRESHOLD - 1)   treeifyBin(tab, hash);**`
2. static final int TREEIFY_THRESHOLD = 8;
3. 如果到了转换条件时，此时这个链表已经有了9个元素
4. 因为binCount从0开始，当binCount >= 7时，binCount表示有八个元素
5. 加上p.next = newNode(hash, key, value, null);指向了一个新元素，所以有九个元素才会触发转换条件
```java
/**
 * 实现map集合添加方法
 *
 * @param key的哈希值
 * @param 需要添加数据的key
 * @param 需要添加数据的value
 * @param onlyIfAbsent 如果为true，则不需要修改value
 * @param evict 如果为 false，则表处于创建模式
 * @return 旧值 or null
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    //初始化数据
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //如果第一次添加数据，先初始化默认容量，并获取长度，默认为16
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // tab[i = (n - 1) & hash] 计算下标位置
    if ((p = tab[i = (n - 1) & hash]) == null)
        //如果该位置是第一次添加则创建一个新Node节点并赋值到该位置
        tab[i] = newNode(hash, key, value, null);
    else { //此时不是第一次添加，需要进行判断
        Node<K,V> e; K k;
        //判断key是否相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode) //待添加的元素是否为树节点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //都不是其他判断时就添加到链表中
            //遍历链表找到插入位置
            for (int binCount = 0; ; ++binCount) {
                //此时已经找到了为节点
                if ((e = p.next) == null) {
                    //将尾节点的next指向待插入的元素
                    p.next = newNode(hash, key, value, null);
                    //判断是否转换成红黑树，（链表数量 >= 8 - 1）
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //若链表里的元素存在相同key时，则返回到最后修改元素
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //如果key相同则修改value值即可
        if (e != null) { // existing mapping for key
            //获取旧的value值
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                //修改value值
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //判断当前元素是否需要扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
#### `resize()`扩容机制

1. 由于博主能力有限，这里只是初步解释了一下代码
2. 如需了解更多，可以通过该链接学习：[https://www.javabetter.cn/collection/hashmap.html](https://www.javabetter.cn/collection/hashmap.html)
```java
/**
* 初始化或加倍表大小。 
*
* @返回表
*/
final Node<K,V>[] resize() {
    // 获取旧的哈希表
    Node<K,V>[] oldTab = table;
    // 获取旧的哈希表的大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 获取旧的阈值
    int oldThr = threshold;
    // 新的哈希表的大小和新的阈值
    int newCap, newThr = 0;
    // 如果旧的哈希表的大小大于0
    if (oldCap > 0) {
        // 如果旧的哈希表的大小已经达到了最大值
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 阈值设置为Integer的最大值
            threshold = Integer.MAX_VALUE;
            // 返回旧的哈希表
            return oldTab;
        }
        // 如果旧的哈希表的大小的两倍小于最大值，并且旧的哈希表的大小大于等于默认的初始容量
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 新的阈值为旧的阈值的两倍
            newThr = oldThr << 1; // double threshold
    }
    // 如果旧的阈值大于0，即初始容量已经在阈值中设置
    else if (oldThr > 0) 
        // 新的哈希表的大小为旧的阈值
        newCap = oldThr;
    // 旧的阈值为0，表示使用默认值
    else {               
        // 新的哈希表的大小为默认的初始容量
        newCap = DEFAULT_INITIAL_CAPACITY;
        // 新的阈值为默认的负载因子乘以默认的初始容量
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 如果新的阈值为0
    if (newThr == 0) {
        // 计算新的阈值
        float ft = (float)newCap * loadFactor;
        // 如果新的哈希表的大小小于最大容量，并且新的阈值小于最大容量
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    // 设置新的阈值
    threshold = newThr;
    // 创建新的哈希表
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    // 将新的哈希表赋值给table
    table = newTab;
    // 如果旧的哈希表不为空
    if (oldTab != null) {
        // 遍历旧的哈希表
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            // 如果旧的哈希表的当前位置不为空
            if ((e = oldTab[j]) != null) {
                // 将旧的哈希表的当前位置设置为null
                oldTab[j] = null;
                // 如果当前节点没有下一个节点
                if (e.next == null)
                    // 将当前节点放到新的哈希表的对应位置
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果当前节点是TreeNode类型
                else if (e instanceof TreeNode)
                    // 将当前节点分裂
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 其他情况
                else { 
                    // 保留顺序
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
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
    // 返回新的哈希表
    return newTab;
}
```
#### `treeifyBin()`是否转成树结构
```java
 /**
 * 替换给定哈希索引处 bin 中的所有链接节点，除非表太小，则会进行扩容
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    //判断当前数组长度是否小于64，即最小树化容量，如果小于则扩容数组
    //static final int MIN_TREEIFY_CAPACITY = 64;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    // 否则，找到哈希值对应的桶
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        // 遍历桶中的所有节点，将它们转化为树节点，并连接成双向链表
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        // 如果头节点不为空，将其设置为桶的头节点，并进行树化
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```
#### `putTreeVal()`添加树节点

1. 因为缺乏对红黑树的知识，这部分代码是通过 `GPT` 解释的
```java
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                               int h, K k, V v) {
    // kc用于缓存k的类，如果k是Comparable的实例
    Class<?> kc = null;
    // searched标记是否已经搜索过
    boolean searched = false;
    // 获取根节点，如果parent不为null，调用root()方法获取根节点，否则当前节点就是根节点
    TreeNode<K,V> root = (parent != null) ? root() : this;
    // 无限循环，直到找到合适的位置插入新节点
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;
        // 如果当前节点的hash值大于h，dir设为-1，表示新节点应该插入到当前节点的左子树
        if ((ph = p.hash) > h)
            dir = -1;
        // 如果当前节点的hash值小于h，dir设为1，表示新节点应该插入到当前节点的右子树
        else if (ph < h)
            dir = 1;
        // 如果当前节点的hash值等于h，并且key也相等，直接返回当前节点
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        // 如果k不是Comparable的实例，或者k和pk不可比，需要进行搜索
        else if ((kc == null &&
                  (kc = comparableClassFor(k)) == null) ||
                 (dir = compareComparables(kc, k, pk)) == 0) {
            // 如果还没有搜索过，进行搜索
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                // 如果左子树或右子树中找到了相等的节点，直接返回该节点
                if (((ch = p.left) != null &&
                     (q = ch.find(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                     (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            // 如果没有找到相等的节点，使用tieBreakOrder方法决定新节点是插入到左子树还是右子树
            dir = tieBreakOrder(k, pk);
        }

        // xp是当前要插入新节点的父节点
        TreeNode<K,V> xp = p;
        // 如果dir小于等于0，新节点应该插入到左子树，否则插入到右子树
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            // xpn是xp的下一个节点
            Node<K,V> xpn = xp.next;
            // 创建新的树节点
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            // 如果dir小于等于0，新节点成为xp的左子节点，否则成为右子节点
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            // xp的下一个节点设置为x，x的父节点和前一个节点都设置为xp
            xp.next = x;
            x.parent = x.prev = xp;
            // 如果xpn不为null，xpn的前一个节点设置为x
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            // 将根节点移动到前面，并进行平衡
            moveRootToFront(tab, balanceInsertion(root, x));
            // 返回null
            return null;
        }
    }
}
```
## 总结
HashMap是Java中最常用的集合之一，它是一种键值对存储的数据结构，可以根据键来快速访问对应的值。以下是对HashMap的总结：

- HashMap采用数组+链表/红黑树的存储结构，能够在O(1)的时间复杂度内实现元素的添加、删除、查找等操作。
- HashMap是线程不安全的，因此在多线程环境下需要使用`ConcurrentHashMapopen`来保证线程安全。
- HashMap的扩容机制是通过扩大数组容量和重新计算hash值来实现的，扩容时需要重新计算所有元素的hash值，因此在元素较多时扩容会影响性能。
- 在Java 8中，HashMap的实现引入了拉链法、树化等机制来优化大量元素存储的情况，进一步提升了性能。
- HashMap中的key是唯一的，如果要存储重复的key，则后面的值会覆盖前面的值。
- HashMap的初始容量和加载因子都可以设置，初始容量表示数组的初始大小，加载因子表示数组的填充因子。一般情况下，初始容量为16，加载因子为0.75。
- HashMap在遍历时是无序的，因此如果需要有序遍历，可以使用`TreeMapopen`。

综上所述，HashMap是一种高效的数据结构，具有快速查找和插入元素的能力，但需要注意线程安全和性能问题。
