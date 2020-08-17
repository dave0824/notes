## 前言
本文为对List集合的再一次整理，从父集接口Collection到顶级接口Iterable再到线程不安全实现类：ArrayList、LinkedList,再到线程安全实现类：Vector(被弃用)、CopyOnWriteArrayList。
## List
List集合扩展了Collection接口，它是一个允许重复的集合，即允许有多个元素引用相同的对象。
我们来看看List接口的源码：
```java
public interface List<E> extends Collection<E> {

void add(int index,Object ele)// 根据下标添加元素
Boolean addAll(int index,Collection eles) // 在下标index插入另一个集合的全部元素
Object get(index) // 获取对应下标的元素
int indexOf(Object ele) // ele在集合中首次出现的位置  如果不存在，返回-1
int lastIndexOf(Obj ele) // ele在集合中最后出现的位置  如果不存在，返回-1
Object remove(int index) 移除指定位置索引的元素
Object set(int index,Object ele); // 在下标在下标index插入一个元素

}
```
可以看到List接口继承了Collection接口，那么接下来看看Collection接口
## Collection
在java类库中，Collection接口是集合类的基本接口，这个接口有两个基本的方法：
```java
public interface Collection<E> extends Iterable<E>
{
    boolean add(E element);
    Iterator<E> iterator();
    ...
}
```
add方法用于向集合中添加元素。如果添加元素确实改变了集合就返回true，如果集合没有发生改变就返回false。(其实后面具体的实现类都会对该方法进行重写)列如，向一个集合中添加一个已经存在的对象，这个添加请求就没有效果会返回false，因为Set集合不允许重复对象。从代码中可以看到， Collection接口还包括了一个iterator()方法，返回类型为Iterator接口对象,那么什么Iterator接口?

Collection接口中的其它方法：
```java
//添加方法：
add(Object o) //添加指定元素
addAll(Collection c) //添加指定集合
//删除方法：
remove(Object o) //删除指定元素
removeAll(Collection c) //输出两个集合的交集
retainAll(Collection c) //保留两个集合的交集
clear() //清空集合
//查询方法：
size() //集合中的有效元素个数
toArray() //将集合中的元素转换成Object类型数组
//判断方法：
isEmpty() //判断是否为空
equals(Object o) //判断是否与指定元素相同
contains(Object o) //判断是否包含指定元素
containsAll(Collection c) //判断是否包含指定集合
```
我们还知道有一个Collections，它是一个包装类。它包含有各种有关集合操作的静态多态方法。此类不能实例化，就像一个工具类，服务于Java的Collection框架。
## Iterable
翻看Iterable源码：
```java
public interface Iterable<T> {
    /**
     * Returns an iterator over elements of type {@code T}.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();
}
```
可以看到在Iterable接口中定义了一个iterator方法返回类型为Iterator，继续跟进这个Iterator:
```java
public interface Iterator<E> {
   
    boolean hasNext();
    E next();
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```
很明显的可以看到，这个Iterator接口就像当于链表中的结点，只不过在C语言里结点里的指针在java中变成了对象的引用。那么 我们就应该知道，通过反复的调用nest()就可以逐个的访问集合中的每个元素，但是当到达了集合的末尾，nest方法将抛出一个NoSuchElementException。因此，每次都用next方法前都应该调用hasNext方法进行判断。hasNext方法的作用是判断对象是否还有下一个元素，有就返回true，否则返回false。remove方法会删除上次调用next方法时返回的元素。就像是删除一个元素之前先看下它是很有必要的，remove方法就是按照这个理念设计的。举一个访问集合中所有元素的案例：
```java
 Collection<String> s = new ArrayList<String>();
        s.add("xiaohong");
        s.add("xionming");
        s.add("wanger");

        Iterator<String> iterator = s.iterator();
        while (iterator.hasNext()) {

            String element = iterator.next();
            System.out.println(element);
```
在Java SE8版本中，新加入了for each循环遍历，编译器简单地将“for each”循环翻译为带有迭代器的循环。
```java
for (String string : s)
 {
    System.out.println(string);
		
 }
```
显然，通过"for each"遍历使得代码更加简洁，所有实现了Iterable接口的对象都可以使用"for each"循环。Collection接口扩展了Iterable接口。
好了，到这里我们就看完了List的父级接口了，接下来我们看看它的实现类。
## AbstractList
如果实现了Collection接口的每一个类都要实现它的所有方法，那么将是一件很烦的事情。此时，AbstractList应运而生。它将基础的iterator抽象化，其它的方法给实现了，此时一个具体的集合类就可以扩展AbstractList类，并且只需提供Iterator方法，当然如果不满意AbstractList类实现的方法也可以在子类重写它的方法。
## 线程不安全实现类
### ArrayList
我们先来看看ArrayList的源码：
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
 	...
    private static final int DEFAULT_CAPACITY = 10;
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	transient Object[] elementData; 
	...
	public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
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
}
```
通过源码我们可以看到：
1. 当通过 new ArrayList()创建对象时，它会分配一个定义好的不能序列化的空的Object数组
2. 当通过 new ArrayList(int initialCapacity)创建对象时，当initialCapacity大于0时，会返回一个初始大小为10的Object数组。
3. ArrayList 继承了AbstractList，AbstractList实现了List接口中的大部分方法，提供了相关的添加、删除、修改、遍历等功能。
4. ArrayList 实现了RandomAccess 接口， RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持快速随机访问的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。
5. ArrayList 实现了Cloneable 接口，即覆盖了函数 clone()，能被克隆。
6. ArrayList 实现java.io.Serializable 接口，这意味着ArrayList支持序列化，能通过序列化去传输。

我们再看看ArrayList是如何扩容的，我们跟进add()：
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{

	private int size;
	 private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
	
	...

	public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
	private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
	private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
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
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
	private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
}
```
分析：
1. 首先我们看到它调用了ensureCapacityInternal(size + 1)方法，并传了一个最小容量过去（minCapacity），其中size初始化大小为0。接着我们跟进ensureCapacityInternal方法；
2. 可以看到ensureCapacityInternal方法中又调用了ensureExplicitCapacity()方法，并且ensureExplicitCapacity()方法的参数为calculateCapacity(elementData, minCapacity)的返回值，那我们跟进calculateCapacity(elementData, minCapacity)方法
3. 在calculateCapacity中进行了一个判断，如果数组为null，则返回默认大小10(DEFAULT_CAPACITY),否则返回minCapacity。接着我们再跟进ensureExplicitCapacity
4. 可以看到在ensureExplicitCapacity方法中进行了一个判断，当minCapacity比数组的长度大就调用grow(minCapacity)方法扩容，否则什么都不干，我们跟进grow(minCapacity)
5. 可以看到新的容量newCapacity为旧的容量oldCapacity加上oldCapacity右移一位，也就是说新的容量是旧的容量的1.5倍，再将新容量和最小容量进行比较，小于就直接将最小容量付给新的容量。如果新的容量大于MAX_ARRAY_SIZE再调用hugeCapacity函数
6. 可以看到最后的扩容会创建一个新的数组，并将老的数组拷贝过来。

