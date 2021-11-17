# CountDownLatch源码分析

我们原来看过CountDownLatch，再来复习一下，他是一个计数器，只有当计数器减为0才能执行下面的代码，用法特别简单。

### 用法

```java
public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(10);//创建一个计数器

        for (int i = 1;i <= 10;i++){
            int finalI = i;
            new Thread(()->{
                System.out.println(finalI + " Go Out");
                countDownLatch.countDown();//计数器减一
            },String.valueOf(finalI)).start();
        }

        countDownLatch.await();//只有计数器等于0之后，才会往下执行
        System.out.println("Close Door");
    }
}
```

### 源码分析

1. #### 概览

   ![](https://pic.imgdb.cn/item/6035123d5f4313ce256a079f.jpg)

   首先是一个内部类Sync，它继承了我们的AQS

   在我们的构造函数中，对其进行了实例化。

   ```java
   public CountDownLatch(int count) {
           if (count < 0) throw new IllegalArgumentException("count < 0");
           this.sync = new Sync(count);
       }
   
   Sync(int count) {
               setState(count);
           }
   ```

   从上面可知，我们定义了一个n的计数器，我们就会在底层的AQS中把他的State定义为n，每次我们执行完一个子线程的countDown方法后，n就减一。

   我们来看看它提供的方法

   ```java
      //只有子线程执行完，这个方法才会继续往下执行
      public void await() throws InterruptedException {
           sync.acquireSharedInterruptibly(1);
       } 
       //带有计时器的版本，在超出时间后，就继续向下执行，即使计数器没有减为1
       public boolean await(long timeout, TimeUnit unit)
           throws InterruptedException {
           return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
       }
       //计数器减一
       public void countDown() {
           sync.releaseShared(1);
       }
       //获取当前计数器的值
       public long getCount() {
           return sync.getCount();
       }
   ```

   可以看到方法特别简单，我们从最简单的说起。

2. #### getCount()

   ```java
    public long getCount() {
           return sync.getCount();
       }
       
       int getCount() {
               return getState();
           }
   ```

   ok，没啥说的，就是返回底层state的值

3. ###  await()

   ```java
   public void await() throws InterruptedException {
           sync.acquireSharedInterruptibly(1);
       }
   
   public final void acquireSharedInterruptibly(int arg)
               throws InterruptedException {
           if (Thread.interrupted())
               throw new InterruptedException();
       	//小于0
           if (tryAcquireShared(arg) < 0)
               //执行
               doAcquireSharedInterruptibly(arg);
       }
   //AQS提供的方法，需要子类去自己实现
   protected int tryAcquireShared(int arg) {
           throw new UnsupportedOperationException();
       }
   //实现类
   protected int tryAcquireShared(int acquires) {
       		//不为0，返回-1
               return (getState() == 0) ? 1 : -1;
           }
   
   private void doAcquireSharedInterruptibly(int arg)
           throws InterruptedException {
       	//老朋友了
           final Node node = addWaiter(Node.SHARED);
           boolean failed = true;
           try {
               //自旋
               for (;;) {
                   //前置节点为head
                   final Node p = node.predecessor();
                   if (p == head) {
                       //-1
                       int r = tryAcquireShared(arg);
                       //不执行
                       if (r >= 0) {
                           setHeadAndPropagate(node, r);
                           p.next = null; // help GC
                           failed = false;
                           return;
                       }
                   }
                   //阻塞，并修改了head的state值为-1
                   if (shouldParkAfterFailedAcquire(p, node) &&
                       //注意，方法阻塞在这一行
                       parkAndCheckInterrupt())
                       throw new InterruptedException();
               }
           } finally {
               if (failed)
                   cancelAcquire(node);
           }
       }
   //入队（不再赘述）
   private Node addWaiter(Node mode) {
       //创建节点
           Node node = new Node(Thread.currentThread(), mode);
           // Try the fast path of enq; backup to full enq on failure
           Node pred = tail;
           if (pred != null) {
               node.prev = pred;
               if (compareAndSetTail(pred, node)) {
                   pred.next = node;
                   return node;
               }
           }
       //初始化队列
           enq(node);
           return node;
       }
   ```

4. #### countDown（）

   我们分两种情况

   - 当计数器从n——>n-1(n != 1)

     ```java
     public void countDown() {
             sync.releaseShared(1);
         }
         
         public final boolean releaseShared(int arg) {
             //false，直接返回，完毕！！
             if (tryReleaseShared(arg)) {
                 doReleaseShared();
                 return true;
             }
            
             return false;
         }
         //跟上面的tryAcquireShared一样的，需要子类去实现
         protected boolean tryReleaseShared(int arg) {
             throw new UnsupportedOperationException();
         }
         //实现
         protected boolean tryReleaseShared(int releases) {
                 // Decrement count; signal when transition to zero
                 for (;;) {
                     int c = getState();
                     if (c == 0)
                         return false;
                     //不为0
                     int nextc = c-1;
                     if (compareAndSetState(c, nextc))
                         //返回false
                         return nextc == 0;
                 }
             }
         }
     ```

     太简单了。

   - 当计数器从1——>0

     ```java
     public void countDown() {
             sync.releaseShared(1);
         }
         
         public final boolean releaseShared(int arg) {
             //true
             if (tryReleaseShared(arg)) {
                 
                 doReleaseShared();
                 return true;
             }
            
             return false;
         }
         
         //实现
         protected boolean tryReleaseShared(int releases) {
                 // Decrement count; signal when transition to zero
                 for (;;) {
                     int c = getState();
                     if (c == 0)
                         return false;
                     //为0
                     int nextc = c-1;
                     if (compareAndSetState(c, nextc))
                         //返回true
                         return nextc == 0;
                 }
             }
         }
     
     private void doReleaseShared() {
             for (;;) {
                 Node h = head;
                 //符合
                 if (h != null && h != tail) {
                     //-1
                     int ws = h.waitStatus;
                     if (ws == Node.SIGNAL) {
                         head.state = 0
                         if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                             continue;            // loop to recheck cases
                         //恢复线程
                         unparkSuccessor(h);
                     }
                     else if (ws == 0 &&
                              !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                         continue;                // loop on failed CAS
                 }
                 if (h == head)                   // loop if head changed
                     break;
             }
         }
     
     //恢复后的线程
     private void doAcquireSharedInterruptibly(int arg)
             throws InterruptedException {
             final Node node = addWaiter(Node.SHARED);
             boolean failed = true;
             try {
                 for (;;) {
                     final Node p = node.predecessor();
                     if (p == head) {
                         //此时tryAcquireShared(arg)返回1
                         int r = tryAcquireShared(arg);
                         if (r >= 0) {
                             setHeadAndPropagate(node, r);
                             p.next = null; // help GC
                             failed = false;
                             //返回
                             return;
                         }
                     }
                     if (shouldParkAfterFailedAcquire(p, node) &&
                         //从这恢复
                         parkAndCheckInterrupt())
                         throw new InterruptedException();
                 }
             } finally {
                 if (failed)
                     cancelAcquire(node);
             }
         }
     
     private void setHeadAndPropagate(Node node, int propagate) {
             Node h = head; // Record old head for check below
         	//狸猫换太子
             setHead(node);
         	//符合
             if (propagate > 0 || h == null || h.waitStatus < 0 ||
                 (h = head) == null || h.waitStatus < 0) {
                 Node s = node.next;
                 if (s == null || s.isShared())
                     doReleaseShared();
             }
         }
     
     private void setHead(Node node) {
             head = node;
             node.thread = null;
             node.prev = null;
         }
     ```

     大体就是这，可能有些地方写的不好，欢迎指正！

   

