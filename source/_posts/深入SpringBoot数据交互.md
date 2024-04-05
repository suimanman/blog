---
title: 深入SpringBoot数据交互
date: 2024-02-08 09:25:00
author: 岁漫
img: /images/2.jpg
top: true
hide: false
cover: true
coverImg: /images/2.jpg
toc: false
mathjax: false
summary: 深入SpringBoot数据交互
categories: Markdown
tags:
  - java后端
  - SpringBoot
---

# JPA

## 什么是JPA

JPA（Java Persistence API）和JDBC类似，也是官方定义的一组接口，但是它相比传统的JDBC，它是为了实现ORM而生的，即Object-Relationl Mapping，它的作用是在关系型数据库和对象之间形成一个映射，这样，我们在具体的操作数据库的时候，就不需要再去和复杂的SQL语句打交道，只要像平时操作对象一样操作它就可以了。

其中比较常见的JPA实现有：

1. Hibernate：Hibernate是JPA规范的一个具体实现，也是目前使用最广泛的JPA实现框架之一。它提供了强大的对象关系映射功能，可以将Java对象映射到数据库表中，并提供了丰富的查询语言和缓存机制。
2. EclipseLink：EclipseLink是另一个流行的JPA实现框架，由Eclipse基金会开发和维护。它提供了丰富的特性，如对象关系映射、缓存、查询语言和连接池管理等，并具有较高的性能和可扩展性。
3. OpenJPA：OpenJPA是Apache基金会的一个开源项目，也是JPA规范的一个实现。它提供了高性能的JPA实现和丰富的特性，如延迟加载、缓存和分布式事务等。
4. TopLink：TopLink是Oracle公司开发的一个对象关系映射框架，也是JPA规范的一个实现。虽然EclipseLink已经取代了TopLink成为Oracle推荐的JPA实现，但TopLink仍然得到广泛使用。

在之前，我们使用JDBC或是Mybatis来操作数据，通过直接编写对应的SQL语句来实现数据访问，但是我们发现实际上我们在Java中大部分操作数据库的情况都是读取数据并封装为一个实体类，因此，为什么不直接将实体类直接对应到一个数据库表呢？也就是说，一张表里面有什么属性，那么我们的对象就有什么属性，所有属性跟数据库里面的字段一一对应，而读取数据时，只需要读取一行的数据并封装为我们定义好的实体类既可以，而具体的SQL语句执行，完全可以交给框架根据我们定义的映射关系去生成，不再由我们去编写，因为这些SQL实际上都是千篇一律的。

而实现JPA规范的框架一般最常用的就是`Hibernate`，它是一个重量级框架，学习难度相比Mybatis也更高一些，而SpringDataJPA也是采用Hibernate框架作为底层实现，并对其加以封装。

官网：https://spring.io/projects/spring-data-jpa

## 快速上手

导入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

接着我们可以直接创建一个类，比如用户类，我们只需要把一个账号对应的属性全部定义好即可：

```java
@Data
public class Account {
    int id;
    String username;
    String password;
}
```

接着，我们可以通过注解形式，在属性上添加数据库映射关系，这样就能够让JPA知道我们的实体类对应的数据库表长啥样，这里用到了很多注解：

```java
@Data
@Entity   //表示这个类是一个实体类
@Table(name = "account")    //对应的数据库中表名称
public class Account {

    @GeneratedValue(strategy = GenerationType.IDENTITY)   //生成策略，这里配置为自增
    @Column(name = "id")    //对应表中id这一列
    @Id     //此属性为主键
    int id;

    @Column(name = "username")   //对应表中username这一列
    String username;

    @Column(name = "password")   //对应表中password这一列
    String password;
}
```

接着我们来修改一下配置文件，把日志打印给打开：

```yaml
spring:
  jpa:
    #开启SQL语句执行日志信息
    show-sql: true
    hibernate:
      #配置为检查数据库表结构，没有时会自动创建
      ddl-auto: update
```