### LinkedList
LinkedList是一个实现了List接口和Deque接口的双端链表。 LinkedList底层的链表结构使它支持高效的插入和删除操作，另外它实现了Deque接口，使得LinkedList类也具有队列的特性; LinkedList不是线程安全的，如果想使LinkedList变成线程安全的，可以调用静态类Collections类中的synchronizedList方法。
需要提一下的是，LinkedList类中有一个ListIterator<E> listIterator方法，listIterator接口中包含一个add方法：
```java
public interface ListIterator<E> extends Iterator<E> {
 
    boolean hasNext();
    E next();
    boolean hasPrevious();
    void set(E e);
    void add(E e);
}
```
因为链表是一个有序的集合，每个对象的位置就显得十分重要。LinkedList中的add方法只能将对象添加到链表尾部，而经常却要将对象添加到链表的中间，迭代器就是用于描述集合中的位置的，所以这种依赖位置的方法就交由迭代器来完成。因为Set集合是无序的，所以在Iterator接口中就没有add方法，而是扩展了一个LinkIterator接口来实现。

值得一提的是：大家都知道，链表是不支持快速随机访问的。如果要查看链表中的第n个元素，就必须从头开始，越过n-1个元素，没有捷径可走，但尽管如此，LinkedList还是提供了一个用来访问某个特定元素的get方法，当然这个方法的效率并不高，如果在使用这个方法，那么可能对于所要解决的问题使用了错误的数据结构。LinkedList类中get方法所谓的随机访问都是需要从列表的头部开始搜索，效率极低。使用链表的唯一理由是尽可能的减少在链表中间插入或删除元素所付出的代价。

**LinkedList中特有的方法**
```java
//查询方法：
getFirst() //获取集合中的第一个元素
getLast() //获取集合中的最后一个元素
//添加方法：
addFirst(Object o) //在集合的第一个位置添加指定元素
addLast(Object o) //在集合的最后一个位置添加指定元素
//删除方法：
removeFirst() //删除集合中的第一个元素
removeLast() //删除集合中的最后一个元素
```
下面程序简单的创建了两个链表，将它们合并在一起，然后从第二个链表中每隔一个元素删除一个元素，最后测试removeAll()方 法 ： 

