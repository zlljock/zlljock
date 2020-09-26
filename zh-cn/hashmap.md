### 简介

**底层原理**：数组+链表+红黑树

**线程安全性**：线程不安全

HashMap 允许 key 和 value 为 null

### 基本属性

```java
// 默认容量16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
 
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;    
 
// 默认负载因子0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f; 
 
// 链表节点转换红黑树节点的阈值, 8个节点转
static final int TREEIFY_THRESHOLD = 8; 
 
// 红黑树节点转换链表节点的阈值, 6个节点转
static final int UNTREEIFY_THRESHOLD = 6;   
 
// 转红黑树时, table的最小长度
static final int MIN_TREEIFY_CAPACITY = 64; 
 
// 链表节点, 继承自Entry
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
 
    // ... ...
}
 
// 红黑树节点
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
   
    // ...
}
```

### 定位哈希桶数组索引位置

不管增加、删除、查找键值对，定位到哈希桶数组的位置都是很关键的第一步。前面说过 HashMap 的数据结构是“数组+链表+红黑树”的结合，所以我们当然希望这个 HashMap 里面的元素位置尽量分布均匀些，尽量使得每个位置上的元素数量只有一个，那么当我们用 hash 算法求得这个位置的时候，马上就可以知道对应位置的元素就是我们要的，不用遍历链表/红黑树，大大优化了查询的效率。HashMap 定位数组索引位置，直接决定了 hash 方法的离散性能。下面是定位哈希桶数组的源码：

```java
// 代码1
static final int hash(Object key) { // 计算key的hash值
    int h;
    // 1.先拿到key的hashCode值; 2.将hashCode的高16位参与运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// 代码2
int n = tab.length;
// 将(tab.length - 1) 与 hash值进行&运算
int index = (n - 1) & hash;
```

整个过程本质上就是三步：

1. 拿到 key 的 hashCode 值
2. 将 hashCode 的高位参与运算，重新计算 hash 值
3. 将计算出来的 hash 值与 (table.length - 1) 进行 & 运算