`ddl-auto`属性用于设置自动表定义，可以实现自动在数据库中为我们创建一个表，表的结构会根据我们定义的实体类决定，它有以下几种：

- `none`: 不执行任何操作，数据库表结构需要手动创建。
- `create`: 框架在每次运行时都会删除所有表，并重新创建。
- `create-drop`: 框架在每次运行时都会删除所有表，然后再创建，但在程序结束时会再次删除所有表。
- `update`: 框架会检查数据库表结构，如果与实体类定义不匹配，则会做相应的修改，以保持它们的一致性。
- `validate`: 框架会检查数据库表结构与实体类定义是否匹配，如果不匹配，则会抛出异常。

这个配置项的作用是为了避免手动管理数据库表结构，使开发者可以更方便地进行开发和测试，但在生产环境中，更推荐使用数据库迁移工具来管理表结构的变更。

我们可以在日志中发现，在启动时执行了如下SQL语句：

![image-20230720235136506](https://image.itbaima.cn/markdown/2023/07/20/kABZVhJ8vjKSqzT.png)

我们的数据库中对应的表已经自动创建好了。

我们接着来看如何访问我们的表，我们需要创建一个Repository实现类：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
}
```

注意JpaRepository有两个泛型，前者是具体操作的对象实体，也就是对应的表，后者是ID的类型，接口中已经定义了比较常用的数据库操作。编写接口继承即可，我们可以直接注入此接口获得实现类：

```java
@Resource
AccountRepository repository;

@Test
void contextLoads() {
    Account account = new Account();
    account.setUsername("小红");
    account.setPassword("1234567");
    System.out.println(repository.save(account).getId());   //使用save来快速插入数据，并且会返回插入的对象，如果存在自增ID，对象的自增id属性会自动被赋值，这就很方便了
}
```

![image-20230720235640148](https://image.itbaima.cn/markdown/2023/07/20/ksI3J5eidzTrvyL.png)

同时，查询操作也很方便：

```java
@Test
void contextLoads() {
  	//默认通过通过ID查找的方法，并且返回的结果是Optional包装的对象，非常人性化
    repository.findById(1).ifPresent(System.out::println);
}
```

![image-20230720235949290](https://image.itbaima.cn/markdown/2023/07/20/TRHOWbop267Al4Q.png)

包括常见的一些计数、删除操作等都包含在里面，仅仅配置应该接口就能完美实现增删改查：

![image-20230721000050875](https://image.itbaima.cn/markdown/2023/07/21/uIBciLqFsH5tdDR.png)

## 方法名称拼接自定义SQL

虽然接口预置的方法使用起来非常方便，但是如果我们需要进行条件查询等操作或是一些判断，就需要自定义一些方法来实现，同样的，我们不需要编写SQL语句，而是通过方法名称的拼接来实现条件判断，这里列出了所有支持的条件判断名称：

| 属性               | 拼接方法名称示例                                             | 执行的语句                                                   |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Distinct           | findDistinctByLastnameAndFirstname                           | select distinct … where x.lastname = ?1 and x.firstname = ?2 |
| And                | findByLastnameAndFirstname                                   | … where x.lastname = ?1 and x.firstname = ?2                 |
| Or                 | findByLastnameOrFirstname                                    | … where x.lastname = ?1 or x.firstname = ?2                  |
| Is，Equals         | findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals  | … where x.firstname = ?1                                     |
| Between            | findByStartDateBetween                                       | … where x.startDate between ?1 and ?2                        |
| LessThan           | xxxxxxxxxx <input type="checkbox" name="remember" class="ad-checkbox">html | … where x.age < ?1                                           |
| LessThanEqual      | findByAgeLessThanEqual                                       | … where x.age <= ?1                                          |
| GreaterThan        | findByAgeGreaterThan                                         | … where x.age > ?1                                           |
| GreaterThanEqual   | findByAgeGreaterThanEqual                                    | … where x.age >= ?1                                          |
| After              | findByStartDateAfter                                         | … where x.startDate > ?1                                     |
| Before             | findByStartDateBefore                                        | … where x.startDate < ?1                                     |
| IsNull，Null       | findByAge(Is)Null                                            | … where x.age is null                                        |
| IsNotNull，NotNull | findByAge(Is)NotNull                                         | … where x.age not null                                       |
| Like               | findByFirstnameLike                                          | … where x.firstname like ?1                                  |
| NotLike            | findByFirstnameNotLike                                       | … where x.firstname not like ?1                              |
| StartingWith       | findByFirstnameStartingWith                                  | … where x.firstname like ?1（参数与附加`%`绑定）             |
| EndingWith         | findByFirstnameEndingWith                                    | … where x.firstname like ?1（参数与前缀`%`绑定）             |
| Containing         | findByFirstnameContaining                                    | … where x.firstname like ?1（参数绑定以`%`包装）             |
| OrderBy            | findByAgeOrderByLastnameDesc                                 | … where x.age = ?1 order by x.lastname desc                  |
| Not                | findByLastnameNot                                            | … where x.lastname <> ?1                                     |
| In                 | findByAgeIn(Collection ages)                                 | … where x.age in ?1                                          |
| NotIn              | findByAgeNotIn(Collection ages)                              | … where x.age not in ?1                                      |
| True               | findByActiveTrue                                             | … where x.active = true                                      |
| False              | findByActiveFalse                                            | … where x.active = false                                     |
| IgnoreCase         | findByFirstnameIgnoreCase                                    | … where UPPER(x.firstname) = UPPER(?1)                       |

比如我们想要实现根据用户名模糊匹配查找用户：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    //按照表中的规则进行名称拼接，不用刻意去记，IDEA会有提示
    List<Account> findAllByUsernameLike(String str);
}
```

