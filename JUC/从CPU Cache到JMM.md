## 前言
随着计算机的飞速发展，cpu从单核到四核，八核。在2020年中国网民数预计将达到11亿人。这些数据都意味着，作为一名java程序员，必须要掌握多线程开发，谈及多线程，绕不开的是对JMM(Java 内存模型)。那么什么是JMM?什么是可见性、原子性、有序性？本文将从CPU的缓存开始谈起，深度解剖JMM底层原理。

## CPU高速缓存（cache）
学过操作系统的同学都应该知道CPU缓存。那么为什么要弄这么一个CPU缓存呢？这是因为缓存的出现主要是为了解决CPU运算速度与内存读写速度不匹配的矛盾，因为CPU运算速度要比内存读写速度快很多，这样会使CPU花费很长时间等待数据到来或把数据写入内存。因此如果任何时候对数据的操作都要通过和内存的交互来进行，会大大降低指令执行的速度。因此在CPU里面就有了高速缓存。也就是，当程序在运行过程中，会将运算需要的数据从主存复制一份到CPU的高速缓存当中，那么CPU进行计算时就可以直接从它的高速缓存读取数据和向其中写入数据，当运算结束之后，再将高速缓存中的数据刷新到主存当中

![CPUCache](..\asset\CPUCache.png)
如图，CPU缓存分为三层（L1,L2,L3 Cache）,L1和L2 Cache都是每个CPU core独立拥有一个，而L3 Cache是几个Cores共享的，可以认为是一个更小但是更快的内存。CPU在做运算时需要先把内存（RAM）中的数据读取到缓存当中，经过运算后再将数据写回内存（RAM）中。这样的操作在单核CPU中当然是没有问题的，但是在多核CPU中会出现**Cache一致性问题**。
### 什么是Cache一致性问题？
比如两个CPU（a和b）同时将内存中的同一个变量i=0加载到了CPU缓存（L1或L2）中，aCPU对变量i进行了++操作后回写到了内存中，此时内存中的i变量值变成了1，但是bCPU不知道，这是 bCPU在缓存中(L1或L2)的i变量还是0，这时bCPU对i变量进行i++运算后回写到内存中，这是内存中的i变量被覆盖，值还是1。这就是Cache一致性问题。
### 如何解决Cache一致性问题？
为了正确性，一旦一个CPU更新了内存中的内容，硬件就必须要保证其他的核心能够读到更新后的数据。目前大多数硬件采用的策略或协议是MESI或基于MESI的变种：
M代表更改（modified），表示缓存中的数据已经更改，在未来的某个时刻将会写入内存；
E代表排除（exclusive），表示缓存的数据只被当前的CPU所缓存；
S代表共享（shared），表示缓存的数据还被其他CPU缓存；
I代表无效（invalid），表示缓存中的数据已经失效，即其他CPU更改了数据。
单个CPU对缓存中数据进行了改动，需要通知给其它CPU，也就是意味着，CPU处理要控制自己的读写操作，还要监听其他CPU发出的通知，从而保证最终一致。
### CPU运行时的指令重排
CPU在对性能的优化除了缓存之外还有运行时指令重排，当CPU写缓存时发现缓存区正被其他CPU占用（例如：三级缓存L3），为了提高CPU处理性能，可能将后面的读缓存命令优先执行。列如：

```
 x = 6;
 y = z;
```
这一段程序的正常执行顺序应该是：
1. 将6写入X
2. 读取z的值
3. 将z值写入y

但是经过CPU指令重排后的执行顺序可能是这样：
1. 读取z的值
2. 将z值写入y
3. 将6写入x
当然，指令重排并非随便重排，是需要遵守 as-if-serial 语义的，as-if-serial 语义的意思是指不管怎么重排序（编译器和处理器为了提高并行度），单线程程序的执行结果不能被改变。编译器，runtime 和处理器都必须遵守 as-if-serial 语义，也就是说编译器和处理器不会对存在数据依赖关系的操作做重排序。但是，虽然遵守了 as-if-serial语义，仅在单CPU自己执行的情况下能保证结果正确。多核多线程中，指令逻辑无法分辨因果关联，可能出现乱序执行，导致程序运行结果错误。为了解决这个问题，就需要引入**内存屏障**了。
### 内存屏障
处理器提供了两个内存屏障（Memory Barrier）指令用于解决上述两个问题：