### get方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
 
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 1.对table进行校验：table不为空 && table长度大于0 && 
    // table索引位置(使用table.length - 1和hash值进行位与运算)的节点不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 2.检查first节点的hash值和key是否和入参的一样，如果一样则first即为目标节点，直接返回first节点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 3.如果first不是目标节点，并且first的next节点不为空则继续遍历
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                // 4.如果是红黑树节点，则调用红黑树的查找目标节点方法getTreeNode
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                // 5.执行链表节点的查找，向下遍历链表, 直至找到节点的key和入参的key相等时,返回该节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    // 6.找不到符合的返回空
    return null;
}
```

#### 如果是红黑树的话

```java
final TreeNode<K,V> getTreeNode(int h, Object k) {
    // 1.首先找到红黑树的根节点；2.使用根节点调用find方法
    return ((parent != null) ? root() : this).find(h, k, null);
}
```



#### find方法

```java
/**
 * 从调用此方法的节点开始查找, 通过hash值和key找到对应的节点
 * 此方法是红黑树节点的查找, 红黑树是特殊的自平衡二叉查找树
 * 平衡二叉查找树的特点：左节点<根节点<右节点
 */
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
    // 1.将p节点赋值为调用此方法的节点，即为红黑树根节点
    TreeNode<K,V> p = this;
    // 2.从p节点开始向下遍历
    do {
        int ph, dir; K pk;
        TreeNode<K,V> pl = p.left, pr = p.right, q;
        // 3.如果传入的hash值小于p节点的hash值，则往p节点的左边遍历
        if ((ph = p.hash) > h)
            p = pl;
        else if (ph < h) // 4.如果传入的hash值大于p节点的hash值，则往p节点的右边遍历
            p = pr;
        // 5.如果传入的hash值和key值等于p节点的hash值和key值,则p节点为目标节点,返回p节点
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if (pl == null)    // 6.p节点的左节点为空则将向右遍历
            p = pr;
        else if (pr == null)    // 7.p节点的右节点为空则向左遍历
            p = pl;
        // 8.将p节点与k进行比较
        else if ((kc != null ||
                  (kc = comparableClassFor(k)) != null) && // 8.1 kc不为空代表k实现了Comparable
                 (dir = compareComparables(kc, k, pk)) != 0)// 8.2 k<pk则dir<0, k>pk则dir>0
            // 8.3 k<pk则向左遍历(p赋值为p的左节点), 否则向右遍历
            p = (dir < 0) ? pl : pr;
        // 9.代码走到此处, 代表key所属类没有实现Comparable, 直接指定向p的右边遍历
        else if ((q = pr.find(h, k, kc)) != null) 
            return q;
        // 10.代码走到此处代表“pr.find(h, k, kc)”为空, 因此直接向左遍历
        else
            p = pl;
    } while (p != null);
    return null;
}
```

### put方法 

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
 
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 1.校验table是否为空或者length等于0，如果是则调用resize方法进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 2.通过hash值计算索引位置，将该索引位置的头节点赋值给p，如果p为空则直接在该索引位置新增一个节点即可
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // table表该索引位置不为空，则进行查找
        Node<K,V> e; K k;
        // 3.判断p节点的key和hash值是否跟传入的相等，如果相等, 则p节点即为要查找的目标节点，将p节点赋值给e节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 4.判断p节点是否为TreeNode, 如果是则调用红黑树的putTreeVal方法查找目标节点
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 5.走到这代表p节点为普通链表节点，则调用普通的链表方法进行查找，使用binCount统计链表的节点数
            for (int binCount = 0; ; ++binCount) {
                // 6.如果p的next节点为空时，则代表找不到目标节点，则新增一个节点并插入链表尾部
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 7.校验节点数是否超过8个，如果超过则调用treeifyBin方法将链表节点转为红黑树节点，
                    // 减一是因为循环是从p节点的下一个节点开始的
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }
                // 8.如果e节点存在hash值和key值都与传入的相同，则e节点即为目标节点，跳出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;  // 将p指向下一个节点
            }
        }
        // 9.如果e节点不为空，则代表目标节点存在，使用传入的value覆盖该节点的value，并返回oldValue
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e); // 用于LinkedHashMap
            return oldValue;
        }
    }
    ++modCount;
    // 10.如果插入节点后节点数超过阈值，则调用resize方法进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);  // 用于LinkedHashMap
    return null;
}
```

#### putTreeVal方法

```java
/**
 * 红黑树的put操作，红黑树插入会同时维护原来的链表属性, 即原来的next属性
 */
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                               int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    // 1.查找根节点, 索引位置的头节点并不一定为红黑树的根节点
    TreeNode<K,V> root = (parent != null) ? root() : this;
    // 2.将根节点赋值给p节点，开始进行查找
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;
        // 3.如果传入的hash值小于p节点的hash值，将dir赋值为-1，代表向p的左边查找树
        if ((ph = p.hash) > h)
            dir = -1;
        // 4.如果传入的hash值大于p节点的hash值， 将dir赋值为1，代表向p的右边查找树
        else if (ph < h)
            dir = 1;
        // 5.如果传入的hash值和key值等于p节点的hash值和key值, 则p节点即为目标节点, 返回p节点
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        // 6.如果k所属的类没有实现Comparable接口 或者 k和p节点的key相等
        else if ((kc == null &&
                  (kc = comparableClassFor(k)) == null) ||
                 (dir = compareComparables(kc, k, pk)) == 0) {
            // 6.1 第一次符合条件, 从p节点的左节点和右节点分别调用find方法进行查找, 如果查找到目标节点则返回
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null &&
                     (q = ch.find(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                     (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            // 6.2 否则使用定义的一套规则来比较k和p节点的key的大小, 用来决定向左还是向右查找
            dir = tieBreakOrder(k, pk); // dir<0则代表k<pk，则向p左边查找；反之亦然
        }
 
        TreeNode<K,V> xp = p;   // xp赋值为x的父节点,中间变量,用于下面给x的父节点赋值
        // 7.dir<=0则向p左边查找,否则向p右边查找,如果为null,则代表该位置即为x的目标位置
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            // 走进来代表已经找到x的位置，只需将x放到该位置即可
            Node<K,V> xpn = xp.next;    // xp的next节点
            // 8.创建新的节点, 其中x的next节点为xpn, 即将x节点插入xp与xpn之间
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            // 9.调整x、xp、xpn之间的属性关系
            if (dir <= 0)   // 如果时dir <= 0, 则代表x节点为xp的左节点
                xp.left = x;
            else        // 如果时dir> 0, 则代表x节点为xp的右节点
                xp.right = x;
            xp.next = x;    // 将xp的next节点设置为x
            x.parent = x.prev = xp; // 将x的parent和prev节点设置为xp
            // 如果xpn不为空,则将xpn的prev节点设置为x节点,与上文的x节点的next节点对应
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            // 10.进行红黑树的插入平衡调整
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```



