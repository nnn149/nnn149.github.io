---
title: SpringCloud学习笔记
comments: true
tags:
  - SpringCloud
categories:
  - java
date: 2021-08-31 15:43:24
updated: 2021-08-31 15:43:24
---

SpringCloud笔记

<!--more-->

## 项目结构

聚合项目没有代码，删除pom.xml内的`properties,dependencies,dependencyManagement,build`等标签。设置`<packaging>pom</packaging>`。设置主项目groupId。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.nicenan.mahumall</groupId>
    <artifactId>mahumall</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>mahumall</name>
    <description>聚合</description>
    <packaging>pom</packaging>
    <modules>
        <module>common</module>
        <module>gateway</module>
    </modules>
</project>

```

再使用Spring Initializr分别创建微服务模块，groupId(组)设置为聚合项目的`groupId.xxx`xxx为项目名。

聚合项目的modules标签添加创建的微服务模块。微服务模块设置服务注册，配置中心等。

可以将共同依赖和用到的类抽离到common模块，设置各个微服务模块再依赖common模块。

## Nacos 服务注册 配置中心

### 依赖添加和配置

[查看commit](https://github.com/nnn149/mahumall/tree/adc675e16ec62a612e689fde7a0a603e7265fc32)

```xml
<dependencies> 
    <!--        服务注册发现  俩依赖都要.因为ribbon不再维护换用loadbalancer-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        <version>3.0.3</version>
    </dependency>

    <!--        配置中心来做配置管理  俩依赖都要-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bootstrap</artifactId>
        <version>3.0.3</version>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2021.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 服务注册

启动类加上`@EnableDiscoveryClient`注解

`application.yml`中配置nacos server的地址，并且指定`application.name`

```yaml
spring:
  cloud:
    nacos:
      server-addr: 192.168.2.211:8848
  application:
    name: mahumall-coupon
```



### 配置中心

使用Nacos作为配置中心统一管理配置

1. 引入依赖  [文档](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/nacos-example/nacos-config-example/readme-zh.md)
2. 创建bootstrap.properties文件，这是springboot的文件，配置应用名字和Nacos Server地址

      ```properties
      spring.application.name=mahumall-third-party
      spring.cloud.nacos.config.server-addr=192.168.2.211:8848
      #命名空间
      spring.cloud.nacos.config.namespace=c31fdda6-b467-42bf-a5d6-83f7e59fa719
      #配置分组
      spring.cloud.nacos.config.group=DEFAULT_GROUP
      #多配置
      spring.cloud.nacos.config.extension-configs[0].data-id=oss.yml
      spring.cloud.nacos.config.extension-configs[0].group=DEFAULT_GROUP
      spring.cloud.nacos.config.extension-configs[0].refresh=true
      ```
3. 进入Nacos Server的配置中心，新建配置。Data ID一定是 xxx.properties。xxx是应用名字。默认规则 应用名.properties
4. 添加任意配置
5. 动态获取配置，使用两个注解`@RefreshScope`和 `@Value("${配置项名字}")`。优先使用配置中心内的值

#### 细节

 1. 命名空间  配置隔离。默认public(保留空间)。默认新增的配置都在public空间内。`bootstrap.properties`里面设置`spring.cloud.nacos.config.namespace`
    可以设置开发，测试，生产命名空间。推荐：也可以每一个微服务之间项目隔离配置，每个微服务创建自己的命名空间(直接用微服务的名字)，只加载自己命名空间的配置，利用分组切换环境。
    
2. 配置集  所有配置的集合

3. 配置集id  类似配置的文件名，就是Data ID。 应用名字.properties

4. 配置分组  所有的配置集都输入:DEFAULT_GROUP组  。组可以自己随便输。

5. 加载多个配置文件。
   
      ```properties
      spring.cloud.nacos.config.extension-configs[0].data-id="xxx.yml"
      spring.cloud.nacos.config.extension-configs[0].group="dev"
      spring.cloud.nacos.config.extension-configs[0].refresh=true
      ```

#### 推荐用法

1. 每个**微服务创建自己的命名空间**，创建Data ID，使用**配置分组区分环境**(dev,test,prod)。
2. 在bootstrap.properties指定命名空间和分组
3. 以前springboot任何方式从配置文件中获取值都能用（@Value，@ConfigurationProperties等），优先使用配置中心的值。

## Openfeign 远程调用

[Nacos 测试远程调用](https://github.com/nnn149/mahumall/commit/e8745ed850e606a31bec7a0d9387a3befc66f6a3)

### 添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2020.0.3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 用法

1. 引入open-feign

2. 编写接口（feign包），告诉SpringCloud这个接口需要调用远程服务

3. 接口加上注解`@FeignClient("mahumall-coupon")`  mahumall-coupon是服务的名字（用Nacos注册）

4. 接口里添加复制的方法签名，需要补全RequestMapping整个路径

5. 开启远程调用功能，启动类注解`@EnableFeignClients(basePackages = "cn.nicenan.mahumall.member.feign")` 自动扫描

6. MemberController写一个测试请求 public R test();

7. SpringCloud新版本需要在Common中排除Ribbon依赖，微服务模块添加loadbalancer依赖 https://blog.csdn.net/weixin_43556636/article/details/110653989

   ```java
   //另一个微服务的controller
   @RestController
   @RequestMapping("coupon/coupon")
   public class CouponController {
       @RequestMapping("/member/list")
       public R memberCoupons() {
           CouponEntity couponEntity = new CouponEntity();
           couponEntity.setCouponName("满99减20");
           return R.ok().put("coupons", List.of(couponEntity));
       }
   }
   //远程调用的接口
   @FeignClient("mahumall-coupon")
   public interface CouponFeignService {
       @RequestMapping("/coupon/coupon/member/list")
       public R memberCoupons();
   }
   //使用远程调用
   @RestController
   @RequestMapping("member/member")
   public class MemberController {
       @Autowired
       CouponFeignService couponFeignService;
   
       @RequestMapping("/coupons")
       public R test() {
           R memberCoupons = couponFeignService.memberCoupons();
           return R.ok().put("coupons", memberCoupons.get("coupons"));
       }
   }
   ```

## Gateway 网关

1. 添加依赖

```xml
<dependencies>
	<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2020.0.3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

2. 配置Nacos服务注册和配置中心
3. 在`application.yml`内配置路由规则

### 路由规则

按yml内从上到下匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: nicenan_route
          uri: https://www.nicenan.cn
          predicates:
            - Query=url,nicenan
        - id: qq_route
          uri: https://www.qq.com
          predicates:
            - Query=url,qq
```

### 统一跨域处理

```java
@Configuration
public class MahumallCorsConfiguration {
    @Bean
    public CorsWebFilter corsWebFilter(){
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addAllowedOriginPattern("*");
        corsConfiguration.setAllowCredentials(true);
        source.registerCorsConfiguration("/**",corsConfiguration );
        return new CorsWebFilter(source);
    }
}
```

