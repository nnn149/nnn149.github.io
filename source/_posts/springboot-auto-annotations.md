---
title: Springboot从手动装配到自动装配
comments: true
date: 2018-12-31 12:14:46
updated: 2018-12-31 12:14:46
tags:
- Springboot
categories:
- java
---

先理解手动装配,再实现自动装配
<!--more-->

## 模式注解（[Stereotpe Annotations](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model#stereotype-annotations)）

> A **stereotype annotation** is an annotation that is used to declare the role that a component plays within the application. For example, the  `@Repository` annotation in the Spring Framework is a marker for any class that fulﬁlls the role or stereotype of a repository (also known as Data Access Object or DAO).
>
> `@Component`  is a generic stereotype for any Spring-managed component. Any component annotated with `@Component`  is a candidate for component scanning. Similarly, any component annotated with an annotation that is itself meta-annotated with  `@Component`  is also a candidate for component scanning. For example, `@Service` is meta-annotated with  `@Component` .

模式注解是一种用于声明在应用中扮演“组件”角色的注解。如 Spring Framework `@Repository` 标注在任何类上 ，用于扮演仓储角色的模式注解。`@Component`  作为一种由 Spring 容器托管的通用模式组件，任何被  @Component  标准的组件均为组件扫描的候选对象。类似地，凡是被  `@Component` 元标注（meta-annotated）的注解，如  `@Service`  ，当任何组件标注它时，也被视作组件扫描的候选对象。

### 模式注解举例

| Spring Framework 注解 | 场景说明         | 起始版本 |
| --------------------- | ---------------- | -------- |
| `@Repository`         | 数据仓储模式注解 | 2.0      |
| `@Component`          | 通用组件模式注解 | 2.5      |
| `@Service`            | 服务模式注解     | 2.5      |
| `@Controller`         | 控制器模式注解   | 2.5      |
| `@Configuration`      | 配置类模式注解   | 3.0      |

### 装配方式

`<context:component-scan>` (配置文件)

`@ComponentScan `(用注解)

两个都可以设置要扫描的包 `base-package`

### 自定义模式注解

`@Component` “派生性"

`@Component` “层次性

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repository 
public @interface FirstLevelRepository {
    String value() default "";
}
```

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@FirstLevelRepository 
public @interface SecondLevelRepository  {
    String value() default "";
}
```

- `@Component`
  - `@Repository`
    - `@FirstLevelRepository`
      - `@SecondLevelRepository`

所以被`@SecondLevelRepository`注解的类本质也是`@Component`



## Spring @Enable 模块装配 

> `@enable*`是Springboot中用来启用某一个功能特性的一类注解。其中包括我们常用的`@SpringBootApplication`注解中用于开启自动注入的annotation`@EnableAutoConfiguration`，开启异步方法的annotation`@EnableAsync`，开启将配置文件中的属性以bean的方式注入到IOC容器的annotation`@EnableConfigurationProperties`等。

Spring Framework 3.1 开始支持”@Enable 模块驱动“。所谓“模块”是指具备相同领域的功能组件集合， 组合所形成一个独立 的单元。比如 Web MVC 模块、AspectJ代理模块、Caching（缓存）模块、JMX（Java 管 理扩展）模块、Async（异步处 理）模块等





### `@Enable` 注解模块举例 

| 框架实现         | `@Enable`注解模块                | 激活模块            |
| ---------------- | -------------------------------- | ------------------- |
| Spring Framework | `@EnableWebMvc`                  | Web MVC 模块        |
|                  | `@EnableTransactionManagement`   | 事务管理模块        |
|                  | `@EnableCaching`                 | Caching 模块        |
|                  | `@EnableMBeanExport`             | JMX 模块            |
|                  | `@EnableAsync`                   | 异步处理模块        |
|                  | `@EnableWebFlux`                 | Web Flux 模块       |
|                  | `@EnableAspectJAutoProxy`        | AspectJ 代理模块    |
|                  |                                  |                     |
| Spring Boot      | `@EnableAutoConfiguration`       | 自动装配模块        |
|                  | `@EnableManagementContext`       | Actuator 管理模块   |
|                  | `@EnableConfigurationProperties` | 配置属性绑定模块    |
|                  | `@EnableOAuth2Sso`               | OAuth2 单点登录模块 |
|                  |                                  |                     |
| Spring Cloud     | `@EnableEurekaServer`            | Eureka服务器模块    |
|                  | `@EnableConfigServer`            | 配置服务器模块      |
|                  | `@EnableFeignClients`            | Feign客户端模块     |
|                  | `@EnableZuulProxy`               | 服务网关 Zuul 模块  |
|                  | `@EnableCircuitBreaker`          | 服务熔断模块        |

