## SpringBoot  整合 Mybatis 、MySql



## 一、 在Mysql 中 创建数据库、创建数据表



```bash
create database  if not exists ssm;

drop table if exists users;

create table users(
    id int,
    username varchar(20),
    password varchar(20)
);
```



## 二、创建 SpringBoot 项目，添加依赖



![1657538566933](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657538566933.png)



## 三、 写代码



## （1）pojo 层



1、创建实体类，与mysql表中数据进行对应



创建pojo包，创建实体类

![1657539327933](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657539327933.png)





实体类的代码，使用Lombok简化实体类的操作



```java
package com.study.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private int id;
    private String username;
    private String password;
}

```





## （2）mapper 层



写mapper接口以及映射文件



创建mapper包，创建UserMapper接口以及对应的映射文件

这里xml映射文件如果想要被SpringBoot识别，需要在pom.xml中进行将这个路径下的xml文件指定为资源文件 



![1657539317457](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657539317457.png)



创建UserMapper接口，@Mapper 使得当前接口有了一个代理类对象，注入了SqlSession 可以操作Mybatis



```java
package com.study.mapper;

import com.study.pojo.User;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

@Mapper
public interface UserMapper {

    /**
     *  查询全部
     * @return 返回数据表中所有的user 用户数据
     */

    List<User> getUsers();

}

```



UserMapper.xml 映射文件，用Sql处理具体的方法



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.study.mapper.UserMapper">
     <select id="getUsers" resultType="com.study.pojo.User">
        select * from users
    </select>
