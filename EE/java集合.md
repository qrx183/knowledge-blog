# Java集合框架

## ArrayList

ArrayList继承自AbstractList,实现了List接口![1661930689059](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661930689059.png)

ArrayList底层是数组,是基于数组实现容量的动态变化的

### 成员变量

ArrayList的成员变量有size和elementdata,其中size表示容器中实际的元素个数,而elementdata.length表示容器中最多可以容纳的元素个数

```Java

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     * 容器最多可以容纳的元素个数,当第一个元素添加进来时,elementData.length从0变为DEFAULT_CAPACITY(10)
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     * 实际包含的元素个数
     * @serial
     */
    private int size;
```

默认初始容量为10

```java

    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;
```

```java

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

上述两个空数组的第一个表示长度为0的空数组,第二个表示在添加元素之前的一个空数组.两个空数组的区别在于ArrayList的elementData是从哪个构造函数初始化得来的(无参/有参)

### 构造方法

```java
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

有参构造方法指定容器大小,当指定大小为0时,将elementData指向EMPTY_ELEMENTDATA;无参构造方法将elementData指向DEFAULTCAPACITY_EMPTY_ELEMENTDATA,**无参构造函数只是实现了赋值,真正的扩容是在第一次添加元素时实现的**

### 操作方法

add()

```java
	public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
	private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
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

add方法首先需要判断是否需要扩容:当elementData=DEAFULTCAPACITY_EMPTY_ELMENETDATA时,返回DEAULT_CAPACITY和minCapactiy的最大值,也就是10.否则返回minCapacity

如果当前所需容器容量minCapacity大于容器最大容量,则进行扩容(grow)

每次扩容时将容器容量扩充为max(minCapacity,1.5*elementData.length)

remove()

```java
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
public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

remove操作分为两种,一种是以下标为命中点进行删除,一种是以元素值为命中点进行删除.然后大体思路都是找到删除元素的下标,然后利用System.arrayCopyOf调整删除后的元素位置.arrayCopyOf是Hot VM内部实现的,通过手写汇编等方式来优化处理,相较于普通的for循环复制数组效率更高

ArrayList是线程不安全的,代替集合有Vector,synchronizedList

## LinkedList

LinkedList继承了AbstractSequentialList,实现了List接口.LinkedList的底层是链表.

### 成员变量

size,first和last.其中size表示容器中实际的元素个数,first指向容器的首元素节点,last指向容器的尾元素节点

```java

```

其中Node是节点类型,由代码可知LinkedList是一个典型的双向链表

### 构造方法

```java
public LinkedList() {
}
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

其中无参构造方法就是一个空方法.有参构造方法首先调用无参构造方法,然后调用addAll:检查数组是否越界,将数组中的元素构造成新的节点插入到链表中

### 操作方法

add:将元素插入到链表末尾

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

add:将元素插入指定位置

```java
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
//找到指定位置的节点
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

remove:移除链表中某个元素

```java
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

remove:移除链表中某个位置的元素

```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

## HashMap(jdk1.8)

HashMap底层是由**数组+链表+红黑树**实现![1662117628239](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662117628239.png)

### HashMap基本属性

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 默认容量

static final int MAXIMUM_CAPACITY = 1 << 30; // 最大容量

static final float DEFAULT_LOAD_FACTOR = 0.75f; // 负载因子

static final int TREEIFY_THRESHOLD = 8; // 链表转为红黑树的节点个数的阈值

static final int UNTREEIFY_THRESHOLD = 6; // 红黑树转为链表的节点个数的阈值

static final int MIN_TREEIFY_CAPACITY = 64; // 转为红黑树的table的最小长度

static class Node<K,V> implements Map.Entry<K,V> {} // HashMap的节点

