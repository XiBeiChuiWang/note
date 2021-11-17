# Java锁机制

1. ### 首先什么是锁呢？

   其实就是一个标志位（保存在对象头中）

   ![](https://pic.imgdb.cn/item/6031f9685f4313ce25fd6d23.jpg)

   那么为什么需要锁呢？

   其实很简单，跟操作系统中的临界资源一样，存在并发安全问题。

   我们Java传统的synchronized就是基于操作系统中对线程的阻塞挂起以及唤醒实现的，这种方式需要切换用户态及内核态，执行速度很慢，对应上表中的重量级锁。

2. ### 锁的升级

   jdk6.0之后，jdk内部对synchronized做了很多优化，不会一上来就给你整个monitor(操作系统中的概念)，而是会给你先来一个偏向锁。(使用重量级锁很耗费时间)

   锁有上表中4种状态：

   - **无锁：**当我们觉着线程不存在竞争问题时，就不需要加锁。

   - **偏向锁：**正如他的名字一般，这个锁是偏心的，他会偏向某个线程，

      作用：偏向锁是为了消除==无竞争情况==下的同步原语，进一步提升程序性能。 

     例：

      当线程1访问代码块并获取锁对象时，会在java对象头和栈帧中记录偏向的锁的threadID，因为**偏向锁不会主动释放锁**，因此以后线程1再次获取锁的时候，需要**比较当前线程的threadID和Java对象头中的threadID是否一致**，如果一致（还是线程1获取锁对象），则无需使用CAS来加锁、解锁；如果不一致（其他线程，如线程2要竞争锁对象，而偏向锁不会主动释放因此还是存储的线程1的threadID），那么需要**查看Java对象头中记录的线程1是否存活**，如果没有存活，那么锁对象被重置为无锁状态，其它线程（线程2）可以竞争将其设置为偏向锁；如果存活，那么立刻**查找该线程（线程1）的栈帧信息，如果还是需要继续持有这个锁对象**，那么暂停当前线程1，撤销偏向锁，升级为轻量级锁，如果线程1 不再使用该锁对象，那么将锁对象状态设为无锁状态，重新偏向新的线程。 

   - **轻量级锁：**当出现了第二个线程后，我们的偏向锁就开始招架不住了，那这个时候该怎么办，假设，我们此时需要此资源的线程不多，只有三四个，难道要直接使用synchronized吗？不，那样太耗费时间了，我们应该使用自旋锁，当一个线程持有锁后，其他的想要临界资源的线程就在原地自旋，等待持有锁的线程释放锁。

     - 线程与轻量级锁的绑定：

       线程1获取轻量级锁时会先把锁对象的**对象头MarkWord复制一份到线程1的栈帧中创建的用于存储锁记录的空间**（称为DisplacedMarkWord），然后**使用CAS把对象头中的内容替换为线程1存储的锁记录（**DisplacedMarkWord**）的地址**； 

     那我们就让他一直自旋吗？当然不行，自旋是要耗费CPU资源的，当我们自旋的线程达到CPU核心数的一半，或者是自旋次数达到10次，我们就会将锁升级为重量级锁。
     
     ```java
     public class JolTest {
         public static void main(String[] args) throws InterruptedException {
             A a1 = new A();
             System.out.println("未启用偏向锁：" + ClassLayout.parseInstance(a1).toPrintable());
     
             TimeUnit.SECONDS.sleep(5);
             A a2 = new A();
             System.out.println("启用偏向锁(101)：" + ClassLayout.parseInstance(a2).toPrintable());
     
     
             for (int i = 1;i <= 2;i++) {
                 int finalI = i;
                 new Thread(() -> {
                     synchronized (a2) {
                         if (finalI == 2){
                             System.out.println("偏向锁--->轻量级锁(00)" + ClassLayout.parseInstance(a2).toPrintable());
                         }
                     }
     
                 }).start();
             }
     
             TimeUnit.SECONDS.sleep(2);
             for (int i = 1;i <= 5;i++) {
                 int finalI = i;
                 new Thread(()->{
                     synchronized (a2) {
                         if (finalI == 5) {
                             System.out.println("轻量级锁--->重量级锁(10)" + ClassLayout.parseInstance(a2).toPrintable());
                         }
                     }
                 }).start();
             }
     
     
         }
     }
     
     class A {
         private int a = 3;
     }
     ```
     
     ![](https://pic.imgdb.cn/item/603246645f4313ce251c3b65.jpg)

   ​	

    **注意：**为了避免无用的自旋，轻量级锁一旦膨胀为重量级锁就不会再降级为轻量级锁了；偏向锁升级为轻量级锁也不能再降级为偏向锁。一句话就是锁可以升级不可以降级，但是偏向锁状态可以被重置为无锁状态。 

3. ### 锁粗化

   按理来说，同步块的作用范围应该尽可能小，仅在共享数据的实际作用域中才进行同步，这样做的目的是为了使需要同步的操作数量尽可能缩小，缩短阻塞时间，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。 
   但是加锁解锁也需要消耗资源，如果存在一系列的连续加锁解锁操作，可能会导致不必要的性能损耗。 
   **锁粗化就是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁，避免频繁的加锁解锁操作。** 

4. ### 锁消除

    Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，经过逃逸分析，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间 

5. ### 自旋锁

    普通自旋锁，问题会出在他是很浪费CPU资源的（CPU空转）。

    ```java
    public class MyLock {
        private AtomicBoolean atomicBoolean = new AtomicBoolean();
    
        public void myLock(){
            while (!atomicBoolean.compareAndSet(false,true)){//若为false则一直自旋,就是阻塞
               
            }
        }
    
        public void myUnLock(){
            atomicBoolean.compareAndSet(true,false);
        }
    }
    ```

    加一个yield方法

    ```java
    public class MyLock {
        private AtomicBoolean atomicBoolean = new AtomicBoolean();
    
        public void myLock(){
            while (!atomicBoolean.compareAndSet(false,true)){//若为false则一直自旋,就是阻塞
                Thread.yield();
            }
        }
    
        public void myUnLock(){
            atomicBoolean.compareAndSet(true,false);
        }
    }
    ```

    确确实实是会提升性能的，但是假设我们线程很多，yield方法就不行了

    因此我们可以想到使用这种方法

    ```java
    public class MyLock {
        private AtomicBoolean atomicBoolean = new AtomicBoolean();
    	private LinkedList linkedList= new LinkedList();
        public void myLock(){
            while (!atomicBoolean.compareAndSet(false,true)){//若为false则一直自旋,就是阻塞
                linkedList.addLast(currentThread)//将当前线程添加到队末
                LockSupport.park();//将当前线程阻塞
            }
        }
    
        public void myUnLock(){
            Thread t = linkedList.getLast()//获取队末线程
            LockSupport.unpark(t);//解锁队末线程
            atomicBoolean.compareAndSet(true,false);
        }
    }
    ```

    ok,那么有没有现成的类呢，答案是肯定的，那就是大名鼎鼎的AQS

     AQS（AbstractQueuedSynchronizer）本身是一个抽象类，主要的使用方法是继承它作为一个内部类，JDK中许多并发工具类的内部实现都依赖于AQS，如ReentrantLock, Semaphore, CountDownLatch等等。 

    **AQS类的属性**

    ![](https://pic.imgdb.cn/item/6033548e5f4313ce25928777.jpg)

    跟我们上面那个自旋锁的双端队列一样，head，tail 

    这个state对应了我们上面的布尔变量，只是这个是int

    而这个Node节点就封装了我们的线程