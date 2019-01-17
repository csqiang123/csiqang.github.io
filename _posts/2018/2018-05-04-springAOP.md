## SpringAOP

### 一，AOP的基本概念

(1)Aspect(切面):通常是一个类，里面可以定义切入点和通知

(2)JointPoint(连接点):程序执行过程中明确的点，一般是方法的调用

(3)Advice(通知):AOP在特定的切入点上执行的增强处理，有before,after,afterReturning,afterThrowing,around

(4)Pointcut(切入点):就是带有通知的连接点，在程序中主要体现为书写切入点表达式

(5)AOP代理：AOP框架创建的对象，代理就是目标对象的加强。Spring中的AOP代理可以使JDK动态代理，也可以是CGLIB代理，前者基于接口，后者基于子类

### 二，Spring AOP 

​	Spring中的AOP代理还是离不开Spring的IOC容器，代理的生成，管理及其依赖关系都是由IOC容器负责，Spring默认使用JDK动态代理，在需要代理类而不是代理接口的时候，Spring会自动切换为使用CGLIB代理，不过现在的项目都是面向接口编程，所以JDK动态代理相对来说用的还是多一些。 

### 三，基于注解的AOP配置方式 

##### 1.启用@AsjectJ支持 

在applicationContext.xml中配置下面一句: 

```xml
<aop:aspectj-autoproxy />
```

##### 2.通知类型介绍 

(1)Before:在目标方法被调用之前做增强处理,@Before只需要指定切入点表达式即可

(2)AfterReturning:在目标方法正常完成后做增强,@AfterReturning除了指定切入点表达式后，还可以指定一个返回值形参名returning,代表目标方法的返回值

(3)AfterThrowing:主要用来处理程序中未处理的异常,@AfterThrowing除了指定切入点表达式后，还可以指定一个throwing的返回值形参名,可以通过该形参名

来访问目标方法中所抛出的异常对象

(4)After:在目标方法完成之后做增强，无论目标方法时候成功完成。@After可以指定一个切入点表达式

(5)Around:环绕通知,在目标方法完成前后做增强处理,环绕通知是最重要的通知类型,像事务,日志等都是环绕通知,注意编程中核心是一个ProceedingJoinPoint

##### 3.举例说明

###### (1)LoggingAspect.java --> 切面类 

```java
package com.spring.test01.aspect;

import java.util.Arrays;
import java.util.List;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Repository;

/*
 * 当前类  就是一个切面类
 * LoggingAspect 日志切面类
 * 想要一个类成为切面类
 * 		1.添加@component 注解 标注 当前类被springIOC容器所管理
 * 		2.@Aspect 标示当前类为一个切面类
 * 		3.优先级 @Order(num) 数字越小优先级越高
 */

@Order(1)
@Aspect
@Repository
public class LoggingAspect {

	/*
	 * 定义一个切点
	 */
	@Pointcut("execution(* com.spring.test01.service.impl.*.*(..))")
	public void declareRepeatJoinPointExpression(){}
	
	/*
	 * 可以使用AspectJ表达式 对目标方法进行抽象的概括
	 * 
	 * execution(* com.spring.test01.service.impl.*.*(..))
	 * 
	 * 第一个*表示匹配所有的访问修饰符 以及所有的返回值类型的方法
	 * 第二个*表示当前包下的所有类
	 * 第三个*表示所有的方法名称
	 * ..表示匹配任意多个参数
	 */
	//@Before 标示前置通知 。指的意思是 在特定位置之前  去执行该方法 。
	//通知 其实就是切面类中一个具体的方法 。  
	//使用前置通知注解。 在目标方法开始之前执行。
	//execution
	//JoinPoint  标示连接点  。
	@Before("declareRepeatJoinPointExpression()")
	public void beforeLog(JoinPoint joinPoint){
		/*
		 * 通过连接点对象可以获得调用目标方法的名称以及参数
		 * 获得方法名称。   能拿到你要调用的方法的名字
		 */
		String method=joinPoint.getSignature().getName();
		//获得调用方法时传递的参数
		List arguments=Arrays.asList(joinPoint.getArgs());
		System.out.println("前置日志");
	}
	
	///注意：后置通知即使方法异常也会成功执行，但是后置通知无法拿到目标方法的返回结果。 需要使用返回通知。。。
	@After("declareRepeatJoinPointExpression()")
	public void afterLog(JoinPoint joinpoint){
		String method = joinpoint.getSignature().getName();
		List arguments = Arrays.asList(joinpoint.getArgs());
		System.out.println("后置日志 。");
	}
	
	/**
	 * 注意：返回通知，其实跟后置通知一样。都是在目标方法执行完之后才会执行
	 * returning="sb" 名字   要跟参数列表中 ）Object对象的名称一致，不然产生异常。
	 * 返回通知：xml  注解
	 */
	@AfterReturning(value="declareRepeatJoinPointExpression()",returning="sb")
	public void testAfterReturning(JoinPoint joinpoint,Object sb){
		//
		String method = joinpoint.getSignature().getName();
		System.out.println("我是返回通知  。 我在目标方法核心业务执行完才会执行 。"+sb);
	}
	
	//异常通知 。
	@AfterThrowing(value="declareRepeatJoinPointExpression()",throwing="e")
	public void testAfterThrowing(JoinPoint joinpoint,Exception e){
		System.out.println("我是异常通知 ，我是在方法产生异常后执行的。"+e);
	}
	
	//环绕通知。  跟动态代理的代码很像
	@Around("declareRepeatJoinPointExpression()")
	public void around(ProceedingJoinPoint pjp){
		//声明一个Object对象 用来表示 目标方法的返回值
		Object obj=null;
		String methodName=pjp.getSignature().getName();
		try {
			
			System.out.println("我是前置日志 。。。"+methodName);
			
			obj=pjp.proceed();
			//调用proceed() 表示执行被代理类的目标方法。
			
			System.out.println("我是返回通知 。。。"+obj+methodName);

		} catch (Throwable e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.out.println("异常通知，产生异常的时候 会执行catch 里面的代码 。");
		}
		System.out.println("我是后置通知 。。。"+obj+methodName);	
	}
}

```

