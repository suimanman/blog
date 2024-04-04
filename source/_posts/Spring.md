---
title: Spring

---

# IoC容器基础

## IoC理论介绍

Inversion of Control，“控制反转”，把复杂系统分解成相互合作的对象，这些对象通过封装以后，内部实现对外部透明，从而降低了解决问题的复杂度，还可以灵活的被重用和扩展。

![image-20240128114815288](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240128114815288.png)

将对象交给IoC容器进行管理，比如当我们需要一个接口的实现时，由它根据配置文件决定给我们哪一个实现类。

高内聚、低耦合，是现代软件开发的设计目标，而Spring框架给我们提供了这样的一个IoC容器进行对象的管理，一个由Spring IoC容器实例化、组装和管理的对象，成为Bean。

## spring项目、Bean注册与配置

给IoC容器编写配置文件，放在*resources* 包下面，通过配置文件告诉容器需要管理哪些Bean以及Bean的属性、依赖关系等。

*test.xml* 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean name="stu" class="org.example.entity.Student"/>
    <!-- 也可以通过name属性设置别名-->
</beans>
```

这里将实体类Student注册为Bean

```java
public class Main {
    public static void main(String[] args) {
        //配置IoC上下文，并通过Bean来获取对象
        ApplicationContext applicationContext=new ClassPathXmlApplicationContext("test.xml");
        Student student= applicationContext.getBean(Student.class);
        System.out.println(student);
    }
}
```

IoC创建的Bean对象默认是通过单例模式创建，因此多次向Bean获取同一个类时，得到的是同一个对象。

但是也可以通过scope属性修改为原型模式，每次得到的都是新对象，只在需要时对象才会被创建，而在单例模式下，只要IoC容器没有被销毁，首次创建的Bean对象会一直被容器存储。

```xml
<bean class="org.example.entity.Student" scope="prototype"/>
```

在初始时所有Bean对象就会被创建（饿汉式），可以设置为懒加载，在获取时才会被创建。

```xml
<bean class="org.example.entity.Student" scope="prototype" lazy-init="true"/>
```

## 依赖注入

### setter方法注入

在IoC配置文件中给Bean注入相应的依赖

比如有三个类，学生类、艺术老师类、编程老师类、老师类接口

```java
public interface Teacher {
}
```

```java
public class ArtTeacher implements Teacher{
}
```

```java
public class ProgramTeacher implements Teacher{
}
```

```java
import lombok.ToString;
@ToString
public class Student {
    public String name;
    public Teacher teacher;
    public void setName(String name) {
        this.name = name;
    }

    public void setTeacher(Teacher teacher) {
        this.teacher = teacher;
    }
}
```

配置文件：在*property* 中通过*name* 属性设置Studeng类中的属性值对应的值，通过value直接设置具体值，通过ref设置为其他的Bean对象。

**！！！** 注意必须对Bean对象起一个名字（即name属性），然后在property中通过name配置相应的Bean对象。

```xml
<bean name="teacher" class="org.example.entity.ArtTeacher"/><!--通过修改这里的Bean即可改变学生的老师 -->
<bean name="student" class="org.example.entity.Student">
    <property name="name" value="小明"/>
    <property name="teacher" ref="teacher"/>
</bean>
```

这样之后，在向IoC容器获取Bean对象后，相应的对象的属性就会直接被按照相应配置进行赋值。

获取对象并进行输出（这里导入了lombok依赖，并在Main上加了@ToString，能够使输出结果更直观）：

![image-20240128165255469](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240128165255469.png)

### 使用有参构造函数注入

```java
import lombok.ToString;
@ToString
public class Student {
    public String name;
    public Teacher teacher;

    public Student(String name,Teacher teacher){
        this.name=name;
        this.teacher=teacher;
    }
}
```

```xml
<bean name="teacher" class="org.example.entity.ArtTeacher"/>
    <bean name="student" class="org.example.entity.Student">
        <constructor-arg name="name" value="小明"/>
        <constructor-arg name="teacher" ref="teacher"/>
    </bean>
