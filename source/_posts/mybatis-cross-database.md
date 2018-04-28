---
title: spring整合mybatis实现跨库(实现主从查询)
date: 2018-03-01 10:18:59
tags: [java]
categories: [java]
---

### 1、简单的实现跨库查询(可以实现主从查询,跨库查询)方法1
1. 配置多个数据源（datasource）
2. 配置多个SqlSessionFactoryBean工场
3. MapperScannerConfigurer根据不同的mapper目录配置不同的SqlSessionFactory和basePage
4. 注意，同一时间内，只能保证一个数据源的事务(一个方法内包含对两个数据库的操作，你只能保证对一个库有效)

##### 实现如下:
###### 数据库表
```sql
CREATE TABLE `tb_user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `age` tinyint(3) unsigned DEFAULT NULL,
  `gender` char(2) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE TABLE `tb_news` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(50) DEFAULT NULL,
  `author` varchar(50) DEFAULT NULL,
  `content` varchar(200) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```
###### 数据库链接配置
```
#tb_user
jdbc.username=root
jdbc.password=root
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?useSSL=true

#tb_news
news.jdbc.username=root
news.jdbc.password=root
news.jdbc.driver=com.mysql.jdbc.Driver
news.jdbc.url=jdbc:mysql://localhost:3306/caogao?useSSL=true
```
##### mybatis+spring文件配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--user库数据源-->
    <bean id="userDatasource" class="com.zaxxer.hikari.HikariDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
    </bean>

    <!--new库数据源-->
    <bean id="newDatasource" class="com.zaxxer.hikari.HikariDataSource">
        <property name="driverClassName" value="${news.jdbc.driver}"/>
        <property name="username" value="${news.jdbc.username}"/>
        <property name="password" value="${news.jdbc.password}"/>
        <property name="jdbcUrl" value="${news.jdbc.url}"/>
    </bean>

    <!--userSqlSessionFactory工场-->
    <bean id="userSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="userDatasource"/>
        <property name="mapperLocations" value="classpath:cqut/wys/dao/mapper/user/*.xml"/>
    </bean>

    <!--newSqlSessionFactory工场-->
    <bean id="newSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="newDatasource"/>
        <property name="mapperLocations" value="classpath:cqut/wys/dao/mapper/news/*.xml"/>
    </bean>

    <!--userMapperScannerConfigurer mapper扫描-->
    <bean id="userMapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cqut.wys.dao.mapper.user"/>
        <property name="sqlSessionFactoryBeanName" value="userSqlSessionFactory"/>
    </bean>

    <!--newMapperScannerConfigurer mapper扫描-->
    <bean id="newMapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cqut.wys.dao.mapper.news"/>
        <property name="sqlSessionFactoryBeanName" value="newSqlSessionFactory"/>
    </bean>

    <!-- userDatasource定义事务管理器  -->
    <bean id="userTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="userDatasource"/>
    </bean>

    <!-- userDatasource事务支持注解  -->
    <tx:annotation-driven transaction-manager="userTransactionManager"/>

    <!-- newDatasource定义事务管理器  -->
    <bean id="newTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="newDatasource"/>
    </bean>

    <!-- newDatasource事务支持注解  -->
    <tx:annotation-driven transaction-manager="newTransactionManager"/>
</beans>
```
#### 此跨库的实现方法是:
- 在mapper包的根目录下，不同数据库新建不同的mapper查询包

![image](/learning/myblog/public/img/mybatis跨库122.png)
- MapperScannerConfigurer根据不同的mapper查询包配置不同的basepage,sqlSessionFactoryBeanName

1. MapperScannerConfigurer配置中cqut.wys.dao.mapper.news mapper查询包就使用newSqlSessionFactory和newDatasource的值

#### 此跨库查询方法的优缺点
1. 实现简单
2. 配置繁索，多个数据库就得新建、配置多个sqlSessionFactory和MapperScannerConfigurer。

#### [实现源码:方式1](https://gitee.com/wuyoushan/learning/tree/master/mybaits-learning/mybatis-cross-database-spring1)

### 实现跨库查询(可以实现主从查询,跨库查询)方法2
1. 配置多个数据源（datasource）
2. 使用spring的AbstractRoutingDataSource实现多数据源切换
3. 使用spring的aop，根据不同mapper包配置不同的数据源
4. 注意，同一时间内，只能保证一个数据源的事务(一个方法内包含对两个数据库的操作，你只能保证对一个库有效)

