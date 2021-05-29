---
title: Java杂记——容器
author: Kzero Coder
date: 2021-05-28 11:00:00 +0800
categories: [Blogging, JavaNotes]
tags: [writing]
math: true
---

<head>
    <script type="text/x-mathjax-config"> 
   		MathJax.Hub.Config({ TeX: { equationNumbers: { autoNumber: "all" } } }); 
   	</script>
    <script type="text/x-mathjax-config">
    	MathJax.Hub.Config({tex2jax: {
             inlineMath: [ ['$','$'], ["\\(","\\)"] ],
             processEscapes: true
           }
         });
    </script>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" 	type="text/javascript">
	</script>
</head>



# 容器

![collection](/assets/img/javaNotes/collection.png)

绿色为接口，黄色为抽象类，蓝色为类，灰线和黑线没有区别，只是用来防止交错的时候产生误导，线上i表示implements，e表示extends



## Collection

> The root interface in the *collection hierarchy*. A collection represents a group of objects, known as its *elements*. **Some collections allow duplicate elements and others do not. Some are ordered and others unordered.** The JDK does not provide any *direct* implementations of this interface: it provides implementations of more specific subinterfaces like `Set` and `List`. **This interface is typically used to pass collections around and manipulate them where maximum generality is desired.**
>
> ***Bags* or *multisets*** (unordered collections that may contain duplicate elements) should implement this interface directly.
>
> All general-purpose `Collection` implementation classes (which typically implement `Collection` indirectly through one of its subinterfaces) should provide **two "standard" constructors**: a void (no arguments) constructor, which creates an empty collection, and a constructor with a single argument of type `Collection`, which creates a new collection with the same elements as its argument. In effect, the latter constructor allows the user to copy any collection, producing an equivalent collection of the desired implementation type. There is no way to enforce this convention (as interfaces cannot contain constructors) but all of the general-purpose `Collection` implementations in the Java platform libraries comply.
>
> **Certain methods are specified to be *optional*.** If a collection implementation doesn't implement a particular operation, it should define the corresponding method to throw `UnsupportedOperationException`. Such methods are marked "optional operation" in method specifications of the collections interfaces.
>
> ***# such as：remove, addAll, removeAll, retainAll, clear***
>
> Some collection implementations have restrictions on the elements that they may contain. For example, **some implementations prohibit null elements, and some have restrictions on the types of their elements**. Attempting to add an ineligible element throws an unchecked exception, typically `NullPointerException` or `ClassCastException`. Attempting to query the presence of an ineligible element may throw an exception, or it may simply return false; some implementations will exhibit the former behavior and some will exhibit the latter. More generally, attempting an operation on an ineligible element whose completion would not result in the insertion of an ineligible element into the collection may throw an exception or it may succeed, at the option of the implementation. Such exceptions are marked as "optional" in the specification for this interface.
>
> It is up to each collection to determine its own synchronization policy. In the absence of a stronger guarantee by the implementation, undefined behavior may result from the invocation of any method on a collection that is being mutated by another thread; this includes direct invocations, passing the collection to a method that might perform invocations, and using an existing iterator to examine the collection.
>
> Many methods in Collections Framework interfaces are defined in terms of the `equals` method. For example, the specification for the `contains(Object o)` method says: "returns `true` if and only if this collection contains at least one element `e` such that `(o==null ? e==null : o.equals(e))`." This specification should *not* be construed to imply that invoking `Collection.contains` with a non-null argument `o` will cause `o.equals(e)` to be invoked for any element `e`. Implementations are free to implement optimizations whereby the `equals` invocation is avoided, for example, by first comparing the hash codes of the two elements. The `Object.hashCode()` specification guarantees that two objects with unequal hash codes cannot be equal.) More generally, implementations of the various Collections Framework interfaces are free to take advantage of the specified behavior of underlying `Object` methods wherever the implementor deems it appropriate.
>
> Some collection operations which perform recursive traversal of the collection may fail with an exception for self-referential instances where the collection directly or indirectly contains itself. This includes the `clone()`, `equals()`, `hashCode()` and `toString()` methods. Implementations may optionally handle the self-referential scenario, however most current implementations do not do so.

一般不会使用Collection类，用其子接口Set和List较多，Collection只能作为传递collections以及在高度generalize情况下操纵它们。



## List

**特点：**

- 有序，可以通过整数index访问某个元素
- 允许重复元素
- 从0开始计数
- 提供特殊的Iterator——ListIterator

**静态方法：**