#### resize()方法

老表的容量为 0，并且老表的阈值大于 0：这种情况是新建 HashMap 时传了初始容量，例如：new HashMap<>(32)，使用这种方式新建 HashMap 时，由于 HashMap 没有 capacity 属性，所以此时的 capacity 会被暂存在 threshold 属性。因此此时的 threshold 的值就是我们要新创建的 HashMap 的 capacity，所以将新表的容量设置为 threshold。

4.如果新表的阈值为空，则通过新的容量 * 负载因子获得阈值（这种情况是初始化的时候传了初始容量，跟第2点相同情况，或者初始容量设置的太小导致老表的容量没有超过 16 导致的）。

如果是红黑树节点，则进行红黑树的重 hash 分布



```java

final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 1.老表的容量不为0，即老表不为空
    if (oldCap > 0) {
        // 1.1 判断老表的容量是否超过最大容量值：如果超过则将阈值设置为Integer.MAX_VALUE，并直接返回老表,
        // 此时oldCap * 2比Integer.MAX_VALUE大，因此无法进行重新分布，只是单纯的将阈值扩容到最大
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 1.2 将newCap赋值为oldCap的2倍，如果newCap<最大容量并且oldCap>=16, 则将新阈值设置为原来的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 2.如果老表的容量为0, 老表的阈值大于0, 是因为初始容量被放入阈值，则将新表的容量设置为老表的阈值
    else if (oldThr > 0)
        newCap = oldThr;
    else {
        // 3.老表的容量为0, 老表的阈值为0，这种情况是没有传初始容量的new方法创建的空表，将阈值和容量设置为默认值
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 4.如果新表的阈值为空, 则通过新的容量*负载因子获得阈值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    // 5.将当前阈值设置为刚计算出来的新的阈值，定义新表，容量为刚计算出来的新容量，将table设置为新定义的表。
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 6.如果老表不为空，则需遍历所有节点，将节点赋值给新表
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {  // 将索引值为j的老表头节点赋值给e
                oldTab[j] = null; // 将老表的节点设置为空, 以便垃圾收集器回收空间
                // 7.如果e.next为空, 则代表老表的该位置只有1个节点，计算新表的索引位置, 直接将该节点放在该位置
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 8.如果是红黑树节点，则进行红黑树的重hash分布(跟链表的hash分布基本相同)
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    // 9.如果是普通的链表节点，则进行普通的重hash分布
                    Node<K,V> loHead = null, loTail = null; // 存储索引位置为:“原索引位置”的节点
                    Node<K,V> hiHead = null, hiTail = null; // 存储索引位置为:“原索引位置+oldCap”的节点
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 9.1 如果e的hash值与老表的容量进行与运算为0,则扩容后的索引位置跟老表的索引位置一样
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null) // 如果loTail为空, 代表该节点为第一个节点
                                loHead = e; // 则将loHead赋值为第一个节点
                            else
                                loTail.next = e;    // 否则将节点添加在loTail后面
                            loTail = e; // 并将loTail赋值为新增的节点
                        }
                        // 9.2 如果e的hash值与老表的容量进行与运算为1,则扩容后的索引位置为:老表的索引位置＋oldCap
                        else {
                            if (hiTail == null) // 如果hiTail为空, 代表该节点为第一个节点
                                hiHead = e; // 则将hiHead赋值为第一个节点
                            else
                                hiTail.next = e;    // 否则将节点添加在hiTail后面
                            hiTail = e; // 并将hiTail赋值为新增的节点
                        }
                    } while ((e = next) != null);
                    // 10.如果loTail不为空（说明老表的数据有分布到新表上“原索引位置”的节点），则将最后一个节点
                    // 的next设为空，并将新表上索引位置为“原索引位置”的节点设置为对应的头节点
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 11.如果hiTail不为空（说明老表的数据有分布到新表上“原索引+oldCap位置”的节点），则将最后
                    // 一个节点的next设为空，并将新表上索引位置为“原索引+oldCap”的节点设置为对应的头节点
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    // 12.返回新表
    return newTab;
}
```