我们来测试一下：

```java
@Test
void contextLoads() {
    repository.findAllByUsernameLike("%明%").forEach(System.out::println);
}
```

![image-20230721001035279](https://image.itbaima.cn/markdown/2023/07/21/mioZaUk7Yj3QDxb.png)

又比如我们想同时根据用户名和ID一起查询：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    List<Account> findAllByUsernameLike(String str);

    Account findByIdAndUsername(int id, String username);
    //也可以使用Optional类进行包装，Optional<Account> findByIdAndUsername(int id, String username);
}
```

```java
@Test
void contextLoads() {
    System.out.println(repository.findByIdAndUsername(1, "小明"));
}
```

比如我们想判断数据库中是否存在某个ID的用户：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    List<Account> findAllByUsernameLike(String str);
    Account findByIdAndUsername(int id, String username);
    //使用exists判断是否存在
    boolean existsAccountById(int id);
}
```

注意自定义条件操作的方法名称一定要遵循规则，不然会出现异常：

```sh
Caused by: org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract  ...
```

# 关联查询

我们知道，在JPA中，每张表实际上就是一个实体类的映射，而表之间的关联关系，也可以看作对象之间的依赖关系，比如用户表中包含了用户详细信息的ID字段作为外键，那么实际上就是用户表实体中包括了用户详细信息实体对象：

```java
@Data
@Entity
@Table(name = "users_detail")
public class AccountDetail {

    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    int id;

    @Column(name = "address")
    String address;

    @Column(name = "email")
    String email;

    @Column(name = "phone")
    String phone;

    @Column(name = "real_name")
    String realName;
}
```

## 一对一

而用户信息和用户详细信息之间形成了一对一的关系，那么这时我们就可以直接在类中指定这种关系：

```java
@Data
@Entity
@Table(name = "users")
public class Account {

    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    @Id
    int id;

    @Column(name = "username")
    String username;

    @Column(name = "password")
    String password;

    @JoinColumn(name = "detail_id")   //指定存储外键的字段名称
    @OneToOne    //声明为一对一关系
    AccountDetail detail;
}
```

