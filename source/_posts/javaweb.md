---
title: javaweb

---

# Lombok

**@Getter @Setter**
加上这三个注解后会自动生成相应属性的**get**和**set**方法，不用再手写
_**@tostring** _
_编译之后_：
```java
public String toString{
    return "Student(sid"+this.getSid()+",name"+this.getName();
}
```
**@EqualsAndHashQode**
重写了equals和hash,并且Equals方法不会比较父类，只比较子类（如果有继承关系）
**@AllArgsConstructor**
自动写了有参（全参数）构造函数
**@NoArgsConstructor**
无参构造函数
**@RequiredArgsConstructor**
默认是无参构造，如果有final类型的属性，就会变成只含该属性的有参构造函数
**@Data**
包含上述所有（除了**@AllArgsContructor**和 **@NoArgsContructor**）注解，不建议有继承关系。
**@Value**
和**@Data**类似，只是编译之后所有的属性都变成了_private final_类型，然后不会生成_setter_。
**@SneakyThrows**
自动加上_try catch_语句进行异常处理。
**@Cleanup**
编译自动加上close语句，比如文件的close。
**@Builder**
建造者模式（一步步构造），调用builder方法来构造对象。
# Mybatis
## mybatis.xml配置
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <!-- 运行环境配置，即数据源配置，default为默认配置 -->
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <!-- 驱动 -->
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <!-- 数据库路径，可以从idea里面的数据库配置里看到-->
        <property name="url" value="jdbc::mysql://localhost:3306/test"/>
        <property name="username" value="root"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    <!-- 如果mapper很多，可以用包定义
      <package name="com.test.mapper"/>
    -->
  </mappers>
</configuration>
```
## 用xml文件来配置mapper的sql语句
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace用来区分不同的mapper -->
<mapper namespace="org.mybatis.example.BlogMapper">
  <!-- id是方法名，resultType是返回值类型 -->
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>

  <!-- 如果返回值类型是一个对象，或者有多个返回值，需要对返回值类型做封装 -->
	<select id="getById" resultMap="asTeacher">
    select *,teacher.name as tname from student,teach,teacher where 
    student.sid=teach.sid and teach.tid=teacher.tid and teach.id=#{tid}
  </select>

  <resultMap id="asTeacher" type="Teacher">
    <id column="tid" property="tid"/>
    <result column="name" property="name"/>
    <!-- 对象返回值映射,并通过collection将查询的所有结果并为一个集合，ofType是list的对象类型 -->
    <collection property="studentList" ofType="Student">
      <id column="sid" property="sid"/>
      <result column="name" property="name"/>
      <result column="sex" property="sex"/>
    </collection>
    <!--	如果查询结果只包含一个对象结果，用association做映射
        <association property="teacher" javaType="Teacher">
          <id column="tid" property="tid"/>
          <result column="tname" property="name"/>
  			</association>
    -->
  </resultMap>
</mapper>
```
## 动态sql
根据不同条件来拼接sql语句
```xml
<select id="selectById" resultType="Student">
	select * from student where sid = #{sid}
  <if test=" sid%2==0">
    and sex='男'
  </if>
</select>
```
```xml
<select id="selectById" resultType="Student">
	select * from student where sid = #{sid}
  <choose>
    <when test="sid==1">
      and sex ='女'
    </when>
    <when test="sid==2">
      and sex = '男'
    </when>
    <otherwise>
      and sex = '男'
    </otherwise>
  </choose>
</select>
```
## 缓存机制
分为一级缓存和二级缓存，默认开启一级缓存（只能调整不能关闭），并且每一个会话都包含唯一一个一级缓存，互不干扰。
一级缓存：发生增删改时会自动清除缓存，避免发生缓存与数据库数据不一致的问题。
二级缓存可以手动开启，开启之后，对于同一个mapper下的所有会话都共用一个二级缓存。
二级缓存避免了同一个mapper下的不同会话（不同的进程）对同一个数据库进行的增删改操作导致其他进程的缓存与数据库数据不一致的问题，[因为一级缓存中，不同会话之间不会相互影响，导致某一个进程对数据进行修改操作后，其他进程还是从缓存中读取数据]。
但是，这个缓存清除的机制是必须要等待事务结束才能清除缓存（也就是二级缓存）中的内容，所以如果说，在一个会话中开启另一个会话，并在外面的会话结束事务之前进行查询操作，还是会导致缓存与数据库数据不一致的问题。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40925307/1706248766924-1bde9fd7-60c0-4f83-9b85-3c5b16952f66.png#averageHue=%23e1ebb2&clientId=u98efc3ff-4517-4&from=paste&height=368&id=ub5b4e685&originHeight=368&originWidth=938&originalType=binary&ratio=1&rotation=0&showTitle=false&size=91779&status=done&style=none&taskId=uf3b893a8-4473-4e50-a543-52ce59c44e5&title=&width=938)
## 注解开发mapper
```java

public Teacher{
    int sid,
    int tid,
    List<Students> studentList
}
```
```java
@Results({
    @Results(column = "tid", property="tid"),
    @Results(column = "name", property="name"),
    <!-- 下面的column是传入的参数，注解@Many是查询集合结果，@One是查询单个结果 -->
    @Results(column = "tid", property="studentList", many=
             <!-- getStudentById就是调用的查询方法 -->
             @Many(select = "getStudentById")
            )
})
@Select("select * from teacher where tid =#{tid}")
Teacher getTeacherById(int tid);

@Select("select * from  student,teach where student.sid=teach.sid and teach.tid=#{tid}")
List<Student> getStudentById(int tid);
```
如果需要传入多个参数
```java
@Select("select * from student where sid =#{sid} and sex=#{s}")
Teacher getTeacherById(@Param("sid") int sid,@Param("s") String sex);
```
如果传入的参数是对象
```java
@Insert("insert into student(sid,name,sex) values (#{sid},#{student.name},#{student.sex})")
int addStudent(@Param("sid") int sid,@Param("student") Student student);
```
# JUnit单元测试框架
## @Test
```java
public class TestMain{
	@Test
    public void method(){
        System.out.println("测试");
    }
}
```
_注意_：

