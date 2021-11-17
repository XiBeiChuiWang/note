# Spring循环依赖

## 概述

Spring框架发展至今，已经是一个十分庞大的框架了，想去通读他的代码，是非常困难的，我们要选择一个切入点去处理，而比较好的选择就是我们的循环依赖。

## 什么是循环依赖

所谓循环依赖就是你中有我，我中有你，形成一个闭环，由这个特性我们可以得出，循环依赖只存在于单例模式，因为在原型模式下，他会形成一个死循环。

实体类：

```java
public class A {

    public B b;
    public A() {
        System.out.println("a");
    }

    public B getB() {
        return b;
    }


    public void setB(B b) {
        this.b = b;
    }
}

public class B {
    public A a;

    public B() {
        System.out.println("b");
    }
    public A getA() {
        return a;
    }
}
```

此中的A和B便形成了循环依赖。

## 如何解决循环依赖

### 第一次尝试

既然是一个容器，我们便直接将容器创建出来，并且这个容器是全局唯一的。

我们就使用HashMap，key为class的全限定类名，而value便是我们的单例对象了

我们来先看这段代码

```java
public class SingletonFactory {
    private static HashMap<String, Object> hashmap = new HashMap<String, Object>();

    public static void createBean(Class... classes) throws IllegalAccessException, InstantiationException {

        for (int i = 0;i < classes.length;i++){
            createBean(classes[i]);
        }
    }

    private static void createBean(Class class1) throws IllegalAccessException, InstantiationException {
        if (hashmap.get(class1.getName()) == null) {
            Object o = class1.newInstance();
            //将对象创建出来后直接放进容器，使得其他类在进行属性填充时，可以获取到这个对象
            hashmap.put(class1.getName(),o);

            //属性填充
            Field[] fields = class1.getFields();
            for (int j = 0;j < fields.length;j++){
                //根据属性名递归创建要注入的对象
                if (hashmap.get(fields[j].getType().getName()) == null){
                    createBean(fields[j].getType());
                }
                //将属性注入
                fields[j].set(o,hashmap.get(fields[j].getType().getName()));
                return;
            }
        }
    }

    public static Object getBean(String s){
        return hashmap.get(s);
    }
}
```

欧克，我们来看看效果

```java
public class Main {
    public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        SingletonFactory.createBean(A.class,B.class);
        A a = (A)SingletonFactory.getBean("com.zx.test.A");
        B b = (B)SingletonFactory.getBean("com.zx.test.B");
        System.out.println(a);
        System.out.println(a.getB());

        System.out.println(b);
        System.out.println(b.getA());

    }
}
```

![](https://pic.imgdb.cn/item/607fbe8902af4a6ed46e5af6.jpg)

好的，看来没有问题，但是仔细想想真的没问题吗？

我们在进行属性填充前就将对象提前暴露给外界，会不会出现问题？

肯定是有问题的，我们获取到的对象可能是一个没有经过属性填充的对象。

### 第二次尝试

```java
public class SingletonFactory2 {
    private static HashMap<String, Object> hashmap = new HashMap<String, Object>();
    private static HashMap<String, Object> earlyHashMap = new HashMap<String, Object>();

    public static void createBean(Class... classes) throws IllegalAccessException, InstantiationException {

        for (int i = 0;i < classes.length;i++){
            createBean(classes[i]);
        }
    }

    private static void createBean(Class class1) throws IllegalAccessException, InstantiationException {
        if (hashmap.get(class1.getName()) == null) {
            Object o = class1.newInstance();
            //将对象创建出来后直接放进容器，使得其他类在进行属性填充时，可以获取到这个对象
            earlyHashMap.put(class1.getName(),o);

            //属性填充
            Field[] fields = class1.getFields();

            for (int j = 0;j < fields.length;j++){
                //根据属性名递归创建要注入的对象
                Object object = null;
                if (hashmap.get(fields[j].getType().getName()) != null){
                    object = hashmap.get(fields[j].getType().getName());
                }

                if (earlyHashMap.get(fields[j].getType().getName()) != null){
                    object = earlyHashMap.get(fields[j].getType().getName());
                }

                if (hashmap.get(fields[j].getType().getName()) == null && earlyHashMap.get(fields[j].getType().getName()) == null){
                    createBean(fields[j].getType());
                    object = hashmap.get(fields[j].getType().getName());
                }

                //将属性注入
                fields[j].set(o,object);
            }


            earlyHashMap.remove(class1.getName());
            hashmap.put(class1.getName(),o);
        }
    }

    public static Object getBean(String s){
        return hashmap.get(s);
    }
}
```

这个代码有很多纰漏，但是无伤大雅，通过这种方法，可以给我们提供一种思路，建造一个半成品仓库，专门存放已经初始化但是还没有完成属性填充的对象。

无形之中，我们已经知道了我们这个bean的生命周期

1. 实例化
2. 属性填充

而我们的这个代码就是基于生命周期去写的（Spring也是如此，当然Spring比这要复杂很多）

