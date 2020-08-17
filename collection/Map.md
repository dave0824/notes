## 前言
本文为对Map集合的再一次整理。内容包括：Map HashMap LinkedHashMap TreeHashMap HashTable ConcurrentHashMap

## Map
Map<k,v>使用键值对存储，map会维护与键k相关联的值v。两个key可以关联相同的对象，但key不能重复，常见的key是String类型，但也可以是任何对象。通过键就可以找到对应的值，这种数据结构就是Map(映射)

Map接口中定义的方法：
```java
void clear() 
//删除所有的映射
default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction) 
//尝试计算指定键的映射及其当前映射的值（如果没有当前映射， null ）。  
default V computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction) 
/*如果指定的键尚未与值相关联（或映射到 null ），则尝试使用给定的映射函数计算其值，并将其输入到此/映射中，除非 null 。 */ 
default V computeIfPresent(K key, BiFunction<? super K,? super V,? extends V> remappingFunction) 
//如果指定的密钥的值存在且非空，则尝试计算给定密钥及其当前映射值的新映射。  
boolean containsKey(Object key) 
//如果此映射包含指定键的映射，则返回 true 。  
boolean containsValue(Object value) 
//如果此map将一个或多个键映射到指定的值，则返回 true 。  
Set<Map.Entry<K,V>> entrySet() 
//返回此map中包含的映射的Set视图。  
boolean equals(Object o) 
//将指定的对象与此映射进行比较以获得相等性。  
default void forEach(BiConsumer<? super K,? super V> action) 
//对此映射中的每个条目执行给定的操作，直到所有条目都被处理或操作引发异常。  
V get(Object key) 
//返回到指定键所映射的值，或 null  
default V getOrDefault(Object key, V defaultValue) 
//返回到指定键所映射的值，或 defaultValue
int hashCode() 
//返回此map的哈希码值。  
boolean isEmpty() 
//如果此map不包含键值映射，则返回 true 。  
Set<K> keySet() 
//返回此map中包含的键的Set视图。  
default V merge(K key, V value, BiFunction<? super V,? super V,? extends V> remappingFunction) 
//如果指定的键尚未与值相关联或与null相关联，则将其与给定的非空值相关联。  
V put(K key, V value) 
//将指定的值与该映射中的指定键相关联（可选操作）。  
void putAll(Map<? extends K,? extends V> m) 
//将指定map的所有映射复制到此映射（可选操作）。  
default V putIfAbsent(K key, V value) 
/*如果指定的键尚未与某个值相关联（或映射到 null ）将其与给定值相关联并返回null，否则返回当前值。  */
V remove(Object key) 
//如果存在（从可选的操作），从该map中删除一个键的映射。  
default boolean remove(Object key, Object value) 
//仅当指定的密钥当前映射到指定的值时删除该条目。  
default V replace(K key, V value) 
//只有当目标映射到某个值时，才能替换指定键的条目。  
default boolean replace(K key, V oldValue, V newValue) 
//仅当当前映射到指定的值时，才能替换指定键的条目。  
default void replaceAll(BiFunction<? super K,? super V,? extends V> function) 
//将每个条目的值替换为对该条目调用给定函数的结果，直到所有条目都被处理或该函数抛出异常。  
int size() 
//返回此map中键值映射的数量。  
Collection<V> values() 
//返回此map中包含的值的Collection视图。  
```
## Map的实现类
## 线程不安全实现类
### 1.HashMap
JDK1.8 之前 HashMap 底层是 数组和链表 结合在一起使用也就是 链表散列。HashMap 通过 key 的 hashCode 经过扰动函数（所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。）处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。数据结构如下图：
![HashMap1.8before](..\asset\collection\HashMap1.8before.png)
JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。数据结构图如下：
![HashMap1.8](..\asset\collection\HashMap1.8.jpg)
HashMap最多只允许一条记录的键为null，允许多条记录的值为null。HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