###### (2)ClientDao.java-->定义dao层

接口类：

```java
package com.spring.test01.dao;

import java.util.List;

public interface Dao {

	List getAll();
	
	void update(Object id);
	
	void delete(String id);
	
	void insert(Object id);
}

```

实现接口类

```java
package com.spring.test01.dao.impl;

import java.util.Arrays;
import java.util.List;

import org.springframework.stereotype.Repository;

import com.spring.test01.dao.Dao;


@Repository
public class ClientDao implements Dao {

	@Override
	public List getAll() {
		// TODO Auto-generated method stub
		return Arrays.asList("汤姆","杰克","露西");
	}

	@Override
	public void update(Object obj) {
		// TODO Auto-generated method stub
		System.out.println("成功修改对象:"+obj);
	}

	@Override
	public void delete(String id) {
		// TODO Auto-generated method stub
		System.out.println("成功删除id为:"+id+"的数据");
	}

	@Override
	public void insert(Object obj) {
		// TODO Auto-generated method stub
		System.out.println("新增对象:"+obj+"成功");
	}
}
```

###### (3)UserService.java --> 定义一些目标方法 

接口类：

```java
package com.spring.test01.service;

import java.util.List;

public interface ClientService {

	List getAll();
	
	void update(Object obj);
	
	void delete(String id);
	
	void insert(Object obj);
	
	void count();
}
```

实现接口类：

```java
package com.spring.test01.service.impl;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.spring.test01.dao.Dao;
import com.spring.test01.service.ClientService;


@Service
public class ClientServiceImpl implements ClientService {

	@Autowired
	private Dao clientDao;
	
	/**
	 * 实际开发中，Service负责处理复杂的逻辑业务。 因为一个完整的业务流程，是需要执行很多个方法才能实现的。
	 * 
	 * 如下：我们现在需要在ClientService的实现类中，加入以下三个方法才能实现一套完整的业务流程。 
	 * 1.check(); 检查安全性。
	 * 2.loggingBefore(); 前置日志方法。 
	 * 3.loggingAfter(); 后置日志方法。
	 * 
	 * 那么现在想要完成一套增删改差，方法的执行顺序应变为。 
	 * 1.首先执行 check(); 
	 * 2.记录前置日志 loggingBefore();
	 * @。@ 3.具体的业务逻辑方法。(根据操作区调用dao中的 CRUD 方法。) 
	 * 4.记录后置日志 loggingAfter();
	 * 
	 * 	这样写代码带来的问题 ：
	 * 		1(代码混乱).业务方法急速膨胀，每个方法除了处理核心业务，还要兼顾其他多个关注点。
	 * 		2(代码分散).如果check()方法发生改变，那么每个业务方法中的代码都要被修改。
	 * 
	 * 	解决方法 。 （动态代理）
	 */

	@Override
	public List getAll() {
		List list = clientDao.getAll();//核心业务  。
		System.out.println("核心业务 getAll执行完毕 。。。");
		return list;
	}

	@Override
	public void update(Object obj) {
		clientDao.update(obj);//核心业务 
	}

	@Override
	public void delete(String id) {
		clientDao.delete(id);
	}

	@Override
	public void insert(Object obj) {
		clientDao.insert(obj);
	}

	@Override
	public void count() {
		int x = 10/0;
		System.out.println(x);
	}

}
```

###### (4).applicationContext.xml -->配置XML文件

注解方式配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">


	<!-- 自动扫描组件 -->
	<context:component-scan base-package="com/spring/test01"></context:component-scan>
		
	<!-- 
		@Aspect 注解生效
		让注解生效，切面中的注解生效
		当调用目标方法，跟Aspect中声明的方法相匹配的时候
		AOP框架会自动的为目标方法所在的类创建代理对象
		
		作用是让注解生效，当调用的方法，跟通知中声明的方法一致的时候。AOP框架会自动的为那个方法所在的类生成代理对象，然后在调用目标方法（之前或者之后）把通知中的方法加进去。
	 -->
	 
 	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