</mapper>
```



在xml中默认资源文件是resources目录下的，mapper.xml 没有识别，所以需要手动指定资源，在build标签下。



```xml
      <resources>
           <resource>
               <directory>src/main/java</directory>
               <includes>
                   <include>**/*.yml</include>
                   <include>**/*.properties</include>
                   <include>**/*.xml</include>
               </includes>
           </resource>
       </resources>
```



&emsp;&emsp;在这里一定要说明一下，SpringBoot 默认识别 resource 目录下的所有资源文件， 如果在其他目录还有资源文件的话，那么需要在pom.xml 中指定资源文件的路径及格式。



## （3）service 层



创建service包，创建UserService 接口以及 实现类



![1657541664939](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657541664939.png)





UserService 实现类代码，注入Usermapper接口，实现具体的业务代码



```java
package com.study.service;

import com.study.mapper.UserMapper;
import com.study.pojo.User;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

@Service
public class UserService implements UserService {

    @Resource
    private UserMapper userMapper;

    public List<User> getUsers(){
        List<User> list = userMapper.getUsers();
        return list;
    }
}
```





## 四、配置文件



```properties
# 应用名称
spring.application.name=demo
# 应用服务 WEB 访问端口
server.port=8080

#===================Mybatis 相关配置==========================

#下面这些内容是为了让MyBatis映射

#指定Mybatis的Mapper文件,记得加上 classpath:
mybatis.mapper-locations=classpath:com/study/mapper/UserMapper.xml

#指定Mybatis的实体目录,起别名默认小写
mybatis.type-aliases-package=com.study.pojo

#===============Mysql 数据库相关 ======================

#数据源,使用默认的 希卡利数据源（可以不写自动匹配）
spring.datasource.type=com.zaxxer.hikari.HikariDataSource

# 数据库驱动，使用默认的 （可以不写自动匹配）
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# 数据库连接地址
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/ssm?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT

# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
```



&emsp;&emsp;在这里说明一下，Mysql的四件套必须配置，**url、username、password、driver-class-Name**，SpringBoot 默认使用 Hikari 数据源，如果需要其他数据源可以再进行配置



&emsp;&emsp;mybatis 的配置是 **mapperLocation** 一定要写，给 对应的mapper映射文件在 mybatis中进行注册，起别名typeAlias 可写可不写，最好写，写一个包名，然后默认实体类别名不用写全限定名，只用写首字母小写即可。



## 五、测试使用运行 Mysql+Mybatis 的代码



在test 文件夹下，进行测试 springboot 下 mysql、mybatis 的正常使用



```java
package com.study;

import com.study.pojo.User;
import com.study.service.UserServiceImpl;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class DemoApplicationTests {

    @Autowired
    private UserServiceImpl userService;

    @Test
    void contextLoads() {
        List<User> list = userService.getUsers();
        System.out.println(list);
    }

}
```



查看运行结果，成功运行查到结果



![1657542169128](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657542169128.png)





## 六、@Mapper 和 @MapperScan



&emsp;&emsp;在 dao层接口上面加上 @mapper ，能够自动生成代理类，实现mybatis核心类 sqlSession的注入，使得当前接口能够使用 mybatis 。



![1657542334323](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657542334323.png)



&emsp;&emsp;如果要写的mapper接口非常多，那么我们需要在每个接口上都加一个@Mapper,只需要在 启动类上加上@MapperScan,注解中加上要扫描的包路径，启动项目时会在包路径自动扫描mapper接口进行生成代理对象。



当前接口没有加 @Mapper 注解



```java
package com.study.mapper;

import com.study.pojo.User;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

// 不加@Mapper 注解
public interface UserMapper {
    List<User> getUsers();

}

```



在测试类上加上 @MapperScan 扫描包路径的接口



```java
package com.study;

import com.study.pojo.User;
import com.study.service.UserServiceImpl;
import org.junit.jupiter.api.Test;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
@MapperScan("com.study.mapper")
class DemoApplicationTests {

    @Autowired
    private UserServiceImpl userService;

    @Test
    void contextLoads() {
        List<User> list = userService.getUsers();
        System.out.println(list);
    }

}
```



运行测试类，查看结果



![1657542640765](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657542640765.png)



成功运行，拿到数据库中的结果



### @Mapper

作用：用在接口类上，在编译之后会生成相应的接口实现类

位置：对应的某个接口类上面 



 **如果每个接口类 都要 @Mapper 注解，是重复而无聊的工作，解决这个问题用 `@MapperScan` 。** 



### @MapperScan

作用：扫描指定包下所有的接口类，然后所有接口在编译之后都会生成相应的实现类

位置：是在 启动类上面添加 。 



在注解中写上扫描的包路径即可在 包下的所有接口生成代理类

（1）写在启动类上

（2）支持扫描多个包

（3）支持表达式，扫描包或子包中的类

```java
@MapperScan("com.*","com..*.mapper")
```



### 总结



（1）`@Mapper` 是对单个接口类的注解。单个操作。



（2）`@MapperScan` 是对整个包下的所有的接口类的注解。是批量的操作。使用 `@MapperScan` 后，接口类 就不需要使用 `@Mapper` 注解。



## 七、接口与 mapper 文件分离



一般mapper的映射文件不放在 java目录下，java目录下只放java代码，而配置文件就放在 resource目录下



**（1）在resouces 目录下创建mapper文件夹，把mapper.xml 文件放在文件夹中**



![1657543302051](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657543302051.png)



**（2）配置文件指定mapper映射文件的位置**



&emsp;&emsp;注意: resouces 目录与 java 目录 属于同一级别的，所以直接写路径的时候相当于直接写 **resources** 文件夹下面的路径即可

![1657543431373](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657543431373.png)



更改 mybatis 注册 mapper 配置文件的路径

```properties
#指定Mybatis的Mapper文件
mybatis.mapper-locations=classpath:mapper/*.xml
```



**（3）可以把之前写的识别资源文件路径的 xml 给消掉（都行不影响）**



因为 在resources 目录下，所有文件都可以被识别为资源进而读取，所以不要在写配置说资源文件的路径



可以不要，删去识别 java目录下的resource 文件的代码

```xml
      <resources>
           <resource>
               <directory>src/main/java</directory>
               <includes>
                   <include>**/*.yml</include>
                   <include>**/*.properties</include>
                   <include>**/*.xml</include>
               </includes>
           </resource>
       </resources>
```

