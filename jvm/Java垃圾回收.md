# Java垃圾回收（一）

1. ### 垃圾回收概述

   1. ##### 垃圾是什么？

      奥，我就是垃圾。。。（爬）

      在Java中，垃圾就是在运行程序中没有任何指针指向的对象。

   2. ##### 为什么需要GC？

      答：你家垃圾多了不得打扫嘛。（下一个）

   3. ##### 早期的垃圾回收

      c/c++人工收集。

      现代语言一般都是有自动垃圾回收的。

2. ### 垃圾回收算法

   **垃圾回收大体可以分为两个阶段——标记阶段以及清除阶段**

   1. ##### 标记阶段

      - **引用计数算法**

        很简单。每一个对象都有一个引用计数器属性，记录对象被引用的情况。

        **优点：**

        - 实现简单，垃圾对象便于辨识，判定效率高。

        **缺点：**

        - 需要单独的字段存储计数器，这样的做法增加了存储空间的开销。

        - 每次赋值都需要更新计数器，并伴随着加法减法操作，增加了时间开销。

        - 致命缺陷，无法处理循环依赖问题

          ![](https://pic.imgdb.cn/item/601eb0dc3ffa7d37b399d708.png)

      - **可达性分析算法**

        ***基本思路：***
        
        - 以根节点集合为起始点，按照从上至下的方式搜索被根对象集合所连接的目标对象是否可达
        - 使用可达性分析算法后，内存中的存活对象都会被根对象集合直接或者间接的连接，搜索时走过的路径称为**引用链**。
        - 如果目标对象不可达，则意味着其已死亡，可以标记为垃圾对象。
        
        ***GC Roots包含以下几类元素***
        
        1. 在虚拟机栈中的引用的对象。
        2. 本地方法栈中引用的对象。
        3. 方法区中类静态属性引用的对象。
        4. 方法区中常量引用的对象。
        5. 所有被同步锁synchronized持有的对象。
        6. Java虚拟机内部的引用。

   2. ##### 清除阶段

      - 标记-清除算法

        ![](https://pic.imgdb.cn/item/6020a0cb3ffa7d37b3688359.png)

      - 标记-整理算法

        ![](https://pic.imgdb.cn/item/6020a1233ffa7d37b368b0b1.png)

      - 复制算法

        ![](https://pic.imgdb.cn/item/6020a1103ffa7d37b368a94a.png)

      |   算法    |                             缺点                             |                             优点                             |
      | :-------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
      | 标记-清除 | 效率一般<br />运行GC时需要暂停用户程序<br />会产生内存碎片，需要维护一个空闲列表 |                             ————                             |
      | 标记整理  | 从效率上说，速度比其他二者都要慢<br />移动对象时，需要调整引用的地址。<br />移动过程中，需要暂停应用程序：STW | 消除了标记-清除算法内存区域分散的缺点。<br />消除了复制算法中，内存减半的高额代价 |
      |   复制    |      需要两倍的空间<br />移动对象时，需要调整引用的地址      |           实现简单，运行高效<br />保证空间的连续性           |

   3. ##### 对象的finalization机制

      永远不要主动去调用某个对象的finalization（）方法，应该将其交给垃圾回收机制调用。理由：

      1. 在finalization（）时，对象可能复活
      2. finalization（）的执行时间没有保证

      由于finalization（）存在，对象处于3种可能的状态：

      1. 可触及的：可达对象
      2. 可复活的：对象不可达。但是可能在finalization（）中复活。
      3. 不可触及的：finalization（）被调用，且没有复活。

      finalization（）只会被调用一次

      ![](https://pic.imgdb.cn/item/6020a3393ffa7d37b369b2b8.png)

      ```java
      public class GCTest1 {
          private static GCTest1 gcTest1;
      
          @Override
          protected void finalize() throws Throwable {
              super.finalize();
              gcTest1 = this;
          }
      
          public static void main(String[] args) {
             gcTest1 = new GCTest1();
             //第一次垃圾回收
              System.out.println("第一次垃圾回收");
             gcTest1 = null;
             System.gc();
              try {
                  Thread.sleep(2000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
      
              if (gcTest1 == null){
                  System.out.println("gcTest1死亡");
              }else {
                  System.out.println("gcTest1自救成功");
              }
      
              //第二次垃圾回收
              System.out.println("第二次垃圾回收");
              gcTest1 = null;
              System.gc();
              try {
                  Thread.sleep(2000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
      
              if (gcTest1 == null){
                  System.out.println("gcTest1死亡");
              }else {
                  System.out.println("gcTest1自救成功");
              }
          }
      }
      
      ```

      ![](https://pic.imgdb.cn/item/602fc0efe7e43a13c69c7337.jpg)

      

3. ### 垃圾回收的相关概念

   1. #### 内存溢出与内存泄漏

      - 内存溢出：没有空闲内存，并且垃圾回收也无法提供更多内存（可能是出现了内存泄漏，也有可能是单纯的内存不足）
      - 内存泄漏：严格来讲，对象不会被程序再用到，但是GC觉着不能回收他们，叫做内存泄漏。
        举例：
        - 单例模式中持有外部对象的引用。
        - 一些需要close的对象未close（数据库连接，io等）

   2. #### Stop The World

      简称STW，在GC时，整个应用程序线程都会停顿。

      原因：如果在GC时对象引用关系不停的变化，那么准确性不可以保证。

      所有的垃圾回收器都不可避免地会出现STW

      在调用`System.gc();`时会触发STW

   3. #### 安全点与安全区域

      1. 安全点

         程序执行时并非在任何地方都能停下，只有在特定的地方才能停下来开始GC

         两种方式：

         - 抢先式中断（不再使用了）

           首先中断所有的线程，如果有线程不在安全点，就恢复他，让其跑到安全点再中断。

         - 主动式中断

           各个线程跑到安全点后主动轮询是否有中断信号，如果有，则自己中断挂起。

      2. 安全区域

         线程处于Sleep或者blocked状态时，线程无法响应中断请求

         为了解决，引入安全区域：

         在一段代码中，引用关系不会发生变化，在这期间任何位置进行GC都是安全的。

   4. #### 引用

      1. ##### 强引用（Strong Reference）

         代码中最普遍的引用赋值，类似于`Object obj = new Object()`，只要强引用还在，这个对象永远不会被回收掉。

         ```java
         public class 强引用 {
             public static void main(String[] args) {
                 Integer integer = new Integer(233);
                 try {
                     byte[] bytes = new byte[1024 * 1024 * 7];
                 } finally {
                     System.out.println(integer);
                 }
             }
         }
         ```

         ![](https://pic.imgdb.cn/item/602fc7b8e7e43a13c69f82f0.png)

         看到了没，都OOM了，还在！这好像都不能用至死方休来形容了吧。

      2. ##### 软引用（Soft Reference）

         讲过强引用，大家肯定对接下来这几个玩意儿充满疑惑，没关系，讲一下概念大家就懂了.

         在系统即将发生内存溢出时，会将这些对象进行回收，如果一直不发生溢出，就一直不会被回收

         那么怎样去实现呢？

         ![](https://pic.imgdb.cn/item/602fc924e7e43a13c6a034c6.png)

         上代码

         ```java
         public class 软引用 {
             public static void main(String[] args) {
                 SoftReference<Integer> integerSoftReference = new SoftReference<>(new Integer(233));
         //        等同于以下写法
         //        Integer integer = new Integer(233);
         //        SoftReference<Integer> integerSoftReference = new SoftReference<>(integer);
         //        integer = null;
         
                 System.out.println(integerSoftReference.get());//233
         
                 System.gc();
                 System.out.println("GC");
                 System.out.println(integerSoftReference.get());//233
         
                 try {
                     byte[] bytes = new byte[1024 * 1024 * 7];
                 }finally {
                     System.out.println("OOM");
                     System.out.println(integerSoftReference.get());//null
                 }
             }
         }
         ```

         ![](https://pic.imgdb.cn/item/602fcb41e7e43a13c6a11f71.png)

         可以看到在没有发生内存溢出时，即使进行GC也不会被回收，只有在内存即将溢出时才会被回收。

      3. ##### 弱引用

         有了上面的基础就比较简单了

         ```java
         public class 弱引用 {
             public static void main(String[] args) {
                 WeakReference<Integer> integerWeakReference = new WeakReference<>(new Integer(233));
         
                 System.out.println(integerWeakReference.get());//233
                 System.out.println("GC");
                 System.gc();
                 System.out.println(integerWeakReference.get());//null
             }
         }
         ```

         ![](https://pic.imgdb.cn/item/602fcca0e7e43a13c6a1b096.png)

         在GC时，弱引用就会被回收

      4. ##### 虚引用

          虚引用，正如其名，对一个对象而言，这个引用形同虚设，有和没有一样。 

         > 如果一个对象与GC Roots之间仅存在虚引用，则称这个对象为`虚可达（phantom reachable）`对象。

         当试图通过虚引用的get()方法取得强引用时，总是会返回null，并且，虚引用必须和引用队列一起使用。既然这么虚，那么它出现的意义何在？？

         别慌别慌，自然有它的用处。它的作用在于跟踪垃圾回收过程，在对象被收集器回收时收到一个系统通知。 当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在垃圾回收后，将这个虚引用加入引用队列，在其关联的虚引用出队前，不会彻底销毁该对象。 所以可以通过检查引用队列中是否有相应的虚引用来判断对象是否已经被回收了。

         如果一个对象没有强引用和软引用，对于垃圾回收器而言便是可以被清除的，在清除之前，会调用其finalize方法，如果一个对象已经被调用过finalize方法但是还没有被释放，它就变成了一个虚可达对象。

         与软引用和弱引用不同，显式使用虚引用可以阻止对象被清除，只有在程序中显式或者隐式移除这个虚引用时，这个已经执行过finalize方法的对象才会被清除。想要显式的移除虚引用的话，只需要将其从引用队列中取出然后扔掉（置为null）即可。

         ```java
         public class 虚引用 {
             static ReferenceQueue<虚引用> integerReferenceQueue = new ReferenceQueue<>();
             static 虚引用 xu = null;
         
             @Override
             protected void finalize() throws Throwable {
                 super.finalize();
                 xu = this;
             }
         
             @Override
             public String toString() {
                 return "虚引用{}";
             }
         }
         class a{
             public static void main(String[] args) throws InterruptedException {
                 Thread t = new Thread(()->{
                     while (true) {
         
                         Reference<? extends 虚引用> poll = 虚引用.integerReferenceQueue.poll();
                         if (poll != null) {
                             System.out.println("--- 虚引用对象被jvm回收了 ---- " + poll);
                             System.out.println("--- 回收对象 ---- " + poll.get());
                             System.out.println(poll);
                         }
         
                     }
                 });
                 t.setDaemon(true);
                 t.start();
         
                 xu = new 虚引用();
                 PhantomReference<虚引用> integerPhantomReference = new PhantomReference<>(xu, 虚引用.integerReferenceQueue);
                 System.out.println(integerPhantomReference.get());//null
         
                 xu = null;
                 System.gc();
                 System.out.println("第一次GC");
                 try {
                     TimeUnit.SECONDS.sleep(1);
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
                 if (xu == null){
                     System.out.println("对象死亡");
                 }else {
                     System.out.println("对象存活");
                 }
         
                 xu = null;
                 System.gc();
                 System.out.println("第二次GC");
                 try {
                     TimeUnit.SECONDS.sleep(1);
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
                 if (xu == null){
                     System.out.println("对象死亡");
                 }else {
                     System.out.println("对象存活");
                 }
         
             }
         }
         
         ```

         ![](https://pic.imgdb.cn/item/602fd970e7e43a13c6a6fdca.png)