- 写内存屏障（Store Memory Barrier）：在指令后插入 Store Barrier，能让写入缓存中的最新数据更新写入主内存，让其他线程可见。强制写入主内存，这种显示调用，CPU 就不会因为性能考虑而去对指令重排。



- 读内存屏障（Load Memory Barrier）：在指令前插入 Load Barrier，可以让高速缓存中的数据失效，强制从新的主内存加载数据。强制读取主内存内容，让 CPU 缓存与主内存保持一致，避免了缓存导致的一致性问题。

好了，到这里总算是将CPU的缓存机制粗略的讲完了，接下来到了文章的重点部分：JMM，其实JMM的实现原理基本上就是照搬的CPU高速缓存的Cache一致性问题和CPU运行时的指令重排问题的解决策略。

## JMM是什么？
JMM(Java内存模型Java Memory Model)本身是一种抽象的概念 并不真实存在,它是Java虚拟机规范中试图定义的一种模型或规范来屏蔽各个硬件平台和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。，通过规范定制了程序中各个变量(包括实例字段,静态字段和构成数组对象的元素)的访问方式.
JMM关于同步规定:


1. 可见性：要求主存中的变量数据对每个线程是可见的，即每个线程要得到主内存中实时的最新数据
2. 原子性：变量的修改是一个不可分割的步骤
3. 有序性：程序执行的顺序按照代码的先后顺序执行
 
由于JVM运行程序的实体是线程,而每个线程创建时JVM都会为其创建一个工作内存(对应JVM内存区域的虚拟机栈),工作内存是每个线程的私有数据区域,而Java内存模型中规定所有变量（这里指的变量为类的成员变量，方法中创建的临时变量不在其中，下同）都存储在主内存（对应JVM内存区域的堆）,主内存是共享内存区域,所有线程都可访问,但线程对变量的操作(读取赋值等)必须在工作内存中进行,首先要将变量从主内存拷贝到自己的工作空间,然后对变量进行操作,操作完成再将变量写回主内存,不能直接操作主内存中的变量,各个线程中的工作内存储存着主内存中的变量副本拷贝,因此不同的线程无法访问对方的工作内存,线程之间的通讯(传值) 必须通过主内存来完成,其简要访问过程如下图:

![JMM](..\asset\JMM.bmp)

### 可见性
在之前的CPU高速缓存中，我们讲解了**Cache一致性问题**，JMM规范中的**可见性**和**Cache一致性问题**是一样一样的。即：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
下面一段代码将描述变量的不可见性：
```java
public class NoVisibility {

    private static int NUM = 0;

    public void numEqTen(){
        NUM = 10;
    }

    public static void main(String[] args) {
        final NoVisibility noVisibility = new NoVisibility();

        // 第一个线程
        new Thread(() -> {
            try {
                // 睡眠1秒钟，保证主线程得到执行
                Thread.sleep(1000L);
                noVisibility.numEqTen();
                System.out.println(Thread.currentThread().getName() + "\t 执行完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"thread1").start();

        while (noVisibility.NUM == 0) {
            //如果myData的num一直为零，main线程一直在这里循环
        }
        System.out.println(Thread.currentThread().getName() + "\t 主线程执行完毕, num 值是 " + noVisibility.NUM);
    }
}
```
该程序的运行结果是：输出thread1执行完毕，后一直停在了主线程的while循环中不能结束。下面解释一下这段代码为什么一直停留在while而无法执行完毕：
在前面已经解释过，每个线程在运行过程中都有自己的工作内存，那么主线程在运行的时候，会将num变量的值拷贝一份放在自己的工作内存当中。那么当线程1更改了num变量的值之后，主线程由于不知道线程1对num变量的更改，因此还会一直循环下去。

### 原子性
即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。对变量的操作，如：i++,该操作是分为三个指令执行的：
1. 先得到i的初始值
2. 对i进行加1操作
3. 把i的累加结果写回给i
在多线程的环境下，线程在运行i++操作时，可能会在第一或第二个指令结束后由于线程的调度而被挂起去执行其它线程导致所得的结果可能会不是预期的结果。所以JMM规范变量的操作必须为原子操作。下面给出程序演示非原子性操作：
```java
public class NoAtomicity {

    private int num;

    public void numPlusPlus(){
        num++;
    }

    public static void main(String[] args) {
        NoAtomicity noAtomicity = new NoAtomicity();

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j <200 ; j++) {
                        noAtomicity.numPlusPlus();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"thread" + String.valueOf(i)).start();
        }

        // 等待上面的线程运行完毕
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "\t num的最终值是：" + noAtomicity.num);
    }
}
```
我们都知道在理想情况下值应该是2000，然而因为num++不是原子性的，所以我执行出来的结果是：main num的最终值是：1600  当然，每次运行的结果可能都不一样。但基本上都是小于2000的。