在修改实体类信息后，我们发现在启动时也进行了更新，日志如下：

![image-20240208205410698](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240208205410698.png)

接着我们往用户详细信息中添加一些数据，一会我们可以直接进行查询：

```java
@Test
void pageAccount() {
    repository.findById(1).ifPresent(System.out::println);
}
```

查询后，可以发现，得到如下结果：

```
Hibernate: select account0_.id as id1_0_0_, account0_.detail_id as detail_i4_0_0_, account0_.password as password2_0_0_, account0_.username as username3_0_0_, accountdet1_.id as id1_1_1_, accountdet1_.address as address2_1_1_, accountdet1_.email as email3_1_1_, accountdet1_.phone as phone4_1_1_, accountdet1_.real_name as real_nam5_1_1_ from users account0_ left outer join users_detail accountdet1_ on account0_.detail_id=accountdet1_.id where account0_.id=?
Account(id=1, username=Test, password=123456, detail=AccountDetail(id=1, address=四川省成都市青羊区, email=8371289@qq.com, phone=1234567890, realName=本伟))
```

也就是，在建立关系之后，我们查询Account对象时，会自动将关联数据的结果也一并进行查询。

那要是我们只想要Account的数据，不想要用户详细信息数据怎么办呢？我希望在我要用的时候再获取详细信息，这样可以节省一些网络开销，我们可以设置懒加载，这样只有在需要时才会向数据库获取：

```java
@JoinColumn(name = "detail_id")
@OneToOne(fetch = FetchType.LAZY)    //将获取类型改为LAZY
AccountDetail detail;
```

接着我们测试一下：

```java
@Transactional   //懒加载属性需要在事务环境下获取，因为repository方法调用完后Session会立即关闭
@Test
void pageAccount() {
    repository.findById(1).ifPresent(account -> {
        System.out.println(account.getUsername());   //获取用户名
        System.out.println(account.getDetail());  //获取详细信息（懒加载）
    });
}
```

接着我们来看看控制台输出了什么：

```java
Hibernate: select account0_.id as id1_0_0_, account0_.detail_id as detail_i4_0_0_, account0_.password as password2_0_0_, account0_.username as username3_0_0_ from users account0_ where account0_.id=?
Test
Hibernate: select accountdet0_.id as id1_1_0_, accountdet0_.address as address2_1_0_, accountdet0_.email as email3_1_0_, accountdet0_.phone as phone4_1_0_, accountdet0_.real_name as real_nam5_1_0_ from users_detail accountdet0_ where accountdet0_.id=?
AccountDetail(id=1, address=四川省成都市青羊区, email=8371289@qq.com, phone=1234567890, realName=卢本)
```

可以看到，获取用户名之前，并没有去查询用户的详细信息，而是当我们获取详细信息时才进行查询并返回AccountDetail对象。

那么我们是否也可以在添加数据时，利用实体类之间的关联信息，一次性添加两张表的数据呢？可以，但是我们需要稍微修改一下级联关联操作设定：

```java
@JoinColumn(name = "detail_id")
@OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL) //设置关联操作为ALL
AccountDetail detail;
```

- ALL：所有操作都进行关联操作
- PERSIST：插入操作时才进行关联操作
- REMOVE：删除操作时才进行关联操作
- MERGE：修改操作时才进行关联操作

可以多个并存，接着我们来进行一下测试：

```java
@Test
void addAccount(){
    Account account = new Account();
    account.setUsername("Nike");
    account.setPassword("123456");
    AccountDetail detail = new AccountDetail();
    detail.setAddress("重庆市渝中区解放碑");
    detail.setPhone("1234567890");
    detail.setEmail("73281937@qq.com");
    detail.setRealName("张三");
  	account.setDetail(detail);
    account = repository.save(account);
    System.out.println("插入时，自动生成的主键ID为："+account.getId()+"，外键ID为："+account.getDetail().getId());
}
```

