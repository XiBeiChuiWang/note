# Lock类以及Condition类

**官方说明：**

![](https://pic.imgdb.cn/item/6021490e3ffa7d37b3bf684e.png)

Lock是专门处理线程安全的。

我们来看看格式。

```java
public class Test3 {
    public static void main(String[] args) {
        T1 t1 = new T1();
        new Thread(()->{while (true) t1.A();}).start();
        new Thread(()->{while (true) t1.B();}).start();
        new Thread(()->{while (true) t1.C();}).start();
    }
}

class T1{
    private Lock lock= new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    private int num = 1;
    public void A(){
        lock.lock();
        try {
            while (num != 1){
                condition1.await();
            }
            System.out.println("A");
            num = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void B(){
        lock.lock();
        try {
            while (num != 2){
                condition2.await();
            }
            System.out.println("B");
            num = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void C(){
        lock.lock();
        try {
            while (num != 3){
                condition3.await();
            }
            System.out.println("C");
            num = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

可以看到使用这种方式可以唤醒指定的线程，这是我们使用以前的方式做不到的。

