# MyBaits3培训

---

> * MyBaits简介
> * HelloWorld实现
> * 配置文件讲解
> * * enviromenets
> * * transactionManager
> * * dataSource
> * * properties
> * * typeAliases
> * * mappers
> * XML配置SQL映射器
> * * INSERT映射语句 
> * * UPDATE映射语句
> * * DELETE映射语句
> * * SELECT映射语句
> * MyBatis关系映射
> * * 一对一关系
> * * 一对多关系
> * Mybatis动态SQL
> * * if条件
> * * choose,when,otherwrise
> * * where
> * * trim
> * * foreach
> * * set
> * MyBatis其他
> * * CLOB类型处理
> * * BLOB类型处理
> * * MyBatis分页
> * * MyBatis缓存
> * 注解配置SQL映射器
> * * 基本映射语句
> * * * @INSERT
> * * * @UPDATE
> * * * @DELETE
> * * * @SELECT
> * * 结果集映射
> * * 关系映射
> * * * 一对一关系
> * * * 一对多关系
> * * 动态SQL
> * * * @InsertProvider
> * * * @UpdateProvider
> * * * @DeleteProvider
> * * * @SelectProvider
> * MyBatis与Spring，Struts2整合
> * * Spring与Struts整合 
> * * Spring与MyBatis整合

---

## MyBaits简介

具体介绍查看百度百科，或者直接查看官网介绍。

## HelloWorld实现

我们将采用最原始的JavaProject来构建项目，已达到最原始的使用体验，具体的构建过程不再赘述，所有内容将围绕MyBatis展开

开始之前，我们需要准备一个MySql数据库，并新建一张 **t_student** 表供测试用
```sql
CREATE DATABASE  DEFAULT CHARACTER SET utf8 ;

USE `db_mybatis`;

DROP TABLE IF EXISTS `t_student`;

CREATE TABLE `t_student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;

insert  into `t_student`(`id`,`name`,`age`) values 
(1,'张三',10),(2,'李四',11),(3,'李四',11),(4,'李四',11),
(5,'李四',11),(6,'李四',11),(7,'李四',11);

```

然后新建一个项目，项目结构大概如下所示:
```
MyBatisPro01
   --src
     --org.mybatis.example
        --mappers
          --StudentMapper.java
          --StudentMapper.xml
        --model
          --Studnet.java
        --service
          --Test.java
        --util
          --SqlSessionFactoryUtil.java
   --lib
     --mysql-connector-java-3.1.12-bin.jar
     --mybatis-3.2.8.jar
    --jdbc.properties
    --mybatis-config.xml
```

配置jdbc.properties
```
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/db_mybatis
jdbc.username=root
jdbc.password=
```

配置mybaties-config.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="jdbc.properties"></properties>
	<typeAliases>
		<package name="com.sue.mybatis.demo.model"/>
	</typeAliases>
	
	<environments default="developer">
		<environment id="developer">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driverClassName}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
	</environments>
	
	<mappers>
		<package name="com.sue.mybatis.demo.mapper"/>
	</mappers>
	
</configuration>
```

创建Student.java
```java

public class Student {

	private Integer id;
	private String name;
	private Integer age;

	public Student() {
		super();
	}

	public Student(String name, Integer age) {
		super();
		this.name = name;
		this.age = age;
	}

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Integer getAge() {
		return age;
	}

	public void setAge(Integer age) {
		this.age = age;
	}

}

```

创建SqlSessionFactoryUtils.java用于获取SqlSession
```java

import java.io.IOException;
import java.io.InputStream;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

public class SqlSessionFactoryUtils {
	
	public static SqlSessionFactory sqlSessionFactory;
	
	public static SqlSessionFactory getSqlSessionFactory(){
		if(sqlSessionFactory == null){
			InputStream is = null;
			try {
				is = Resources.getResourceAsStream("mybatis-config.xml");
				sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return sqlSessionFactory;
	}
	
	public static SqlSession opSession(){
		return getSqlSessionFactory().openSession();
	}

}

```

创建StudentMapper.java
```java

import com.sue.mybatis.demo.model.Student;

public interface StudentMapper {

	public int add(Student student);
}

```

创建StudentMapper.xml
```xml
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    
<mapper namespace="com.sue.mybatis.demo.mapper.StudentMapper">

	<insert id="add" parameterType="Student" useGeneratedKeys="true" keyProperty="id">
		insert into t_student(name, age) values(#{name}, #{age})
	</insert>
	
</mapper>
```

