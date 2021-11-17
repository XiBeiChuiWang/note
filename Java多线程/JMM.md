# Volatile/CAS/自旋锁

1. ###  请你谈谈你对Volatile的理解 

   Volatile是Java虚拟机提供轻量级的同步机制 

   - 保证可见性
   - 不保证原子性
   - 禁止指令重排

2. ### 什么是JMM

   JMM定义了Java 虚拟机(JVM)在计算机内存(RAM)中的工作方式。JVM是整个计算机虚拟模型，所以JMM是隶属于JVM的。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。

   ![](https://pic.imgdb.cn/item/60289d32d2a061fec7b57bf2.png)

   关于主内存与工作内存之间的交互协议，即一个变量如何从主内存拷贝到工作内存。如何从工作内存同步到主内存中的实现细节。java内存模型定义了8种操作来完成。**这8种操作每一种都是原子操作。8种操作如下：**

   - lock(锁定)：作用于主内存，它把一个变量标记为一条线程独占状态；
   - read(读取)：作用于主内存，它把变量值从主内存传送到线程的工作内存中，以便随后的load动作使用；
   - load(载入)：作用于工作内存，它把read操作的值放入工作内存中的变量副本中；
   - use(使用)：作用于工作内存，它把工作内存中的值传递给执行引擎，每当虚拟机遇到一个需要使用这个变量的指令时候，将会执行这个动作；
   - assign(赋值)：作用于工作内存，它把从执行引擎获取的值赋值给工作内存中的变量，每当虚拟机遇到一个给变量赋值的指令时候，执行该操作；
   - store(存储)：作用于工作内存，它把工作内存中的一个变量传送给主内存中，以备随后的write操作使用；
   - write(写入)：作用于主内存，它把store传送值放到主内存中的变量中。
   - unlock(解锁)：作用于主内存，它将一个处于锁定状态的变量释放出来，释放后的变量才能够被其他线程锁定；

3. ### 理解Volatile

   1. **保证可见性**
   
      ```java
      public class VolatileTest1 {
          static int num = 0;
          public static void main(String[] args) {
                  new Thread(()->{
                      while (num == 0){}
                  }).start();
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              num++;
              System.out.println(num);
          }
      }
      ```
   
      执行结果
   
      ![](C:\Users\wwwe\AppData\Roaming\Typora\typora-user-images\1613278205025.png)
   
      num确实变为1，但是程序没有结束。
   
      这是因为num并没有重新写入内存中。
   
   2. **不保证原子性**
   
      ```java
      public class VolatileTest2 {
          static volatile int num = 0;
          static void add(){
              num++;
          }
          public static void main(String[] args) {
              for (int i = 1;i <= 20;i++){
                  new Thread(()->{
                      for (int j = 1;j <= 1000;j++){
                          add();
                      }
                  }).start();
              }
      
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
      
              System.out.println(num);
          }
      }
      ```
   
      输出结果不是20000
   
      **解决方法**
   
      1. 把add方法改成一个同步方法。
   
      2. 加lock锁
   
      3. 还可以用CAS算法去解决。
   
      4. 采用原子类
   
         ![](https://pic.imgdb.cn/item/6028b4fdd2a061fec7c42738.jpg)
   
         ```
         public class VolatileTest2 {
             static AtomicInteger num = new AtomicInteger(0);
             static void add(){
                 num.getAndIncrement();
             }
             public static void main(String[] args) {
                 for (int i = 1;i <= 20;i++){
                     new Thread(()->{
                         for (int j = 1;j <= 1000;j++){
                             add();
                         }
                     }).start();
                 }
         
                 try {
                     TimeUnit.SECONDS.sleep(1);
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
         
                 System.out.println(num);
             }
         }
         ```
   
   3. **禁止指令重排**
   
      理解就好
   
4. ### 深入理解CAS

   ```java
   public class CASTest2 {
       public static void main(String[] args) {
           AtomicInteger atomicInteger = new AtomicInteger(2020);
   
           new Thread(()->{
               System.out.println(atomicInteger.compareAndSet(2020, 2021));//true
               System.out.println(atomicInteger.compareAndSet(2021, 2020));//true
           },"A").start();
   
           new Thread(()->{
               try {
                   TimeUnit.SECONDS.sleep(1);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               System.out.println(atomicInteger.compareAndSet(2020, 2021));//true
           },"B").start();
       }
   }
   ```

   B线程compareAndSet成功了，貌似没啥问题，可是这个2020已经不是原来的2020了，他已经被A线程修改过了。

   这就是传说中的**ABA问题**，怎样解决呢

   ```java
   public class CASTest1 {
       public static void main(String[] args) {
           //正常在开发中都是比较对象
           AtomicStampedReference<Integer> integerAtomicStampedReference = new AtomicStampedReference<Integer>(1,1);
   
           new Thread(()->{
               System.out.println(integerAtomicStampedReference.getStamp()+"  A1");
               try {
                   TimeUnit.SECONDS.sleep(1);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               System.out.println(integerAtomicStampedReference.compareAndSet(1, 2, integerAtomicStampedReference.getStamp(), integerAtomicStampedReference.getStamp() + 1));
               System.out.println(integerAtomicStampedReference.compareAndSet(2, 1, integerAtomicStampedReference.getStamp(), integerAtomicStampedReference.getStamp() + 1));
               System.out.println(integerAtomicStampedReference.getStamp()+"  A2");
           },"A").start();
   
           new Thread(()->{
               try {
                   TimeUnit.SECONDS.sleep(2);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               System.out.println(integerAtomicStampedReference.getStamp()+"  B1");
               System.out.println(integerAtomicStampedReference.compareAndSet(1, 3, 1, 2));
               System.out.println(integerAtomicStampedReference.getStamp()+"  B2");
           },"B").start();
       }
   }
   ```

   这样就没问题了，我们使用了时间戳去处理

   **大坑：**

   ![](https://pic.imgdb.cn/item/6028e41fd2a061fec7d9f3a0.jpg)

5. ### 自旋锁

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
   class MyLockTest{
       public static void main(String[] args) throws InterruptedException {
           MyLock myLock = new MyLock();
   
           new Thread(()->{
               myLock.myLock();
               System.out.println("A锁");
               try {
                   System.out.println("A操作");
               } finally {
                   System.out.println("A解锁");
                   myLock.myUnLock();
               }
   
           },"A").start();
   
   //        TimeUnit.SECONDS.sleep(1);
           new Thread(()->{
               myLock.myLock();
               System.out.println("B锁");
               try {
                   System.out.println("B操作");
               } finally {
                   System.out.println("B解锁");
                   myLock.myUnLock();
               }
           }).start();
   
       }
   }
   ```
   
   这就是我们自己实现的自旋锁，自旋锁的应用很多。
   
   
   
   