```

### 集合类属性的配置

```java
//map
public Map<Integer,String> map;
//list
public List<String> list;
```

```xml
<bean name="student" class="org.example.entity.Student">
        <property name="list">
            <list>
                <value>AAA</value>
                <value>BBB</value>
            </list>
        </property>
</bean>

<bean name="student" class="org.example.entity.Student">
        <property name="map">
            <map>
                <entry key="1" value="a"/>
                <entry key="2" value="b"/>
            </map>
        </property>
</bean>
```

## 自动装配

*autowire* ：byType可以自动根据类型来配置（类型是setter方法，比如setTeacher）

```xml
    <bean name="teacher" class="org.example.entity.ArtTeacher"/>
    <bean name="student" class="org.example.entity.Student" autowire="byType"/>
```

byName根据名字（也是setter方法，但是改成setArt等等）

```xml
<bean name="art" class="org.example.entity.ArtTeacher"/>
<bean name="program" class="org.example.entity.ProgramTeacher"/>
<bean name="student" class="org.example.entity.Student" autowire="byName"/>
```

构造方法也支持自动装配: autowire=“constructor"

使用byType时，如果有多个可以候选的bean对象时，可以不需要的bean对象后面手动添加配置将其去除。

```xml
<bean name="art" class="org.example.entity.ArtTeacher"/>
<bean name="program" class="org.example.entity.ArtTeacher" autowire-candidate="false"/>
<bean name="student" class="org.example.entity.Student" autowire="byType"/>
```

或者通过配置优先选择的bean对象来避免歧义

```xml
<bean name="art" class="org.example.entity.ArtTeacher" />
<bean name="program" class="org.example.entity.ArtTeacher" primary="true"/>
<bean name="student" class="org.example.entity.Student" autowire="byType"/>
```

## 生命周期与继承

### 生命周期

除了修改构造方法，我们也可以为Bean指定初始化方法和销毁方法，以便在对象创建和被销毁时执行一些其他的任务。

Student类

```java
import lombok.ToString;
@ToString
public class Student {
    public void init(){
        System.out.println("hello init!");
    }
    public void destroy(){
        System.out.println("被销毁了");
    }
}
```

IoC配置文件

```xml
<bean name="student" class="org.example.entity.Student" init-method="init" destroy-method="destroy"/>
```

调用类

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("test.xml");
        Student student= context.getBean(Student.class);
        System.out.println(student);
        context.close();//关闭会将其管理的Bean对象销毁
    }
}
```

测试结果

![image-20240128210944020](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240128210944020.png)

**！！！** 但是销毁只在**单例模式** 下才会调用，如果设置为**原型模式（懒加载）** ，只会调用初始化方法，并**不会** 调用销毁方法。

### 继承

Bean对象之间可以通过继承获取其他Bean对象的配置，但是子Bean必须包含父Bean配置的所有属性，当然，可以进行覆盖和单独的配置。

```xml
<bean name="art" class="org.example.entity.ArtTeacher">
   <property name="name" value="小明"/>
</bean>
<bean name="program" class="org.example.entity.ProgramTeacher" parent="art">
   <property name="name" value="小红"/>
   <property name="id" value="1"/>
</bean>
```

如果将一个Bean对象单独作为一个模版来用的话，可以对其进行配置。

```xml
<bean name="art" class="org.example.entity.ArtTeacher" abstract="true">
        <property name="name" value="小明"/>
</bean>
```

### 统一配置

![image-20240128215406466](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240128215406466.png)

## 工厂模式和工厂Bean

Student类

```java
@ToString
public class Student {
    Student(){
        System.out.println("你好，我是学生");
    }
}
```

工厂类(这里写的是static方法)

```java
public class StudentFactory {
    public static Student getStudent(){
        System.out.println("学生工厂");
        return new Student();
    }
}
```

**bean配置**（这里本质注册的还是Student类的Bean对象，而不是工厂）

```xml
<bean name="factory" class="org.example.entity.StudentFactory" factory-method="getStudent"/>
```

