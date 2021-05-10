## ArrayList

> 快速概述

- 底层基于数组，**“动态”增长**
- 有序
- 线程不安全
- 允许 null 的存在
- 默认初始容量大小为 10
- 使用无参构造器构造时，内部数组为空数组，首次添加增长为容量为10的数组。如果构造时指定了容量，初始化为对应容量
- 容量不足时扩容至1.5倍
- 删除元素时不会减少容量，**若希望减少容量则调用trimToSize()**
- 添加和删除时，时间复杂度为O(n)
- 查询快，增删慢

### 源码

```java
------------------------------------------------------域
//初始化默认容量。
private static final int DEFAULT_CAPACITY = 10;
//指定该ArrayList容量为0时，返回该空数组。
private static final Object[] EMPTY_ELEMENTDATA = {};
/**
 * 当调用无参构造方法，返回的是该数组。刚创建一个ArrayList 时，其内数据量为0。
 * 它与EMPTY_ELEMENTDATA的区别就是：该数组是默认返回的，而后者是在用户指定容量为0时返回。
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
/**
 * 保存添加到ArrayList中的元素。
 * ArrayList的容量就是该数组的长度。
 * 该值为DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时，当第一次添加元素进入ArrayList中时，数组将扩容值DEFAULT_CAPACITY。
 * 被标记为transient，在对象被序列化的时候不会被序列化。
 */
transient Object[] elementData; 
//ArrayList的实际大小（数组包含的元素个数）。
private int size;
------------------------------------------------------构造方法
 /**
 * 构造一个指定初始化容量为capacity的空ArrayList。
 *
 * @param  initialCapacity  ArrayList的指定初始化容量
 * @throws IllegalArgumentException  如果ArrayList的指定初始化容量为负。
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}

/**
 * 构造一个初始容量为 10 的空列表。
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。
 *
 * @param c 其元素将放置在此列表中的 collection
 * @throws NullPointerException 如果指定的 collection 为 null
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
------------------------------------------------------核心方法
/**
 * 返回list中索引为index的元素
 *
 * @param  index 需要返回的元素的索引
 * @return list中索引为index的元素
 * @throws IndexOutOfBoundsException 如果索引超出size
 */
public E get(int index) {
    //越界检查
    rangeCheck(index);
    //返回索引为index的元素
    return elementData(index);
}

/**
 * 越界检查。
 * 检查给出的索引index是否越界。
 * 如果越界，抛出运行时异常。
 * 这个方法并不检查index是否合法。比如是否为负数。
 * 如果给出的索引index>=size，抛出一个越界异常
 */
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
 * 返回索引为index的元素
 */
@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}

-------------------------
1、进行空间检查，决定是否进行扩容，以及确定最少需要的容量
2、如果确定扩容，就执行grow(int minCapacity)，minCapacity为最少需要的容量
3、第一次扩容，逻辑为newCapacity = oldCapacity + (oldCapacity >> 1);即在原有的容量基础上增加一半。
4、第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。
5、对扩容后的容量进行判断，如果大于允许的最大容量MAX_ARRAY_SIZE，则将容量再次调整为MAX_ARRAY_SIZE。至此扩容操作结束。

/**
 * 添加元素到list末尾.
 *
 * @param e 被添加的元素
 * @return true
 */
public boolean add(E e) {
    //确认list容量，如果不够，容量加1。注意：只加1，保证资源不被浪费
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}


/**
* 增加ArrayList容量。
* 
* @param   minCapacity   想要的最小容量
*/
public void ensureCapacity(int minCapacity) {
    // 如果elementData等于DEFAULTCAPACITY_EMPTY_ELEMENTDATA，最小扩容量为DEFAULT_CAPACITY，否则为0
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)? 0: DEFAULT_CAPACITY;
    //如果想要的最小容量大于最小扩容量，则使用想要的最小容量。
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
/**
* 数组容量检查，不够时则进行扩容，只供类内部使用。
* 
* @param minCapacity    想要的最小容量
*/
private void ensureCapacityInternal(int minCapacity) {
    // 若elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA，则取minCapacity为DEFAULT_CAPACITY和参数minCapacity之间的最大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
/**
* 数组容量检查，不够时则进行扩容，只供类内部使用
* 
* @param minCapacity 想要的最小容量
*/
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 确保指定的最小容量 > 数组缓冲区当前的长度  
    if (minCapacity - elementData.length > 0)
        //扩容
        grow(minCapacity);
}

/**
 * 分派给arrays的最大容量
 * 为什么要减去8呢？
 * 因为某些VM会在数组中保留一些头字，尝试分配这个最大存储容量，可能会导致array容量大于VM的limit，最终导致OutOfMemoryError。
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
* 扩容，保证ArrayList至少能存储minCapacity个元素
* 第一次扩容，逻辑为newCapacity = oldCapacity + (oldCapacity >> 1);即在原有的容量基础上增加一半。第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。
* 
* @param minCapacity 想要的最小容量
*/
private void grow(int minCapacity) {
    // 获取当前数组的容量
    int oldCapacity = elementData.length;
    // 扩容。新的容量=当前容量+当前容量/2.即将当前容量增加一半。
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //如果扩容后的容量还是小于想要的最小容量
    if (newCapacity - minCapacity < 0)
        //将扩容后的容量再次扩容为想要的最小容量
        newCapacity = minCapacity;
    //如果扩容后的容量大于临界值，则进行大容量分配
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData,newCapacity);
}
/**
* 进行大容量分配
*/
private static int hugeCapacity(int minCapacity) {
    //如果minCapacity<0，抛出异常
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    //如果想要的容量大于MAX_ARRAY_SIZE，则分配Integer.MAX_VALUE，否则分配MAX_ARRAY_SIZE
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}

------------------------------------------
/**
 * 在制定位置插入元素。当前位置的元素和index之后的元素向后移一位
 *
 * @param index 即将插入元素的位置
 * @param element 即将插入的元素
 * @throws IndexOutOfBoundsException 如果索引超出size
 */
public void add(int index, E element) {
    //越界检查
    rangeCheckForAdd(index);
    //确认list容量，如果不够，容量加1。注意：只加1，保证资源不被浪费
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 对数组进行复制处理，目的就是空出index的位置插入element，并将index后的元素位移一个位置
    System.arraycopy(elementData, index, elementData, index + 1,size - index);
    //将指定的index位置赋值为element
    elementData[index] = element;
    //实际容量+1
    size++;
}

private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}


--------------------------------------
/**
 * 删除list中位置为指定索引index的元素
 * 索引之后的元素向左移一位
 *
 * @param index 被删除元素的索引
 * @return 被删除的元素
 * @throws IndexOutOfBoundsException 如果参数指定索引index>=size，抛出一个越界异常
 */
public E remove(int index) {
    //检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
    rangeCheck(index);
    //结构性修改次数+1
    modCount++;
    //记录索引为inde处的元素
    E oldValue = elementData(index);

    // 删除指定元素后，需要左移的元素个数
    int numMoved = size - index - 1;
    //如果有需要左移的元素，就移动（移动后，该删除的元素就已经被覆盖了）
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // size减一，然后将索引为size-1处的元素置为null。为了让GC起作用，必须显式的为最后一个位置赋null值
    elementData[--size] = null; // clear to let GC do its work

    //返回被删除的元素
    return oldValue;
}

/**
 * 越界检查。
 * 检查给出的索引index是否越界。
 * 如果越界，抛出运行时异常。
 * 这个方法并不检查index是否合法。比如是否为负数。
 * 如果给出的索引index>=size，抛出一个越界异常
 */
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}


--------------------------------------
/**
 * 替换指定索引的元素
 *
 * @param 被替换元素的索引
 * @param element 即将替换到指定索引的元素
 * @return 返回被替换的元素
 * @throws IndexOutOfBoundsException 如果参数指定索引index>=size，抛出一个越界异常
 */
public E set(int index, E element) {
    //检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
    rangeCheck(index);

    //记录被替换的元素
    E oldValue = elementData(index);
    //替换元素
    elementData[index] = element;
    //返回被替换的元素
    return oldValue;
}
```

