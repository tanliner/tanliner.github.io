---
layout:     post
title:      Spring+MyBatis
subtitle:   Spring server demo
date:       2019-10-16
header-img: img/post-bg-desk.jpg
catalog:    true
tags:
    - Spring
    - MyBatis

---

### 环境
>MacOS Catalina 10.15<br>
>IntelliJ IDEA 2019.1<br>
>JDK-1.8.0_151<br>
>MySQL 8.0.17, mysql driver 8.0.15<br>
>Tomcat 9.0.21<br>
>Spring 4.3.18<br>

### 创建Maven项目
File -> New project -> Maven
![][create]
填写groupid(一般为包名)，artifactid(项目名)，version
![][proj-id]
选择maven环境，确认填写信息，最好加上`archetypeCatalog=internal`这个属性，不然在点击下一步时会卡很久
![][confirm]
然后，等待 IDE 创建maven工程(可能比较慢)

添加`Spring`框架，右键project -> add framework support -> spring mvc -> download 作为全局lib保存下来
>如果之前已经下载过SpringMVC的库，可以勾选Use library，自动识别已下载好的 Spring

确定后，此时会下载SpringMVC的其他包，如果下载慢，请配置代理(SS)
配置方法如下
```text
IntelliJ IDEA -> Preferences -> Appearance & Behavior -> System Settings -> HTTP Proxy

由于[SS]()升级后，没有开启全局的代理，IDE中有可能读取不到代理配置，所以这里选择手动配置，然后选择HTTP

host => 127.0.0.1
port => 10086
no proxy for => *.baidu.com,*jetbrains.com

port 可以通过查看SS客户端的设置项
也可以通过SS客户端的 `Copy HTTP Proxy Shell Export Line` 复制出来查看
```

在`main` 目录下创建`java`文件夹并mark as source root<br>
在`main` 目录下创建`resources`文件夹并mark as resources root<br>
然后再java目录下创建service/dao/entity等包，最后的框架如下图<br>
![][struc]

### 开始配置spring
pom.xml
```
  <properties>
    ...
    <!--定义一系列的依赖版本号-->
    <jackson.version>2.9.9.3</jackson.version>
    <spring.version>4.3.18.RELEASE</spring.version>
    <mybatis.version>3.5.2</mybatis.version>
    <mybatis.spring.version>2.0.1</mybatis.spring.version>
    <aspectj.version>1.7.4</aspectj.version>
    <driver.version>8.0.15</driver.version>
    <c3p0.version>0.9.5.4</c3p0.version>
  </properties>
  ...
  <dependencies>
    ...
    <!-- spring framework -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!-- aspectj -->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>${aspectj.version}</version>
    </dependency>
    <!-- servlet, controller -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>

    <!-- database -->
    <dependency>
      <groupId>com.mchange</groupId>
      <artifactId>c3p0</artifactId>
      <version>${c3p0.version}</version>
    </dependency>
    <!-- driver -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>${driver.version}</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>${mybatis.spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>${mybatis.version}</version>
    </dependency>

    <!-- 类似 Gson 的一个转换库 -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>${jackson.version}</version>
    </dependency>
  </dependencies>
```
添加完后，执行
```
// shell 里通过代理下载依赖
mvn clean install -DproxySet=true -DproxyHost=127.0.0.1 -DproxyPort=10086
或者直接在IDEA工具内执行import，只是可能较慢
```