测试类（**获取bean对象时，不能写工厂的对象，而是应该写要获取的Bean对象**）

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("test.xml");
        Student student= context.getBean(Student.class);//这里很重要，不能写StudentFactory.class
        System.out.println(student);
        context.close();
    }
}
```

测试结果，可以看到bean对象是通过工厂创建的。

![image-20240128221609962](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240128221609962.png)

## 使用注解开发

配置类

```java
@Import("father.class")//在这里import父类Bean
@Configuration
public class MainConfiguration {
    @Bean(value = "name",initMethod = "",destroyMethod = "")//其他配置
    @Lazy()//懒加载，默认是true
    public Student student(){
        return new Student();
    }
}
```

测试类

```java
public class Main {
    public static void main(String[] args) {
        //这里new的对象换成注解形式
        ApplicationContext context=new AnnotationConfigApplicationContext(MainConfiguration.class);
        System.out.println(context.getBean("name"));
    }
}
```

需要注入其他Bean时，直接将其作为形参放到方法中。

```java
@Bean
public Teacher teacher(){
   return new ArtTeacher();
}

@Bean
public Student student(Teacher teacher){
   Student student=new Student();
   student.setTeacher(teacher);
   return new Student();
}
```

### 自动注入(@Autowired)

```java
@ToString
public class Student {
    @Autowired
    Teacher teacher;
}
```

配置类(不需要任何多余的配置)

```java
@Configuration
public class MainConfiguration {
    @Bean
    public Teacher teacher(){
        return new ArtTeacher();
    }

    @Bean
    public Student student(){
        return new Student();
    }
}
```

自动注入默认是byType，如果相同类型有多个Bean，可以配置name，然后在@Autowired下面加上@Qualifier("name")注解即可。

### 更加简单的Bean配置

在要注册为Bean对象的类上加注解@Component

```java
@Component("name")//也可以配置别名
@ToString
public class Student {
    @Autowired
    Teacher teacher;
}
```

在IoC配置文件中加注解ComponentScan来进行扫描，每一个Component只能扫描一个包，如果有多个包，则写ComponentScans。

```java
@ComponentScans({
        @ComponentScan("org.example.entity")
})//如果只有一个包，就单独写一个@ComponentScan就可以
@Configuration
public class MainConfiguration {
}
```

## Spring高级特性

### Bean Aware

### 异步执行

多线程执行

*注意* ：异步执行只针对于注册为Bean对象的成员方法，如果是new的对象，则执行异步方法时仍然是同步执行。

IoC配置文件（@EnableAsync）

```java
@EnableAsync//加这个注解
@ComponentScans({
        @ComponentScan("org.example.entity")
})
@Configuration
public class MainConfiguration {

}
```

注册为Bean的类（@Async）

```java
@Component
@ToString
public class Student {
    public void syncTest() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"我是同步执行的方法,开始");
        Thread.sleep(3000);
        System.out.println("我是同步执行的方法,结束");
    }
    @Async//在需要异步执行的方法上加这个注解
    public void asyncTest() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"我是异步执行的方法,开始");
        Thread.sleep(3000);
        System.out.println("我是异步执行的方法,结束");
    }
}
```

测试类

* 执行同步执行的方法

  ```java
  public static void main(String[] args) throws InterruptedException {
          ApplicationContext context=new AnnotationConfigApplicationContext(MainConfiguration.class);
          Student student=context.getBean(Student.class);
          student.syncTest();
          System.out.println("main结束");
      }
  ```

  测试结果：

  ![image-20240129115814997](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240129115814997.png)

* 执行异步执行的方法

  ```java
  public static void main(String[] args) throws InterruptedException {
          ApplicationContext context=new AnnotationConfigApplicationContext(MainConfiguration.class);
          Student student=context.getBean(Student.class);
          student.asyncTest();
          System.out.println("main结束");
      }
  ```

  测试结果：

  ![image-20240129115904260](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240129115904260.png)

上面报红信息不用管，可以看到异步执行方法中的线程方法已经不是main了，而是开了另一个线程。

### 定时任务

让某方法定时执行，每隔一定时间或者在特定时间执行。

在配置上加注解@EnableScheduling

```java
@EnableScheduling
@ComponentScans({
        @ComponentScan("org.example.entity")
})
@Configuration
public class MainConfiguration {

}
```

在需要执行定时任务的类方法上加上注解@Scheduled

```java
@Scheduled(fixedRate = 2000)
    public void task(){
        System.out.println("我是定时任务"+ new Date());
    }
