## Java集合

### 一. 简介

Java集合分为Set、List、Queue和Map，其中Set无序、不可重复；List有序、可重复；Queue指队列；Map为映射。集合就像一种容器，保存对象实际上只是保存对象的引用变量。

Java集合和数组的区别：① 数组长度固定，在初始化时指定；集合可以保存数量不确定的数据，还可以保存映射

​                                          ② 数组元素可以是基本数据类型或对象；集合只能保存对象，基本类型要转换成包装类

Java集合类之间的继承关系：Set、List、Queue由Collection派生；Map可以派生出HashMap、TreeMap等



### 二. Collection接口

接口中定义的方法：`add()`、`addAll()`、`clear()`、`contaions()`、`equals()`、`remove()`等增删改查

遍历集合元素使用迭代器Iterator（Collection接口的父接口），常用方法有`hasNext()`和`next()`

Set：与Collection基本相同，只是不包含重复元素

List：有序、可重复，可以根据顺序索引操作集合元素

Queue：队列，先进先出，不允许随机访问元素



### 三. Map集合

接口中定义的方法：`get()`、`put()`、`containsKey()`、`isEmpty()`、`remove()`等增删改查

Map中还包括一个内部类Entry，该类封装了一个key-value对，包含方法有`getKey()`、`getValue()`、`setValue()`

与Set集合的关系：把Map的所有key放在一起就组成了一个Set集合（key无序且不可重复），Map有`keySet()`

与List集合的关系：把Map的所有Value放在一起就组成了一个List集合（Value可重复可根据索引查找）



### 四. ArrayList

ArrayList的重点是自动扩容，可以理解为“动态数组”，主要方法为`add()`、`set()`、`get()`和`remove()`。

`add`函数，阅读源码知其实现最核心的内容是`ensureCapacityInternal`，这个函数也是自动扩容机制的核心，源码如下

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1); // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity); 
    }
    ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code 
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 扩展为原来的1.5倍 
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果扩为1.5倍还不满足需求，直接扩为需求值 
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity); 
}
```

由上可知，增加数据时，若ArrayList大小不满足需求，那么就将数组变为原长度的1.5倍，之后将旧数组拷贝到新数组中；若扩为1.5倍还不满足需求则会直接扩为需求值。

`set`和`get`函数的实现，先做index检查，然后执行赋值或访问操作，源码如下

```java
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue; 
}
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}
```

`remove`函数，先做index检查再操作，前面元素下标不变，后面元素前移，容量减1，源码如下

```java
public E remove(int index) {
	rangeCheck(index);
	modCount++;
	E oldValue = elementData(index);
	int numMoved = size - index - 1;
    if (numMoved > 0)
        // 把后面的往前移 
        System.arraycopy(elementData, index+1, elementData, inde x, numMoved);
    //把最后的置null
    elementData[--size] = null; // clear to let GC do its work 
    return oldValue; 
}
```



### 五. LinkedList

LinkedList基于（双向）链表实现，主要方法为`set()`和`get()`。

`set`和`get`函数都调用了`node`函数，该函数会以O(n/2)的性能去获取一个结点，源码如下

```java
Node<E> node(int index) {
	// assert isElementIndex(index);
	if (index < (size >> 1)) {
    	Node<E> x = first;
        for (int i = 0; i < index; i++) 
            x = x.next; return x; 
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    } 
}

public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal; 
}
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

`node`的实现就是判断index在前半区间还是后半区间，决定从head还是tail开始搜索，而不是一直从头到尾搜索。



### 六. HashMap

HashMap基于Map接口实现，允许空键/值，非同步，不保证有序（比如插入的顺序），不保证序不随时间变化。

HashMap有两个重要参数：容量(Capacity)和负载因子(Load factor)，Capacity就是bucket的大小，Load factor就是bucket填满程度的最大比例；对迭代性能要求高的话，不要把capacity设置过大，也不要把load factor设置过小；当bucket中的entries的数目大于capacity*load factor时需要调整bucket的大小为当前的2倍。

`put`函数的实现思路为：① 对key的hashCode()做hash，然后再计算index；

​                                          ② 如果没碰撞直接放到bucket里；

​                                          ③ 如果碰撞以链表的形式存在buckets后；