### 有序性
在前面我们讲解了**CPU运行时的指令重排**，这里的**有序性**也是同样的问题。计算机在执行程序时,为了提高性能（原因在CPU运行时的指令重排有说）,编译器和处理器常常会做指令重排,一把分为以下3中：

![ORDER](..\asset\order.bmp) 

线程环境里面确保程序最终执行结果和代码顺序执行的结果一致，处理器在进行重新排序是必须要考虑指令之间的数据依赖性。多线程环境中线程交替执行,由于编译器优化重排的存在,两个线程使用的变量能否保持一致性是无法确定的,所以所得的结果无法预测。
重排代码实例：

声明变量：int a,b,x,y=0

| 线程1          | 线程2          |
   | -------------- | -------------- |
   | x = a;         | y = b;         |
   | b = 1;         | a = 2;         |
   | 结          果 | x = 0      y=0 |

   如果编译器对这段程序代码执行重排优化后，可能出现如下情况：

   | 线程1          | 线程2          |
   | -------------- | -------------- |
   | b = 1;         | a = 2;         |
   | x= a;          | y = b;         |
   | 结          果 | x = 2      y=1 |
这个结果说明在多线程环境下，由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的。
另外，Java内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 happens-before 原则。如果两个操作的执行次序无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。

下面就来具体介绍下happens-before原则（先行发生原则）：

1. 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
2. 锁定规则：一个unLock操作先行发生于后面对同一个锁额lock操作
3. volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
4. 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
5. 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作
6. 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
7. 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
8. 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始
　　这8条原则摘自《深入理解Java虚拟机》。

这8条规则中，前4条规则是比较重要的，后4条规则都是显而易见的。

下面我们来解释一下前4条规则：

1. 对于程序次序规则来说，我的理解就是一段程序代码的执行在单个线程中看起来是有序的。注意，虽然这条规则中提到“书写在前面的操作先行发生于书写在后面的操作”，这个应该是程序看起来执行的顺序是按照代码顺序执行的，因为虚拟机可能会对程序代码进行指令重排序。虽然进行重排序，但是最终执行的结果是与程序顺序执行的结果一致的，它只会对不存在数据依赖性的指令进行重排序。因此，在单个线程中，程序执行看起来是有序执行的，这一点要注意理解。事实上，这个规则是用来保证程序在单线程中执行结果的正确性，但无法保证程序在多线程中执行的正确性。
2. 第二条规则也比较容易理解，也就是说无论在单线程中还是多线程中，同一个锁如果出于被锁定的状态，那么必须先对锁进行了释放操作，后面才能继续进行lock操作。
3. 第三条规则是一条比较重要的规则，直观地解释就是，如果一个线程先去写一个变量，然后一个线程去进行读取，那么写入操作肯定会先行发生于读操作。
4. 第四条规则实际上就是体现happens-before原则具备传递性。
## 如何实现JMM规范？
在了解了JMM规范后，那么如何保证变量的可见性、原子性和有序性呢？可爱的java为我们提供了一些关键字如：synchronized、volatile。还有一个诚意满满的类库：JUC，是不是很感动？哈哈~ 接下来我们来介绍几种实现。
### synchronized
谈及synchronized，这家伙在在JavaSE 1.6之前可是一个重量级锁，在JavaSE 1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化之后变得在某些情况下并不是那么重了。synchronized的底层实现主要依靠 Lock-Free 的队列，基本思路是 自旋后阻塞，竞争切换后继续竞争锁，稍微牺牲了公平性，但获得了高吞吐量。synchronized有三种使用方式：


1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象
3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象
4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象

当某部分被sychronized关键字修饰后，该部分在任意时刻只能有一个线程执行（得到锁的线程）,既然只能有一个线程执行，那么JMM中的可见性，原子性它都能够保证了。那么有序性呢？sychronized还是不能阻止指令重排，在双重检验+锁实现单例模式时还是会出现空指针异常，这个我们后面会讲到。

### volatile
volatile是Java虚拟机提供的轻量级的同步机制，作用在变量上（类成员变量、类的静态成员变量），它能对作用的变量保证可见性和禁止指令重排，但是并不能保证原子性。