```

其中@Scheduled有很多参数，且必须指定‘cron’，‘fixedRate’，‘fixedDelay'的其中一个，否则无法创建定时任务。

区别：

* fixedDelay: 在上一次定时任务执行完之后，间隔多久继续执行。

* fixedRate：无论上一次定时任务有没有完成，两次任务的时间间隔。

* cron：可以通过cron表达式灵活指定任务计划。

  测试结果：

  ![image-20240129130101159](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240129130101159.png)

### 监听器

等待某个事件的触发，当事件触发时，对应事件的监听器就会被通知。

需要实现监听器接口:ApplicationListener<T> ,其中T为监听的事件，下面的方法即监听到时所触发的方法。

```java
public class EventListen implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("监听到事件");
    }
}
```

## SpringEL表达式

###  外部属性注入

可以读入properties配置文件里面的属性配置

test.properties：

```java
test.name=test
```

IoC配置文件加上注解读取配置文件：

```java
@PropertySource("classpath:test.properties")
```

需要配置的类的属性上加注解@Value，写上要注入的属性

```java
@Value("${test.name}")
    String name;
```

## AOP面向切面

### xml配置

application.xml配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
</beans>
```

导入依赖：

```xml
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-aop</artifactId>
   <version>6.1.2</version>
</dependency>
```

将要插入的切点类和插入的切面类都注册为Bean对象，用execution指定切点方法，可以指定为一个包、类、方法等。

```xml
<bean class="org.example.entity.Student"/>
<bean id="studentAop" class="org.example.entity.StudentAop"/>

<aop:config>
  <aop:pointcut id="point" expression="execution(* org.example.entity.Student.test())"/>
  <aop:aspect ref="studentAop"><!--指定切面方法，下面是指定切入方式 -->
        <aop:after method="afterStudy" pointcut-ref="point"></aop:after>
   </aop:aspect>
</aop:config>
```

切点也可以通过JoinPoint 拿到切面传入的参数。

比如在测试类中传入String类型参数：

```java
public void test(String Str){
        System.out.println("测试方法");
    }
```

切点类：

```java
public void afterStudy(JoinPoint joinPoint){

        System.out.println("Aop切面参数："+joinPoint.getArgs()[0]);//这里getArgs方法就是获取参数，0即第一个参数
    }
```

测试类：

```java
public static void main(String[] args) throws InterruptedException {
        ApplicationContext context=new ClassPathXmlApplicationContext("application.xml");
        Student student=context.getBean(Student.class);
        student.test("切面参数");
    }
```

测试结果：

![image-20240129165729629](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240129165729629.png)

aroud切点：环绕方法执行切点，需要在切点方法中手动调用切面方法，对于有参数的切面，可以对其参数进行修改，对于有返回值的切面，也必须要有返回值。

```java
public Object afterStudy(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("方法开始之前");
        String arg=joinPoint.getArgs()[0]+"新加参数";
        Object value=joinPoint.proceed(new Object[]{arg});
        System.out.println("方法执行结果为："+value);
        return value;
    }
```

测试结果：

![image-20240129174431760](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240129174431760.png)

### 接口配置

advice

### 注解配置

IoC配置文件加上注解@EnableAspectJAutoProxy

在切点类中加相应注解：

```java
@Aspect
@Component
public class StudentAop {
    @After("execution(* org.example.entity.Student.test())")//指定切面
    public void afterStudy(){
        System.out.println("我是后置");
    }
}
```