#### HashMap的初始化源码：
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

	static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
	static final int MAXIMUM_CAPACITY = 1 << 30;
	static final float DEFAULT_LOAD_FACTOR = 0.75f;
	transient Node<K,V>[] table; // 节点数组
	final float loadFactor; // 加载因子也称填充因子
    int threshold; // 阀值，计算：threshold = loadFactor * capcity(节点数组容量)
	...

	public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
	 public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
	 public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
	static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
}
```
从源码中可以看到：
1. DEFAULT_INITIAL_CAPACITY = 1 << 4；节点数组默认大小为16，这里使用位运算，相比乘法运算性能更快。
2. Node<K,V>[] table，节点数组（也叫桶数组），就相当于是链表中的节点。需要注意的是Node类中的K是使用final关键字修饰的，也就说明了Map中的键（key）是不能修改的。
3. threshold ：阀值，计算方式为threshold = loadFactor * capcity(节点数组容量)，当节点数组的使用容量超过阀值时就会以2的幂次方扩充节点数组。
3. HashMap空参初始化的时候并没有初始化节点数组，而只是设置了默认的加载因子，初始化大小会在第一次调用put方法后调用resize()方法进行初始化，其实数组的扩容也是调用这个方法进行的。


#### HashMap的put方法
HashMap的put方法的源码有些多和难以理解（这里就不展示和解读了，感兴趣的可以自己去看源码），我们看一张java8中put方法的处理逻辑图：
![HashMap-put](..\asset\collection\HashMap-put.png)
												-- 图片来源：https://www.jianshu.com/p/e2f75c8cce01
这里我们重点说说resize()方法，在两个地方会调用到resize方法：
1. 当节点数组table为null时，会调用resize方发完成初始化
2. 当节点数组table使用的容量大于阀值threshold时会调用resize方法扩容。

看看resize方法的部分源码：
```java
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
		...
}
```
从源码中我们可以看到，初始化时table为空，紧接着就执行newCap = DEFAULT_INITIAL_CAPACITY;初始化新的容量newCap大小为默认容量大小16，最后再new一个容量为16的节点数组再将成员变量中的节点数组table的引用指向新的节点数组。所以说HashMap是在执行put()方法才初始化数组大小的。

我们看当oldCap>0，容量大于等于默认值且未超过最大值时，会通过newCap = oldCap << 1 和newThr = oldThr << 1; 将容量和加载因子都扩大一倍。也就是说，HashMap的扩容机制就是重新申请一个容量是当前的2倍的节点数组，然后将原先的记录逐个重新映射到新的节点数组里面，然后将原先的节点数组里的节点逐个置为null使得引用失效。

剩余部分的代码就是通过新插入的节点（每一个put(key,value)都会被封装成一个节点）通过e.hash & (newCap - 1)找到节点数组的下标判断是否有值...自己看图。

#### 为什么HashMap是不安全的？
其实HashMap之所以线程不安全就是出在了这个resize方法上，在resize操作的时候会造成线程不安全，如：put的时候导致的多线程数据不一致。

这个问题比较好想象，比如有两个线程A和B，首先A希望插入一个key-value对到HashMap中，首先计算记录所要落到的桶的索引坐标，然后获取到该桶里面的链表头结点，此时线程A的时间片用完了，而此时线程B被调度得以执行，和线程A一样执行，只不过线程B成功将记录插到了桶里面，假设线程A插入的记录计算出来的桶索引和线程B要插入的记录计算出来的桶索引是一样的，那么当线程B成功插入之后，线程A再次被调度运行时，它依然持有过期的链表头但是它对此一无所知，以至于它认为它应该这样做，如此一来就覆盖了线程B插入的记录，这样线程B插入的记录就凭空消失了，造成了数据不一致的行为。

### 2.LinkedHashMap
LinkedHashMap是HashMap的一个子类，保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的，也可以在构造时带参数，按照访问次序排序。看名字也可以猜到，LinkedHashMap就是在HashMap的基础上加了一个链表来保存记录的插入顺序。结构图如下：

![LinkedHashMap](..\asset\collection\LinkedHashMap.jpg)

### 3.TreeMap
TreeMap实现SortedMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。如果使用排序的映射，建议使用TreeMap。在使用TreeMap时，key必须实现Comparable接口或者在构造TreeMap传入自定义的Comparator，否则会在运行时抛出java.lang.ClassCastException类型的异常

这里顺便提一下Comparable与Comparator的简单区别：

- comparable是java.lang包下的一个接口，使用compareTo(Object obj)方法来进行排序的。
- comparator是java.util包下的一个接口，使用compare（Object obj1,Object obj2）方法来进行排序的。

## 线程安全类
### 1.HashTable
HashTable和HashMap的实现原理几乎一样，差别无非是1.HashTable不允许key和value为null；2.HashTable是线程安全的。但是HashTable线程安全的策略实现代价却太大了，简单粗暴，get/put所有相关操作都是synchronized的，这相当于给整个哈希表加了一把大锁，多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞，相当于将所有的操作串行化，在竞争激烈的并发场景中性能就会非常差。Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。数据结构如下图：

![HashTable](..\asset\collection\HashTable.jpg)

### 2.ConcurrentHashMap
ConcurrentHashMap是Java并发包中提供的一个线程安全且高效的HashMap实现

1.分段锁机制
概念由来：上面我们已经说了HashTable线程安全的策略实现代价太大了，它将get/put所有相关操作都是使用synchronized修饰的，这相当于给整个哈希表加了一把大锁，多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞，相当于将所有的操作串行化，在竞争激烈的并发场景中性能就会非常差。所以在jdk1.5~1.7版本提出了分段机制概念。

原理：分段锁对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。结构图如下：
![ConcurrentHashMap1.8befor](..\asset\collection\ConcurrentHashMap1.8befor.jpg)


2.jdk1.8版本（现在所用的实现方式）
在jdk1.8版本中，由于采用分段锁时，最大并发数受限于segment个数，有很大的局限性，所以在jdk1.8版本中ConcurrentHashMap取消了Segment分段锁，采用CAS和synchronized来保证并发安全。数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树。Java 8在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为O(N)）转换为红黑树（寻址时间复杂度为O(log(N))）synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，性能得到了很大的提升。

JDK1.8的ConcurrentHashMap（TreeBin: 红黑二叉树节点 Node: 链表节点）：

![ConcurrentHashMap1.8](..\asset\collection\ConcurrentHashMap1.8.jpg)

## 参考
https://www.jianshu.com/p/e2f75c8cce01
https://snailclimb.top/JavaGuide/#/java/Multithread
