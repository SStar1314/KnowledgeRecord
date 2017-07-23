---
title: Spring JDBC
tags: Spring
categories: Spring
---

### Spring JDBC
Database Driver:
- mysql-connector-java (com.mysql.jdbc.Driver)
- commons-dbcp (org.apache.commons.dbcp)
- spring-jdbc (org.springframework.jdbc.datasource [DriverManagerDataSource] + org.springframework.jdbc.core [JdbcTemplate])

```xml
    <!--数据源的配置 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql:///spring"></property>
        <property name="username" value="root"></property>
        <property name="password" value=""></property>
    </bean>
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <bean id="userDao" class="com.curd.spring.impl.UserDAOImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"></property>
    </bean>
```

#### JdbcTemplate主要提供下列方法：

　　1、execute方法：可以用于执行任何SQL语句，一般用于执行DDL语句；

　　2、update方法及batchUpdate方法：update方法用于执行新增、修改、删除等语句；batchUpdate方法用于执行批处理相关语句；

　　3、query方法及queryForXXX方法：用于执行查询相关语句；

　　4、call方法：用于执行存储过程、函数相关语句。
