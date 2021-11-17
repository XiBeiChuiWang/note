# SpringCloud入门

我们承接上文，服务端提供接口供客户端使用，并且将服务进行注册

## 首先先建主项目，并配置pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>com.example.springcloud</artifactId>
    <version>1.0-SNAPSHOT</version>
<!--    子服务-->
    <modules>
        <module>springcloudapi</module>
        <module>springcloud-privider-company-8001</module>
        <module>springcloud-consumer-company-80</module>
        <module>springcloud-eureka-7001</module>
        <module>springcloud-eureka-7002</module>
        <module>springcloud-eureka-7003</module>
    </modules>


    <packaging>pom</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.10</junit.version>
    </properties>

<!--    依赖管理-->
    <dependencyManagement>
        <dependencies>
<!--            springcloud依赖-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR9</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

<!--            springboot依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.5.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.47</version>
            </dependency>

            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.1.10</version>
            </dependency>

            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>1.3.2</version>
            </dependency>

            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>

            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.16.10</version>
            </dependency>

            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>1.2.17</version>
            </dependency>

        </dependencies>
    </dependencyManagement>
</project>
```

## 第一个模块儿——API

一个JavaBean

```java
package com.pojo;

import lombok.Data;

import java.io.Serializable;
public class Company implements Serializable {
    private int id;
    private String name;
    private String ads;
    private String data_source;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAds() {
        return ads;
    }

    public void setAds(String ads) {
        this.ads = ads;
    }

    public String getData_source() {
        return data_source;
    }

    public void setData_source(String data_source) {
        this.data_source = data_source;
    }
}
```

不需要额外的jar包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>com.example.springcloud</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springcloudapi</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```

## 服务提供者

pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>com.example.springcloud</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springcloud-privider-company-8001</artifactId>

    <dependencies>
<!--        将api网关进行注入-->
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>springcloudapi</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

<!--        加入eureka依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>

    </dependencies>
</project>
```

application.yml

```yaml
server:
  port: 8001
mybatis:
  type-aliases-package: com.pojo
  mapper-locations: classpath:mapper/*.xml

spring:
  application:
    name: springcloud-provider
  datasource:
    url: jdbc:mysql://localhost:3306/db01?useUnicode=true&characterEncoding=utf-8
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    username: root
    password: 123456
#eureka配置
eureka:
  client:
    service-url:
      defaultZone: http://7001.com:7001/eureka/,http://7002.com:7002/eureka/,http://7003.com:7003/eureka/  #将服务注册到哪里？
```

持久层

```java
@Mapper
@Repository
public interface CompanyMapper {

    public boolean addCompany(Company company);
    public Company findById(int id);
    public List<Company> findAll();
}
```

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.mapper.CompanyMapper">
    <insert id="addCompany" parameterType="Company">
        insert into company (name,ads,data_source)
        values (#{name},#{ads},DATABASE())
    </insert>

    <select id="findById" parameterType="Integer" resultType="Company">
        select * from company
        where id = #{id}
    </select>

    <select id="findAll" resultType="Company">
        select * from company
    </select>
</mapper>
```

service层省略吧

controller层

```java
@RestController
public class CompanyController {

    @Autowired
    private CompanyServiceImpl companyService;

    @Autowired
    private DiscoveryClient client;

    @PostMapping("/add")
    public boolean addCompany(@RequestBody Company company){
        return companyService.addCompany(company);
    }
    @GetMapping("/findbyid/{id}")
    public Company findById(@PathVariable int id){
        return companyService.findById(id);
    }
    @GetMapping("/findall")
    public List<Company> findAll(){
        return companyService.findAll();
    }

    @GetMapping("/getclient")
    public Object di(){
        List<String> services = client.getServices();
        System.out.println(services);
        return true;
    }
}
```

## 客户端

我们先来测试一下上面的

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>com.example.springcloud</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springcloud-consumer-company-80</artifactId>

    <dependencies>
<!--        对api进行注入-->
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>springcloudapi</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
<!--        我们使用restful风格-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

application.yml

```yaml
server:
  port: 80
```

我们想要调用服务端的接口就得使用模板去发送请求，我们先得注册我们的模板

```java
@Configuration
public class Config {

    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

编写controller直接通过模板进行调用

```java
@RestController
public class CompanyConsumerController {

    @Autowired
    private RestTemplate restTemplate;

    private static final String REST_URL_PREFIX = "http://localhost:8001/";

    @RequestMapping("/consumer/get/{id}")
    public Company get(@PathVariable int id){
        return restTemplate.getForObject(REST_URL_PREFIX+"/findbyid/"+id,Company.class);
    }

    @RequestMapping("/consumer/getall")
    public List<Company> findAll(){
        return restTemplate.getForObject(REST_URL_PREFIX+"/findall",List.class);
    }

    @RequestMapping("/consumer/add")
    public boolean addCompany(Company company){
        return restTemplate.postForObject(REST_URL_PREFIX+"/add",company,Boolean.class);
    }
}
```

## 服务注册中心

到最后就是我们的服务注册中心了，我们可以配置我们的eureka集群

我们先将3个不同的域名指向我们的localhost（127.0.0.1）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>com.example.springcloud</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springcloud-eureka-7001</artifactId>
    <dependencies>
<!--        eureka-server jar包-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

application.yml

```yaml
server:
  port: 7001
#Eureka配置
eureka:
  instance:
    hostname: localhost #Eureka服务端实例名称
  client:
    register-with-eureka: false #表示是否注册自己
    fetch-registry: false  #表示自己为注册中心
    service-url:  #监控页面
      defaultZone: http://7002.com:7002/eureka/,http://7003.com:7003/eureka/#与其他注册中心进行绑定
```

其他两个注册中心同理