​                                          ④ 如果碰撞导致链表过长(≥TREEIFY_THRESHOLD)，就把链表转为红黑树；

​                                          ⑤ 如果结点已存在就替换old value(保证key的唯一性)；

​                                          ⑥ 如果bucket满了(超过load factor*current capacity)，就要resize

`get`函数的实现思路为：① 如果是bucket里的第一个结点，直接命中；

​                                          ② 如果有冲突，则通过key.quals(k)去查找对应的entry

​                                               若为树，则在树中通过key.quals(k)查找，O(logn)；

​                                               若为链表，则在链表中通过key.quals(k)查找，O(n)

`hash`函数的实现思路：对hashCode进行计算，高16bit不变，低16bit和高16bit异或`hash=h^(h>>>16)`（考虑到下标计算的实现为`(n-1)&hash`）

`resize`函数的实现思路：把bucket扩充为2倍，之后重新计算index，把结点再放到新的bucket中；不需要重新计算hash，只需看原来的hash值新增的那个bit是1还是0，是0的话索引没变，是1的话索引变成“原索引+oldCap”

经典面试题如下

**1.什么时候会使用HashMap？它有什么特点？**

是基于Map接口的实现，存储键值对时，可以接受null的键值，是非同步的，HashMap存储着Entry(hash, key, value, next)对象。

**2.HashMap的工作原理？**

通过hash的方法，通过put和get存储和获取对象。

存储对象时，将K/V传给put方法，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量（超过Load Factor则resize为原来的2倍）。

获取对象时，将K传给get，它条用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。

如果发生碰撞，HashMap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制（默认是8），则使用红黑树来替换链表，从而提高速度。

**3.put和get的原理？equals()和hashCode()都有什么作用？**

通过对key的hashCode()进行hashing，并计算下标（`(n-1)&hash`），从而获得buckets的位置。如果产生碰撞，则利用key.equals()方法去链表或树中查找对应的结点。

**4.hash的实现？为什么要这样实现？**

在Java 1.8的实现中，是通过hashCode()的高16位异或低16位实现的（`(h=k.hashCode())^(h>>>16)`），主要是从速度、功效、质量来考虑的，这么做可以再bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销。

**5.如果HashMap的大小超过了负载因子(load factor)定义的容量怎么办？**

如果超过了负载因子（默认0.75），则会重新resize一个雨来长度两倍的HashMap，并且重新调用hash方法。



### 七. LinkedHashMap

LinkedHashMap是Hash表和链表的实现，并且依靠着双向链表保证了迭代顺序是插入的顺序。

LinkedHashMap继承与于HashMap，重新实现了三个重要函数：`afterNodeAccess`、`afterNodeInsertion`、`afterNodeRemoval`，作用分别是在结点访问后、插入后、移除后进行操作。`afterNodeAccess`函数将当前访问到的结点移动至内部的双向链表的尾部；`afterNodeInsertion`函数，如果用户定义了`removeEldestEntry`的规则便可执行相应的移除操作；`afterNodeRemoval`函数将结点从双向链表中删除。这些函数基本上都是为了保证双向链表中的结点次序或者双向链表容量所做的一些额外的事情，目的就是保持双向链表中结点的顺序要从eldest到youngest。

`put`函数在LinkedHashMap中未重新实现，只是实现了`afterNodeAccess`和`afterNodeInsertion`两个回调函数；`get`函数则重新实现并加入了`afterNodeAccess`来保证访问顺序。值得注意的是，在accessOrder模式（会把访问过的元素放在链表后面）下，只要执行`get`或`put`等操作，就会产生`structural modification`。



### 八. TreeMap

TreeMap通过红黑树实现，可以保持key的大小顺序。

`put`函数的实现思路：如果key存在，old value被替换；如果不存在，新增一个结点，然后做红黑树的平衡操作

`get`函数的实现思路：二分查找，以O(logn)的复杂度进行

TreeMap通过`successor`方法保证其迭代输出有序，`successor`函数返回当前结点的后继结点，可以理解成树的中序遍历(LDR)，当前结点为空结点，没有后继；有右子树的结点，后继是右子树的”最左结点“；无右子树的结点，后继是该结点所在左子树的第一个祖先结点。