static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {// 红黑树节点
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // 红黑树中也要维护链表结构
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
    // ...

```

### HashMap常用方法

1. hash:用来确认哈希桶数组中节点的位置

   ```java
   static final int hash(Object key) {
       int h;
       return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   }
   
   int n = tab.length;
   
   int index= (n-1) & hash
   ```

   Hash值是拿key的hashCode与key的hashCode的高16位异或.  使得key的高位和低位都可以得到充分利用,这种方式叫做**扰动函数**

   **之所以这么做,是因为得到的hash值之后还要通过与table.length-1进行与运算,而当table的长度没有那么大时,有效用到的位数都是hash中的低位,因此如果直接用hashCode去进行与运算发生碰撞的概率非常大,因此将高16位和低16位异或,既能减少冲突的发生概率,也能变相的保存高16位的信息**

   而哈系统中节点的位置则是将hash和(table长度-1)进行与.这种算法是对模运算的一种改善.传统的hash算法应该是拿hash值对数组长度取模.但为了提升效率,**利用位运算来代替了模运算:hash%(2^n) == hash & (2^n-1).而table的长度是2的k次方**

2. get:从哈希表找找到某个具体的值

   ```java
   public V get(Object key) {
       Node<K,V> e;
       return (e = getNode(hash(key), key)) == null ? null : e.value;
   }
   final Node<K,V> getNode(int hash, Object key) {
       Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
       if ((tab = table) != null && (n = tab.length) > 0 &&
           (first = tab[(n - 1) & hash]) != null) {
           if (first.hash == hash && // always check first node
               ((k = first.key) == key || (key != null && key.equals(k))))
               return first;
           if ((e = first.next) != null) {
               if (first instanceof TreeNode)
                   return ((TreeNode<K,V>)first).getTreeNode(hash, key);
               do {
                   if (e.hash == hash &&
                       ((k = e.key) == key || (key != null && key.equals(k))))
                       return e;
               } while ((e = e.next) != null);
           }
       }
       return null;
   }
   
   final TreeNode<K,V> getTreeNode(int h, Object k) {
       return ((parent != null) ? root() : this).find(h, k, null);
   }
   
   final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
       TreeNode<K,V> p = this;
       do {
           int ph, dir; K pk;
           TreeNode<K,V> pl = p.left, pr = p.right, q;
           if ((ph = p.hash) > h)
               p = pl;
           else if (ph < h)
               p = pr;
           else if ((pk = p.key) == k || (k != null && k.equals(pk)))
               return p;
           else if (pl == null)
               p = pr;
           else if (pr == null)
               p = pl;
           else if ((kc != null ||
                     (kc = comparableClassFor(k)) != null) &&
                    (dir = compareComparables(kc, k, pk)) != 0)
               p = (dir < 0) ? pl : pr;
           else if ((q = pr.find(h, k, kc)) != null)
               return q;
           else
               p = pl;
       } while (p != null);
       return null;
   }
   ```

   首先判断table是否为空或table的长度为0或key对应的table中的下标是否为空

   然后检查key对应的数组下标中第一个是否为要找的key:地址相同或值相同,如果是则直接返回first

   然后判断对应哈希桶是链表还是红黑树,如果是红黑树调用getTreeNode,否则继续向下遍历链表

   最后如果差找不到直接返回null

   > getTreeNode
   >
   > 首先判断当前节点的hash和目标节点的hash的大小,如果前者大,去左子树找,如果后者大,去右子树找
   >
   > 如果hash值相同,判断当前节点的key值是否相同,如果相同直接返回该节点,否则继续判断
   >
   > 如果左树为null,去右子树找,如果右树为null,去左子树找
   >
   > 如果key实现了Comparable接口,就根据Compareable的比较规则进行比较然后去左树或右树找
   >
   > 否则指定去右树找,找不到就去左子树找
   >
   > 如果最后还是找不到就返回null

3. put:将一个键值对加入哈希表中

   ```java
   public V put(K key, V value) {
      return putVal(hash(key), key, value, false, true);
   }
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
   ```

   首先判断table是否为空,如果为空则调用resize方法进行扩容

   利用hash计算待插入位置是否有节点,如果没有则直接创建一个新的节点

   否则先判断第一个节点的key值和hash值是否和待插入节点一致,如果一致则直接将新的值赋给该节点

   如果不一样判断节点组是红黑树还是链表,如果是红黑树就进入putTreeVal方法,否则就先在链表中查找是否存在"相同"的节点,如果存在直接跳出循环,否则查到链表的最后,同时判断链表个数是否到达了8,如果到达则进行树化treeifyBin

   判断e是否为空,如果不为空说明链表中存在已有节点和待插入节点相同,将新的值赋给该节点并返回原节点的value值

   链表大小+1,然后判断是是否需要扩容

   > putTreeVal
   >
   > ```java
   > 
   > ```
   >
   >

   > treeifyBin
   >
   > ```java
   > 
   > ```
   >
   > 如果table的长度小于64,则进行扩容
   >
   > 然后找到对应桶内的链表,分别对链表中的每个节点链表节点到红黑树节点的转化:replacementTreeNode
   >
   > 并维护链表结构(双向)
   >
   > 然后返回该链表的头结点,然后调用treeify方法来构建红黑树
   >
   > > ```java
   > > final void treeify(Node<K,V>[] tab) {
   > >     TreeNode<K,V> root = null;
   > >     for (TreeNode<K,V> x = this, next; x != null; x = next) {
   > >         next = (TreeNode<K,V>)x.next;
   > >         x.left = x.right = null;
   > >         if (root == null) {
   > >             x.parent = null;
   > >             x.red = false;
   > >             root = x;
   > >         }
   > >         else {
   > >             K k = x.key;
   > >             int h = x.hash;
   > >             Class<?> kc = null;
   > >             for (TreeNode<K,V> p = root;;) {
   > >                 int dir, ph;
   > >                 K pk = p.key;
   > >                 if ((ph = p.hash) > h)
   > >                     dir = -1;
   > >                 else if (ph < h)
   > >                     dir = 1;
   > >                 else if ((kc == null &&
   > >                           (kc = comparableClassFor(k)) == null) ||
   > >                          (dir = compareComparables(kc, k, pk)) == 0)
   > >                     dir = tieBreakOrder(k, pk);
   > > 
   > >                 TreeNode<K,V> xp = p;
   > >                 if ((p = (dir <= 0) ? p.left : p.right) == null) {
   > >                     x.parent = xp;
   > >                     if (dir <= 0)
   > >                         xp.left = x;
   > >                     else
   > >                         xp.right = x;
   > >                     root = balanceInsertion(root, x);
   > >                     break;
   > >                 }
   > >             }
   > >         }
   > >     }
   > >     moveRootToFront(tab, root);
   > > }
   > > ```
   > >
   > >

4. resize:扩容

   ```java
   
   ```

   首先判断旧表为空,如果不为空则判断旧表容量是否大于最大容量,如果大于,将阈值设置为Integer.MAX_VALUE.如果旧表容量的2倍小于最大容量且旧表容量大于等于16,则将新表阈值设置为旧表阈值的2倍

   如果旧表容量为0但阈值不为0,则将新表容量设置为旧表的阈值

   如果旧表容量为0且阈值也为0,则新表容量为16,阈值为0.75*16

   如果新表的阈值为空,则通过新表的容量*负载因子得到新表的阈值

   然后定义新表的结构

   如果旧表不为空,则将旧表中的节点存储到新表中.在存储过程中,首先判断链表是否只有一个节点,如果只有一个节点则直接存储.否则判断节点是否为红黑树节点,如果是则调用split方法赋值

   如果为普通的链表节点,则判断节点的hash与旧表容量与运算是否为0,若为0这说明节点在新表的位置与旧表一致,如果不为0,说明节点在新表的位置为旧表位置+旧表容量.将节点直接插入到新表的对应位置的链表的最后.

   然后设置新表对应位置的第一个节点的指向.然后返回新表

   > split方法
   >
   > ```java
   > final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
   >     TreeNode<K,V> b = this;
   >     // Relink into lo and hi lists, preserving order
   >     TreeNode<K,V> loHead = null, loTail = null;
   >     TreeNode<K,V> hiHead = null, hiTail = null;
   >     int lc = 0, hc = 0;
   >     for (TreeNode<K,V> e = b, next; e != null; e = next) {
   >         next = (TreeNode<K,V>)e.next;
   >         e.next = null;
   >         if ((e.hash & bit) == 0) {
   >             if ((e.prev = loTail) == null)
   >                 loHead = e;
   >             else
   >                 loTail.next = e;
   >             loTail = e;
   >             ++lc;
   >         }
   >         else {
   >             if ((e.prev = hiTail) == null)
   >                 hiHead = e;
   >             else
   >                 hiTail.next = e;
   >             hiTail = e;
   >             ++hc;
   >         }
   >     }
   > 
   >     if (loHead != null) {
   >         if (lc <= UNTREEIFY_THRESHOLD)
   >             tab[index] = loHead.untreeify(map);
   >         else {
   >             tab[index] = loHead;
   >             if (hiHead != null) // (else is already treeified)
   >                 loHead.treeify(tab);
   >         }
   >     }
   >     if (hiHead != null) {
   >         if (hc <= UNTREEIFY_THRESHOLD)
   >             tab[index + bit] = hiHead.untreeify(map);
   >         else {
   >             tab[index + bit] = hiHead;
   >             if (loHead != null)
   >                 hiHead.treeify(tab);
   >         }
   >     }
   > }
   > ```
   >
   > 大致思路与链表中的节点的赋值相同
   >
   > 只是在赋值完后需要判断老表索引位置与老表容量+老表索引位置的节点个数是否需要链表化或树化

   为什么扩容后老表中某个位置的节点只可能出现在新表中老表位置或老表位置+老表容量这两个位置

   >扩容后的新表的大小是旧表的一倍(如果旧表的大小不为0)
   >
   >![1662126222880](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662126222880.png)

5. remove:清除掉某个键值对

   ```java
   
   ```

   首先查找到待删除的节点:如果是红黑树就走getTreeNode,否则在链表中遍历查找

   找到待删除节点后,根据节点类型进行红黑树式删除或普通链表型的删除

   > ```java
   > final void removeTreeNode(HashMap<K,V> map, Node<K,V>[] tab,
   >                           boolean movable) {
   >     int n;
   >     if (tab == null || (n = tab.length) == 0)
   >         return;
   >     int index = (n - 1) & hash;
   >     TreeNode<K,V> first = (TreeNode<K,V>)tab[index], root = first, rl;
   >     TreeNode<K,V> succ = (TreeNode<K,V>)next, pred = prev;
   >     if (pred == null)
   >         tab[index] = first = succ;
   >     else
   >         pred.next = succ;
   >     if (succ != null)
   >         succ.prev = pred;
   >     if (first == null)
   >         return;
   >     if (root.parent != null)
   >         root = root.root();
   >     if (root == null
   >         || (movable
   >             && (root.right == null
   >                 || (rl = root.left) == null
   >                 || rl.left == null))) {
   >         tab[index] = first.untreeify(map);  // too small
   >         return;
   >     }
   >     TreeNode<K,V> p = this, pl = left, pr = right, replacement;
   >     if (pl != null && pr != null) {
   >         TreeNode<K,V> s = pr, sl;
   >         while ((sl = s.left) != null) // find successor
   >             s = sl;
   >         boolean c = s.red; s.red = p.red; p.red = c; // swap colors
   >         TreeNode<K,V> sr = s.right;
   >         TreeNode<K,V> pp = p.parent;
   >         if (s == pr) { // p was s's direct parent
   >             p.parent = s;
   >             s.right = p;
   >         }
   >         else {
   >             TreeNode<K,V> sp = s.parent;
   >             if ((p.parent = sp) != null) {
   >                 if (s == sp.left)
   >                     sp.left = p;
   >                 else
   >                     sp.right = p;
   >             }
   >             if ((s.right = pr) != null)
   >                 pr.parent = s;
   >         }
   >         p.left = null;
   >         if ((p.right = sr) != null)
   >             sr.parent = p;
   >         if ((s.left = pl) != null)
   >             pl.parent = s;
   >         if ((s.parent = pp) == null)
   >             root = s;
   >         else if (p == pp.left)
   >             pp.left = s;
   >         else
   >             pp.right = s;
   >         if (sr != null)
   >             replacement = sr;
   >         else
   >             replacement = p;
   >     }
   >     else if (pl != null)
   >         replacement = pl;
   >     else if (pr != null)
   >         replacement = pr;
   >     else
   >         replacement = p;
   >     if (replacement != p) {
   >         TreeNode<K,V> pp = replacement.parent = p.parent;
   >         if (pp == null)
   >             root = replacement;
   >         else if (p == pp.left)
   >             pp.left = replacement;
   >         else
   >             pp.right = replacement;
   >         p.left = p.right = p.parent = null;
   >     }
   > 
   >     TreeNode<K,V> r = p.red ? root : balanceDeletion(root, replacement);
   > 
   >     if (replacement == p) {  // detach
   >         TreeNode<K,V> pp = p.parent;
   >         p.parent = null;
   >         if (pp != null) {
   >             if (p == pp.left)
   >                 pp.left = null;
   >             else if (p == pp.right)
   >                 pp.right = null;
   >         }
   >     }
   >     if (movable)
   >         moveRootToFront(tab, r);
   > }
   > ```

### 死循环问题

在jdk1.7之前,在进行扩容将旧表中的元素复制到新表中时会发生元素位置对调的行为,这种行为在多线程下会发生死循环:Race Condition![1662127547921](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662127547921.png)

而在jdk1.8中扩容不会发生元素位置的对调,所以不会产生死循环问题,但HashMap仍然是一个线程不安全的容器

### HashMap和HashTable的区别

1. HashMap允许key和value
2. HashMap的默认容量为16,HashTable的默认容量为11
3. HashMap的扩容大小为x2,HashTable的扩容大小为x2+1
4. HashMap是线程不安全的,HashTable是线程安全的
5. HashMap的hash是hashCode再次经过计算得到的,HashTable的hash直接使用hashCode
6. HashMap继承AbstractHashMap,HashTable继承Dictionary

## ConcurrentHashMap(jdk1.8)

ConcurrentHashMap是一个**线程安全**的类.利用**CAS算法**保证了线程安全.ConcurrentHashMap的底层是数组+链表+红黑树

### 主要属性

```java
private transient volatile int sizeCtl;
// sizeCtl是一个控制符,sizeCtl取-1时表示正在进行初始化,-N表示有N-1个线程正在扩容,非负数代表哈希表还没有被初始化,正数值相当于是扩容阈值