xml方式配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">


	<!-- 自动扫描组件 -->
	<context:component-scan base-package="com/spring/test01"></context:component-scan>
		
	<!-- 
		@Aspect 注解生效
		让注解生效，切面中的注解生效
		当调用目标方法，跟Aspect中声明的方法相匹配的时候
		AOP框架会自动的为目标方法所在的类创建代理对象
		
		作用是让注解生效，当调用的方法，跟通知中声明的方法一致的时候。AOP框架会自动的为那个方法所在的类生成代理对象，然后在调用目标方法（之前或者之后）把通知中的方法加进去。
	 -->
	 
<!-- 	<aop:aspectj-autoproxy></aop:aspectj-autoproxy> -->
	
	<!-- 配置AOP -->
	<aop:config>
		<!-- 配置切面表达式 -->
		<aop:pointcut expression="execution(* com.spring.test01.service.impl.ClientServiceImpl.*(..))" id="pointcut"/>
		<!-- 配置切面以及通知 -->
		<aop:aspect ref="loggingAspect" order="2">
			<aop:before method="beforeLog" pointcut-ref="pointcut"/>
		</aop:aspect>
	</aop:config>
</beans>
```

###### (5).Test.java  -->测试类

```java
package com.spring.test01;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.spring.test01.service.ClientService;

public class Test {

	public static void main(String[] args) {
		ApplicationContext ac=new ClassPathXmlApplicationContext("springaop.xml");
		ClientService bean = (ClientService) ac.getBean("clientServiceImpl");
		bean.getAll();
	}
}

```

##### 4.注意

​	做环绕通知的时候，调用ProceedingJoinPoint的proceed()方法才会执行目标方法。 

##### 5.通知执行的优先级 

​	进入目标方法时,先织入Around,再织入Before，退出目标方法时，先织入Around,再织入AfterReturning,最后才织入After。

​	注意:Spring AOP的环绕通知会影响到AfterThrowing通知的运行,不要同时使用!同时使用也没啥意义。

##### 6.切入点的定义和表达式 

切入点表达式的定义算是整个AOP中的核心，有一套自己的规范

Spring AOP支持的切入点指示符：

(1)execution:用来匹配执行方法的连接点

​	A:@Pointcut("execution(* com.aijava.springcode.service..*.*(..))")

第一个*表示匹配任意的方法返回值，..(两个点)表示零个或多个，上面的第一个..表示service包及其子包,第二个*表示所有类,第三个*表示所有方法，第二个..表示

方法的任意参数个数

​	B:@Pointcut("within(com.aijava.springcode.service.*)")

within限定匹配方法的连接点,上面的就是表示匹配service包下的任意连接点

​	C:@Pointcut("this(com.aijava.springcode.service.UserService)")

this用来限定AOP代理必须是指定类型的实例，如上，指定了一个特定的实例，就是UserService

​	D:@Pointcut("bean(userService)")

bean也是非常常用的,bean可以指定IOC容器中的bean的名称

### 四，JDK动态代理介绍 

##### 1，定义被代理的接口类

```java
package com.spring.test08.MeiNv;

public interface MeiNv {

	void play();
}
```

##### 2，实现接口类

```java
package com.spring.test08.MeiNv;

public class PanMeiNv implements MeiNv {

	@Override
	public void play() {
		// TODO Auto-generated method stub
		System.out.println("和潘美女玩");
	}

}

```

##### 3，创建JDK代理类

```java
package com.spring.test08;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxy implements InvocationHandler {

	private Object obj;
	
	public Object getProxy(Object obj){
		this.obj=obj;
		return Proxy.newProxyInstance(obj.getClass().getClassLoader(),obj.getClass().getInterfaces(),this);
	}
	@Override
	public Object invoke(Object objs, Method method, Object[] args) throws Throwable {
		// TODO Auto-generated method stub
		
		check();
		beforeLogging();
		
		Object returninvoke = method.invoke(obj, args);
		
		afterLoggging();
		
		return returninvoke;
		
	}
	
	public void beforeLogging(){
		System.out.println("事前日志 。");
	}
	public void afterLoggging(){
		System.out.println("事后日志。");
	}
	public void check(){
		System.out.println("安全检查。");
	}

}
```

##### 4，创建测试类

```java
package com.spring.test08;

import com.spring.test08.MeiNv.MeiNv;
import com.spring.test08.MeiNv.PanMeiNv;
import com.spring.test08.fang.Fang;
import com.spring.test08.fang.ShangHaiFang;

public class Test {

	public static void main(String[] args) {
		//创建被代理对象 。
		PanMeiNv pan = new PanMeiNv();
		//创建代理类 。
		DynamicProxy dp = new DynamicProxy();
		//通过getProxy方法拿到代理对象。
		Object proxy = dp.getProxy(pan);
		//把代理对象强转成美女类型。
		MeiNv mv = (MeiNv) proxy;
		mv.play();
	}
}
```

