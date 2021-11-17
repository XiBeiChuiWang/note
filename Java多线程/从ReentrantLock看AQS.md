# 从ReentrantLock看AQS

1. ### 首先我们看ReentrantLock的构造方法，一个bool变量

   ```java
   public ReentrantLock(boolean fair) {
           sync = fair ? new FairSync() : new NonfairSync();
       }
   ```

   就是公平以及不公平的问题，一会儿会讲到。

2. ### lock方法（FairSync式）

   1. **当前只有一个线程时，我们可以想，AQS是为了提升效率，必不会有什么太难的操作。**

      ![](https://pic.imgdb.cn/item/603353765f4313ce2591ed2e.png)

      我们来进行分析

      ```java
      public final void acquire(int arg) {
      			//首先我们直接看tryAcquire(arg)，作用是尝试直接获取锁
              if (!tryAcquire(arg) && 
                  acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                  selfInterrupt();
       }
      
      protected final boolean tryAcquire(int acquires) {
      			//获得当前线程
                  final Thread current = Thread.currentThread();
                  获得状态值
                  int c = getState();
                  对锁的状态进行判断
                  if (c == 0) {
                  		//检查队列是否有其他节点（看下面代码）若没有元素则这为true
                      if (!hasQueuedPredecessors() &&
                      	//因此我们直接CAS修改状态位
                          compareAndSetState(0, acquires)) {
                          //设置当前线程
                          setExclusiveOwnerThread(current);
                          //一直返回执行用户自己编写的代码
                          return true;
                      }
                  }
                  else if (current == getExclusiveOwnerThread()) {
                      int nextc = c + acquires;
                      if (nextc < 0)
                          throw new Error("Maximum lock count exceeded");
                      setState(nextc);
                      return true;
                  }
                  return false;
              }
        }
        
        //检查队列是否有其他节点
       public final boolean hasQueuedPredecessors() {
              // The correctness of this depends on head being initialized
              // before tail and on head.next being accurate if the current
              // thread is first in queue.
              Node t = tail; // Read fields in reverse initialization order
              Node h = head;
              Node s;
              //若无元素，返回false
              return h != t &&
                  ((s = h.next) == null || s.thread != Thread.currentThread());
          } 
      ```

   2. **2个线程并发**

      ```java
      public final void acquire(int arg) {
      			//true
              if (!tryAcquire(arg) &&
              	//addWaiter(Node.EXCLUSIVE)作用是将当前线程封装为节点添加进阻塞队列
                  //acquireQueued(addWaiter(Node.EXCLUSIVE), arg))负责阻塞线程以及之后唤醒后的尝试获取锁
                  acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                  selfInterrupt();
          }
          
      protected final boolean tryAcquire(int acquires) {
                  final Thread current = Thread.currentThread();
                  int c = getState();
                  //不为0
                  if (c == 0) {
                      if (!hasQueuedPredecessors() &&
                          compareAndSetState(0, acquires)) {
                          setExclusiveOwnerThread(current);
                          return true;
                      }
                  }
                  else if (current == getExclusiveOwnerThread()) {
                      int nextc = c + acquires;
                      if (nextc < 0)
                          throw new Error("Maximum lock count exceeded");
                      setState(nextc);
                      return true;
                  }
                  //返回false
                  return false;
              }
          }
          
      private Node addWaiter(Node mode) {
      		//创建当前线程节点
              Node node = new Node(Thread.currentThread(), mode);
              // Try the fast path of enq; backup to full enq on failure
              //pred，tail均为null
              Node pred = tail;
              if (pred != null) {
                  node.prev = pred;
                  if (compareAndSetTail(pred, node)) {
                      pred.next = node;
                      return node;
                  }
              }
              //进入
              enq(node);
              return node;
          }    
        //负责初始化队列
        private Node enq(final Node node) {
        		//自旋锁
              for (;;) {
              	//1.t为null
              	//2.t不为null
                  Node t = tail;
                  if (t == null) { // Must initialize
                  	//1.CAS设置一个新的傀儡头节点
                      if (compareAndSetHead(new weijiedianNode()))
                      	//将tail也指向该傀儡节点
                          tail = head;
                  } else {
                  	将当前线程添加到尾节点
                      node.prev = t;
                      //CAS修改尾节点
                      if (compareAndSetTail(t, node)) {
                          t.next = node;
                          return t;
                      }
                  }
              }
          }
      
      final boolean acquireQueued(final Node node, int arg) {
              boolean failed = true;
              try {
                  boolean interrupted = false;
                  for (;;) {
                      //指向头节点
                      final Node p = node.predecessor();
                      //假设获取锁失败
                      //假设自旋第二次仍旧失败
                      if (p == head && tryAcquire(arg)) {
                          setHead(node);
                          p.next = null; // help GC
                          failed = false;
                          return interrupted;
                      }
                      //第一次返回false，自旋
                      //第二次返回true
                      if (shouldParkAfterFailedAcquire(p, node) &&
                          //阻塞当前线程
                          parkAndCheckInterrupt())
                          interrupted = true;
                  }
              } finally {
                  if (failed)
                      cancelAcquire(node);
              }
          }
      
       private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
           	//head.waitStatus默认为0
              int ws = pred.waitStatus;
           	//Node.SIGNAL == -1
           	//第二次直接返回true
              if (ws == Node.SIGNAL)
                  /*
                   * This node has already set status asking a release
                   * to signal it, so it can safely park.
                   */
                  return true;
              if (ws > 0) {
                  /*
                   * Predecessor was cancelled. Skip over predecessors and
                   * indicate retry.
                   */
                  do {
                      node.prev = pred = pred.prev;
                  } while (pred.waitStatus > 0);
                  pred.next = node;
              } else {
                  /*
                   * waitStatus must be 0 or PROPAGATE.  Indicate that we
                   * need a signal, but don't park yet.  Caller will need to
                   * retry to make sure it cannot acquire before parking.
                   */
                  //将head.waitStatus设为-1
                  compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
              }
              return false;
          }
      
       /** waitStatus value to indicate thread has cancelled */
              static final int CANCELLED =  1;
              /** waitStatus value to indicate successor's thread needs unparking */
              static final int SIGNAL    = -1;
              /** waitStatus value to indicate thread is waiting on condition */
              static final int CONDITION = -2;
              /**
               * waitStatus value to indicate the next acquireShared should
               * unconditionally propagate
               */
              static final int PROPAGATE = -3;
       //阻塞当前线程
      private final boolean parkAndCheckInterrupt() {
          	//阻塞线程，底层使用UnSafe类，线程会被阻塞在这一句，恢复线程后从这执行
              LockSupport.park(this);
              return Thread.interrupted();
          }
      ```

   3. **3个线程并发**

      ```java
      final boolean acquireQueued(final Node node, int arg) {
              boolean failed = true;
              try {
                  boolean interrupted = false;
                  for (;;) {
                      //指向t2
                      final Node p = node.predecessor();
                     
                      if (p == head && tryAcquire(arg)) {
                          setHead(node);
                          p.next = null; // help GC
                          failed = false;
                          return interrupted;
                      }
                      //第一次返回false，自旋
                      //第二次返回true
                      if (shouldParkAfterFailedAcquire(p, node) &&
                          //阻塞当前线程
                          parkAndCheckInterrupt())
                          interrupted = true;
                  }
              } finally {
                  if (failed)
                      cancelAcquire(node);
              }
          }
      
       private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
           	//t2.waitStatus默认为0
              int ws = pred.waitStatus;
           	//Node.SIGNAL == -1
           	//第二次直接返回true
              if (ws == Node.SIGNAL)
                  /*
                   * This node has already set status asking a release
                   * to signal it, so it can safely park.
                   */
                  return true;
              if (ws > 0) {
                  /*
                   * Predecessor was cancelled. Skip over predecessors and
                   * indicate retry.
                   */
                  do {
                      node.prev = pred = pred.prev;
                  } while (pred.waitStatus > 0);
                  pred.next = node;
              } else {
                  /*
                   * waitStatus must be 0 or PROPAGATE.  Indicate that we
                   * need a signal, but don't park yet.  Caller will need to
                   * retry to make sure it cannot acquire before parking.
                   */
                  //t2.waitStatus设为-1
                  compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
              }
              return false;
          }
      ```

      可以看到，如果t3之前还有t1，t2，那么t3应该直接加入队列，然后阻塞

3. ### unLock（FairSync式）

   ```java
   public void unlock() {
           sync.release(1);
       }
       
   public final boolean release(int arg) {
       //恢复锁的状态位（释放锁）
           if (tryRelease(arg)) {
               Node h = head;
               if (h != null && h.waitStatus != 0)
                   //解锁第一个排队的线程
                   unparkSuccessor(h);
               return true;
           }
           return false;
       } 
   //恢复锁的状态位（释放锁）
   protected final boolean tryRelease(int releases) {
               int c = getState() - releases;
               if (Thread.currentThread() != getExclusiveOwnerThread())
                   throw new IllegalMonitorStateException();
               boolean free = false;
               if (c == 0) {
                   free = true;
                   setExclusiveOwnerThread(null);
               }
               setState(c);
               return free;
           }
   //解锁第一个排队的线程
   private void unparkSuccessor(Node node) {
           /*
            * If status is negative (i.e., possibly needing signal) try
            * to clear in anticipation of signalling.  It is OK if this
            * fails or if status is changed by waiting thread.
            */
           int ws = node.waitStatus;
           if (ws < 0)
               compareAndSetWaitStatus(node, ws, 0);
   
           /*
            * Thread to unpark is held in successor, which is normally
            * just the next node.  But if cancelled or apparently null,
            * traverse backwards from tail to find the actual
            * non-cancelled successor.
            */
       	//head.next 第一个排队的
           Node s = node.next;
           if (s == null || s.waitStatus > 0) {
               s = null;
               for (Node t = tail; t != null && t != node; t = t.prev)
                   if (t.waitStatus <= 0)
                       s = t;
           }
       	//解锁这个第一个排队的线程
           if (s != null)
               LockSupport.unpark(s.thread);
       }
   
   //解锁之后
   private final boolean parkAndCheckInterrupt() {
           LockSupport.park(this);
       	//从这恢复执行
           return Thread.interrupted();
       }
   
   final boolean acquireQueued(final Node node, int arg) {
           boolean failed = true;
           try {
               boolean interrupted = false;
               //进行自旋
               for (;;) {
                   final Node p = node.predecessor();
                   //拿到锁且通过条件
                   if (p == head && tryAcquire(arg)) {
                       //狸猫换太子
                       setHead(node);
                       //将彻底断开之前的head
                       p.next = null; // help GC
                       failed = false;
                       //退出循环
                       return interrupted;
                   }
                   if (shouldParkAfterFailedAcquire(p, node) &&
                       parkAndCheckInterrupt())
                       interrupted = true;
               }
           } finally {
               if (failed)
                   cancelAcquire(node);
           }
       }
   
   private void setHead(Node node) {
      		//将当前节点设为head
           head = node;
       	//将当前节点打造成head
           node.thread = null;
           node.prev = null;
       }
   ```

