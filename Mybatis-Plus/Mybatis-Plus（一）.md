# Mybatis-Plus（一）

我们在使用Mybatis框架时，经常会因为大量的简单sql语句而心烦意乱，那我们有办法去解决吗？

## Mybatis-Plus优点

- **润物无声**

  只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑。

- **效率至上**

   只需简单配置，即可快速进行单表 CRUD 操作，从而节省大量时间。 

- **丰富功能**

   代码生成、物理分页、性能分析等功能一应俱全。 

**Mybatis-Plus愿景**

我们的愿景是成为 MyBatis 最好的搭档，就像魂斗罗中的 1P、2P，基友搭配，效率翻倍。

## 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

## 快速入门

很简单，我们只需要导入jar包

 Maven： 

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.x.x</version>
</dependency>
```

然后写mapper接口

```java
@Repository
public interface UserMapper extends BaseMapper<User> {
}

```

启动类

```java
@SpringBootApplication
@MapperScan("com.mapper")
public class MybatisApp {
    public static void main(String[] args) {
        SpringApplication.run(MybatisApp.class,args);
    }
}

```

测试

```java
@SpringBootTest
public class Test1 {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void a(){
        System.out.println(userMapper);
        System.out.println(userMapper.selectList(null));
    }
}
```

我们会发现从头到尾没有sql语句，Mybatis-Plus都帮我们生成了，那我们怎么看呢

## 日志输出

很简单，我们只需要在配置文件中新增一句

```yml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC
    username: root
    password: 123456
//日志
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 插入

```java
 @Test
    public void b(){
        User user = new User();
        user.setName("狗蛋");
        user.setAge(18);
        System.out.println(userMapper.insert(user));
        System.out.println(user);
    }
```

