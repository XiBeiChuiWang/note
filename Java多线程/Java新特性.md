# Java新特性

1. ### 四大函数式接口

   **函数式接口： 只有一个方法的接口**  @FunctionalInterface 

   ![](https://pic.imgdb.cn/item/6023d4273ffa7d37b3ca75fe.png)

   1. ##### Function函数式接口

      ```java
      /**
       * Function,函数型接口，有一个输入参数，有一个输出
       * 只要是函数型接口都能用lambda表达式简化
       */
      public class FunctionDemo {
          public static void main(String[] args) {
              // Function<String, Integer> function = new Function<String, Integer>() {
              //
              //     @Override
              //     public Integer apply(String str) {
              //         return 123;
              //     }
              // };
              Function<String, Integer> function=(str)->{return 123;};
              System.out.println(function.apply("123"));
          }
      }
      ```

   2. #####  Predicate：断定型接口 

      ```java
      /**
       * 断定型接口，有一个输入参数，返回值只能是布尔值！
       */
      public class PredicateDemo {
          public static void main(String[] args) {
              // 判断字符串是否为空
              // Predicate<String> predicate = new Predicate<String>() {
              //     @Override
              //     public boolean test(String s) {
              //         return s.isEmpty();
              //     }
              // };
              Predicate<String> predicate =  (s)->{return s.isEmpty();};
              System.out.println(predicate.test(""));
          }
      }
      ```

   3. #####  Consumer：消费型接口 

      ```java
      public class ConsumerDemo {
          public static void main(String[] args) {
              // Consumer消费型接口，只有输入，没有返回值
              // Consumer<String> consumer = new Consumer<String>() {
              //     @Override
              //     public void accept(String s) {
              //         System.out.println(s);
              //     }
              // };
              Consumer<String> consumer = (s)->{System.out.println(s);};
              System.out.println(consumer);
          }
      }
      ```

   4. #####  Supplier：供给型接口 

      ```java
      public class SupplierDemo {
          public static void main(String[] args) {
              // Supplier,没有参数，只有返回值
              // Supplier<String> supplier = new Supplier<String>() {
              //     @Override
              //     public String get() {
              //         return "123";
              //     }
              // };
              Supplier<String> supplier = ()->{return "123";};
              System.out.println(supplier.get());
          }
      }
      ```

2. ### Stream流式计算

    **什么是流式计算？** 

   大数据：存储 + 计算

   集合、MySQL本质就是存储东西的；

   计算都应该交给流来操作！

   ![](https://pic.imgdb.cn/item/6023d5f23ffa7d37b3cb3ef8.png)

   ```java
   public class StreamTest {
       public static void main(String[] args) {
           User user1 = new User("张三", 18);
           User user2 = new User("李四", 19);
           User user3 = new User("王五", 20);
   
           Arrays.asList(user1,user2,user3).stream()
                   .filter(user ->{return user.getAge() % 2 == 0;})
                   .forEach((user -> {
               System.out.println(user);
           }));
       }
   }
   
   class User{
       private String name;
       private int age;
   
       public User(String name, int age) {
           this.name = name;
           this.age = age;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public int getAge() {
           return age;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       @Override
       public String toString() {
           return "User{" +
                   "name='" + name + '\'' +
                   ", age=" + age +
                   '}';
       }
   }
   ```

   

3. 