transient volatile Node<K,V>[] table;
// 哈希数组

static final int MOVED     = -1; // hash值是-1，表示这是一个链表节点
static final int TREEBIN   = -2; // hash值是-2  表示这时一个红黑树节点

```

### 主要内部类

```java
 static class Node<K,V> implements Map.Entry<K,V> {
     final int hash;
     final K key;
     volatile V val;
     volatile Node<K,V> next;

     Node(int hash, K key, V val, Node<K,V> next) {
         this.hash = hash;
         this.key = key;
         this.val = val;
         this.next = next;
     }
     public final K getKey()       { return key; }
     public final V getValue()     { return val; }
     public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
     public final String toString(){ return key + "=" + val; }
     public final V setValue(V value) {
         throw new UnsupportedOperationException();
     }
     
     Node<K,V> find(int h, Object k) {
         Node<K,V> e = this;
         if (k != null) {
             do {
                 K ek;
                 if (e.hash == h &&
                     ((ek = e.key) == k || (ek != null && k.equals(ek))))
                     return e;
             } while ((e = e.next) != null);
         }
         return null;
     }
 }
```

Node节点是哈希表存储的基本节点,与HashMap相比对val和next属性添加了volatile同步锁,同时不允许直接点用setValue方法去修改节点的值.添加了find方法来辅助map.get()方法

```java
 static final class TreeNode<K,V> extends Node<K,V> {}