开始创建java文件
#### Entity Person.java
```
/**
 * Detail:
 * <p>
 * Created by tanlin on 2019-10-16
 */
public class Person {
    private long id;
    private String name;
    private String email;
    private int status;
    // getter & setter
}
```
#### Service, PersonService.java
```
import com.ltan.server.entity.Person;
import org.springframework.stereotype.Service;

/**
 * Detail:
 * <p>
 * Created by tanlin on 2019-10-16
 */
@Service
public interface PersonService {
    Person findPersonById(long id);
}
```
#### Dao PersonMapperDao.java
```
import com.ltan.server.entity.Person;
import org.springframework.stereotype.Repository;

/**
 * Detail:
 * <p>
 * Created by tanlin on 2019-10-16
 */
@Repository
public interface PersonMapperDao {
    Person findPersonById(long id);
}
```
#### ServiceImpl PersonServiceImpl.java
```
import com.ltan.server.dao.PersonMapperDao;
import com.ltan.server.entity.Person;
import com.ltan.server.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * Detail:
 * <p>
 * Created by tanlin on 2019-10-16
 */
@Component
public class PersonServiceImpl implements PersonService {
    @Autowired
    private PersonMapperDao personMapperDao;

    public Person findPersonById(long id) {
        return personMapperDao.findPersonById(id);
    }
}
```
#### Controller PersonController.java
```
import com.fasterxml.jackson.databind.ObjectMapper;
import com.ltan.server.entity.Person;
import com.ltan.server.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * Detail: http://localhost:8080/spring_war_exploded
 * <p>
 * Created by tanlin on 2019-10-16
 */
@Controller
@RequestMapping("/person")
public class PersonController {
    @Autowired
    private PersonService personService;

    @RequestMapping("/selectPerson")
    public void selectPerson(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // URL: http://localhost:8080/spring_war_exploded/person/selectPerson
        request.setCharacterEncoding("utf-8");
        response.setCharacterEncoding("utf-8");

        long personId = Long.parseLong(request.getParameter("id"));
        System.out.println("req id = " + personId);
        System.out.println("personService " + personService);
        Person person = personService.findPersonById(personId);
        System.out.println("query person = " + person);

        ObjectMapper mapper = new ObjectMapper();

        response.getWriter().write(mapper.writeValueAsString(person));
        response.getWriter().close();
    }
}
```
### 创建数据库配置文件
#### resources/jdbc.properties
路径: /src/main/resources/
```
# 老版本 driver
#jdbc.driver=com.mysql.jdbc.Driver
# 当前是8.0, 提示升级为这个 driver
jdbc.driver=com.mysql.cj.jdbc.Driver
#数据库地址
jdbc.url=jdbc:mysql://localhost:3306/person?useUnicode=true&characterEncoding=utf8
#用户名
jdbc.username=java
#密码
jdbc.password=java
#最大连接数
c3p0.maxPoolSize=30
#最小连接数
c3p0.minPoolSize=10
#关闭连接后不自动commit
c3p0.autoCommitOnClose=false
#获取连接超时时间
c3p0.checkoutTimeout=10000
#当获取连接失败重试次数
c3p0.acquireRetryAttempts=2
```
### 创建Spring映射文件

#### spring-mvc.xml
路径: /src/main/resources/config/spring/spring-mvc.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd">

    <!-- 扫描web相关的bean -->
    <context:component-scan base-package="com.ltan.server"/>
    <!--<context:component-scan base-package="com.ltan.server"/>-->
    <!--<context:component-scan base-package="com.ltan.server.dao"/>-->
    <!--<context:component-scan base-package="com.ltan.server.controller"/>-->
    <context:component-scan base-package="com.ltan.server.service"/>
    <context:component-scan base-package="com.ltan.server.service.impl"/>

    <!-- 开启SpringMVC注解模式 -->
    <mvc:annotation-driven/>

    <!-- 静态资源默认servlet配置 -->
    <mvc:default-servlet-handler/>

    <!-- 配置jsp 显示ViewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".html"/>
    </bean>
</beans>
```

#### spring-mybatis.xml
路径: /src/main/resources/config/spring/spring-mybatis.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context

       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 配置数据库相关参数properties的属性：${url} -->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="maxPoolSize" value="${c3p0.maxPoolSize}"/>
        <property name="minPoolSize" value="${c3p0.minPoolSize}"/>
        <property name="autoCommitOnClose" value="${c3p0.autoCommitOnClose}"/>
        <property name="checkoutTimeout" value="${c3p0.checkoutTimeout}"/>
        <property name="acquireRetryAttempts" value="${c3p0.acquireRetryAttempts}"/>
    </bean>
    <bean id="namedParameterJdbcTemplate"  class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
        <constructor-arg ref="dataSource"/>
    </bean>

    <!-- 配置SqlSessionFactory对象 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 扫描model包 使用别名 -->
        <property name="typeAliasesPackage" value="com.ltan.server.entity"/>
        <!-- 扫描sql配置文件:mapper需要的xml文件 -->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>

    <!-- 配置扫描Dao接口包，动态实现Dao接口，注入到spring容器中 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 注入sqlSessionFactory -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!-- 给出需要扫描Dao接口包 -->
        <property name="basePackage" value="com.ltan.server.dao"/>
    </bean>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置基于注解的声明式事务 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

#### mapper MyBatis.xml
路径: /src/main/resources/mapper/PersonMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 设置为IUserDao接口方法提供sql语句配置 -->
<mapper namespace="com.ltan.server.dao.PersonMapperDao">
    <select id="findPersonById" resultType="Person" parameterType="long">
        SELECT * FROM person WHERE id = #{id}
    </select>
</mapper>
```

