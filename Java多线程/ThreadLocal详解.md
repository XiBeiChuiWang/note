# ThreadLocal详解

1. ### 定义

    ThreadLocal提供了线程的局部变量，每个线程都可以通过`set()`和`get()`来对这个局部变量进行操作，但不会和其他线程的局部变量进行冲突，**实现了线程的数据隔离**～。 

   直接上代码

   ```java
   public class ThreadLocalTest {
       public static void main(String[] args) {
           ThreadLocal<String> stringThreadLocal1 = new ThreadLocal<String>();
           ThreadLocal<String> stringThreadLocal2 = new ThreadLocal<String>();
   
           for (int i = 0; i < 5;i++) {
               new Thread(()->{
                   stringThreadLocal1.set(Thread.currentThread().getName()+"a");
                   stringThreadLocal2.set(Thread.currentThread().getName()+"b");
                   System.out.println(stringThreadLocal1.get());
                   System.out.println(stringThreadLocal2.get());
               }).start();
           }
       }
   }
   ```

   ![](https://pic.imgdb.cn/item/603ce0335f4313ce2569aa8c.png)

   ok，其实就是一个线程隔离的类似于容器的东东。

2. ### 原理

   上源码！

   ```java
   //先看set
   public void set(T value) {
       	//获取当前线程
           Thread t = Thread.currentThread();
       	//根据当前线程获取map ?? 莫着急，点进去
           ThreadLocalMap map = getMap(t);
           if (map != null)
               map.set(this, value);
           else
               createMap(t, value);
       }
       
      ThreadLocalMap getMap(Thread t) {
          //继续点
           return t.threadLocals;
       }
       //ok，这个属性，我们来看，ThreadLocalMap类型的，奥这样啊，应该是个键值对
   	//注意，这个属性是Thread类的普通属性，这就说明它是线程私有的，这也就解释了为什么它实现了线程的数据隔离
       ThreadLocal.ThreadLocalMap threadLocals = null;
   
   
   //我们来看看ThreadLocalMap，好多方法，不要紧，我们只看重要的
   static class ThreadLocalMap {
       //Entry内部类，注意这的key是使用的弱引用指向ThreadLocal
       static class Entry extends WeakReference<ThreadLocal<?>> {
              //我们保存的值
               Object value;
   			
               Entry(ThreadLocal<?> k, Object v) {
                   super(k);
                   value = v;
               }
           }
       //Entry数组默认长度为16
       private static final int INITIAL_CAPACITY = 16;
       //底层是一个Entry数组 
       private Entry[] table;
   }
   
   //我们接着set往下说
   public void set(T value) {
       	//获取当前线程
           Thread t = Thread.currentThread();
       	//根据当前线程获取map ?? 莫着急，点进去
           ThreadLocalMap map = getMap(t);
       	//如果map不为空，就set进去
           if (map != null)
           
               map.set(this, value);
           else
               //如果map为空，就创建一个新的map（初始化）
               createMap(t, value);
       }
   
   //注意这是ThreadLocalMap的set
   private void set(ThreadLocal<?> key, Object value) {
   			//拿到Entry数组
               Entry[] tab = table;
               int len = tab.length;
       		//根据threadLocal（key）算出索引
               int i = key.threadLocalHashCode & (len-1);
   			//循环，相信大家也注意到了，Entry里面并没有出现next，也就是说不存在链表，那他是咋处理			hash冲突的呢，其实很简单，一直往后遍历就可以了，找到第一个为空的位子，但是因为这种原因，查				找起来的速度肯定比较慢
               for (Entry e = tab[i];
                    e != null;
                    e = tab[i = nextIndex(i, len)]) {
                   ThreadLocal<?> k = e.get();
   
                   if (k == key) {
                       e.value = value;
                       return;
                   }
   
                   if (k == null) {
                       replaceStaleEntry(key, value, i);
                       return;
                   }
               }
   
               tab[i] = new Entry(key, value);
               int sz = ++size;
               if (!cleanSomeSlots(i, sz) && sz >= threshold)
                   rehash();
           }
   
   //get大家看一看
   public T get() {
           Thread t = Thread.currentThread();
           ThreadLocalMap map = getMap(t);
           if (map != null) {
               ThreadLocalMap.Entry e = map.getEntry(this);
               if (e != null) {
                   @SuppressWarnings("unchecked")
                   T result = (T)e.value;
                   return result;
               }
           }
       //需要注意的是这里如果不存在，不会返回空，而是会自己造一个出来
           return setInitialValue();
       }
   //造的过程
   private T setInitialValue() {
           T value = initialValue();
           Thread t = Thread.currentThread();
           ThreadLocalMap map = getMap(t);
           if (map != null)
               map.set(this, value);
           else
               createMap(t, value);
           return value;
       }
   ```

   说了这么多，有个问题一直被忽略了——弱引用

   我们来看堆栈图（我的画图技术越来越好了呢）

   ![](https://pic.imgdb.cn/item/603cea415f4313ce2570509f.jpg)

   原来是这样，一下子就明白了

   为什么要使用弱指针呢？

   因为一旦栈中的ThreadLocal引用没有了，如果使用强指针的话，ThreadLocal实例会产生内存泄漏吗

   那么使用了指针后就没有问题了吗？

   **不**，栈中的ThreadLocal引用没有了，弱引用会导致在下一次GC时，ThreadLocal实例没有了，这个时候我们的key变为空，但是！value还在，还是内存泄漏，所以我们的解决方案只能是使用完毕后记着remove一下

   由此可见使用弱指针只是为了让我们在忘记remove时产生内存泄漏，这个泄露的空间能小一点，煞费苦心啊！

   **哈哈哈，如果只是这样的话，那你未免太小看作者了，实际上这种做法是为了应对线程池，使用线程池的话，我们的线程对象是被复用的，不会消亡。因此如果使用了强指针，那么ThreadLocal的实例对象永远不会被GC，这太可怕了**
   
   