可以看到日志结果：

```java
Hibernate: insert into users_detail (address, email, phone, real_name) values (?, ?, ?, ?)
Hibernate: insert into users (detail_id, password, username) values (?, ?, ?)
插入时，自动生成的主键ID为：6，外键ID为：3
```

结束后会发现数据库中两张表都同时存在数据。

## 一对多

接着我们来看一对多关联，比如每个用户的成绩信息：

```java
@JoinColumn(name = "uid")  //注意这里的name指的是Score表中的uid字段对应的就是当前的主键，会将uid外键设置为当前的主键
@OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.REMOVE)   //在移除Account时，一并移除所有的成绩信息，依然使用懒加载
List<Score> scoreList;
```

```java
@Data
@Entity
@Table(name = "users_score")   //成绩表，注意只存成绩，不存学科信息，学科信息id做外键
public class Score {

    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    @Id
    int id;

    @OneToOne   //一对一对应到学科上
    @JoinColumn(name = "cid")
    Subject subject;

    @Column(name = "socre")
    double score;

    @Column(name = "uid")
    int uid;
}
```

```java
@Data
@Entity
@Table(name = "subjects")   //学科信息表
public class Subject {

    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cid")
    @Id
    int cid;

    @Column(name = "name")
    String name;

    @Column(name = "teacher")
    String teacher;

    @Column(name = "time")
    int time;
}
```

在数据库中填写相应数据，接着我们就可以查询用户的成绩信息了：

```java
@Transactional
@Test
void test() {
    repository.findById(1).ifPresent(account -> {
        account.getScoreList().forEach(System.out::println);
    });
}
```

成功得到用户所有的成绩信息，包括得分和学科信息。

## 多对一

同样的，我们还可以将对应成绩中的教师信息单独分出一张表存储，并建立多对一的关系，因为多门课程可能由同一个老师教授（千万别搞晕了，一定要理清楚关联关系，同时也是考验你的基础扎不扎实）：

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "tid")   //存储教师ID的字段，和一对一是一样的，也会当前表中创个外键
Teacher teacher;
```

接着就是教师实体类了：

```java
@Data
@Entity
@Table(name = "teachers")
public class Teacher {

    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    int id;

    @Column(name = "name")
    String name;

    @Column(name = "sex")
    String sex;
}
```

最后我们再进行一下测试：

```java
@Transactional
@Test
void test() {
    repository.findById(3).ifPresent(account -> {
        account.getScoreList().forEach(score -> {
            System.out.println("课程名称："+score.getSubject().getName());
            System.out.println("得分："+score.getScore());
            System.out.println("任课教师："+score.getSubject().getTeacher().getName());
        });
    });
}
```

## 多对多

最后我们再来看最复杂的情况，现在我们一门课程可以由多个老师教授，而一个老师也可以教授多个课程，那么这种情况就是很明显的多对多场景，现在又该如何定义呢？我们可以像之前一样，插入一张中间表表示教授关系，这个表中专门存储哪个老师教哪个科目：

```java
@ManyToMany(fetch = FetchType.LAZY)   //多对多场景
@JoinTable(name = "teach_relation",     //多对多中间关联表
        joinColumns = @JoinColumn(name = "cid"),    //当前实体主键在关联表中的字段名称
        inverseJoinColumns = @JoinColumn(name = "tid")   //教师实体主键在关联表中的字段名称
)
List<Teacher> teacher;
```

接着，JPA会自动创建一张中间表，并自动设置外键，我们就可以将多对多关联信息编写在其中了。

# Mybatis Plus

## 快速上手

添加依赖

```xml
<dependency>
     <groupId>com.baomidou</groupId>
     <artifactId>mybatis-plus-boot-starter</artifactId>
     <version>3.5.3.1</version>