#### web.xml
路径: /src/main/webapp/WEB-INF/web.xml
更改context配置文件
```xml
<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <!-- 编码过滤器 -->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <!-- 配置DispatcherServlet -->
  <servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 配置springMVC需要加载的配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:/config/spring/spring-*.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    <!--<async-supported>true</async-supported>-->
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

### 开启Tomcat
Run -> Edit configurations -> Tomcat server -> local
![][tomcat]
![][tomcat-war]

### MySQL新增测试数据
#### 创建数据库、用户、授权
```
CREATE database person;
// 创建一个测试用户
CREATE USER 'java'@'localhost' IDENTIFIED BY 'java';
// 为java用户授权person的所有表的所有权限
GRANT privileges ON person.* TO 'java'@'localhost'
// 只授权部分权限
// GRANT SELECT, INSERT ON db.table TO 'java'@'host';
```
#### IDEA 配置 MySQL
![][mysql-driver]
![][mysql]

#### 创建表
```
create table person(
	id int auto_increment,
	name varchar(20) null,
	email varchar(30) null,
	status int default 0 null,
	constraint person_pk primary key (id)
);
```
#### 插入测试数据
```
INSERT into person (name, email, status) value ('ltan', 'lintan.spring@gmail.com', 0);
INSERT into person (name, email, status) value ('ltan2', 'lintan2.spring@gmail.com', 0);
// 插入多行数据
INSERT into person (name, email, status) values
('ltan', 'lintan.spring@gmail.com', 10),
('ltan2', 'lintan2.spring@gmail.com', 200);
```
![][mysql-db]
`sql: show create table person;`<br>
output:
```
CREATE TABLE `person` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  `email` varchar(30) DEFAULT NULL,
  `status` int(11) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

### 前端界面
路径:src/main/webapp/index.jsp

```jsp
<html>
<script>
    function selectUser() {
        var xmlhttp = new XMLHttpRequest();
        xmlhttp.onreadystatechange = function () {
            if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
                document.getElementById("test").innerHTML = xmlhttp.responseText;
            }
        };
        xmlhttp.open("POST", "person/selectPerson", true);
        xmlhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
        xmlhttp.send("id=1");
    }
</script>
<body>
<h2>Hello World!</h2>

<p id="test">click 'Click Me' button, here will show response if server running</p>
<button type="button" onclick="selectUser()">Click Me</button>
</body>
</html>
```

配置好Tomcat后，点击Run，浏览器自动打开`http://localhost:8080/spring_war_exploded/`
![][test]
点击Click Me，输出数据库的结果到页面上
![][done]

#### 总结
[参考-掘金](https://juejin.im/post/5cefa4455188253f9e24df74#heading-0), 他没写详细，这个过程遇到各种问题<br>

1. `java.lang.NoClassDefFoundError: org/apache/ibatis/session/SqlSession` --> 没有添加 mybatis 依赖<br>
2. `java.lang.NoClassDefFoundError: org/springframework/dao/support/DaoSupport` --> 没有添加 spring-tx 依赖<br>
3. `java.lang.NoClassDefFoundError: org/springframework/jdbc/datasource/TransactionAwareDataSource` --> 没有添加 spring-jdbc 依赖<br>
4. `java.lang.NoClassDefFoundError: xxxx DataSourceTransactionManager` --> 没有添加 spring-jdbc 依赖<br>
5. `expected at least 1 bean which qualifies as autowire candidate for this dependency.` --> spring 配置文件不对，没识别到bean<br>
6. `org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'xxx' ` --> spring 配置文件不对<br>
7. `java.sql.SQLException: No suitable driver` --> 没有添加 mysql 的 driver 依赖<br>
8. `java.sql.SQLException: Connections could not be acquired from the underlying database!` --> 注意 `c3p0` 依赖配置<br>

针对spring配置文件报错，大多是`Autowired`注解进来的对象找不多或者无法创建
```text
// 仔细看看 spring-mvc.xml 是否添加了scan, 根据需要加上service/dao相关的包
<context:component-scan base-package="com.ltan.server"/>
<!--<context:component-scan base-package="com.ltan.server"/>-->
<!--<context:component-scan base-package="com.ltan.server.dao"/>-->
<!--<context:component-scan base-package="com.ltan.server.controller"/>-->
<!--<context:component-scan base-package="com.ltan.server.service"/>-->
<!--<context:component-scan base-package="com.ltan.server.service.impl"/>-->
```

[Github-Source-Code](https://github.com/tanliner/spring.git)

[create]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-create.jpg
[proj-id]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-create-proj-id.jpg
[confirm]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-create-confirm.jpg
[struc]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-src-struc.jpg

[tomcat]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-tomcat.jpg
[tomcat-war]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-tomcat-war.jpg

[mysql]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-mysql.jpg
[mysql-db]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-mysql-db.jpg
[mysql-driver]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-mysql-driver.jpg

[test]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-proj-test.jpg
[done]:https://raw.githubusercontent.com/tanliner/ImageHolder/master/img/20191016/spring-mybatis-proj-done.jpg