##### 实现如下:
###### 数据库链接配置
```
#tb_user
jdbc.username=root
jdbc.password=root
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?useSSL=true

#tb_news
news.jdbc.username=root
news.jdbc.password=root
news.jdbc.driver=com.mysql.jdbc.Driver
news.jdbc.url=jdbc:mysql://localhost:3306/caogao?useSSL=true
```
##### mybatis+spring文件配置
```xml
<?xml version="1.0" encoding="GBK"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--user库数据源-->
    <bean id="userDatasource" class="com.zaxxer.hikari.HikariDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
    </bean>

    <!--new库数据源-->
    <bean id="newDatasource" class="com.zaxxer.hikari.HikariDataSource">
        <property name="driverClassName" value="${news.jdbc.driver}"/>
        <property name="username" value="${news.jdbc.username}"/>
        <property name="password" value="${news.jdbc.password}"/>
        <property name="jdbcUrl" value="${news.jdbc.url}"/>
    </bean>

    <!--动态数据源datasource-->
    <bean id="datasource" class="cqut.wys.utils.DynamicDataSource">
        <!--设置默认数据库源-->
        <property name="defaultTargetDataSource" ref="userDatasource"/>
        <property name="targetDataSources">
            <map key-type="java.lang.String">
                <entry key="userDatasource" value-ref="userDatasource"/>
                <entry key="newDatasource" value-ref="newDatasource"/>
            </map>
        </property>
    </bean>

    <!--数据源切面,选择不同数据源-->
    <bean id="datasourceAspect" class="cqut.wys.utils.DataSourceAspect"/>

    <aop:config>
        <aop:aspect ref="datasourceAspect">
            <!--数据源选择,news包选择news库-->
            <!--同时可以根据mapper前缀不同实现主从分离(选择不同的主从)-->
            <aop:pointcut id="datasourceSelect" expression="execution(* cqut.wys.dao.mapper.news.*.*(..))"/>
            <aop:before method="before" pointcut-ref="datasourceSelect"/>
        </aop:aspect>
    </aop:config>

    <!--sqlSessionFactory工场-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="datasource"/>
        <property name="mapperLocations" value="classpath:cqut/wys/dao/mapper/**/*.xml"/>
    </bean>

    <!--mapperScannerConfigurer mapper扫描-->
    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cqut.wys.dao.mapper"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

    <!--定义事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="datasource"/>
    </bean>

    <!-- 事务支持注解  -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
```
##### 动态数据源选择实现
```java
public class DataSourceAspect{

    private final static String NEWS = "newDatasource";
    private final static String USER = "userDatasource";

    public void before(){
        if (DataSourceContextHolder.getDbType() == null) {
            DataSourceContextHolder.setDbType(NEWS);
        }
    }
}

public class DataSourceContextHolder {

    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

    public static void setDbType(String dbType) {
        contextHolder.set(dbType);
    }

    public static String getDbType() {
        return contextHolder.get();
    }

    public static void clearDbType() {
        contextHolder.remove();
    }
}

public class DynamicDataSource extends AbstractRoutingDataSource{

    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContextHolder.getDbType();
    }
}
```
#### 此跨库查询方法的优缺点
1. 此实现相对方法1来说较复杂
2. 配置简单，使用比方法1灵活

#### [实现源码:方式2](https://gitee.com/wuyoushan/learning/tree/master/mybaits-learning/mybatis-cross-database-spring1)


#### springboot整合mybatis实现跨库

- springboot整合mybatis实现跨库,实现原理和上面一致。在此不再赘述。
- 源码地址[springboot实现方式1](https://gitee.com/wuyoushan/learning/tree/master/mybaits-learning/mybatis-cross-database-springboot1)
- 源码地址[springboot实现方式2](https://gitee.com/wuyoushan/learning/tree/master/mybaits-learning/mybatis-cross-database-springboot2)


#### 参考文档
http://blog.csdn.net/millery22/article/details/49465425

http://blog.csdn.net/neosmith/article/details/61202084

https://www.jianshu.com/p/a042ff2ee2ae

http://blog.csdn.net/ggjlvzjy/article/details/51544016
