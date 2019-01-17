## Spring整合JDBC

### 一，外部配置文件

​	放在项目跟目录下

```java
jdbc.user=root
//MySQL用户名
jdbc.password=root
//MySql密码
jdbc.driverClass=com.mysql.jdbc.Driver
//启动数据库
jdbc.jdbcUrl=jdbc:mysql:///goods
//连接的数据库（为goods）

//连接池初始数量
jdbc.initPoolSize=5
//最大连接池数量
jdbc.maxPoolSize=10
```

### 二，配置spring.XMl配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd">
		
    <!-- 自动扫描 -->
	<context:component-scan base-package="com"></context:component-scan>
	
	<!-- 导入资源文件 -->
	<context:property-placeholder location="classpath:db.properties"/>
	
	<!-- 配置 C3P0 数据源 -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="user" value="${jdbc.user}"></property>
		<property name="password" value="${jdbc.password}"></property>
		<property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
		<property name="driverClass" value="${jdbc.driverClass}"></property>

		<property name="initialPoolSize" value="${jdbc.initPoolSize}"></property>
		<property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
	</bean>
	
	<!-- 配置 Spirng 的 JdbcTemplate -->
	<bean id="jdbcTemplate" 
		class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
</beans>
```

### 三，测试连接

输出jdbcTemplate和ds有内存地址，而不是null。说明连接成功

```java
public class TestDao {

	private ApplicationContext ac=null;
	private JdbcTemplate jdbcTemplate;
	
	{
		ac=new ClassPathXmlApplicationContext("applicationContext.xml");
		jdbcTemplate=(JdbcTemplate) ac.getBean("jdbcTemplate");
	}
	
	//spring整合JDBC之后，使用JdbcTemplate对象 ，去完成对数据库的各种操作 。
	//注意：JdbcTemplate对象中需要注入 DataSource对象 。。。
	//JdbcTemplate 是由Spring框架提供，这个类对象JDBC操作，进行了封装 。
	//使用者 只需要编写SQL语句 ，处理数据封装就可以了 。。。
	// JdbcTemplate 多表操作的时候比较麻烦 ******
	
	//1.测试数据库连接 。
	@Test
	public void test(){
		System.out.println(jdbcTemplate);
		DataSource ds=ac.getBean(DataSource.class);
		System.out.println(ds);
	}
}
```

### 四，对数据库的基本操作

##### 1，修改单行数据

```java
//修改单行数据
 	@Test
	public void testUpdate(){
		String sql="update user set u_name =? where u_id = ?";
		jdbcTemplate.update(sql,"qnm",2);
	}
```

##### 2，修改多行数据  插入 修改 删除操作

```java
	//批量执行
 	@Test
 	public void testBatchUpdate(){
 		String sql="insert into user (u_name,u_jifen) values(?,?)";
 		//需要声明一个对象数组 。List嵌套Object [] 类型的数据 。
		//jdbcTemplate 对象中，想要完成批量操作 。 
		//List <Object [] > list 容器对象 。
 		List<Object []> list =new ArrayList<>();
 		list.add(new Object[] {"zs01",100});
 		list.add(new Object[] {"zs02",100});
 		list.add(new Object[] {"zs03",100});
 		list.add(new Object[] {"zs04",100});	
 		jdbcTemplate.batchUpdate(sql,list);
 	}
```

##### 3，查询单行数据

```java
	@Test
 	public void testSelectOneColumn(){
 		String sql="select u_jifen from user where u_id=?";
 		
 		int jifen = jdbcTemplate.queryForObject(sql,Integer.class,1);
 		System.out.println(jifen);
 	}		
```

##### 4，查询单一操作

```java
@Test
 	public void testSelectOne(){
 		String sql="select * from user where u_id=?";
 		
 		// 在JdbcTemplate对象中 ，想要查询数据 ， 一定要指定数据的封装规则 。。
		//需要创建 一个RowMapper对象  。然后指定泛型。  接口 常用的实现类 。BeanPropertyRowMapper ...可以在构造器中指定，
		//你要把查询到的数据对应到那个类的属性当中 。
 		RowMapper<User> rm=new BeanPropertyRowMapper<>(User.class);
 		
 		User user=jdbcTemplate.queryForObject(sql, rm,"1");
 		System.out.println(user);
 	}
```

##### 5，查询多条数据

```java
	@Test
 	public void testSelectlist(){
 		String sql="select * from user where u_id>?";
 		
 		RowMapper<User> rm=new BeanPropertyRowMapper<>(User.class);
 		
 		List<User> list = jdbcTemplate.query(sql,rm,1);
 		
 		System.out.println(list);
 	}
```

