# spring事物的配置方式

## 注解式事务

### spring+mybatis 事务配置
***

```xml
    <!-- 定义事务管理器 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>
	<!--使用注释事务 -->
	<tx:annotation-driven  transaction-manager="transactionManager" />
```

### spring+hibernate 事务配置
***

```xml
    <!-- 事务管理器配置,单数据源事务 -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>
	
	<!-- 使用annotation定义事务 -->
	<tx:annotation-driven  transaction-manager="transactionManager"/>
```

### @Transactional

1. 这里说明一下，有的把这个注解放在类名称上面了，这样你配置的这个`@Transactional` 对这个类中的所有public方法都起作用

2. `@Transactional` 方法方法名上，只对这个方法有作用，同样必须是`public`的方法

* 事务的传播性：@Transactional(propagation=Propagation.REQUIRED) 

      如果有事务, 那么加入事务, 没有的话新建一个(默认情况下)

* 事务的超时性：@Transactional(timeout=30) //默认是30秒 

      注意这里说的是事务的超时性而不是Connection的超时性，这两个是有区别的

* 事务的隔离级别：@Transactional(isolation = Isolation.READ_UNCOMMITTED)

      读取未提交数据(会出现脏读, 不可重复读) 基本不使用

* 回滚：
        指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)

        指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, Exception.class})

        该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。

* 只读：@Transactional(readOnly=true)

## 使用AOP--拦截器的方式实现事务的配置

```xml
    <!-- 定义事务管理器 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>
	<!-- 下面使用aop切面的方式来实现 -->
	<tx:advice id="TestAdvice" transaction-manager="transactionManager">
		<!--配置事务传播性，隔离级别以及超时回滚等问题 -->
		<tx:attributes>
			<tx:method name="save*" propagation="REQUIRED" />
			<tx:method name="del*" propagation="REQUIRED" />
			<tx:method name="update*" propagation="REQUIRED" />
			<tx:method name="add*" propagation="REQUIRED" />
			<tx:method name="*" rollback-for="Exception" />
		</tx:attributes>
	</tx:advice>
	<aop:config>
		<!--配置事务切点 -->
		<aop:pointcut id="services"
			expression="execution(* com.website.service.*.*(..))" />
		<aop:advisor pointcut-ref="services" advice-ref="TestAdvice" />
	</aop:config>
```
