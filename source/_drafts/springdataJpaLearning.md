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

参考:

-  [工作上关于spring data jpa 的使用详细总结](https://github.com/zempty-zhaoxuan/spring-data-jpa)
- [Spring Data JPA - Reference Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.named-queries)

<!--more-->

#### jpa关键字使用

定义一个名为xxxRepository的接口继承`JpaRepository<xxx,ID>` 泛型1(xxx)是Entity，泛型2(ID)是主键类型。

##### 查询关键字

| Keyword                                                      | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `find…By`, `read…By`, `get…By`, `query…By`, `search…By`, `stream…By` | General query method returning typically the repository type, a `Collection` or `Streamable` subtype or a result wrapper such as `Page`, `GeoResults` or any other store-specific result wrapper. Can be used as `findBy…`, `findMyDomainTypeBy…` or in combination with additional keywords. |
| `exists…By`                                                  | Exists projection, returning typically a `boolean` result.   |
| `count…By`                                                   | Count projection returning a numeric result.                 |
| `delete…By`, `remove…By`                                     | Delete query method returning either no result (`void`) or the delete count. |
| `…First<number>…`, `…Top<number>…`                           | Limit the query results to the first `<number>` of results. This keyword can occur in any place of the subject between `find` (and the other keywords) and `by`. |
| `…Distinct…`                                                 | Use a distinct query to return only unique results. Consult the store-specific documentation whether that feature is supported. This keyword can occur in any place of the subject between `find` (and the other keywords) and `by`. |

##### 条件关键字

| 关键字                 | 例子                                                         | JPQL 片段                                                    |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `Distinct`             | `findDistinctByLastnameAndFirstname`                         | `select distinct … where x.lastname = ?1 and x.firstname = ?2` |
| `And`                  | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                   | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is`, `Equals`         | `findByFirstname`,,`findByFirstnameIs``findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`              | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`             | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`        | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
| `GreaterThan`          | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`     | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`                | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`               | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`, `Null`       | `findByAge(Is)Null`                                          | `… where x.age is null`                                      |
| `IsNotNull`, `NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`                 | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`              | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`         | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`           | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`           | `findByFirstnameContaining`                                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`              | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`                  | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                   | `findByAgeIn(Collection<Age> ages)`                          | `… where x.age in ?1`                                        |
| `NotIn`                | `findByAgeNotIn(Collection<Age> ages)`                       | `… where x.age not in ?1`                                    |
| `True`                 | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`                | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`           | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstname) = UPPER(?1)`                     |

##### 更新删除

先查询，然后更改属性，最后save()保存即可。

#### SQL语句使用

```java
public interface ClassRoomRepository extends JpaRepository<ClassRoom,Integer> {

    //使用的 JPQL 的 sql 形式 from 后面是类名
    // ?1 代表是的是方法中的第一个参数
    @Query("select s from ClassRoom s where s.name =?1")
    List<ClassRoom> findClassRoom1(String name);

    //这是使用正常的 sql 语句去查询
    // :name 是通过 @Param 注解去确定的
    @Query(nativeQuery = true,value = "select * from class_room c where c.name =:name")
    List<ClassRoom> findClassRoom2(@Param("name")String name);
}
```

参考上述的案例我们可以发现，**sql 有两种呈现形式**：

1. JPQL 形式的 sql 语句，from 后面是以类名呈现的。
2. 原生的 sql 语句，需要使用 nativeQuery = true 指定使用原生 sql

**sql 中的参数传递也有两种形式**：

1. 使用问号 ?，紧跟数字序列，数字序列从1 开始，如 ?1 接收第一个方法参数的值。
2. 使用冒号：，紧跟参数名，参数名是通过@Param 注解来确定。



##### Sort排序

spring data jpa 提供了一个 Sort 类来进行分类排序，下面通过代码来说明 Sort 的使用：

```
public interface TeacherRepositoty extends JpaRepository<Teacher,Integer> {

    // 正常使用，只是多加了一个 sort 参数而已
    @Query("select t from Teacher t where t.subject = ?1")
    List<Teacher> getTeachers(String subject, Sort sort);
}
```

上述代码正常的写 sql 语句，只是多加了一个 Sort 参数而已，如何实例化 Sort 应该是我们关注的重点，现在看测试代码如下：

```java
  @GetMapping("/sort/{subject}")
    public List<Teacher> getTeachers(@PathVariable("subject") String subject) {
        // 第一种方法实例化出 Sort 类，根据年龄进行升序排列
        Sort sort1 = Sort.by(Sort.Direction.ASC, "age");

        //定义多个字段的排序
        Sort sort2 = Sort.by(Sort.Direction.DESC, "id", "age");

        // 通过实例化 Sort.Order 来排序多个字段
        List<Sort.Order> orders = new ArrayList<>();
        Sort.Order order1 = new Sort.Order(Sort.Direction.DESC, "id");
        Sort.Order order2 = new Sort.Order(Sort.Direction.DESC, "age");
        orders.add(order1);
        orders.add(order2);
        Sort sort3 = Sort.by(orders);

        //可以传不同的 sort1,2,3 去测试效果
        return teacherRepositoty.getTeachers(subject, sort1);
    }
```

Sort 是用来用来给字段排序的，可以根据一个字段进行排序，也可以给多个字段设置排序规则，但是个人之见使用Sort 对一个字段排序就好。



##### jpa 的分页操作

数据多的时候就需要分页，spring data jpa 对分页提供了很好的支持，下面通过一个 demo 来展示如何使用分页：

```java
public interface TeacherRepositoty extends JpaRepository<Teacher,Integer> {

    //正常使用，只是多加了一个 Pageable 参数而已
    @Query("select t from Teacher t where t.subject = :subject")
    Page<Teacher> getPage(@Param("subject") String subject, Pageable pageable);
}
```

按照正常的查询步骤，多加一个 Pageable 的参数而已，如何获取 Pageable 呢？

Pageable 是一个接口类，它的实现类是 PageRequest ,下面就通过测试代码来研究一下如何来实例化 PageRequest:

```java
    @GetMapping("page/{subject}")
    public Page<Teacher> getPage(@PathVariable("subject") String subject) {
        // 第一种方法实例化 Pageable
        Pageable pageable1 = PageRequest.of(1, 2);

        //第二种实例化 Pageable
        Sort sort = Sort.by(Sort.Direction.ASC, "age");
        Pageable pageable2 = PageRequest.of(1, 2, sort);

        //第三种实例化 Pageable
        Pageable pageable3 = PageRequest.of(1, 2, Sort.Direction.DESC, "age");


        //可以传入不同的 Pageable,测试效果
        Page page = teacherRepositoty.getPage(subject, pageable3);
        System.out.println(page.getTotalElements());
        System.out.println(page.getTotalPages());
        System.out.println(page.hasNext());
        System.out.println(page.hasPrevious());
        System.out.println(page.getNumberOfElements());
        System.out.println(page.getSize());
        return page;
    }
```

PageRequest 一共有三个可以实例化的静态方法：

1. `public static PageRequest of(int page, int size)`
2. `public static PageRequest of(int page, int size, Sort sort) `分页的同时还可以针对分页后的结果进行一个排序。
3. `public static PageRequest of(int page, int size, Direction direction, String... properties) `直接针对字段进行排序。

##### Specification

暂无

##### Projection (投影映射）

现在的需求是我只需要 Teacher 类对应的表 teacher 中的 name 和 age 的数据，其他数据不需要。 定义一个如下的接口：

```java
public interface TeacherProjection {

    String getName();

    Integer getAge();

    @Value("#{target.name +' and age is' + target.age}")
    String getTotal();
}
```

接口中的方法以 get 开头 + 属性名，属性名首字母大写， 例如 getName(), 也可以通过 @Value 注解中使用 target.属性名获取属性，也可以把多个属性值拼接成一个字符串。

定义好一个接口后，在查询方法中指定返回接口类型的数据即可，参考代码如下：

```java
public interface TeacherRepositoty extends JpaRepository<Teacher,Integer>, JpaSpecificationExecutor {

   // 返回 TeacherProjection 接口类型的数据
    @Query("select t from Teacher t ")
    List<TeacherProjection> getTeacherNameAndAge();
}
```

测试结果：

```java
  @GetMapping("/projection")
    public List<TeacherProjection> projection() {
       // 返回指定字段的数据
        List<TeacherProjection> projections = teacherRepositoty.getTeacherNameAndAge();

       // 打印字段值
        projections.forEach(teacherProjection -> {
            System.out.println(teacherProjection.getAge());
            System.out.println(teacherProjection.getName());
            System.out.println(teacherProjection.getTotal());
        });
        return projections;
    }
```

运行测试，查看结果我们会发现，我们仅仅只是拿到 name 和 age 的字段值而已。



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

   ​	级联保存记得在外键的一方设置外键

2. CascadeType.MERGE 该级联是级联更新

3. CascadeType.REMOVE 该级联是级联删除

4. CascadeType.REFRESH 该级联是级联刷新(不常用）

5. CascadeType.DETACH 该级联是级联托管（不常用）

6. CascadeType.ALL 具有上述五个级联的功能

orphanRemoval 这个属性只存在两类关系注解中 `@OneToOne` 和` @OneToMany`中，为true有级联删除的效果。mappedBy这一方可以使用。(谨慎：例如删除一个学生级联删除教室，进一步导致教室内所有学生被删除)