# gradle-spring-dbutil

## 说明
基于Spring的 *AbstractRoutingDataSource* 进行简单的封装,方便进行数据源的切换，目前主要用于主从数据库的读写切换, 以及指定数据库的操作。
目前代码正在整理中,稍后会发布出来.


## 使用 

### 添加依赖
	compile "org.nimbus:spring-dbutil:+"
		
### 配置xml (spring + mybatis)

	<bean id="dataSource" ...></bean>
	<bean id="slaveDataSource1" ...></bean>
	<bean id="slaveDataSource2" ...></bean>
	<bean id="mysqlDataSource" ...></bean>
	<bean id="oracleDataSource" ...></bean>
	<bean id="db1DataSource" ...></bean>
	<bean id="db2DataSource" ...></bean>
		
	<bean id="dynamicDataSource" class="org.nimbus.spring.dbutil.datasource.DynamicDataSource">
		<property name="master" ref="dataSource"/>
		<property name="slaves">
			<list>
				<value ref="slaveDataSource1"/>
				<value ref="slaveDataSource2"/>
			</list>
		</property>
		<property name="others">
            <map>
                <entry key="mysql" value-ref="mysqlDataSource"></entry>
                <entry key="oracle" value-ref="oracleDataSource"></entry>
                <entry key="db1" value-ref="db1DataSource"></entry>
                <entry key="db2" value-ref="db2DataSource"></entry>
            </map>
        </property>
	</bean>
		
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
			<property name="dataSource" ref="dynamicDataSource" />
			...
	</bean>
		
### 代码里使用

	public void queryXXX(){
		DynamicDataSource.use("master");
		try{
			...
		}finally{
			DynamicDataSource.reset();
		}
	}
	public void queryXXX(){
		DynamicDataSource.use("slave");
		try{
			...
		}finally{
			DynamicDataSource.reset();
		}
	}
	public void queryXXX(){
		DynamicDataSource.use("mysql");
		try{
			...
		}finally{
			DynamicDataSource.reset();
		}
	}
		
### 扩展项
可以使用Spring-AOP进行扩展，减少对代码的入侵。目前支持Aspect和Spring-AOP方式。

#### Aspect
*	需要依赖spring-aspects、aspectjrt、aspectjweaver
*	spring的xml配置：

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns="http://www.springframework.org/schema/beans"
			xmlns:aop="http://www.springframework.org/schema/aop"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi:schemaLocation="
		    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
		    <bean id="DsChangeAspect" class="org.nimbus.spring.dbutil.aop.DataSourceAspect"/>
		    <!--proxy-target-class = true 使用cglib代理，否则使用Java动态代理-->
		    <aop:aspectj-autoproxy proxy-target-class="true" />
		</beans>
		
*	代码示例：

		@DataSourceChange(value="master")
		public void queryXXX(){
			...
		}
		@DataSourceChange(value="slave")
		public void queryXXX(){
			...
		}
		@DataSourceChange(value="mysql")
		public void queryXXX(){
			...
		}
		
#### SpringAOP
不使用aspect，这种方式提供了支持@See DataSourceAdvisor.java，目前还没用到，示例略，只是配置上和Aspect不同，使用方式同样是通过注解来进行改变当前使用的数据源
以下是参考例子:

		<bean id="advisor" class="org.nimbus.spring.dbutil.aop.DataSourceAdvisor" />
		<aop:config proxy-target-class="true">
			<aop:advisor advice-ref="advisor"
				pointcut="@annotation(org.nimbus.spring.dbutil.aop.DataSourceChange)" />
		</aop:config>