```java
// of是用来初始化List的，其中返回的是Immutable对象，进行修改会报UnsupportedOperationException
static <E> List<E> of(E... elements) {...}

// copyOf返回的也是Immutable对象
static <E> List<E> copyOf(Collection<? extends E> coll) {...}
```



### ArrayList

**特点：**

- 允许所有元素，包括null
- 可变长
- 非线程安全
- 1.5倍扩容

```java
transient Object[] elementData; // non-private to simplify nested class access
```

可以看到ArrayList是通过数组来实现的。

*注：此处虽然elementData被transient标注，但是在writeObject的时候会将数组内的元素一一取出序列化，所以实际上没有序列化的只是数组本身*

由于是通过数组实现，所以对ArrayList变更操作往往通过Array.copyOf实现，花费较大，但是带来的好处是随机访问代价低。



### LinkedList

**特点：**

- 允许所有元素，包括null
- 可变长
- 双向
- 非线程安全
- 不需要扩容

```java
    transient int size = 0;

    /**
     * Pointer to first node.
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     */
    transient Node<E> last;

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

	 /**
     * Adapter to provide descending iterators via ListItr.previous
     */
    private class DescendingIterator implements Iterator<E> {
        private final ListItr itr = new ListItr(size());
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
    }
```

可以看到LinkedList通过Node类来存储数据和连接，是双向的，且新定义了DescendingIterator，使得LinkedList能够逆向遍历。

由于LinkedList通过链表方式连接，查询操作需要花费链表长时间，但是适合修改操作，同时由于实现了Deque接口，能够当作双向队列使用。



### Vector

Vector和ArrayList相差不大，唯一区别是在扩容的时候，加入了capacityIncrement进行等步长扩容（当不设置capacityIncrement的时候，采用倍增，即扩容为2倍）

```java
	/**
     * The array buffer into which the components of the vector are
     * stored. The capacity of the vector is the length of this array buffer,
     * and is at least large enough to contain all the vector's elements.
     *
     * <p>Any array elements following the last element in the Vector are null.
     *
     * @serial
     */
    protected Object[] elementData;

    /**
     * The number of valid components in this {@code Vector} object.
     * Components {@code elementData[0]} through
     * {@code elementData[elementCount-1]} are the actual items.
     *
     * @serial
     */
    protected int elementCount;

    /**
     * The amount by which the capacity of the vector is automatically
     * incremented when its size becomes greater than its capacity.  If
     * the capacity increment is less than or equal to zero, the capacity
     * of the vector is doubled each time it needs to grow.
     *
     * @serial
     */
    protected int capacityIncrement;
```



## Set

**特点：**

- 无重复元素

提供的静态方法和List类似，都是返回Immutable对象，不允许修改

*注：实现Set方法时，如果内部元素可变，需要注意，防止出现可重复。Set一般通过equals方法判断两元素是否相等，所以需要严格实现equals和hashcode方法*

如：

```java
	public static void main(String[] args) {
        Set<temp> temps = new HashSet<>();
        temp t1 = new temp(1,1);
        temp t2 = new temp(1,2);
        temps.add(t1);
        temps.add(t2);
        for(temp t : temps){
            System.out.println(t.toString());
        }
        System.out.println();
        t1.setI2(2);
        for(temp t : temps){
            System.out.println(t.toString());
        }
    }

    static class temp{
        int i1;
        int i2;
        
        ...
    }
```

输出结果为：

```
temp{i1=1, i2=1}
temp{i1=1, i2=2}

temp{i1=1, i2=2}
temp{i1=1, i2=2}
```

所以对于mutable对象还是不要在加入set后进行修改



### HashSet

```java
    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
```

HashSet是基于HashMap实现的，实际上只是使用了HashMap的键值，初始默认长度为16。

具体功能将在HashMap中介绍。



### TreeSet

```java
	/**
     * The backing map.
     */
    private transient NavigableMap<E,Object> m;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
```

和HashSet类似，只不过采用了树形结构，使用TreeMap实现。

TreeSet可以用于排序，而不是像HashSet一样无序。



## Map

Map不是Collection的子接口或者实现类，其存储的是键值的映射关系，其中键具有唯一性

内部定义Entry接口，用于提取单一键值属性。



### HashMap

```java
	/**
     * Basic hash bin node, used for most entries.  (See below for
     * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
     */
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }

	/**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;
```

HashMap是通过数组+链表实现的，其中table存储hash值相等的头节点，Node类能够向后查询到所有hash值相等的结点。

其中table一开始并不建立，而是在putVal后才建立。

```java
	/**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
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
        return newTab;
    }
```

通过resize可以得知数组table的扩充是以倍增的，所以一直是2的次幂。

