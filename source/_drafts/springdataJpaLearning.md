---
title: springdataJpaLearning
comments: true
tags:
  - SpringDataJpa
categories:
  - Java
date: 2021-08-10 15:19:35
updated:
---



<!--more-->

#### 继承关系

##### SINGLE_TABLE

只生成父类一个表(复合父类和子类的字段)，子类相当于表中一条记录用dtype字段区分。用在比如key-value的形式。很多数据都是key-value形式，根据不同的类型代表不同的涵义。例如父表有字段id,key,value。子表1的DiscriminatorValue为color，key为red,value为FF0000。子表2的DiscriminatorValue为language,key为chinese,中文等。特征为不同类型子表的结构相同。

父类注解:`@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`数据库表生成dtype字段。

子类继承父类使用注解`@DiscriminatorValue("a1")`指定表中dtype字段值。



##### JOINED

生成父类和子类所有的表，子表带有父类表的外键,也是子表的主键。插入记录时自动插入父表和子表数据。 父表和子表数据一对一(自动生成父表id时无法指定id，所以每次插入子表时会新建一条主表数据)。感觉只是把父表的字段放到子表中存储。

父类注解:`@Inheritance(strategy = InheritanceType.JOINED)`

子类自动生成 子类名_id 的主键和外键(关联父类生成表的主键)



##### TABLE_PER_CLASS 

父类注解：`@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)` 父类不生成表。主键生成策略不可以是`@GeneratedValue(strategy = GenerationType.IDENTITY)` ，我定义成了 `@GeneratedValue(strategy = GenerationType.AUTO) `

只会生成子类对应的表。结构为父类+子类的字段。抽象出不同子类共同拥有的字段，id在父类设置。

另外父类可以使用`@MappedSuperclass`效果等同当前，只是父类不用写` @Entity`注解



#### 类与类关系常用注解

##### @JoinTable

用于生成一张关联外键表(自身表就不会产生外键字段)，name指定表名，表里分别有两个外键JoinColumn和inverseJoinColumns(name指定外键列名)。

如果不用`@JoinTable`就会在Student类自动生成一个computer_id的外键(多对一)

```java
@Getter
@Setter
@Accessors(chain = true)
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    private long id;


// 这里是定义学生和课桌的一对一的关系
    @OneToOne
// 下面的这个注解用来生成第三张表，来维护学生和课桌的关系
    @JoinTable( name = "stu_desk",joinColumns = @JoinColumn(name="student_id"),inverseJoinColumns = @JoinColumn(name="desk_id") )
    private Desk desk;
}
```

```java
@Getter
@Setter
@Accessors(chain = true)
@Entity
public class Computer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;

    @OneToOne(mappedBy = "computer")
    private Student student;
}
```

单个变量用`@JoinTable`就是一对一关系。Set集合变量用`@JoinTable`就是多对多关系



##### mappedBy

只有OneToOne,OneToMany,ManyToMany上才有mappedBy属性,若不设置，系统自动生成一个外键表做关联，ManyToOne不存在该属性；一般用在一的这一方。

使用mappedBy后，这个类生成的表将不会有这个字段的外键，外键应该在需要关联的表中使用`@JoinColumn`设置。例如`@OneToOne(mappedBy = "computer")` mappedBy的值为设置外键那一方(Entity类)的变量名，而不是表中的字段名。

mappedBy跟JoinColumn/JoinTable总是处于互斥的一方



#### 级联

1. CascadeType.PERSIST 该级联是级联保存。
2. CascadeType.MERGE 该级联是级联更新
3. CascadeType.REMOVE 该级联是级联删除
4. CascadeType.REFRESH 该级联是级联刷新(不常用）
5. CascadeType.DETACH 该级联是级联托管（不常用）
6. CascadeType.ALL 具有上述五个级联的功能