# Spring 事务管理
Spring事务管理分为三个组成部分：DataSource、TransactionManager、代理机制。在实际编码中，DataSource和TransactionManager的写法不变，只是不同数据源会有不同的实现方式，主要不同的是代理的实现机制。
## DataSource TransactionManager
```
<!-- 创建数据源 -->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
....
</bean>

<!-- 声明事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```

## 代理机制
Spring声明式的事务管理都是通过动态代理实现。

### 注解@Transactional
主要参数：
* propagation：事务传播方式
* isolation： 事务隔离级别
* rollbackFor：事务回滚的异常类型
* timeout：事务处理超时
* readOnly： 只读

### 拦截器
```
<tx:advice id="transactionAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED" isolation="READ_COMMITTED"
                rollback-for="Exception" />
    </tx:attributes>
</tx:advice>
<aop:config>
    <aop:pointcut id="transactionPointcut"
            expression="execution(* cc.laop.service..*Service.*(..))" />
    <aop:advisor pointcut-ref="transactionPointcut"
            advice-ref="transactionAdvice" order="2" />
</aop:config>
```

# 事务特性
事务主要特性包括：事务传播方式、事务隔离级别。

## 事务传播方式
* REQUIRED -- 支持当前事务，如果当前没有事务就新创建一个事务
* SUPPORTS -- 支持当前事务，如果当前没有事务，就在非事务环境下执行
* REQUIRES_NEW -- 新建事务，如果当前存在事务，就把当前事务挂起
* NOT_SUPPORTED -- 在非事务环境下执行，如果当前存在事务，就把当前事务挂起
* NEVER -- 在非事务环境下执行，如果当前存在事务就抛出异常
* MANDATORY -- 支持当前事务，如果当前没有事务，就抛出异常
* NESTED -- 如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则进行与REQUIRED类似的操作；拥有多个可以回滚的保存点，内部回滚不会对外部事务产生影响

## 事务隔离级别
* DEFAULT -- 使用数据库默认事务隔离级别
* READ_UNCOMMITED -- 允许读取其他事务未提交的数据，可能导致脏读、幻读、不可重复读
* READ_COMMITTED -- 只允许读取已经提交的数据
* REPEATABLE_READ -- 对相同字段的多次读取是一致的，除非数据被事务本身改变
* SERIALIZABLE -- 事务顺序执行，在并发环境下，会强制事务顺序执行，效率很差，但是数据安全性高，可防止脏读、幻读、不可重复读。