**可见性**：
我们回到前面讲可见性时举的例子：
```java
public class NoAtomicity {

    private volatile int num;

    public void numPlusPlus(){
        num++;
    }

    public static void main(String[] args) {
        NoAtomicity noAtomicity = new NoAtomicity();

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j <200 ; j++) {
                        noAtomicity.numPlusPlus();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"thread" + String.valueOf(i)).start();
        }

        // 等待上面的线程运行完毕
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "\t num的最终值是：" + noAtomicity.num);
    }
}
```
通过之前的分析，我们知道主线程会在while循环中一直循环下去出不来，那么，如果在num变量前面加上关键字volatile修饰，情况就不一样了：

1. 使用volatile关键字会强制将修改的值立即写入主存
2. 使用volatile关键字的话，当线程1进行修改时，会导致主线程的工作内存中缓存变量num的缓存行无效（反映到硬件层的话，就是CPU的L1或者L2缓存中对应的缓存行无效）
3. 由于主线程的工作内存中缓存变量num的缓存行无效，所以主线程再次读取变量num的值时会去主存读取

那么在线程1修改num值时（当然这里包括2个操作，修改线程1工作内存中的值，然后将修改后的值写入内存），会使得主线程的工作内存中缓存变量num的缓存行无效，然后主线程读取时，发现自己的缓存行无效，它会等待缓存行对应的主存地址被更新之后，然后去对应的主存读取最新的值。那么主线程读取到的就是最新的正确的值。

**有序性**
在前面提到volatile关键字能禁止指令重排序，所以volatile能在一定程度上保证有序性。
volatile关键字禁止指令重排序有两层意思：
1. 当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；
2. 在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。

我们前面讲了**CPU运行时的指令重排**底层原理其实是内存屏障，volatile关键字禁止指令重排其实就是利用了内存屏障的原理：
“观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”
lock前缀指令实际上相当于一个内存屏障（也叫内存栅栏），内存屏障会提供3个功能：
1. 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
2. 它会强制将对缓存的修改操作立即写入主存；
3. 如果是写操作，它会导致其他CPU中对应的缓存行无效。

**原子性**：
在前面我们讲原子性的时候已经讲过，比举了一个例子，现在我们再对刚才那个例子进行讲解：

```java
public class NoAtomicity {

    private volatile int num;

    public void numPlusPlus(){
        num++;
    }

    public static void main(String[] args) {
        NoAtomicity noAtomicity = new NoAtomicity();

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j <200 ; j++) {
                        noAtomicity.numPlusPlus();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"thread" + String.valueOf(i)).start();
        }
		// 等待上面的线程运行完毕
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "\t num的最终值是：" + noAtomicity.num);
    }
}
```
之前我们讲过，在变量num没有加上volatile关键字修饰时，最后num的结果会是小于2000，那么加上之后呢？我们来分析分析：假设此时num的值为10
1. 线程1对变量num进行自增操作，即线程1读取num的原始值，并且对num进行了加1操作，此时等式右边的值已经变成了11（将 num++ 看成 num = num + 1 ），但还没有将11赋值给num，然后线程1被阻塞了。
2. 然后线程2对变量num进行自增操作，线程2也去读取原始值，但这时由于线程1并没有将11赋值给num,即没有对变量进行修改操作，所以不会导致线程2的工作内存中缓存变量num的缓存行无效，所以这时num读取到的值还是10，然后进行加1操作，并把11赋值给num写入工作内存，最后写入主存。
3. 然后线程1接着进行赋值操作，将11赋值给变量num，然后将11写入工作内存，最后写入主存。

此时可以发现，两次自增操作下来，由于num++不是原子操作，从而导致变量num只增加了1。
那么如何保证原子性？有三种解决办法：
1. 在numPlusPlus方法前面加上sychronized关键字修饰
2. 使用Lock锁
3. 使用JUC.Atomic包下的AtomicInteger（后面细讲）

理解了volatile和sychronized关键字后，我们来举个常用的懒汉式双重判断+锁的单例模式的实现：