# **总结**

1. HashMap 的底层是个 Node 数组（Node<K,V>[] table），在数组的具体索引位置，如果存在多个节点，则可能是以链表或红黑树的形式存在。
2. 增加、删除、查找键值对时，定位到哈希桶数组的位置是很关键的一步，源码中是通过下面3个操作来完成这一步：1）拿到 key 的 hashCode 值；2）将 hashCode 的高位参与运算，重新计算 hash 值；3）将计算出来的 hash 值与 “table.length - 1” 进行 & 运算。
3. HashMap 的默认初始容量（capacity）是 16，capacity 必须为 2 的幂次方；默认负载因子（load factor）是 0.75；实际能存放的节点个数（threshold，即触发扩容的阈值）= capacity * load factor。
4. HashMap 在触发扩容后，阈值会变为原来的 2 倍，并且会对所有节点进行重 hash 分布，重 hash 分布后节点的新分布位置只可能有两个：“原索引位置” 或 “原索引+oldCap位置”。例如 capacity 为16，索引位置 5 的节点扩容后，只可能分布在新表 “索引位置5” 和 “索引位置21（5+16）”。
5. 导致 HashMap 扩容后，同一个索引位置的节点重 hash 最多分布在两个位置的根本原因是：1）table的长度始终为 2 的 n 次方；2）索引位置的计算方法为 “(table.length - 1) & hash”。HashMap 扩容是一个比较耗时的操作，定义 HashMap 时尽量给个接近的初始容量值。
6. HashMap 有 threshold 属性和 loadFactor 属性，但是没有 capacity 属性。初始化时，如果传了初始化容量值，该值是存在 threshold 变量，并且 Node 数组是在第一次 put 时才会进行初始化，初始化时会将此时的 threshold 值作为新表的 capacity 值，然后用 capacity 和 loadFactor 计算新表的真正 threshold 值。
7. 当同一个索引位置的节点在增加后达到 9 个时，并且此时数组的长度大于等于 64，则会触发链表节点（Node）转红黑树节点（TreeNode），转成红黑树节点后，其实链表的结构还存在，通过 next 属性维持。链表节点转红黑树节点的具体方法为源码中的 treeifyBin 方法。而如果数组长度小于64，则不会触发链表转红黑树，而是会进行扩容。
8. 当同一个索引位置的节点在移除后达到 6 个时，并且该索引位置的节点为红黑树节点，会触发红黑树节点转链表节点。红黑树节点转链表节点的具体方法为源码中的 untreeify 方法。
9. HashMap 在 JDK 1.8 之后不再有死循环的问题，JDK 1.8 之前存在死循环的根本原因是在扩容后同一索引位置的节点顺序会反掉。
10. HashMap 是非线程安全的，在并发场景下使用 ConcurrentHashMap 来代替