</dependency>
<dependency>
     <groupId>com.mysql</groupId>
     <artifactId>mysql-connector-j</artifactId>
</dependency>
```

配置文件数据源：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

实体类，可以直接映射到数据库中的表：

```java
@Data
@TableName("user")  //对应的表名
public class User {
    @TableId(type = IdType.AUTO)   //对应的主键
    int id;
    @TableField("name")   //对应的字段
    String name;
    @TableField("email")
    String email;
    @TableField("password")
    String password;
}
```

编写一个Mapper

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
  	//使用方式与JPA极其相似，同样是继承一个基础的模版Mapper
  	//这个模版里面提供了预设的大量方法直接使用，跟JPA如出一辙
}
```

写一个简单测试用例：

```java
@SpringBootTest
class DemoApplicationTests {

    @Resource
    UserMapper mapper;

    @Test
    void contextLoads() {
        System.out.println(mapper.selectById(1));  //同样可以直接selectById，非常快速方便
    }
}
```

## 条件构造器

对于一些复杂查询的情况，MybatisPlus支持我们自己构造QueryWrapper用于复杂条件查询：

```java
@Test
void contextLoads() {
    QueryWrapper<User> wrapper = new QueryWrapper<>();    //复杂查询可以使用QueryWrapper来完成
  	wrapper
            .select("id", "name", "email", "password")    //可以自定义选择哪些字段
            .ge("id", 2)     			//选择判断id大于等于1的所有数据
            .orderByDesc("id");   //根据id字段进行降序排序
    System.out.println(mapper.selectList(wrapper));   //Mapper同样支持使用QueryWrapper进行查询
}
```

通过使用上面的QueryWrapper对象进行查询，也就等价于下面的SQL语句：

```sql
select id,name,email,password from user where id >= 2 order by id desc
```

我们可以在配置中开启SQL日志打印：

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

最后得到的结果如下：