```java
public class Singleton {

    private static volatile Singleton instance;

    private Singleton(){ }

    public Singleton getInstance(){
        if (instance == null){
            synchronized (Singleton.class){
                if (instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
这里将变量instance使用volatile修饰的原因是为了防止指令重排，导致空指针异常，具体原因：

在 instance = new Singleton();这个操作不是原子操作，可能存在着指令重排,正常顺序是：
1. 为Singleton()分配内存
2. 初始化Singleton()
3. 将instance变量指向Singleton()对象在堆内存中的地址

然而出现指令重排后，可能的顺序会变成132，这样就会导致线程1在执行到第3步时线程1被阻塞，这时虽然第2步还没有执行，但是instance已经不为null了
然后线程2获得执行，在if判断时，因为instance不为null了，此时将会直接返回instance。这时线程2在通过instance访问其成员变量时（如：instance.getName()）就会报空指针异常。

这里使用的双重if判断的原因：
1. 第一个if判断主要是为了提高速率，因为绝大部分的线程都会在第一个if判断后就直接返回instance从而跳过了synchronize这个略重的线程锁。
2. 第二个判断是为了防止有两个或以上线程同时通过了第一个if判读进而挣抢锁，线程1第一个获取到了锁创建了实例释放锁后，线程2竞争到了锁，如果这时没有加if判断，那么线程2也会创建实例。

好了，我们回到刚刚说的使用JUC.Atomic包下的AtomicInteger解决volatile关键字不能实现原子性而导致上面程序的结果不为2000的解决办法。那么何为AtomicInteger？

### AtomicInteger
AtomicInteger类是java.util.concurrent.atomic下的类。java在atomic包下提供了基本变量和引用变量的原子类，支持单个变量上的无锁线程安全编程。使用AtmoicInteger + volatile关键字实现上面所提到的程序结果不为2000的程序：
```java
public class Atomicity {
    private volatile AtomicInteger num = new AtomicInteger(0);

    public void numIncrement(){

        num.getAndIncrement();
    }
    public int getNum(){
        return num.get();
    }

    public static void main(String[] args) {
        Atomicity atomicity = new Atomicity();

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j <200 ; j++) {
                        atomicity.numIncrement();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"thread" + String.valueOf(i)).start();
        }

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "\t num的最终值是：" + atomicity.getNum());
    }
}
```
运行此代码的结果是2000，那么为什么使用了AtomicInteger后就能保证原子性了呢？
我们翻看AtomicInteger的源码：
```java
 /**
     * Atomically increments by one the current value.
     *
     * @return the previous value
     */
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
```
发现调用的是unsafe的方法，那么usafe又是什么呢？
UnSafe是CAS的核心类 由于Java 方法无法直接访问底层 ,需要通过本地(native)方法来访问,UnSafe相当于一个后面,基于该类可以直接操作特额定的内存数据.UnSafe类在于sun.misc包中,其内部方法操作可以向C的指针一样直接操作内存,因此Java中CAS操作依赖于UNSafe类的方法.
注意UnSafe类中所有的方法都是native修饰的,也就是说UnSafe类中的方法都是直接调用操作底层资源执行响应的任务。
好了，现在了解了UnSafe是CAS的核心类，那么CAS又是什么？

### CAS
CAS的全称为Compare-And-Swap ,它是一条CPU并发原语.
它的功能是判断内存某个位置的值是否为预期值,如果是则更新为新的值,这个过程是原子的.
CAS并发原语提现在Java语言中就是sun.miscUnSafe类中的各个方法.调用UnSafe类中的CAS方法,JVM会帮我实现CAS汇编指令.这是一种完全依赖于硬件 功能,通过它实现了原子操作,再次强调,由于CAS是一种系统原语,原语属于操作系统用于范畴,是由若干条指令组成,用于完成某个功能的一个过程,并且原语的执行必须是连续的,在执行过程中不允许中断,也即是说CAS是一条原子指令,不会造成所谓的数据不一致的问题。
了解了CAS后，现在我们继续跟进unsafe.getAndAddInt(this, valueOffset, 1)方法：
```java
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```
那么这个方法又是如何实现原子操作的呢？
先对方法的参数进行解读：
1. 其中var1为AtomicInteger的对象，在上面的程序中，就是num对象
2. valueOffset为地址偏移量，即为num对象在主内存中的地址
3. var4为1，也就是每次要增加的值

对这个方法的解读：
假设线程A和线程B两个线程同时执行getAndAddInt操作(分别在不同的CPU上):

1. AtomicInteger里面的value原始值为3,即主内存中AtomicInteger的value为3,根据JMM模型,线程A和线程B各自持有一份值为3的value的副本分别到各自的工作内存。
2. 线程A通过getIntVolatile(var1,var2) 拿到value值3,这是线程A被挂起。
3. 线程B也通过getIntVolatile(var1,var2) 拿到value值3,此时刚好线程B没有被挂起并执行compareAndSwapInt方法比较内存中的值也是3 成功修改内存的值为4 线程B运行完毕。
4. 这时线程A恢复,执行compareAndSwapInt方法比较,发现自己手里的数值和内存中的数字4不一致,说明该值已经被其他线程抢先一步修改了,那A线程修改失败,只能重新来一遍了。
5. 线程A重新获取value值,因为变量value是volatile修饰,所以其他线程对他的修改,线程A总是能够看到,线程A继续执行compareAndSwapInt方法进行比较替换,直到成功。

好了，到这里就解释清楚了AtomicInteger是如何保证原子性的，但是它的缺点也很明显：
1. 循环时间长，开销很大。我们可以看到有个do while循环，若CAS一直失败，会一直重试。
2. 只能保证一个共享变量的原子性。一个变量可用使用CAS来保证原子性，若是涉及多个变量那就得使用锁来保证原子性了。
3. 会导致ABA问题

### ABA问题

什么是ABA问题？简单点的回答就是：狸猫换太子！
因为CAS在取出主存中的数据，然后再进行比较，在这两个步骤中会有一个时间差，即这两个步骤不是原子性的。那么就有可能线程2在线程1取完数据A后，也将数据A取出并将它改为B然后又将它改回A写回内存。这是线程1在进行CAS操作时发现内存中的数据还是A，然后线程1就执行成功了。这就是ABA问题。
ABA问题程序实现：
```java
public class ABA {

