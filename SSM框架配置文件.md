# SSM框架配置文件

##  sqlMapConfig.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<settings>
		<!-- 开启驼峰命名法 -->
		<setting name="mapUnderscoreToCamelCase" value="true" />
	</settings>
	
	<!--typeAliases进行别名设置  -->
	<typeAliases>
		<package name="tel.bean" />
	</typeAliases>

	<plugins>
		<plugin interceptor="com.github.pagehelper.PageInterceptor">
			<!--分页参数合理化 -->
			<property name="reasonable" value="true" />
		</plugin>
	</plugins>

</configuration>
```

## applicationContext.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

	<!-- 开启注解扫描 ，扫描@controller和扫描@service-->
	<context:component-scan base-package="tel">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>
	
	<!--=================== 数据源，事务控制，xxx ================-->
	<context:property-placeholder location="classpath:jdbc.properties" />
	<bean id="pooledDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
		<property name="driverClassName" value="${jdbc.driverClassName}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
		<property name="minIdle" value="3" />
		<property name="maxActive" value="15" />
		<property name="initialSize" value="3" />
		<property name="maxWait" value="1800" />
		<property name="testOnBorrow" value="true" />
		<property name="testOnReturn" value="true" />
		<property name="testWhileIdle" value="true" />
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<property name="minEvictableIdleTimeMillis" value="300000" />
		<property name="removeAbandoned" value="true" />
		<property name="removeAbandonedTimeout" value="300" />
		<!-- #链接回收的时候控制台打印信息，测试环境可以加上true，线上环境false。会影响性能。 -->
		<property name="logAbandoned" value="true" />
		<property name="validationQuery" value="select 1 " />
	</bean>
	
	<!--================== 配置和MyBatis的整合=============== -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<!-- 指定mybatis全局配置文件的位置 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"></property>
		<property name="dataSource" ref="pooledDataSource"></property>
		<!-- 指定mybatis中数据库语句的mapper文件的位置 -->
		<property name="mapperLocations" value="classpath:mapper/*.xml"></property>
	</bean>

	<!-- 配置扫描器，将mybatis接口的实现加入到ioc容器中 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!--扫描所有dao接口的实现，加入到ioc容器中 -->
		<property name="basePackage" value="tel.mapper"></property>
	</bean>
	<!-- 配置一个可以执行批量的sqlSession -->
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"></constructor-arg>
		<constructor-arg name="executorType" value="BATCH"></constructor-arg>
	</bean>
	<!--=============================================  -->

	<!-- ===============事务控制的配置 ================-->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!--控制住数据源  -->
		<property name="dataSource" ref="pooledDataSource"></property>
	</bean>
	<!--配置事务增强，事务如何切入  -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<!-- 所有方法都是事务方法 -->
			<tx:method name="*"/>
			<!--以get开始的所有方法  -->
			<tx:method name="get*" read-only="true"/>
		</tx:attributes>
	</tx:advice>
	<!--开启基于注解的事务，使用xml配置形式的事务（必要主要的都是使用配置式）  -->
	<aop:config>
		<!-- 切入点表达式 -->
		<aop:pointcut expression="execution(* tel.service..*(..))" id="txPoint"/>
		<!-- 配置事务增强 -->
		<aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"/>
	</aop:config>
</beans>

```

## springmvc.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd">

	<!--SpringMVC的配置文件，包含网站跳转逻辑的控制，配置 -->
	<context:component-scan base-package="tel.controller" use-default-filters="false">
		<!--只扫描控制器。 -->
		<context:include-filter type="annotation"
			expression="org.springframework.stereotype.Controller" />
	</context:component-scan>


	<!--配置视图解析器，方便页面返回 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
	<!--两个标准配置 -->
	<!-- 将springmvc不能处理的请求交给tomcat -->
	<mvc:default-servlet-handler />
	<!-- 能支持springmvc更高级的一些功能，JSR303校验，快捷的ajax...映射动态请求 -->
	
	
	<mvc:annotation-driven />
	<!--直接配置处理器映射器和处理器适配器比较麻烦，可以使用注解驱动来加载。
	SpringMVC使用<mvc:annotation-driven>自动加载
	RequestMappingHandlerMapping和RequestMappingHandlerAdapter
	  -->


</beans>

```

## web.xml

```java
<!-- 
  		1. /*  拦截所有   jsp  js png .css  真的全拦截   建议不使用
  		2. *.action *.do 拦截以do action 结尾的请求     肯定能使用   ERP  
  		3. /  拦截所有 （不包括jsp) (包含.js .png.css)  强烈建议使用     前台 面向消费者  www.jd.com/search   /对静态资源放行
-->
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">

	<!-- 1.启动Spring容器 -->
	<!-- needed for ContextLoaderListener -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>
	<!-- Bootstraps the root web application context before servlet initialization -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- 2.SpringMVC的前端控制器，拦截 所有请求 -->
	<!-- The front controller of this Spring Web application, responsible for 
		handling all application requests -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<!-- Map all requests to the DispatcherServlet for handling -->
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<!-- 3、字符编码过滤器，一定要放在所有过滤器之前 -->
	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceRequestEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
		<init-param>
			<param-name>forceResponseEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

	<!-- 4、使用Rest风格的URI，将页面普通的post请求转为指定的delete或者put请求 -->
	<filter>
		<filter-name>HiddenHttpMethodFilter</filter-name>
		<filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>HiddenHttpMethodFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	<!-- 这个拦截器能够让Tomcat接受put请求时，将数据封装在request中，否则默认只讲post请求封装 -->
	<filter>
		<filter-name>HttpPutFormContentFilter</filter-name>
		<filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>HttpPutFormContentFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

</web-app>
```



## jdbc.properties

```java
#Driver
jdbc.driverClassName=com.mysql.jdbc.Driver
#数据库链接，
jdbc.url=jdbc:mysql://118.89.237.132:3306/netcloud?useUnicode=true&characterEncoding=UTF-8
#帐号
jdbc.username=root
#密码
jdbc.password=123
#检测数据库链接是否有效，必须配置
jdbc.validationQuery=SELECT 'x'
#初始连接数
jdbc.initialSize=3
#最大连接池数量
jdbc.maxActive=10
#去掉，配置文件对应去掉
#jdbc.maxIdle=20
#配置0,当线程池数量不足，自动补充。
jdbc.minIdle=0
#获取链接超时时间为1分钟，单位为毫秒。
jdbc.maxWait=60000
#获取链接的时候，不校验是否可用，开启会有损性能。
jdbc.testOnBorrow=false
#归还链接到连接池的时候校验链接是否可用。
jdbc.testOnReturn=false
#此项配置为true即可，不影响性能，并且保证安全性。意义为：申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
jdbc.testWhileIdle=true
#1.Destroy线程会检测连接的间隔时间
#2.testWhileIdle的判断依据
jdbc.timeBetweenEvictionRunsMillis=60000
#一个链接生存的时间（之前的值：25200000，这个时间有点BT，这个结果不知道是怎么来的，换算后的结果是：25200000/1000/60/60 = 7个小时）
jdbc.minEvictableIdleTimeMillis=300000
#链接使用超过时间限制是否回收
jdbc.removeAbandoned=true
#超过时间限制时间（单位秒），目前为5分钟，如果有业务处理时间超过5分钟，可以适当调整。
jdbc.removeAbandonedTimeout=300
#链接回收的时候控制台打印信息，测试环境可以加上true，线上环境false。会影响性能。
jdbc.logAbandoned=false

```

## log4j.properties

```
log4j.rootLogger = debug,stdout,D,E
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

```

