# Java多线程基础

多线程，是什么？这个我们在操作系统中已经有很详细的描述了。[链接](https://star.rainbowsea.top/blog/14)

那我们就直接进入正题

1. ### 线程的生命周期

   这个我们在操作系统中的进程中说过，就不再赘述了。![](https://p.pstatp.com/origin/137ee0002505e716276a3)

2. ### Java中创建多线程的两种常见方式

   1. ##### 继承Thread类

      ```java
      public class Test {
          public static void main(String[] args) {
              Ticket ticketA = new Ticket();
              ticketA.setName("A");
              Ticket ticketB = new Ticket();
              ticketB.setName("B");
              ticketA.start();
              ticketB.start();
          }
      }
      
      class Ticket extends Thread{
          private static int ticket = 50;
      
          private void sale(){
              while (ticket > 0){
                  try {
                      Thread.sleep(10);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread().getName()+": "+ticket--);
              }
          }
      
          @Override
          public void run() {
              this.sale();
          }
      }
      ```

      需要重写run方法，启动时需要使用start方法。

   2. ##### 实现Runnable接口

      ```java
      public class Test1 {
          public static void main(String[] args) {
              Ticket1 ticket1 = new Ticket1();
              new Thread(ticket1,"A").start();
              new Thread(ticket1,"B").start();
          }
      }
      
      class Ticket1 implements Runnable{
          private int ticket = 50;
      
          private void sale(){
              while (ticket > 0){
                  try {
                      Thread.sleep(10);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread().getName()+": "+ticket--);
              }
          }
      
          @Override
          public void run() {
              this.sale();
          }
      }
      ```

3. ### 线程安全问题

   上面的代码确实是多线程，可是他们的执行结果却是乱七八糟的，就是因为当一张票已经减掉，可是还没打印的时候，线程切换，就出现了不可预期的错误。那我们应该怎样去解决呢？

   1. ##### 同步代码块

      依旧是卖票。

      ```java
      public class Test1 {
          public static void main(String[] args) {
              Ticket1 ticket1 = new Ticket1();
              new Thread(ticket1,"A").start();
              new Thread(ticket1,"B").start();
          }
      }
      
      class Ticket1 implements Runnable{
          private int ticket = 100;
      
          private void sale(){
              while (true){
                  synchronized (this){
                      if (ticket>0){
                          try {
                              Thread.sleep(10);
                          } catch (InterruptedException e) {
                              e.printStackTrace();
                          }
                          System.out.println(Thread.currentThread().getName()+"卖出一张票，票号为"+ticket--);
                      }else break;
                  }
              }
          }
      
          @Override
          public void run() {
              this.sale();
          }
      }
      ```

   2. 同步方法

      ```java
      public class Test1 {
          public static void main(String[] args) {
              Ticket1 ticket1 = new Ticket1();
              new Thread(ticket1,"A").start();
              new Thread(ticket1,"B").start();
          }
      }
      
      class Ticket1 implements Runnable{
          private int ticket = 100;
      
          private synchronized void sale(){
              while (true){
                      if (ticket>0){
                          try {
                              Thread.sleep(10);
                          } catch (InterruptedException e) {
                              e.printStackTrace();
                          }
                          System.out.println(Thread.currentThread().getName()+"卖出一张票，票号为"+ticket--);
                      }else break;
              }
          }
      
          @Override
          public void run() {
              this.sale();
          }
      }
      ```

   3. 其他的方法咱们先按下不表，后续的章节中会学到

4. ### 基础方法

   **Thread：**

   - currentThread()：返回当前正在执行的线程对象的引用。
   - sleep（）:抱着锁睡大觉。
   - yield（）：让出处理器（孔融让梨）。

   **Object：**

   - notify（）：唤醒单个线程
   - notifyAll（）：唤醒所有线程
   - wait（）：让出锁，进入阻塞状态。（等待唤醒）

5. 