    private static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);

    public static void main(String[] args) {

        new Thread(() ->{
            atomicReference.compareAndSet(100,101);
            atomicReference.compareAndSet(101,100);
        },"thread1").start();

        new Thread(() ->{
            try {
                // 睡眠1秒，保证完成ABA
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicReference.compareAndSet(100,2020);
            System.out.println(atomicReference.get());
        },"thread1").start();
    }
}
```
执行的最终结果为2020，没有解决ABA问题

那么如何解决ABA问题？
我们想想每次完成CAS操作后都给它加上一个版本号不就可以知道它有没有被改过了嘛？那既然我们都能想到，可爱的Java早就想到了并且为我们提供了一个叫AtomicStampedReference的类，它也是在JUC.atomic包下。
```java
public class ABAResolve {

    private static AtomicStampedReference<Integer> stampedReference = new AtomicStampedReference<>(100,1);

    public static void main(String[] args) {

        new Thread(()->{
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t 第1次版本号"+stamp+"\t值是"+stampedReference.getReference());
            // 睡眠1s让线程2获取值和版本号
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            stampedReference.compareAndSet(100,101,stampedReference.getStamp(),stampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 第2次版本号"+stampedReference.getStamp()+"\t值是"+stampedReference.getReference());
            stampedReference.compareAndSet(101,100,stampedReference.getStamp(),stampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 第3次版本号"+stampedReference.getStamp()+"\t值是"+stampedReference.getReference());
        },"thread1").start();

        new Thread(()->{
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t 第1次版本号"+stamp+"\t值是"+stampedReference.getReference());
            //保证线程1完成1次ABA
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean result = stampedReference.compareAndSet(100, 2019, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName()+"\t 修改成功否"+result+"\t最新版本号"+stampedReference.getStamp());
            System.out.println("最新的值\t"+stampedReference.getReference());
        },"thread2").start();

    }
}
```
运行结果为：
```
thread1	 第1次版本号1	值是100
thread2	 第1次版本号1	值是100
thread1	 第2次版本号2	值是101
thread1	 第3次版本号3	值是100
thread2	 修改成功否false	最新版本号3
最新的值	100
```
至此ABA问题解决。

## 代码
本文所涉及的所有代码都在我的GitHub上：https://github.com/dave0824/jmm

## 推荐阅读
1. [程序员应该吃透的集合List](https://blog.csdn.net/qq_41950229/article/details/105689062)
2. [Java集合之并发容器](https://blog.csdn.net/qq_41950229/article/details/102174177)
3. [Java集合详解之Map](https://blog.csdn.net/qq_41950229/article/details/102168779)

## 参考

1. 《深入理解Java虚拟机》
2. [volatile关键字之全面深度剖析](https://blog.csdn.net/qq_41950229/article/details/102332857)
3. [原来 CPU 为程序性能优化做了这么多](https://baijiahao.baidu.com/s?id=1662492394540513173&wfr=spider&for=pc)