同时在扩容的时候，还需要重新计算index=e.hash & (newCap - 1)。由于newCap保持2的次幂，在减一后保持低位全1，在扩容后，明显知道newCap右移了一位，此时会使得newCap - 1的低位1数量加一，使得可用哈希位加一，得到新的index，完成链表的分裂。

在HashMap的doc中提到了：

>  This map usually acts as a binned (bucketed) hash table, but when bins get too large, they are transformed into bins of TreeNodes, each structured similarly to those in java.util.TreeMap.

这个too large在HashMap中以属性标识出来

```java
	/**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

也即当结点数量大于64且链表结点数大于8的时候，采用红黑树结构，当树结点数降为6，则回到链表形式。

通过treeifyBin和untreeify实现转换（实际上还有treeify，但是是通过treeifyBin调用的）

```java
	/**
     * Replaces all linked nodes in bin at index for given hash unless
     * table is too small, in which case resizes instead.
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
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
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
	static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        ...
            
        /**
         * Forms tree of the nodes linked from this node.
         */
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (root == null) {
                    x.parent = null;
                    x.red = false;
                    root = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);

                        TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            moveRootToFront(tab, root);
        }
            
		/**
         * Returns a list of non-TreeNodes replacing those linked from
         * this node.
         */
        final Node<K,V> untreeify(HashMap<K,V> map) {
            Node<K,V> hd = null, tl = null;
            for (Node<K,V> q = this; q != null; q = q.next) {
                Node<K,V> p = map.replacementNode(q, null);
                if (tl == null)
                    hd = p;
                else
                    tl.next = p;
                tl = p;
            }
            return hd;
        }
        ...
    }
```

接下来看put过程：

```java
/**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with {@code key}, or
     *         {@code null} if there was no mapping for {@code key}.
     *         (A {@code null} return can also indicate that the map
     *         previously associated {@code null} with {@code key}.)
     */
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
	
	/**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

	/**
     * Computes key.hashCode() and spreads (XORs) higher bits of hash
     * to lower.  Because the table uses power-of-two masking, sets of
     * hashes that vary only in bits above the current mask will
     * always collide. (Among known examples are sets of Float keys
     * holding consecutive whole numbers in small tables.)  So we
     * apply a transform that spreads the impact of higher bits
     * downward. There is a tradeoff between speed, utility, and
     * quality of bit-spreading. Because many common sets of hashes
     * are already reasonably distributed (so don't benefit from
     * spreading), and because we use trees to handle large sets of
     * collisions in bins, we just XOR some shifted bits in the
     * cheapest possible way to reduce systematic lossage, as well as
     * to incorporate impact of the highest bits that would otherwise
     * never be used in index calculations because of table bounds.
     */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

所以key的存储位置获取过程为：$key\rightarrow^{hashcode()}h\rightarrow^{hash()}h\bigoplus h>>16\rightarrow h \& (newCap - 1)$

*注：$\bigoplus$表示异或*



### TreeMap

TreeMap是基于红黑树实现的。

```java
	private transient Entry<K,V> root;

	...
    
    static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;

        /**
         * Make a new cell with given key, value, and parent, and with
         * {@code null} child links, and BLACK color.
         */
        Entry(K key, V value, Entry<K,V> parent) {
            this.key = key;
            this.value = value;
            this.parent = parent;
        }

        /**
         * Returns the key.
         *
         * @return the key
         */
        public K getKey() {
            return key;
        }

        /**
         * Returns the value associated with the key.
         *
         * @return the value associated with the key
         */
        public V getValue() {
            return value;
        }

        /**
         * Replaces the value currently associated with the key with the given
         * value.
         *
         * @return the value associated with the key before this method was
         *         called
         */
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;

            return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
        }

        public int hashCode() {
            int keyHash = (key==null ? 0 : key.hashCode());
            int valueHash = (value==null ? 0 : value.hashCode());
            return keyHash ^ valueHash;
        }

        public String toString() {
            return key + "=" + value;
        }
    }
```

由于在插入之后会维护红黑树结构，所以TreeMap的遍历顺序并不符合插入顺序，是无序的。

红黑树本质上是一种二叉平衡查找树，但是给每个结点标记了颜色，并给出了以下要求：

- 树中每个节点必须是有颜色的，要么红色，要么黑色
- 树中的根节点必须是黑色的
- 树中的叶节点必须是黑色的，也就是为NIL节点
- 树中任意一个节点如果是红色的，那么它的两个子节点一点是黑色的
- 任意节点到叶节点（树最下面一个节点）的每一条路径所包含的黑色节点数目一定相同

