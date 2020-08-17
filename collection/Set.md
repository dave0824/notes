## 前言
本文为对Set集合的整理

## 先看看集合框架接口图
![Set](../asset\collection\Set.jpg)
 注：图不包括线程安全集合

## 散列集
谈Set前先说说散列集，有一种总所周知的数据结构，可以快速的查找所需要的对象，它就是散列表(hash table)。散列表为每一个对象计算一个整数，称为散列码，也叫哈希码。散列码是有对象的实例域产生的一个整数。更确切的说，具有不同数据域的对象将产生不同的散列码，当然这是理想的情况下。自定义的类，就要负责实现这个类的hashCode方法，然而有些程序设计者重写hashCode()所使用的杂凑算法也许刚好会让多个对象传回相同的值，越是糟糕的杂凑算法越容易碰撞，这也与数据值域分布的特性有关。

在java中，散列表用链表数组实现。每个列表被称作为桶。想要查找表中对象的位置，就要计算它的散列码，然后与桶的总数取余，所得到的结果就是保存这个元素的桶的索引。列如，如果某个对象的散列码为76268，并且有128个桶，对象就应该保存在108号桶中（76268%128=108）。若桶中没有其它元素就可以直接将元素插入桶中，若桶中有其它元素就需要用新对象与桶中的所有对象进行比较，查看这个对象是否已经存在，这种现象被称为散列冲突。这种方法也是Set找重时所用到的方法，稍后会再次讲到

散列表可以用于实现几个重要的数据结构。其中最简单的就是set类型，set是没有重复元素的元素集合，set的add方法首先会在集中查找要添加的对象，如果不存在，就将这个对象添加进去。

## Set
Set集合扩展了Collection接口，它是一个不允许重复的集合，不会有多个元素引用相同的对象。

更正式地，集合不包含一对元素e1和e2 ，使得e1.equals(e2) ，并且最多一个空元素。 正如其名称所暗示的那样，这个接口模拟了数学集抽象。

Set接口中的方法：

```java
boolean add(E e) 
//添加指定的元素  
boolean addAll(Collection<? extends E> c) 
//将指定集合中的所有元素添加到此集合  
void clear() 
//从此集合中删除所有元素  
boolean contains(Object o) 
//如果此集合包含指定的元素，则返回 true  
boolean containsAll(Collection<?> c) 
//如果此集合包含所有指定集合的元素返回 true 
boolean equals(Object o) 
//将指定的对象与此集合进行比较以实现相等
int hashCode() 
//返回此集合的哈希码值。  
boolean isEmpty() 
//如果此集合不包含元素，则返回 true
Iterator<E> iterator() 
//返回此集合中元素的迭代器。  
boolean remove(Object o) 
//如果存在，则从该集合中删除指定的元素  
boolean removeAll(Collection<?> c) 
//从此集合中删除指定集合中包含的所有元素  
boolean retainAll(Collection<?> c) 
//仅保留该集合中包含在指定集合中的元素  
int size() 
//返回此集合中的元素数  
default Spliterator<E> spliterator() 
//在此集合中的元素上创建一个 Spliterator 。  
Object[] toArray() 
//返回一个包含此集合中所有元素的数组。  
<T> T[] toArray(T[] a) 
//返回一个包含此集合中所有元素的数组; 返回的数组的运行时类型是指定数组的运行时类型 
```
## Set接口的具体实现类
### 线程安全类
#### HashSet
我们先看看它的源码：
```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
	 private transient HashMap<E,Object> map;
	 private static final Object PRESENT = new Object();
	 ...

	 public HashSet() {
        map = new HashMap<>();
    }
	public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
	
	...

	// 下面是HashMap中的方法，上面的put方法会层层调用到这个方法
	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict){
		...
		if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
		...
	}
	

}
```
通过源码我们可以看到，其实HashSet的底层就是HashMap，那么问题来了：我们都知道HashMap的put方法是存键值对的，那么HashSet呢？可以看到源码中的add方法，在HashSet中，添加的元素是存在HashMap的键中，而值是统一的一个无意义的Object对象。
当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals（）方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。

**HashSet中的方法**：
```java
boolean add(E e) 
//将指定的元素添加到此集合
void clear() 
//从此集合中删除所有元素
Object clone() 
//返回此 HashSet实例的浅层副本：元素本身不被克隆。  
boolean contains(Object o) 
//如果此集合包含指定的元素，则返回 true 。  
boolean isEmpty() 
//如果此集合不包含元素，则返回 true 。  
Iterator<E> iterator() 
//返回此集合中元素的迭代器。  
boolean remove(Object o) 
//如果存在，则从该集合中删除指定的元素。  
int size() 
//返回此集合中的元素数（其基数）。  
Spliterator<E> spliterator() 
//在此集合中的元素上创建late-binding和故障快速 Spliterator 
```
### TreeSet
TreeSet类与散列集十分类似，不过它比散列集有所改进。TreeSet是一个有序集合。可以以任意顺序将元素插入集合中。在对集合进行遍历时，每个值将自动地按照排序后的顺序呈现。排序使用树结构完成的（当前使用的是红黑树，这里不做介绍，详情请移步：[五分钟搞懂什么是红黑树](http://www.360doc.com/content/18/0904/19/25944647_783893127.shtml)）。如果要使用TreeSet方法就需要提供排序方法，否则就会报错。详情可阅读：[TreeSet的两种排序方式](https://blog.csdn.net/xiaofei__/article/details/53138681)

### 线程安全类
#### CopyOnWriteArraySet
先看看源码：
```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
	private final CopyOnWriteArrayList<E> al;

	public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }

	 public boolean add(E e) {
        
        return al.addIfAbsent(e);
    }
	public boolean addIfAbsent(E e) {
        Object[] snapshot = getArray();
        return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
            addIfAbsent(e, snapshot);
    }

	//这段代码也很好理解就是首先检查原来的数组里面有没有要添加的元素，如果有的话就不要再添加了，如果没有的话，创建一个新的数组，复制之前数组元素并且添加新的元素
	public boolean addIfAbsent(E e, Object[] snapshot) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] current = getArray();
            int len = current.length;
            if (snapshot != current) {
                // Optimize for lost race to another addXXX operation
                int common = Math.min(snapshot.length, len);
                for (int i = 0; i < common; i++)
                    if (current[i] != snapshot[i] && eq(e, current[i]))
                        return false;
                if (indexOf(e, current, common, len) >= 0)
                        return false;
            }
            Object[] newElements = Arrays.copyOf(current, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
}
```
从源码中我们可以看到它的底层实现就是CopyOnWriteArrayList，关于CopyOnWriteArrayList可以看我的另一篇博客：[List详解](https://blog.csdn.net/qq_41950229/article/details/106393194)