创建测试类Test.java，并运行
```java

import org.apache.ibatis.session.SqlSession;

import com.sue.mybatis.demo.mapper.StudentMapper;
import com.sue.mybatis.demo.model.Student;
import com.sue.mybatis.demo.util.SqlSessionFactoryUtils;

public class Test {
	
	public static void main(String[] args) {
		SqlSession sqlSession = SqlSessionFactoryUtils.opSession();
		
		StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
		
		Student s = new Student("wangwu---", 11);
		studentMapper.add(s);
		sqlSession.commit();
		System.out.println(s.getId());
	}

}

```

---

## 配置文件讲解

以下讲解均是相对于mybatis-config.xml中的标签

### enviromenets

该标签下可以存在多个 **environment** ，实际开发一般都存在3个环境：开发环境（dev），生产环境（pro），测试环境（test），这时可以在 **environments** 中配置3个 **environment**，通过 **environments** 的 **default** 属性指定所用环境
```xml
<environments default="developer">
		<environment id="developer">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driverClassName}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
		<environment id="production">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driverClassName}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
		<environment id="test">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driverClassName}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
	</environments>
```

### transactionManager

MyBatis支持2种类型是事物管理器：**JDBC** 和 **MANAGED(托管)**

- JDBC：应用程序负责数据库连接的生命周期
- MANAGED：由应用服务器负责管理数据库连接的生命周期（一般商业类型的服务器才有该功能，入JBoss，Wblogic）

```
<transactionManager type="JDBC" />
```

### dataSource

数据源配置，类型有：UNPOOLED，POOLED，JNDI

- UNPOOLED：没有连接池，每次数据库操作，MyBatis都会创建一个新的连接，用完后关闭该连接。适合小并发的项目
- POOLED：连接池
- JNDI：使用应用服务器配置JNDI数据源获取数据库连接

```
<dataSource type="POOLED">
	<property name="driver" value="${jdbc.driverClassName}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</dataSource>
```
### properties

属性配置，建议采用下面写法

```
<properties resource="jdbc.properties"></properties>
```

### typeAliases

给类的完全限定名取别名，建议采用下面写法

```
<typeAliases>
	<package name="com.sue.mybatis.demo.model"/>
</typeAliases>
```

### mappers

引入映射文件，建议采用下面写法

```
<mappers>
	<package name="com.sue.mybatis.demo.mapper"/>
</mappers>
```

## XML配置SQL映射器

开始之前，我们现在StudentMapper.java中新增几个常用方法,后面每一小节只需着重针StudentMapper.xml的配置和测试用例即可

```
public interface StudentMapper {

    public int add(Student student);
	
    public int update(Student student);

    public int delete(Integer id);

    public Student findById(Integer id);

    public List<Student> findAll();
}
```

### INSERT映射语句 

```
<insert id="add" parameterType="Student" useGeneratedKeys="true" keyProperty="id">
	insert into t_student(name ,age) values (#{name} ,#{age})
</insert>
```

### UPDATE映射语句

```
<update id="update" parameterType="Student">
	update t_student set name=#{name},age=#{age} 
	where id=#{id}
</update>
```

### DELETE映射语句

```
<delete id="delete" parameterType="Integer">
	delete from t_student where id=#{id}
</delete>
```

### SELECT映射语句

在 **findAll** 中返回的集合可以用到 **resultMap**，此处先熟悉，后续复杂返回需要频繁使用

```
<resultMap type="Student" id="StudentResult">
	<id property="id" column="id"/>
	<result property="name" column="name"/>
	<result property="age" column="age"/>
</resultMap>
```

```
<select id="findById" parameterType="Integer" resultType="Student">
	select * from t_student where id=#{id}
</select>
	
<select id="findAll" resultMap="StudentResult">
	select * from t_student
</select>
```

### 测试案例

```
public class StudentTest2 {

	private SqlSession sqlSession = null;
	private StudentMapper sm = null;

	@Before
	public void setUp() throws Exception {
		sqlSession = SqlSessionFactoryUtil.openSession();
		sm = sqlSession.getMapper(StudentMapper.class);
	}

	@Test
	public void testAdd() {
		Student s = new Student("李四", 12);
		sm.add(s);
		sqlSession.commit();
	}
	
	@Test
	public void testUpdate(){
		Student s = new Student(3,"李四3", 3);
		sm.update(s);
		sqlSession.commit();
	}
	
	@Test
	public void testDelete(){
		sm.delete(5);
		sqlSession.commit();
	}
	
	@Test
	public void testFindById(){
		Student s = sm.findById(2);
		System.out.println(s);
	}
	
	@Test
	public void testFindAll(){
		List<Student> stuList = sm.findAll();
		for(Student s : stuList){
			System.out.println(s);
		}
	}
	
	@After
	public void tearDown() throws Exception {
		sqlSession.close();
	}

}

```

## MyBatis关系映射
### 一对一关系
### 一对多关系






-------------

-------------

-------------

-------------

[MyBatis3官方中文手册](http://www.mybatis.org/mybatis-3/zh/index.html)