```java
package listdemo;
 
import java.util.LinkedList;
import java.util.List;
import java.util.ListIterator;
 
public class LinkedListTest {
 
	public static void main(String[] args) {
		
		List<String> a = new LinkedList<>();
		a.add("aaa");
		a.add("bbb");
		a.add("eee");
		
		List<String> b = new LinkedList<>();
		b.add("AAA");
		b.add("BBB");
		b.add("EEE");
		
		ListIterator<String> aIter = a.listIterator();
		ListIterator<String> bIter = b.listIterator();
		
		//a集合合并b集合
		
		while(bIter.hasNext()) {
			
			if(aIter.hasNext())
				 aIter.next();
			aIter.add(bIter.next());
		}
		
		System.out.println(a);
		
		//从b链表中每间隔一个元素删除一个元素
		
		while(bIter.hasNext()) {
			
			bIter.next();//跳过一个元素
			if(bIter.hasNext()) {
				
				bIter.next();
				bIter.remove();//先查后删
			}
		}
		
		System.out.println(b);
		
		//测试删除所有
		a.removeAll(a);
		System.out.println(a);
		
	}
}
```
### ArrayList与LinkedList的区别
1. 是否保证线程安全： ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；
2. 底层数据结构： Arraylist 底层使用的是 Object 数组；LinkedList 底层使用的是 双向链表 数据结构（JDK1.6之前为循环链表，JDK1.7取消了循环。注意双向链表和双向循环链表的区别）
3. 插入和删除是否受元素位置的影响： ① ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行add(E e) 方法的时候， ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是O(1)。但是如果要在指定位置 i 插入和删除元素的话（add(int index, E element) ）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 ② LinkedList 采用链表存储，所以插入，删除元素时间复杂度不受元素位置的影响，都是近似 O（1）而数组为近似 O（n）。
4. 是否支持快速随机访问： LinkedList 不支持高效的随机元素访问，而 ArrayList 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index) 方法)。
5. 内存空间占用： ArrayList的空间浪费主要体现在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。

### ArrayList不安全例子

```java
public class ListTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(1,8));
                System.out.println(list);
            },Integer.toString(i)).start();
        }
    }
}
```
运行结果：报java.util.ConcurrentModificationException异常。
导致原因：并发争抢修改导致
如何解决？有三种方法：
1. 使用Collections工具类中的synchronizedList包装ArrayList方法即：Collections.synchronizedList(new ArrayList<>());其底层实现是根据是否实现RandomAccess接口而new两个不同的内部类（SynchronizedRandomAccessList<>(list)，SynchronizedList<>(list)），然后在添加方法subList中使用了synchronized锁。
2. 使用Vector类（该类因为读写方法都用sychronizd关键字修饰，性能差，基本已弃用）
3. 使用CopyOnWriteArrayList类

## 线程安全类
### Vector
vector类和ArrayList类的差别就是Vector在每个方法前都加了个sychronized锁，其它地方和ArrayList基本一致，由于性能太差，基本已被弃用。
### CopyOnWriteArrayList
CopyOnWrite的意思：在计算机，如果你想要对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后再将指向原来内存的指针指向新的内存，原来的内存就可以等着被GC回收了。
再来看看源码：
```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
	
	final transient ReentrantLock lock = new ReentrantLock();
	private transient volatile Object[] array;
	 
	 ...

	public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }
	final void setArray(Object[] a) {
        array = a;
    }
	public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
	
	public E get(int index) {
        return get(getArray(), index);
    }
	final Object[] getArray() {
        return array;
    }
}
```
通过源码我们可以看到：

- CopyOnWriteArrayList同样实现了List<E>, RandomAccess, Cloneable, java.io.Serializable接口。
- 定义了一个可从入锁lock，在Object数组前加了volatile关键字修饰，使得Object线程可见。（关于volatile关键字可以看我的另一篇博客：[从底层吃透java内存模型（JMM）、volatile、CAS](https://blog.csdn.net/qq_41950229/article/details/106332389)）
- 需要注意的是：CopyOnWriteArrayList只提供了三个构造器：无参构造器、一个参数为List构造器，一个参数为数组构造器。并没有提供指定初始容量的构造器，这是因为每次的添加操作都是复制一个新的数组来取代旧的数组的，这也就无需指定初始的数组容量大小了
- CopyOnWriteArrayList 类的所有可变操作（add，set等等）都是通过创建底层数组的新副本来实现的。当 List 需要被修改的时候，并不修改原有内容，而是对原有数据进行一次复制，将修改的内容写入副本。写完之后，再将修改完的副本替换原来的数据，这样就可以保证写操作不会影响读操作了。
- 在所有的读取操作时是不加锁的，所以CopyOnWriteArrayList适用于读多写少的场景中，事实上就算是写多的场景CopyOnWriteArrayList在性能上也要好于Vector，所以这就导致着Vector基本被弃用。

最后附上在最开始说的ArrayList线程不安全使用CopyOnWriteArrayList解决的代码：
```java
public class CopyOnWriteArrayListTest {
    public static void main(String[] args) {
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(1,8));
                System.out.println(list);
            },Integer.toString(i)).start();
        }
    }
}
```

## 代码
本文所涉及的所有代码都在我的GitHub上：https://github.com/dave0824/jmm

## 推荐阅读

[从底层吃透java内存模型（JMM）、volatile、CAS](https://blog.csdn.net/qq_41950229/article/details/106332389)









