# String详解

我们从学Java的第一天开始就接触过String类型，当时说他可以使用基本类型的写法，但是属于引用类型。

我们来看看具体的特性。

1. ### 不可变性

   ```java
    @Test
       public void test1(){
           String s1 = "abc";
           String s2 = "abc";
           System.out.println(s1 == s2);//true
   
           s1 = "a";
           System.out.println(s1 == s2);//false
           //按照这种方式给的字符串存在字符串常量池中
       }
   ```

   可以看到这种情况，字符串是创建在字符串常量池中的。

2. ### 字符串的内存分配

   底层使用StringTable（固定大小的Hashtable），默认大小为1009（jdk6）

   使用-XX:StringTableSize可设置StringTable的长度。

   - jdk6：StringTable固定，大小为1009。

     ![](https://pic.imgdb.cn/item/60181fbe3ffa7d37b3097cfd.png)

   - jdk7：StringTable的长度默认值为60013，最小值无要求。

     ![](https://pic.imgdb.cn/item/60181fcf3ffa7d37b3098517.png)

   - jdk8：StringTable的长度默认值为60013，最小值为1009。

     ![](https://pic.imgdb.cn/item/60181fef3ffa7d37b30992d0.png)

3. ### String的基本操作

   ![](https://pic.imgdb.cn/item/601ab4ca3ffa7d37b30a7688.png)

   ![](https://pic.imgdb.cn/item/601ab56a3ffa7d37b30ac275.png)

   1. **字符串拼接操作规则**

      1. 常量与常量的拼接结果在常量池，原理为编译期优化
      2. 常量池中不会存在相同的常量
      3. 只要被拼接的任意一方为变量，则创建在堆中，变量拼接原理为StringBuilder。
      4. 如果拼接的结果调用intern（）方法，则主动入池。

   2. **编译期优化举例**

      ```java
      @Test
          public void test2(){
              String s1 = "abc";
              String s2 = "ab"+"c";
              System.out.println(s1 == s2); //true
      
              final String s3 = "ab";
              final String s4 = "c";
              String s5 = s3 + s4;
              System.out.println(s1 == s5);//true
          }
      ```

      ![](https://pic.imgdb.cn/item/601ab82c3ffa7d37b30bbe4a.jpg)

   3. **变量拼接**

      ```java
      @Test
          public void test3(){
              String s1 = "ab";
              String s2 = "c";
              String s3 = s1 + s2;
              String s4 = "abc";
              System.out.println(s3 == s4);//false
          }
      ```

      ![](https://pic.imgdb.cn/item/601aba0c3ffa7d37b30c6b87.png)

      可以看到变量拼接底层用的是StringBuilder，步骤为

      1. new StringBuilder();
      2. append();
      3. append();
      4. toString()

   4. **我们再来看看用new的方式创建String**

      ```java
      @Test
          public void test4(){
              String s1 = new String("abc");
              String s2 = s1.intern();
              String s3 = "abc";
              System.out.println(s1 == s2);//false
              System.out.println(s2 == s3);//true
      
          }
      ```

      ![](https://pic.imgdb.cn/item/601bd9c13ffa7d37b3734d18.png)

      **我们可以看到，使用new的方式会直接在字符串常量池创建一个此字符串**

   5. **我们看一个比较难的，new方式拼接，这种情况是有一些不同的。**

      我们先看jdk7及之后的

      ```java
      @Test
          public void test5(){
              String s1 = new String("ab") + new String("c");
              String s2 = "abc";
              System.out.println(s1 == s2);//false
          }
      ```

      ![](https://pic.imgdb.cn/item/601bdb2e3ffa7d37b374025c.png)

      可以看到最后是调用了StringBuilder的toString方法，那么这个方法内部会把这个字符串拼接后的值入池吗？（至少我们从这里看是没有）我们来看看字节码

      ![](https://pic.imgdb.cn/item/601bdcb43ffa7d37b374964d.png)

      可以看到并没有入池

      我们继续看一个

      ```java
      @Test
          public void test6(){
              String s1 = "abc";
              String s2 = new String("ab") + new String("c");
              s2.intern();
              System.out.println(s1 == s2);//false
          }
      ```

      这个不就和上面的一样吗，除了多了一个入池操作。

      那我们就来点不一样的

      ```java
      @Test
          public void test6(){
      
              String s2 = new String("ab") + new String("c");
              s2.intern();
              String s1 = "abc";
              System.out.println(s1 == s2);//true
          }
      ```

      ？？？？？？

      不要慌，刚看到这个的时候我也很慌，首先我们看到跟上面的代码完全相同，只是挪动了一个语句的顺序。

      1. 首先，我们拼接了一个s2（”abc“），但是并没有入池。
      2. 手动入池，等等，你以为是在字符串常量池中添加一个”abc“吗？不不不，并不是，注意，他是在字符串常量池中**创建一个指针**指向s2这个堆中的对象。（到这里大家应该明白了）
      3. 然后s1也通过这个指针指向了s2的实例对象。因此为true。

      **值得注意的是，这是在jdk7及之后，7之前还是false。**

4. ### 总结intern()

   1. 在jdk6中，尝试放入串池（字符串常量池）。
      - 如果串池有，不会放入，返回串池中的对象的地址。
      - 如果没有，会把此对象复制一份，放入串池，返回串池中的对象地址。
   2. 在jdk7开始，尝试放入串池（字符串常量池）。
      - 如果串池有，不会放入，返回串池中的对象的地址。
      - 如果没有，则会把对象的引用地址复制一份，放入串池，并返回串池中的引用地址。

5. ### 补充

   - new String("ab")会创建几个对象？

     String和常量池中的ab

     2个

   - new String("a") + new String("b")会创建几个对象？

     首先一个StringBuilder的对象，两个String对象，两个常量池对象，在StringBuilder的toString方法中又创建了一个String对象。

     6个

6. ### G1中的String去重操作

   [官方链接](http://openjdk.java.net/jeps/192)

   