# List

有序列表，内部数据结构为数组，数组大小支持自动扩展

## 子类

### ArrayList

非线程安全；数组采用 1/2 倍扩展；实现 **Fail-Fast 机制**

#### 扩容

默认情况下，

初始容量为 0

首次执行插入时进行扩容，默认为容量为 10

当 size + 1: （10， Intager.MAX_VALUE - 8），进行 0.5 倍扩容（取整）

当 size + 1: （Intager.MAX_VALUE - 8, Intager.MAX_VALUE），容量固定为：Intager.MAX_VALUE

当 size + 1 < 0：数组长度溢出，抛出oom；

```java
	public boolean add(E e) {
        // 插入之前先判断数组容量是否满足
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

	private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
	
	private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // elementData 列表内部存储数据的数组
        // DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为默认构造创建的空数组
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // DEFAULT_CAPACITY：默认容量10
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

	private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        // minCapacity 等于 DEFAULT_CAPACITY(10) 或者 size + 1
        // 当 minCapacity 大于当前数组长度，则进行扩容；
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

	private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 0.5倍（取整）扩容；
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
        // 部分虚拟机对数组保留了头字，直接分配最大数组可能会导致oom
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

	private static int hugeCapacity(int minCapacity) {
        // 小于0, 说明Integer溢出了，抛出oom
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```



### Vector

线程安全，通过使用 **Synchronzied** 对容器加锁，效率低，已经极少使用；

### Stack

Vector 子类；支持使用 push / pop 进行入栈出栈操作（后进先出）

Java6之后，建议使用 **Deque** 

### LinkedList

链表，内部数据结构为链表结构，容量为：Integer.MAX_VALUE，非线程安全；实现 **Fail-Fast 机制**

### CopyOnWriteArrayList

线程安全，通过 **ReentrantLock** 对写相关方法加锁；支持读写分离；

写入操作在复制的数组副本上进行，执行完成之后，将数组指针指向前面复制出来的副本；

读操作，比如Get，支持并发读取，无需加锁；

遍历操作同样是对数组的副本和快照进行处理，支持遍历时的修改；

缺点：

1. 修改和遍历操作，都需要复制当前list，需要额外的空间和时间开销；

2. 不能保证遍历的是最新列表；

### SynchronizedList

Collections 工具类的内部类，支持将 **List** 包装为线程安全的实现；通过 **Synchronized** 实现线程安全；

```java
 	/**
     * @serial include
     */
    static class SynchronizedCollection<E> implements Collection<E>, Serializable {
        private static final long serialVersionUID = 3053995032091335093L;

        final Collection<E> c;  // Backing Collection
        final Object mutex;     // Object on which to synchronize

        SynchronizedCollection(Collection<E> c) {
            this.c = Objects.requireNonNull(c);
            mutex = this;
        }

        SynchronizedCollection(Collection<E> c, Object mutex) {
            this.c = Objects.requireNonNull(c);
            this.mutex = Objects.requireNonNull(mutex);
        }

        public int size() {
            synchronized (mutex) {return c.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return c.isEmpty();}
        }
        public boolean contains(Object o) {
            synchronized (mutex) {return c.contains(o);}
        }
        public Object[] toArray() {
            synchronized (mutex) {return c.toArray();}
        }
        public <T> T[] toArray(T[] a) {
            synchronized (mutex) {return c.toArray(a);}
        }

        public Iterator<E> iterator() {
            return c.iterator(); // Must be manually synched by user!
        }

        public boolean add(E e) {
            synchronized (mutex) {return c.add(e);}
        }
        public boolean remove(Object o) {
            synchronized (mutex) {return c.remove(o);}
        }

        public boolean containsAll(Collection<?> coll) {
            synchronized (mutex) {return c.containsAll(coll);}
        }
        public boolean addAll(Collection<? extends E> coll) {
            synchronized (mutex) {return c.addAll(coll);}
        }
        public boolean removeAll(Collection<?> coll) {
            synchronized (mutex) {return c.removeAll(coll);}
        }
        public boolean retainAll(Collection<?> coll) {
            synchronized (mutex) {return c.retainAll(coll);}
        }
```