1. 方法必须是public的
2. 不能是静态方法
3. 返回值必须是void
4. 必须是没有任何参数的方法
## 断言
```java

public class TestMain{
	@Test
    public void method(){
        System.out.println("测试");
        // 断言表达式，参数1.message 表示测试错误打印的信息，参数2.excepted 期盼值 参数3.actual 实际测试结果
        Assert.assertEquals("断言表达式",1,2);
    }
}
```
```java
Student stu=mapper.getStuByIdAndSex(1,"男");
Assert.assertEquals(new Student().setName("小明").setSex("男").setId(1),student);
```
## 开始和结束都要进行的动作
对需要执行的方法加上注解**@Before**或者**@After**
# Maven
## 依赖作用域

- **type**: 依赖的类型，对于项目坐标定义的packaging，默认是jar
- **scope**: 依赖的范围
- **optional**：标记依赖是否可选
- **exclusions**:用来排除依赖传递性
### scope

- **compile：**默认范围，编译、运行、测试均有效。
- **provided**：编译测试有效，运行无效（在项目运行时，不需要此依赖），比如lombok，在编译阶段所需已经转换为相应的代码了。
- **runtime**: 在运行、测试时有效，编译时无效
- **test**:	只在测试时有效，比如JUnit
- **system**: 导入本地jar包
```xml
<dependency>
  <groupId>javax.jntm</groupId>
  <artifactId>hhhh</artifactId>
  <version>2.0</version>
  <scope>system</scope>
  <systemPath>C://ss/test.jar</systemPath>
</dependency>
```
## 继承
父类将可以直接继承给子类的放入_<dependencies>_，将需要统一管理，然后子类选择去继承的放入_<dependencyManagement>_
## 常用命令
IDEA右上角有一个生命周期，是Maven的一些插件，有不同的功能。
### clean
执行后会清理掉整个target文件，在之后编写Springboot项目是可以解决一些缓存没更新的问题。
### validate
验证项目的可用性
### compile
将项目编译为.class文件
### install
可以将当前项目安装到本地仓库，以供其他项目导入作为依赖使用
### verify
按顺序执行每个默认生命周期阶段（validate,compile,package等）
### test
运行测试类
### xxxxxxxxxx <input type="checkbox" name="remember" class="ad-checkbox">html
打包成jar包（在同级目录下输入 java -jar xxxx.jar 来运行打包好的jar可执行程序）