### 实现方式

#### 注解驱动方式

以SpringMvc为例:`@EnableWebMvc`注解有个`@Import`指向了DelegatingWebMvcConfiguration类,不够灵活

```java 
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.TYPE) 
@Documented 
@Import(DelegatingWebMvcConfiguration.class) 
public @interface EnableWebMvc { }
```

```java 
@Configuration 
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport{
...
}
```

#### 接口编程方式

`@EnableCaching` 注解中`@Import`指向一个ImportSelector接口的实现类,这个接口只需要重写selectImports这个方法,可以在方法里写判断,能灵活的返回一个String[]

```java 
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME) 
@Documented 
@Import(CachingConfigurationSelector.class) 
public @interface EnableCaching{
...
}
```

```java
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {
	@Override
	public String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return getProxyImports();
			case ASPECTJ:
				return getAspectJImports();
			default:
				return null;
		}
	}
}
```

### 自定义`@Enable` 模块 

#### 基于注解驱动实现  - `@EnableHelloWorld `

不详写



#### 基于接口驱动实现 - `@EnableHelloWorld`

`@EnableHelloWorld`->`HelloWorldImportSelector `->`HelloWorldConfiguration `->helloWorld

准备一个helloWorld的Bean

```java 
public class HelloWorldConfiguration {
    @Bean
    public String helloWorld() {
        return "Hello World 2018";
    }
}
```

自定义一个helloworld的选择器继承ImportSelector,实现接口方法,返回上面的Bean

```java
public class HelloWorldImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{HelloWorldConfiguration.class.getName()};
    }
}
```

自定义个 `@EnableHelloWorld` 注解,`@Import`指向自定义的HelloWorldImportSelector

```java 
@Target(ElementType.TYPE)
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Import(HelloWorldImportSelector.class)
public @interface EnableHelloWorld {
}
```

在启动类加上`@EnableHelloWorld`,就可以使用helloworld这个Bean了

```java
@EnableHelloWorld
public class EnableHelloWorldBootstrap {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(EnableHelloWorldBootstrap.class).web(WebApplicationType.NONE).run(args);
        String helloWorld = context.getBean("helloWorld", String.class);
        System.out.println(helloWorld);
        context.close();

    }
}
```

## Spring 条件装配 

> 从 Spring Framework 3.1 开始，允许在 Bean 装配时增加前置条件判断。spring条件装配实现主要有两种实现方式，一种是配置化条件装配，使用`@Profile`注解来实现。另一种比较灵活，底层实现Condition接口来进行实现。

### 条件注解举例

| Spring 注解  | 场景说明                        | 起始版本 |
| ------------ | ------------------------------- | -------- |
| @Profile     | 配置化条件装配(不灵活,条件单一) | 3.1      |
| @Conditional | 编程条件装配(灵活)              | 4.0      |

### 自定义条件装配

#### 配置方式- `@Profile`

```java
//定义一个计算接口
public interface CalculateService {
    Integer sum(Integer... values);
}

//实现类Java7版本,用@Profile("Java7")注解
@Profile("Java7")
@Service
public class Java7CalculateService implements CalculateService{
    @Override
    public Integer sum(Integer... values) {
        System.out.println("Java7实现");
        int sum = 0;
        for (Integer value : values) {
            sum += value;
        }
        return sum;
    }
}

//实现类Java8版本,用@Profile("Java8")注解
@Profile("Java8")
@Service
public class Java8CalculateService implements CalculateService{
    @Override
    public Integer sum(Integer... values) {
        System.out.println("Java8实现");
        int sum = Stream.of(values).reduce(0, Integer::sum);
        return sum;
    }
}

//引导类里面指定profiles("Java8")
@SpringBootApplication(scanBasePackages = "cn.nicenan.springbootdemo.service")
public class CalculateServiceBootstrap {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(CalculateServiceBootstrap.class).web(WebApplicationType.NONE).profiles("Java8").run(args);
        CalculateService calculateService = context.getBean(CalculateService.class);
        Integer sum = calculateService.sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        System.out.println(sum);
        context.close();
    }
}
```