![](https://pic.imgdb.cn/item/6067155f8322e6675c20a56c.jpg)

可以看到确实添加成功了

## 主键策略

想改变主键策略，我们需要在实体类上添加注解

```java
public class User {
    @TableId(type = IdType.ID_WORKER)
    private Long id;
    private String name;
    private int age;
    //  get/set方法略
}
```

|      值       |                             描述                             |
| :-----------: | :----------------------------------------------------------: |
|     AUTO      |                         数据库ID自增                         |
|     NONE      | 无状态,该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT) |
|     INPUT     |                    insert前自行set主键值                     |
|   ASSIGN_ID   | 分配ID(主键类型为Number(Long和Integer)或String)(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextId`(默认实现类为`DefaultIdentifierGenerator`雪花算法) |
|  ASSIGN_UUID  | 分配UUID,主键类型为String(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextUUID`(默认default方法) |
|   ID_WORKER   |     分布式全局唯一ID 长整型类型(please use `ASSIGN_ID`)      |
|     UUID      |           32位UUID字符串(please use `ASSIGN_UUID`)           |
| ID_WORKER_STR |     分布式全局唯一ID 字符串类型(please use `ASSIGN_ID`)      |

一共有这么多

例子（这是采用ID_WORKER（**雪花算法**））

![](https://pic.imgdb.cn/item/606719888322e6675c250303.jpg)

## 更新

```java
@Test
    public void c(){
        User user = new User();
        user.setId(5l);
        user.setName("钢蛋");
        user.setAge(18);
        System.out.println(userMapper.updateById(user));
    }
```

很简单吧

## 自动填充

#### 数据库自动更新

![](https://pic.imgdb.cn/item/60671bd98322e6675c276e26.jpg)

但是只有mysql5.7以上才支持哦

#### 代码更新

1. 添加数据库字段（不再赘述）

2. 实体类添加字段及注解

   ```java
   public class User {
       @TableId(type = IdType.AUTO)
       private Long id;
       private String name;
       private int age;
   
       @TableField(fill = FieldFill.INSERT)
       private Date create_time;
       @TableField(fill = FieldFill.INSERT_UPDATE)
       private Date update_time;
   }
   ```

   |       属性       |             类型             | 必须指定 |          默认值          |                             描述                             |
   | :--------------: | :--------------------------: | :------: | :----------------------: | :----------------------------------------------------------: |
   |      value       |            String            |    否    |            ""            |                         数据库字段名                         |
   |        el        |            String            |    否    |            ""            | 映射为原生 `#{ ... }` 逻辑,相当于写在 xml 里的 `#{ ... }` 部分 |
   |      exist       |           boolean            |    否    |           true           |                      是否为数据库表字段                      |
   |    condition     |            String            |    否    |            ""            | 字段 `where` 实体查询比较条件,有值设置则按设置的值为准,没有则为默认全局的 `%s=#{%s}`,[参考(opens new window)](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/SqlCondition.java) |
   |      update      |            String            |    否    |            ""            | 字段 `update set` 部分注入, 例如：update="%s+1"：表示更新时会set version=version+1(该属性优先级高于 `el` 属性) |
   |  insertStrategy  |             Enum             |    N     |         DEFAULT          | 举例：NOT_NULL: `insert into table_a(column) values (#{columnProperty})` |
   |  updateStrategy  |             Enum             |    N     |         DEFAULT          | 举例：IGNORED: `update table_a set column=#{columnProperty}` |
   |  whereStrategy   |             Enum             |    N     |         DEFAULT          |      举例：NOT_EMPTY: `where column=#{columnProperty}`       |
   |       fill       |             Enum             |    否    |    FieldFill.DEFAULT     |                       字段自动填充策略                       |
   |      select      |           boolean            |    否    |           true           |                     是否进行 select 查询                     |
   | keepGlobalFormat |           boolean            |    否    |          false           |              是否保持使用全局的 format 进行处理              |
   |     jdbcType     |           JdbcType           |    否    |    JdbcType.UNDEFINED    |           JDBC类型 (该默认值不代表会按照该值生效)            |
   |   typeHandler    | Class<? extends TypeHandler> |    否    | UnknownTypeHandler.class |          类型处理器 (该默认值不代表会按照该值生效)           |
   |   numericScale   |            String            |    否    |            ""            |                    指定小数点后保留的位数                    |

3. 编写处理器处理此注解

   ```java
   @Component
   public class MyHandler implements MetaObjectHandler {
       @Override
       public void insertFill(MetaObject metaObject) {
           this.setFieldValByName("createTime",new Date(),metaObject);
           this.setFieldValByName("updateTime",new Date(),metaObject);
       }
   
       @Override
       public void updateFill(MetaObject metaObject) {
           this.setFieldValByName("updateTime",new Date(),metaObject);
       }
   }
   ```

## 乐观锁

1. 添加数据库字段

   ![](https://pic.imgdb.cn/item/606721018322e6675c2cfa96.jpg)

2. 实体类添加注解

   ```java
   @Version
       private int version;
   ```

3. 添加配置类

   ```java
   @Configuration
   @MapperScan("com.mapper")
   @EnableTransactionManagement
   public class MybatisPlusConfig {
   
       @Bean
       public OptimisticLockerInterceptor optimisticLockerInterceptor(){
           return new OptimisticLockerInterceptor();
       }
   }
   ```

   我们将@MapperScan挪到这儿。

4. 测试

   成功的：

   ```java
   @Test
       public void d(){
           User user = userMapper.selectById(1l);
           user.setName("钢蛋");
           user.setAge(18);
           userMapper.updateById(user);
       }
   ```

   ![](https://pic.imgdb.cn/item/606723858322e6675c2fbeac.jpg)

   失败的：

   ```java
       @Test
       public void e(){
           //模拟线程1
           User user1 = userMapper.selectById(3l);
           user1.setName("钢蛋");
           user1.setAge(18);
           //模拟线程2插队
           User user2 = userMapper.selectById(3l);
           user1.setName("钢蛋3");
           user1.setAge(18);
           userMapper.updateById(user1);
   //        userMapper.updateById(user2);
       }
   ```

   修改失败！

   