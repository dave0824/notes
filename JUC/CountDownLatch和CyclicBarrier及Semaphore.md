## 前言
本文为对CountDownLatch、CyclicBarrier、Semaphore的整理使用

## CountDownLatch
CountDownLatch类位于java.util.concurrent包下，CountDownLatch的作用是让一个或多个线程阻塞直到另外一些线程完成后才被唤醒，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

CountDownLatch主要有两个方法,当一个或多个线程调用await方法时,调用线程会被阻塞。其他线程调用countDown方法计数器减1(调用countDown方法时线程不会阻塞),当计数器的值变为0,此时调用await方法被阻塞的线程会被唤醒,继续执行。
```java
public void await() throws InterruptedException { };   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public void countDown() { };  //将count值减1
```
举个简单的例子：班长要等到其它66个同学上完晚自习才能离开教室锁门,代码实现：

```java
public class CountDownLatchDemo {
    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(66);
        for (int i = 1; i <= 66; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t" + "上完自习");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        try {
            // 等待其它66个线程执行完毕
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "\t班长锁门离开教室");
    }
}
```
## CyclicBarrier

CyclicBarrier的字面意思是可循环(Cyclic),使用的屏障(barrier)。它要做的事情是,让一组线程到达一个屏障(也可以叫做同步点)时被阻塞,直到最后一个线程到达屏障时,屏障才会开门,所有被屏障拦截的线程才会继续干活,线程进入屏障通过CyclicBarrier的await()方法。CyclicBarrier刚好和CountDownLatch相反，一个是增，一个是减。

CyclicBarrier类位于java.util.concurrent包下，CyclicBarrier提供2个构造器：
```java
public CyclicBarrier(int parties, Runnable barrierAction) {}
 
public CyclicBarrier(int parties) {}
```
参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。
然后CyclicBarrier中最重要的方法就是await方法，它有2个重载版本：
```java
public int await() throws InterruptedException, BrokenBarrierException { };//挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };//线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。
```
下面举个集齐七颗龙珠召唤神龙的例子：
```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier=new CyclicBarrier(7,()->{
            System.out.println("召唤神龙");
        });

        for (int i = 1; i <=7; i++) {
            final int temp = i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t 收集到第"+ temp +"颗龙珠");
                try {
                    cyclicBarrier.await(); // 等待其它线程收集龙珠
                    System.out.println(Thread.currentThread().getName()+"\t 召唤神龙后，释放第"+ temp +"颗龙珠");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

## Semaphore
Semaphore翻译成字面意思为 信号量，信号量的主要用户两个目的,一个是用于多个共享资源的相互排斥使用,另一个用于并发资源数的控制。Semaphore可以控制同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。其实就是操作系统中进程共享资源访问使用Semaphore信号量控制的java实现。

Semaphore类位于java.util.concurrent包下，我们看下它的常用方法：
```java
public class Semaphore implements java.io.Serializable {
	 public Semaphore(int permits) { // permits表示许可数目，即同时可以允许多少线程进行访问
        sync = new NonfairSync(permits);
    }
	public Semaphore(int permits, boolean fair) { // fair参数表示是否为公平锁
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }

	public void acquire() // 获取一个许可，获取不到将被阻塞，直至获取到为止。
	public void acquire(int permits) // 获取permits个许可,获取不到将被阻塞，直至获取到为止。
	public void release() //释放一个许可
	public void release(int permits) //释放permits个许可
	
	
	public boolean tryAcquire() // 尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
	public boolean tryAcquire(long timeout, TimeUnit unit) // 尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
	public boolean tryAcquire(int permits) // 尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
	public boolean tryAcquire(int permits, long timeout, TimeUnit unit) // 尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false

	public int availablePermits() // 获取可用的许可数目。
}
```
下面举一个抢车位的例子：
```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        //模拟3个停车位
        Semaphore semaphore = new Semaphore(3);
        //模拟6部汽车
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                try {
                    //抢到资源
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "\t抢到车位");
                    try {
                        TimeUnit.SECONDS.sleep(3);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "\t 停3秒离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //释放资源
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

## 代码
本文所涉及的所有代码都在我的GitHub上：[代码](https://github.com/dave0824/jmm)


