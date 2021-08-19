---
title: SpringbootLearning
comments: true
tags:
  - Default
categories:
  - Default
date: 2021-08-13 15:52:38
updated:
---



<!--more-->

## 概念

### 方法参数注入

> @Configuration标注在类上，在`@Bean`标注的方法上(如果未通过@Bean指定bean的名称，则默认与标注的方法名相同)，如果你传入了参数，springboot会自动会为这个参数在spring上下文里寻找这个类型的引用。并先初始化这个类的实例。

```java
@Bean
public FanoutExchange fanout() {
    return new FanoutExchange("tut.fanout");
}

@Bean
public Queue autoDeleteQueue1() {
    return new AnonymousQueue();
}

////Spring自动会执行这个方法，自动填入fanout和queue，这样exchange和queue绑定关系就建成了，并不需要使用这个bean。所以后面直接用fanout就可以了。
@Bean
public Binding binding1(FanoutExchange fanout,
                        @Qualifier("autoDeleteQueue1") Queue qqq) {
    return BindingBuilder.bind(qqq).to(fanout);
}
```

需要注意的是，springboot会按类型去寻找，。如果这个类型有多个实例被注册到spring上下文，按形参的名字注册bean，若按类型和名字都找不到，那你就需要加上`@Qualifier("Bean的名称")`来指定。`FanoutExchange fanout`这个形参可以自动装入上面配置的`public FanoutExchange fanout()`。`@Qualifier("autoDeleteQueue1") Queue qqq`这个形参可以按指定的名称装入`public Queue autoDeleteQueue1()`。

### 注解动态参数

>注解只能配置常量，在一些构架的开发中，有时候我们需要给注解动态配置一些值，或者想从配置文件中读取配置。直接在注解上配置是无法实现的，但是我们可以在拿到注解的值之后，再对这些值进行另外的操作。比如在注解上面配置占位符，在使用的时候，再对这些占位符进行替换。在RocketMQ的监听配置中，就使用了这个技术。

**${key名称}**

1. 用户获取外部文件中指定key的值
2. 可以在xml中配置，也可以出现在@value注解中
3. 一般用于获取数据库配置内容信息

**#{表达式}**

1. spring中el表达式的格式
2. 可以在xml中配置，也可以出现在@value注解中
3. 可以任意表达式，支持运算符

[SpringBoot之Spring@Value属性注入使用详解](https://zhuanlan.zhihu.com/p/99272510)



## 注解

## 功能

### CommandLineRunner 启动时自动执行方法

SpringBoot项目在启动时会**遍历所有的CommandLineRunner的实现类**并调用其中的run方法。

- 如果整个系统中有多个CommandLineRunner的实现类，那么可以使用@Order注解对这些实现类的调用顺序进行排序，数字越小越先执行。
- run方法的参数是系统启动是传入的参数，即入口类中main方法的参数。在调用SpringApplication.run方法时被传入 SpringBoot项目中。

```java
@Component
@Order(1)
public class CommandLineRunner1 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner1:" + Arrays.toString(args));
    }
}
 
@Component
@Order(2)
public class CommandLineRunner2 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner2:" + Arrays.toString(args));
    }
}
```

### Scheduled 定时任务

在方法上使用`@Scheduled`注解启动定时任务。

fixedRate属性设置再次调用的延时(不等待调用完成)，可能造成重复执行问题。

fixedDelay属性等到方法执行完毕后延时。推荐。

initialDelay配合上面其一使用，配置第一次执行的延时。

#### cron属性

> cron属性值是一个String类型的时间表达式，各部分的含义如下：
>
> Seconds : 可出现", - * /“四个字符，有效范围为0-59的整数
> Minutes : 可出现”, - * /“四个字符，有效范围为0-59的整数
> Hours : 可出现”, - * /“四个字符，有效范围为0-23的整数
> DayofMonth : 可出现”, - * / ? L W C"八个字符，有效范围为0-31的整数
> Month : 可出现", - * /“四个字符，有效范围为1-12的整数或JAN-DEc
> DayofWeek : 可出现”, - * / ? L C #“四个字符，有效范围为1-7的整数或SUN-SAT两个范围。1表示星期天，2表示星期一，依次类推
> Year : 可出现”, - * /"四个字符，有效范围为1970-2099年
>
> 举几个例子如下：
>
> “0 0 12 * * ?” 每天中午十二点触发
> “0 30 10 ? * *” 每天早上10：30触发
> “0 15 10 * * ?” 每天早上10：15触发
> “0 15 10 * * ? *” 每天早上10：15触发
> “0 15 10 * * ? 2018” 2018年的每天早上10：15触发
> “0 * 14 * * ?” 每天从下午2点开始到2点59分每分钟一次触发
> “0 0/5 14 * * ?” 每天从下午2点开始到2：55分结束每5分钟一次触发
> “0 0/5 14,18 * * ?” 每天的下午2点至2：55和6点至6点55分两个时间段内每5分钟一次触发
> “0 0-5 14 * * ?” 每天14:00至14:05每分钟一次触发
> “0 10,44 14 ? 3 WED” 三月的每周三的14：10和14：44触发
> “0 15 10 ? * MON-FRI” 每个周一、周二、周三、周四、周五的10：15触发



### Profile 环境配置

profile 可以让 Spring 对不同的环境提供不同配置的功能，可以通过激活、指定参数等方式快速切换环境。

#### 创建自定义的properties文件

如果需要创建自定义的的properties文件时，可以新建名为application-xxx.properties的文件。

若我们需要在两种环境下进行切换，只需要在application.properties中加入`spring.profiles.active=xxx`即可。或者在命令行后追加`--spring.profiles.active=xxx,xxx；`

.properties替换成.yml同理。

#### @Profile注解

`@profile`注解的作用是指定类或方法在特定的 Profile 环境生效，任何`@Component`或`@Configuration`注解的类都可以使用`@Profile`注解。在使用DI来依赖注入的时候，能够根据`profile`标明的环境，将注入符合当前运行环境的相应的bean。

`@Profile`可以用在类和方法上。

`@Profile`可以填入字符串数组` @Profile({"dev","test"})`

`@Profile`加上感叹号代表不在此环境下生效` @Profile("!dev")`

#### 配合CommandLineRunner

```java
@Profile("usage_message")
@Bean
public CommandLineRunner usage() {
    return args -> {
        System.out.println("This app uses Spring Profiles to control its behavior.\n");
    };
}
```

### StopWatch 计时器

```java
StopWatch watch = new StopWatch();
watch.start();
doWork(xxx);
watch.stop();
System.out.println(watch.getTotalTimeSeconds() + "s");
```



