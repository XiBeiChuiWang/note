# Mybatis-Plus（二）

## 查询

```java
@Test
    public void f(){
        HashMap<String, Object> stringObjectHashMap = new HashMap<>();
        stringObjectHashMap.put("name","钢蛋");
        List<User> users = userMapper.selectByMap(stringObjectHashMap);
        System.out.println(users);
    }
```

这是使用map方式去查询

## 分页查询

### 分页查询方式：

1. **limit语句**

2. **分页插件**

3. **mp提供的分页插件**

   ##### 使用方式：

   1. 注册插件

      ```java
       @Bean
          public PaginationInterceptor paginationInterceptor() {
              PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
              // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
              // paginationInterceptor.setOverflow(false);
              // 设置最大单页限制数量，默认 500 条，-1 不受限制
              // paginationInterceptor.setLimit(500);
              // 开启 count 的 join 优化,只针对部分 left join
              paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
              return paginationInterceptor;
          }
      ```

   2. 使用

      ```java
      @Test
          public void g(){
              Page<User> userPage = new Page<>(2, 5);
              userMapper.selectPage(userPage,null);
              System.out.println(userPage.getRecords());
          }
      ```

## 删除

算了，不说了（同增删改）

## 逻辑删除

1. 添加数据库字段

   ![](https://pic.imgdb.cn/item/6067369b8322e6675c45e6d8.jpg)

2. 注册插件

   ```java
   @Bean
       public ISqlInjector sqlInjector(){
           return new LogicSqlInjector();
       }
   ```

3. 测试

   ```java
   @Test
       public void h(){
           userMapper.deleteById(2l);
       }
   ```

   ![](https://pic.imgdb.cn/item/6067372a8322e6675c46dae3.jpg)

   可以看到实际上是使用了update语句

最新版本可能有些不同，[详见](https://mp.baomidou.com/guide/logic-delete.html#%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95)

## 条件查询器

[详见](https://mp.baomidou.com/guide/wrapper.html)

## 代码自动生成器

```java
// 需要构建一个 代码自动生成器 对象
        AutoGenerator mpg = new AutoGenerator();
        // 配置策略
        // 1、全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");//获取目录
        gc.setOutputDir(projectPath + "/src/main/java");//输出目录
        gc.setAuthor("zx");//设置作者
        gc.setOpen(false);//是否打开资源管理器
        gc.setFileOverride(false); // 是否覆盖
        gc.setServiceName("%sService"); // 去Service的I前缀
        gc.setIdType(IdType.AUTO);//设置主键的算法 //对应数据的主键(uuid,自增ID，雪花算法，redis，zookeeper)
        gc.setDateType(DateType.ONLY_DATE);//日期类型
        gc.setSwagger2(true);//是否配置Swagger2
        mpg.setGlobalConfig(gc);
        //2、设置数据源
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/test?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        dsc.setDbType(DbType.MYSQL);//配置数据库类型 mysql
        mpg.setDataSource(dsc);
        //3、包的配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName("blog");//模块名称
        pc.setParent("com");//配置工程包名称
        pc.setEntity("entity");//配置entity的包名
        pc.setMapper("mapper");//配置mapper的包名
        pc.setService("service");//配置service的包名
        pc.setController("controller");//配置controller的包名
        mpg.setPackageInfo(pc);//配置包
        //4、策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setInclude("user"); ////设置映射的表名 多个表strategyConfig.setInclude("user","cloud","...");
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);//字段名转驼峰命名
        strategy.setEntityLombokModel(true); // 自动lombok；
        strategy.setLogicDeleteFieldName("deleted");//逻辑删除的字段名称
        // 自动填充配置
        TableFill create_time = new TableFill("create_time", FieldFill.INSERT);//自动填充更新的字段
        TableFill update_time = new TableFill("update_time", FieldFill.INSERT_UPDATE);//自动更新的字段
        ArrayList<TableFill> tableFills = new ArrayList<>();
        tableFills.add(create_time);//创建的策略
        tableFills.add(update_time);//更新的策略
        strategy.setTableFillList(tableFills);
        // 乐观锁
        strategy.setVersionFieldName("version");
        strategy.setRestControllerStyle(true);
        strategy.setControllerMappingHyphenStyle(true);
        mpg.setStrategy(strategy);
        mpg.execute(); //执行
```