```

TreeNode节点是哈希表存储的红黑树节点,与HashMap相比,TreeNode并不是直接转换成红黑树节点,而是作为节点存储到TreeBin对象中,由TreeBin实现红黑树的转化.而且ConcurrentHashMap中的TreeNode继承自Node,拥有Node的next指针,**便于TreeBin的访问**

```java
 static final class TreeBin<K,V> extends Node<K,V> {
     TreeNode<K,V> root;
     volatile TreeNode<K,V> first;
     volatile Thread waiter;
     volatile int lockState;
     // values for lockState
     static final int WRITER = 1; // set while holding write lock
     static final int WAITER = 2; // set when waiting for write lock
     static final int READER = 4; // increment value for setting read lock
 }
```

TreeBin中存储了许多TreeNode节点,代替TreeNode成为了哈希表中的红黑树.TreeBin类中自带**读写锁**

```java
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }
}
```

ForwardingNode节点用于连接两个table.该节点的key,value,next均为null,hash为-1

### Unsafe和CAS

```java
 private static final sun.misc.Unsafe U;
```

在ConcurrentHashMap中维护了一个Unsafe的U属性,通过调用U.compareAndSwap方法来实现CAS操作,CAS是一种无锁化的修改值的操作,CAS操作是原子性的.**CAS的思想就是不断比较内存中的变量值和你指定的变量值是否相等,如果相等则接收你指定的修改的值,否则拒绝你的操作.因为当内存中的值和你指定的值不同时,当前的值很大可能已经不是最新的值,而你的修改操作会覆盖到其他线程修改的结果.**

```java
// 获取指定i位置上的节点
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