![image-20230721160951500](https://image.itbaima.cn/markdown/2023/07/21/FxOfrnERhVPi8tu.png)

有些时候我们遇到需要批处理的情况，也可以直接使用批处理操作：

```java
@Test
void contextLoads() {
    //支持批处理操作，我们可以一次性删除多个指定ID的用户
    int count = mapper.deleteBatchIds(List.of(1, 3));
    System.out.println(count);
}
```

![image-20230721190139253](https://image.itbaima.cn/markdown/2023/07/21/lwaJUF3g2opbWZG.png)

我们也可以快速进行分页查询操作，不过在执行前我们需要先配置一下：

```java
@Configuration
public class MybatisConfiguration {
    @Bean
    public MybatisPlusInterceptor paginationInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
      	//添加分页拦截器到MybatisPlusInterceptor中
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

这样我们就可以愉快地使用分页功能了：

```java
@Test
void contextLoads() {
    //这里我们将用户表分2页，并获取第一页的数据
    Page<User> page = mapper.selectPage(Page.of(1, 2), Wrappers.emptyWrapper());
    System.out.println(page.getRecords());   //获取分页之后的数据
}
```

![image-20230721185519292](https://image.itbaima.cn/markdown/2023/07/21/XMPLWB3N6VpHUkG.png)

对于数据更新操作，我们也可以使用UpdateWrapper非常方便的来完成：

```java
@Test
void contextLoads() {
    UpdateWrapper<User> wrapper = new UpdateWrapper<>();
    wrapper
            .set("name", "lbw")
            .eq("id", 1);
    System.out.println(mapper.update(null, wrapper));
}
```

![image-20230721162409308](https://image.itbaima.cn/markdown/2023/07/21/W1e8fFuUwSpi7Cg.png)

## 数据校验

导入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

接口上添加注解@Valid

```java
@PostMapping("list")
    public DaoCaoResult submitMenuList(@Valid @RequestBody UmsMenu umsMenu){
        return DaoCaoResult.success(menuService.saveMenu(umsMenu));
    }
```

实体类中字段上添加注解：

```java
@NotNull(message = "请填写菜单名")
    private Long parentId;
```



# 接口基本操作

虽然使用MybatisPlus提供的BaseMapper已经很方便了，但是我们的业务中，实际上很多时候也是一样的工作，都是去简单调用底层的Mapper做一个很简单的事情，那么能不能干脆把Service也给弄个模版？MybatisPlus为我们提供了很方便的CRUD接口，直接实现了各种业务中会用到的增删改查操作。

我们只需要继承即可：

```java
@Service
public interface UserService extends IService<User> {
  	//除了继承模版，我们也可以把它当成普通Service添加自己需要的方法
}
```

接着我们还需要编写一个实现类，这个实现类就是UserService的实现：

```java
@Service   //需要继承ServiceImpl才能实现那些默认的CRUD方法
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

使用起来也很方便，整合了超多方法：

![image-20230721181359616](https://image.itbaima.cn/markdown/2023/07/21/l5Vkb9dgtJcyL4R.png)

比如我们想批量插入一组用户数据到数据库中：

```java
@Test
void contextLoads() {
    List<User> users = List.of(new User("xxx"), new User("yyy"));
  	//预设方法中已经支持批量保存了，这相比我们直接用for效率高不少
    service.saveBatch(users);
}
```

还有更加方便快捷的保存或更新操作，当数据不存在时（通过主键ID判断）则插入新数据，否则就更新数据：

```java
@Test
void contextLoads() {
    service.saveOrUpdate(new User("aaa"));
}
```

我们也可以直接使用Service来进行链式查询，写法非常舒服：

```java
@Test
void contextLoads() {
    User one = service.query().eq("id", 1).one();
    System.out.println(one);
}
```

# 新版代码生成器

它能够根据数据库做到代码的一键生成，能做到什么程度呢？

![image-20230721200757985](https://image.itbaima.cn/markdown/2023/07/21/lGT4g5Y6Heqavsw.png)

导入依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3.1</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.3.1</version>
</dependency>
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.3</version>
</dependency>
```

配置一下数据源：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

接着我们就可以开始编写自动生成脚本了，这里依然选择测试类，用到`FastAutoGenerator`作为生成器：

```java
@Test
    void contextLoads() {
        FastAutoGenerator
          			//首先使用create来配置数据库链接信息
                .create(new DataSourceConfig.Builder(dataSource))
                .execute();
    }
```

接着我们配置一下全局设置，这些会影响一会生成的代码：

```java
@Test
void contextLoads() {
    FastAutoGenerator
            .create(new DataSourceConfig.Builder(dataSource))
            .globalConfig(builder -> {
                builder.author("lbw");              //作者信息，一会会变成注释
                builder.commentDate("2024-01-01");  //日期信息，一会会变成注释
                builder.outputDir("src/main/java"); //输出目录设置为当前项目的目录
            })
            .execute();
}
```

然后是打包设置，也就是项目的生成的包等等，这里简单配置一下：

```java
@Test
void contextLoads() {
    FastAutoGenerator
            ...
      			//打包设置，这里设置一下包名就行，注意跟我们项目包名设置为一致的
      			.packageConfig(builder -> builder.parent("com.example"))
      			.strategyConfig(builder -> {
                    //设置为所有Mapper添加@Mapper注解
                    builder
                            .mapperBuilder()
                            .mapperAnnotation(Mapper.class)
                            .build();
            })
            .execute();
}
```

接着我们就可以直接执行了这个脚本了：

![image-20230721203819514](https://image.itbaima.cn/markdown/2023/07/21/SdDRqZPnNrkeKjG.png)

现在，可以看到我们的项目中已经出现自动生成代码了：

![image-20230721204011913](https://image.itbaima.cn/markdown/2023/07/21/pKMnwFZEOBmLXDy.png)