其中TreeMap的comparator是可以自定义的，所以可以传入自定义的Comparator类，用于TreeMap的排序。



## Qeque

Qeque为队列，提供了FIFO的基本方法。



### PriorityQueue

PriorityQueue 优先队列 顾名思义，就是通过某个属性对进入队列内的元素进行排序得到的队列。

```java
	private static final int DEFAULT_INITIAL_CAPACITY = 11;

    /**
     * Priority queue represented as a balanced binary heap: the two
     * children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  The
     * priority queue is ordered by comparator, or by the elements'
     * natural ordering, if comparator is null: For each node n in the
     * heap and each descendant d of n, n <= d.  The element with the
     * lowest value is in queue[0], assuming the queue is nonempty.
     */
    transient Object[] queue; // non-private to simplify nested class access

    /**
     * The number of elements in the priority queue.
     */
    int size;

    /**
     * The comparator, or null if priority queue uses elements'
     * natural ordering.
     */
    private final Comparator<? super E> comparator;
```

PriorityQueue采用的是堆结构来维持优先性，默认初始大小为11，且可自定义comparator。

```java
	/**
     * Increases the capacity of the array.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50%
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity < 64 ? oldCapacity + 2 : oldCapacity >> 1
                                           /* preferred growth */);
        queue = Arrays.copyOf(queue, newCapacity);
    }
```

在数组大小小于64时，采用倍增（实际为倍增后+2）；当数组大小不小于64的时候，采用1.5扩容



### ArrayDeque

ArrayDeque采用数组实现双端队列，默认初始化大小为16（+1是为了保持elements[tail] = null）

```java
	/**
     * The array in which the elements of the deque are stored.
     * All array cells not holding deque elements are always null.
     * The array always has at least one null slot (at tail).
     */
    transient Object[] elements;

    /**
     * The index of the element at the head of the deque (which is the
     * element that would be removed by remove() or pop()); or an
     * arbitrary number 0 <= head < elements.length equal to tail if
     * the deque is empty.
     */
    transient int head;

    /**
     * The index at which the next element would be added to the tail
     * of the deque (via addLast(E), add(E), or push(E));
     * elements[tail] is always null.
     */
    transient int tail;

	/**
     * Constructs an empty array deque with an initial capacity
     * sufficient to hold 16 elements.
     */
    public ArrayDeque() {
        elements = new Object[16 + 1];
    }
```

通过查看dec和inc操作可以得知，其实ArrayDeque维护的实际上是一个头尾相接的循环数组

```java
	/**
     * Circularly decrements i, mod modulus.
     * Precondition and postcondition: 0 <= i < modulus.
     */
    static final int dec(int i, int modulus) {
        if (--i < 0) i = modulus - 1;
        return i;
    }

    /**
     * Circularly adds the given distance to index i, mod modulus.
     * Precondition: 0 <= i < modulus, 0 <= distance <= modulus.
     * @return index 0 <= i < modulus
     */
    static final int inc(int i, int distance, int modulus) {
        if ((i += distance) - modulus >= 0) i -= modulus;
        return i;
    }
```

当空间不足扩容的时候，ArrayDeque和Priority的扩容方式类似（小倍增，大1.5扩容）

```java
	/**
     * Increases the capacity of this deque by at least the given amount.
     *
     * @param needed the required minimum extra capacity; must be positive
     */
    private void grow(int needed) {
        // overflow-conscious code
        final int oldCapacity = elements.length;
        int newCapacity;
        // Double capacity if small; else grow by 50%
        int jump = (oldCapacity < 64) ? (oldCapacity + 2) : (oldCapacity >> 1);
        if (jump < needed
            || (newCapacity = (oldCapacity + jump)) - MAX_ARRAY_SIZE > 0)
            newCapacity = newCapacity(needed, jump);
        final Object[] es = elements = Arrays.copyOf(elements, newCapacity);
        // Exceptionally, here tail == head needs to be disambiguated
        if (tail < head || (tail == head && es[head] != null)) {
            // wrap around; slide first leg forward to end of array
            int newSpace = newCapacity - oldCapacity;
            System.arraycopy(es, head,
                             es, head + newSpace,
                             oldCapacity - head);
            for (int i = head, to = (head += newSpace); i < to; i++)
                es[i] = null;
        }
    }
```



## ListIterator

ListIterator在Iterator给的hasNext和next基础上，还给出了：

- hasPrevious，previous进行逆序遍历
- nextIndex，previousIndex进行Index获取
- remove在遍历时删除元素（在Iterator中，remove会直接抛出错误）
- set在遍历时插入元素