// 利用CAS来为i位置上的节点赋值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
// 利用volatile方法设置i位置上结点的值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}

```

ConcurrentHashMap中定义了**3个原子性的操作用于对指定位置的节点进行赋值或者获取指定位置的节点**,正是这3个原子性的操作保证了ConcurrentHashMap的线程安全

### 初始化方法initTable

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2); //sc = 0.75*n
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

ConcurrentHashMap中的构造方法仅仅是设置一些参数,真正的初始化是在想ConcurrentHashMap中插入元素的时候发生的.检查table==null

initTable方法主要取决于sizeCtl,当sizeCtl<0,说明有其他线程正在初始化,于是放弃本线程的初始化操作,ConcurrentHashMap的初始化只能由1个线程完成,否则运用CAS将sizeCtl的值设为-1,之后将sizeCtl的设置为0.75*n(扩容阈值)

### 扩容方法transfer

ConcurrentHashMap的扩容思路和HashMap类似,但因为支持**并发扩容**,而且没有加锁.这样可以通过并发去减少扩容的时间复杂度.

扩容操作分为两个部分

1. 构建一个nextTable,容量是原来的2倍,在单线程下执行
2. 将原来table中的元素复制到nextTable中,复制操作允许并发执行



扩容流程

要完成的任务就是遍历table中的所有节点,将table中的所有节点复制到nextTable中

1. 遍历table中的节点,如果该节点为空,则设置forwardNode节点
2. 如果该节点为Node节点,就将该节点下的链表放到新table的i位置,并将该链表的反序链表放到新table的(i+n)位置
3. 如果该节点为TreeBin节点,就将该节点下的红黑树放到新table的i位置,并对该树作反序处理放到新table的(i+n)位置
4. 遍历完所有节点后,令nextTable为新的table,并更新sizeCtl为新容量的0. 75倍



上述流程是在单线程下完成的扩容.而在多线程中,当某个线程在遍历到forward节点时就跳过遍历其他节点,当遍历其他类型的节点时就对节点进行加锁,在完成复制后将该节点设置为forward节点![1662300211713](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662300211713.png)

### put()方法

ConcurrentHashMap中的put方法的思路和HashMap中类似,但在多线程的环境下,会有特殊的处理

首先**ConcurrentHashMap中的key和value不允许为null值**,在这个基础上

1. 当存在一个或多个线程在进行扩容的操作时,当前线程在进行插入操作前,需要协助进行扩容(**即插入节点的位置为forward节点时就帮助扩容**)
2. 如果检测到要插入的节点**非空且非forward节点**,就对该节点加锁,保证线程安全.之后就跟HashMap的put方法思路一致

### get()方法

get()方法和HashMap的get()方法思路一致

### size()方法

因为是ConcurrentHashMap是支持多线程的,所以在一个线程中调用size()方法统计table的数量时不能被确定的,因此size()得到的table的容量是一个**估计值**

size()方法是基于CAS实现的.



总的来说,ConcurrentHashMap在HashMap的基础上保证了线程安全.其中线程安全是基于CAS,volatile(读不加锁,给变量加volatile保证读到的值源自内存)以及减小锁的粒度(从锁整个哈希表减小到锁每个哈希桶).同时在扩容的时候也不是一次性就全部扩容完毕,而是利用多线程使多个线程同时辅助扩容,提高扩容的效率.

## 关于HashMap的面试题

1. HashMap和HashTable的区别

   1. HashTable是线程安全的,支持并发操作,HashMap是线程不安全的

   2. HashTable的key和value不允许为null,HashMap允许key和value为null

      HashTable之所以不允许key和value为null,原因是因为在多线程的环境下无法判断map.get(key)的值为null的原因是因为key不存在还是key对应的value为null,而HashMap中可以在单线程中通过containsKey方法检测对应的key是否存在.而containsKey方法在多线程是不安全的,索引HashTable没有改方法,因此也就避免key和value为null造成的困扰.

      HashMap中针对key为null时key的hashCode值返回0,所以HashMap是允许key为null的

   3. HashTable的迭代器是Enumeration,HashMap的迭代器是Iterator

   4. HashTable的默认数组大小是11,扩容的方式是原来的2倍+1,HashMap的默认数组大小是16,扩容的方式是原来的2倍

   5. HashTable中key的hash即为key的hashCode,而HashMap中key的hash是用key的hashCode的低16位和高16位异或

2. HashMap与ConcurrentHashMap的区别(感觉没啥意义)

   1. HashMap是线程不安全的,ConcurrentHashMap是线程安全的
   2. ConcurrentHashMap在HashMap的基础上为了保证线程安全使用了更复杂的机制,比如CAS,锁粒度更小的锁,volatile等

3. ConcurrentHashMap和HashTable的区别

   1. HashTable是对整个数组加锁,而ConcurrentHashMap的锁粒度更细,是对哈希表中每个哈希桶加锁

4. HashMap与HashSet的区别

   1. HashMap实现了Map接口,HashSet实现了Set接口
   2. HashMap中存储的是键值对,而HashSet中只存储key

## Arrays.sort原理

1. 首先判断待排序数组的元素个数是否小于**QUICKSORT_THRESHOLD**(快排-286),如果小于则判断元素个数是否小于**INSERTION_SORT_THRESHOLD**(插入-47),如果小于则使用**改进的插入排序**,否则使用改进的**双轴快排**.当元素个数大于286时,判断数组的无序程度:无序程度通过将数组划分为多个有序序列的个数来判定,如果有序序列的个数大于**MAX_RUN_COUNT**(67),则认为原数组是基本无序的,所以仍然使用**改进的双轴快排**,否则使用**归并排序**进行排序

## 哈希表

1. 定义:哈希表又称为散列表，可以实现数据增删查改的时间复杂度为O(1)。哈希方法中使用的转换函数称为哈希函数。

2. 哈希表中数组的大小:哈希表中数组的大小最好是一个形如4K+3的素数,因为实践表名一个元素的hash值对这样的素数取模后的余数分布更加均匀,发生冲突的概率更低.但存在一个散列表的负载因子 α = 填入表中的个数/散列表的长度。α是散列表装满程度的标志因子,α越大发生冲突的概率越大,理想的α的大小为0.7-0.8之间,因此当数组中填入一定元素后要进行扩容

3. 哈希函数的选择

   > 哈希函数设置标准：
   >
   > 1. 哈希函数应该尽可能的简单
   >
   > 2. 哈希函数计算出来的哈希地址应该尽可能地均匀分布到整个哈希表中
   >
   >
   > 常见哈希函数：
   >
   > 1. 直接定制法：Hash(key) = A*key + B。优点:简单、均匀。缺点：需要事先知道数据的分布情况。适用场景：适合查找比较小且连续的情况。
   > 2. 除留余数法：设散列表中允许的地址数为m，取一个不大于m但最接近m的**质数**(其实影响不大)作为除数：Hash(key) = key % p(p<=m)
   >
   > 不常见哈希函数：
   >
   > 1. 平方取中法：对关键字key去平方然后取平方后的中间位数的数字作为哈希地址，适合不知道关键字的分布情况但是位数不是很大的情况。
   > 2. 折叠法：将关键字key尽可能分成长度相同的部分，然后对这些部分求和，按照散列表的长度取求和后的后几位作为哈希地址。适合不知道关键字的分布情况但是位数很多的情况。
   > 3. 随机数法：Hash(key) = random(key)。适用于关键字长度不等时
   > 4. 数学分析法：对关键字key进行分析，取key中出现概率比较均匀的几位数作为哈希地址，如身份证号的统计可以统计其后四位。

4. 
   冲突解决：

- 闭散列
  - 线性探测：(H0 + i)%m把冲突的元素向后放到没有冲突的第一个位置。线性探测会导致冲突的元素扎堆，不利于数据的删除。
  - 二次探测：(H0+i^2)%m。缺点是装载因子最好不要超过0.5，超过要考虑扩容，因此二次探测的空间利用率比较低。
- 开散列(哈希桶):将发生冲突的元素组成一个列表(HashMap中就是通过这种方式,HashMap中每个哈希桶由链表或红黑树构成)

## 快速失败(fail-fast)和安全失败(fail-safe)

**快速失败(fail-fast)**

在用迭代器对集合中的对象遍历时,当对集合中的元素进行更新操作时,会抛出Concurrent Modification Exception.

原理:迭代器在遍历集合时,是**直接访问集合中的对象的**,在遍历过程中会维护一个modCount变量,每次在调用.hasNext()方法后,会首先检查modCount变量的值和expectModCount的值是否相等,如果相等则执行这次遍历,否则抛出异常.

modCount变量用来保证遍历的过程中值和预期的值相等,但不能保证对象的值没有发生改变,因为有可能发生把值a改成值b再改成值a的操作(aba),因此**无法通过modCount变量来检测是否发生过并发操作**

Java.util包下的集合都是fail-fast的,即用迭代器遍历的过程中不允许对对象的值进行修改





**安全失败(fail-safe)**

采用了安全失败策略的集合容器,在遍历集合的对象时,不是直接访问对象本身,而是**先复制集合的内容,在拷贝的集合上进行遍历**

原理:由于遍历的是拷贝的集合,因此在遍历的过程中,即使其他线程对集合中的元素发生了改变,也不会影响遍历的过程,但相应的,**采用了安全失败策略的集合在用迭代器进行遍历的过程中访问到的是开始遍历那一刻拿到的拷贝到的值,而无法遍历到最新的值**

Java.Concurrent包下的集合都是fail-safe的,可以在多线程下并发使用



**快速失败和安全失败都是基于迭代器而言的**