### SynchronizedRandomAccessList

**SynchronizedList** 的子类，同时实现了 **RandomAccess** 接口；

## Fail-Fast机制

Fail-Fast 字面意思就是 快速失败， 当我们在使用迭代器遍历集合的时候，如果集合发生变更，就会停止遍历并抛出异常 `ConcurrentModificationException`, 这就是fail-fast机制；

ArrayList的注释中有如下描述：

```java
*
* <p><a name="fail-fast">
* The iterators returned by this class's {@link #iterator() iterator} and
* {@link #listIterator(int) listIterator} methods are <em>fail-fast</em>:</a>
* if the list is structurally modified at any time after the iterator is
* created, in any way except through the iterator's own
* {@link ListIterator#remove() remove} or
* {@link ListIterator#add(Object) add} methods, the iterator will throw a
* {@link ConcurrentModificationException}.  Thus, in the face of
* concurrent modification, the iterator fails quickly and cleanly, rather
* than risking arbitrary, non-deterministic behavior at an undetermined
* time in the future.
*
* <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
* as it is, generally speaking, impossible to make any hard guarantees in the
* presence of unsynchronized concurrent modification.  Fail-fast iterators
* throw {@code ConcurrentModificationException} on a best-effort basis.
* Therefore, it would be wrong to write a program that depended on this
* exception for its correctness:  <i>the fail-fast behavior of iterators
* should be used only to detect bugs.</i>
    
```

意思就是，通过 iterator() 和 listIterator(int) 方法创建的迭代器是实现了fail-fast机制，当迭代器创建之后，除非是通过迭代器提供的 add/remove 方法，其他任何时候对list进行修改，都会抛出 ConcurrentModificationException。

ArrayList中的具体实现

modCount:  ArrayList的成员变量，记录列表的修改次数，每次执行add/remove等操作的时候都会加1；

迭代器在初始化的时候会记录当前的modCount，迭代过程中每次获取next，都会检测当前modCount与期望值是否一致，不一致就会立即结束遍历，抛出异常；

```java
    public Iterator<E> iterator() {     return new Itr();    }

    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
		
        ...

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```



## RandomAccess接口

JDK源码注释如下：

```java
/**
 * Marker interface used by <tt>List</tt> implementations to indicate that
 * they support fast (generally constant time) random access.  The primary
 * purpose of this interface is to allow generic algorithms to alter their
 * behavior to provide good performance when applied to either random or
 * sequential access lists.
 *
 * <p>The best algorithms for manipulating random access lists (such as
 * <tt>ArrayList</tt>) can produce quadratic behavior when applied to
 * sequential access lists (such as <tt>LinkedList</tt>).  Generic list
 * algorithms are encouraged to check whether the given list is an
 * <tt>instanceof</tt> this interface before applying an algorithm that would
 * provide poor performance if it were applied to a sequential access list,
 * and to alter their behavior if necessary to guarantee acceptable
 * performance.
 *
 * <p>It is recognized that the distinction between random and sequential
 * access is often fuzzy.  For example, some <tt>List</tt> implementations
 * provide asymptotically linear access times if they get huge, but constant
 * access times in practice.  Such a <tt>List</tt> implementation
 * should generally implement this interface.  As a rule of thumb, a
 * <tt>List</tt> implementation should implement this interface if,
 * for typical instances of the class, this loop:
 * <pre>
 *     for (int i=0, n=list.size(); i &lt; n; i++)
 *         list.get(i);
 * </pre>
 * runs faster than this loop:
 * <pre>
 *     for (Iterator i=list.iterator(); i.hasNext(); )
 *         i.next();
 * </pre>
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.4
 */
public interface RandomAccess {
}
```

标记接口，用于标记List是否支持快速随机访问；在遍历方式上，**fori** 比 **iterator迭代器** 具有更快的访问速度；