#### 编程方式- `@Conditional `

自定义`@ConditionalOnSystemProperty`注解,指向OnSystemPropertyCondition.class,这是我们自定义的一个类,

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE,ElementType.METHOD})
@Documented
@Conditional(OnSystemPropertyCondition.class)
public @interface ConditionalOnSystemProperty {
    String name();

    String value();
}
```

必须要实现Condition接口,重写matches方法,这里用AnnotatedTypeMetadata获取注解的元信息,取出来后比较和System里配置的值是否相等 

```java
public class OnSystemPropertyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Map<String, Object> attributes = metadata.getAnnotationAttributes(ConditionalOnSystemProperty.class.getName());
        String propertyName = String.valueOf(attributes.get("name"));
        String propertyValue = String.valueOf(attributes.get("value"));
        String javaPropertyValue = System.getProperty(propertyName);
        return propertyValue.equals(javaPropertyValue);
    }
}
```

在启动类中测试,`@ConditionalOnSystemProperty`注解了helloWorld这个Bean,当系统用户名和10418相等的时候就装载这个Bean

```java
public class ConditionalOnSystemPropertyBootstrap {
    @ConditionalOnSystemProperty(name = "user.name", value = "10418")
    @Bean
    public String helloWorld() {
        return "Hello world nnn";
    }
    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(ConditionalOnSystemPropertyBootstrap.class).web(WebApplicationType.NONE).run(args);
        String helloWorld = context.getBean("helloWorld", String.class);
        System.out.println(helloWorld);
        context.close();
    }
}
```



## Spring Boot 自动装配 

在 Spring Boot 场景下，基于约定大于配置的原则，实现 Spring 组件自动装配的目的。其中使用了

### 底层装配技术 

- Spring 模式注解装配 
- Spring `@Enable` 模块装配 
- Spring 条件装配装配 
- Spring 工厂加载机制 
  - 实现类： `SpringFactoriesLoader `
  - 配置资源： `META-INF/spring.factories`

### 自动装配举例 

参考 `META-INF/spring.factories` idea中搜索

### 实现方法 

1. 激活自动装配 -  `@EnableAutoConfiguration ` 

   > 一旦加上此注解，那么将会开启自动装配功能，简单点讲，Spring会试图在你的classpath下找到所有配置的Bean然后进行装配。当然装配Bean时，会根据若干个(Conditional)定制规则来进行初始化。

2. 实现自动装配 -  `XXXAutoConfiguration `

3. 配置自动装配实现 - `META-INF/spring.factories`

### 自定义自动装配 

创建一个自动配置类,`@EnableHelloWorld`基于上面的自定义的`@Enable`模块,`@ConditionalOnSystemProperty` 同上

```java
@Configuration
@EnableHelloWorld
@ConditionalOnSystemProperty(name = "user.name", value = "10418")
public class HelloWorldAutoConfiguration {
}
```

在classpath目录下的META-INF/spring.factories配置好自动装配

```META-INF/spring.factories
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
cn.nicenan.springbootdemo.configuration.HelloWorldAutoConfiguration
```

启动类开启`@EnableAutoConfiguration`测试

```java
@EnableAutoConfiguration
public class EnableAutoConfigurationBootstrap {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(EnableAutoConfigurationBootstrap.class).web(WebApplicationType.NONE).run(args);
        String helloWorld = context.getBean("helloWorld", String.class);
        System.out.println(helloWorld);
        context.close();
    }
}

```

`HelloWorldAutoConfiguration `

- 条件判断： `user.name == "10418" `

- 模式注解： `@Configuration `

- `@Enable` 模块： `@EnableHelloWorld` -> `HelloWorldImportSelector` ->  `HelloWorldConfiguration` - > `helloWorld`