## Vector

> 快速概述

- 底层是数组，现在已少用
- Vector所有方法都是同步，**有性能损失**。
- Vector初始length是10 超过length时 以100%比率增长，**相比于ArrayList更多消耗内存**。

## LinkedList

>  快速概述

- LinkedList底层是**双向链表**
- 有序，输出顺序与输入顺序一致
- 增删快，查询慢
- 允许元素为 null
- 可用作栈、队列、双端队列
- 不是线程安全的
- 没有固定容量，不需要扩容；
- 基于双端链表，添加/删除元素只会影响周围的两个节点，开销很低；
- 只能顺序遍历，无法按照索引获得元素，因此查询效率不高；

### 源码

```java
----------------------------------------全局变量
/**
 * LinkedList节点个数
 */
transient int size = 0;

/**
 * 指向头节点的指针
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
transient Node<E> first;

/**
 * 指向尾节点的指针
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
transient Node<E> last;

----------------------------------------构造方法
/**
 * 构造一个空链表.
 */
public LinkedList() {
}

/**
 * 根据指定集合c构造linkedList。先构造一个空linkedlist，在把指定集合c中的所有元素都添加到linkedList中。
 *
 * @param  c 指定集合
 * @throws NullPointerException 如果特定指定集合c为null
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}

----------------------------------------常用方法
/**
 * 在表头添加元素
 */
private void linkFirst(E e) {
    //使节点f指向原来的头结点
    final Node<E> f = first;
    //新建节点newNode，节点的前指针指向null，后指针原来的头节点
    final Node<E> newNode = new Node<>(null, e, f);
    //头指针指向新的头节点newNode 
    first = newNode;
    //如果原来的头结点为null，更新尾指针，否则使原来的头结点f的前置指针指向新的头结点newNode
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}

/**
 * 在表尾插入指定元素e
 */
void linkLast(E e) {
    //使节点l指向原来的尾结点
    final Node<E> l = last;
    //新建节点newNode，节点的前指针指向l，后指针为null
    final Node<E> newNode = new Node<>(l, e, null);
    //尾指针指向新的头节点newNode
    last = newNode;
    //如果原来的尾结点为null，更新头指针，否则使原来的尾结点l的后置指针指向新的头结点newNode
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}

/**
 * 在指定节点succ之前插入指定元素e。指定节点succ不能为null。
 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    //获得指定节点的前驱
    final Node<E> pred = succ.prev;
    //新建节点newNode，前置指针指向pred，后置指针指向succ
    final Node<E> newNode = new Node<>(pred, e, succ);
    //succ的前置指针指向newTouch
    succ.prev = newNode;
    //如果指定节点的前驱为null，将newTouch设为头节点。否则更新pred的后置节点
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}

/**
 * 删除头结点f，并返回头结点的值
 */
private E unlinkFirst( Node<E> f) {
    // assert f == first && f != null;
    // 保存头结点的值
    final E element = f.item;
    // 保存头结点指向的下个节点
    final Node<E> next = f.next;
    //头结点的值置为null
    f.item = null;
    //头结点的后置指针指向null
    f.next = null; // help GC
    //将头结点置为next
    first = next;
    //如果next为null，将尾节点置为null，否则将next的后置指针指向null
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    //返回被删除的头结点的值
    return element;
}

/**
 * 删除尾节点l.并返回尾节点的值
 */
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    // 保存尾节点的值
    final E element = l.item;
    //获取新的尾节点prev
    final Node<E> prev = l.prev;
    //旧尾节点的值置为null
    l.item = null;
    //旧尾节点的后置指针指向null
    l.prev = null; // help GC
    //将新的尾节点置为prev
    last = prev;
    //如果新的尾节点为null，头结点置为null，否则将新的尾节点的后置指针指向null
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    //返回被删除的尾节点的值
    return element;
}

/**
 * 删除指定节点，返回指定元素的值
 */
E unlink(Node<E> x) {
    // assert x != null;
    // 保存指定节点的值
    final E element = x.item;
    // 获取指定节点的下个节点next
    final Node<E> next = x.next;
    // 获取指定节点的下个节点prev
    final Node<E> prev = x.prev;
    //如果prev为null，那么next为新的头结点，否则将prev的后置指针指向next，x的前置指针指向null
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
    //如果next为null，那么prev为新的尾结点，否则将next的前置指针指向prev，x的后置指针指向null
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    //x的值置为null
    x.item = null;
    size--;
    modCount++;
    //返回被删除的节点的值
    return element;
}

/**
 * 返回链表中的头结点的值.
 *
 * @return 返回链表中的头结点的值
 * @throws NoSuchElementException 如果链表为空
 */
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

/**
 * 返回链表中的尾结点的值.
 *
 * @return 返回链表中的头结点的值
 * @throws NoSuchElementException 如果链表为空
 */
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}

/**
 * 删除并返回表头元素.
 *
 * @return 表头元素
 * @throws NoSuchElementException 链表为空
 */
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

/**
 * 删除并返回表尾元素.
 *
 * @return 表尾元素
 * @throws NoSuchElementException 链表为空
 */
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

/**
 * 在表头插入指定元素.
 *
 * @param e 插入的指定元素
 */
public void addFirst(E e) {
    linkFirst(e);
}

/**
 * 在表尾插入指定元素.
 * 
 * 该方法等价于add()
 *
 * @param e 插入的指定元素
 */ 
public void addLast(E e) {
    linkLast(e);
}


/**
 * 判断链表是否包含指定对象o
 * @param o 指定对象
 * @return 是否包含指定对象
 */
public boolean contains(Object o) {
    return indexOf(o) != -1;
}

/**
 * 在表尾插入指定元素.
 *
 * 该方法等价于addLast
 *
 * @param e 插入的指定元素
 * @return true
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}

/**
 * 正向遍历链表，删除出现的第一个值为指定对象的节点
 *
 * @param o 要删除的节点置
 * @return 如果o在链表中存在，返回true
 */
public boolean remove(Object o) {
    //遍历链表，如果o为null，删除第一个值为null的节点，返回true。如果不为null，删除第一个值为o的节点。如果链表中存在o，就返回true。
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

/**
 * 插入指定集合到链尾
 *
 * @param c 指定集合
 * @return 如果链表改变，返回true
 * @throws NullPointerException 如果指定集合为null
 */
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

/**
 * 插入指定集合到链尾的指定位置
 *
 * @param index 指定的插入位置
 * @param c 插入的指定集合
 * @return 如果链表改变，返回true
 * @throws IndexOutOfBoundsException 如果index<0或index>size
 * @throws NullPointerException 如果指定集合为null
 */
public boolean addAll(int index, Collection<? extends E> c) {
    //检查插入的位置是否合法
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;//如果c是空的话那么就返回false

    //定义两个节点指针，指向插入点前后的节点元素
    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    // 插入集合中所有元素
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    // 修改插入后的指针问题
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

/**
 * 删除链表中的所有元素
 */
public void clear() {
    // Clearing all of the links between nodes is "unnecessary", but:
    // - helps a generational GC if the discarded nodes inhabit
    //   more than one generation
    // - is sure to free memory even if there is a reachable Iterator
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}

/**
 * 返回指定索引处的元素
 *
 * @param index 指定索引
 * @return 指定索引处的元素
 * @throws IndexOutOfBoundsException 如果索引index越界
 */
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

/**
 * 替换指定索引处的元素为指定元素element
 *
 * @param index 被替换的元素的索引
 * @param element 
 * @return 指定元素element
 * @throws IndexOutOfBoundsException 索引越界
 */
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}


/**
 * 插入指定元素到指定索引处
 *
 * @param index 指定索引
 * @param element 指定元素
 * @throws IndexOutOfBoundsException 索引越界
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

/**
 * 删除指定索引处的元素
 *
 * @param 指定索引
 * @return 指定索引处的元素
 * @throws IndexOutOfBoundsException 索引越界
 */
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}

/**
 * 返回在指定索引处的非空元素
 */
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

/**
 * 正向遍历链表，返回指定元素第一次出现时的索引。如果元素没有出现，返回-1.
 * 
 * @param o 需要查找的元素
 * @return 指定元素第一次出现时的索引。如果元素没有出现，返回-1。
 */
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}


/**
 * 逆向遍历链表，返回指定元素第一次出现时的索引。如果元素没有出现，返回-1.
 * 
 * @param o 需要查找的元素
 * @return 指定元素第一次出现时的索引。如果元素没有出现，返回-1。
 */
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}

```

