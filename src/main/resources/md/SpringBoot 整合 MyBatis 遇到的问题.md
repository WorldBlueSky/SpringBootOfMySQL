# SpringBoot 整合 MyBatis 遇到的问题 



尽量不要用 jUnit 提供的单元测试



提一个要求必须使用SpringBoot 提供的测试类进行测试



还有如果有组件 中存在注入对象的话，那么必须在SpringBoot容器中取出 这个组件，进而使用注入的对象的功能！！！





今天有个错误，花了很长时间来解决，最后发现是一个很低级很基础的错误！



**这是mapper接口，使用@mapper 相当于将接口的代理对象注册进入bean中，但是上下文中找不到（其实是正常）**

```txt
因为 @Mapper 这个注解是 Mybatis 提供的，而 @Autowried 注解是 Spring 提供的，IDEA能理解 Spring 的上下文，但是却和 Mybatis 关联不上。而且我们可以根据 @Autowried 源码看到，默认情况下，@Autowried 要求依赖对象必须存在，那么此时 IDEA 只能给个红色警告了。
```



```java
package com.bit.mapper;

import com.bit.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;


@Mapper
public interface UserMapper {
    User selectById(@Param("userid") Integer id);
}

```



**这是与mapper接口对应的xml文件，同样也没有问题**



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.bit.mapper.UserMapper">
        <select id="selectById" resultType="com.bit.pojo.User">
            select * from users where id = #{userid}
        </select>

</mapper>
```



**将java目录下的xml文件加入resource资源中,在build 标签中嵌套，同样没有问题**



```xml
<resources>    
    <resource>        
        <directory>src/main/java</directory>        
        <includes>            
            <include>**/*.xml</include>        
        </includes>    
    </resource>
</resources>
```



然后我们写service层，写了一个UserService接口，有些了一个UserServiceImpl 接口的实现类



在这个实现类中，注入UserMapper 一直提示无法注入，我一直认为有问题（但是最后发现没问题）

![1656780581069](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656780581069.png)





把service实现类写完了，也没问题



```java
package com.bit.service;

import com.bit.mapper.UserMapper;
import com.bit.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService{
    @Autowired
    private UserMapper userMapper;

    @Override
    public User queryById(Integer id) {
        System.out.println("进入了service");
        return userMapper.selectById(id);
    }
}
```



然后我直接去测试了，我测试的呢？



实例化了UserService，new了一个对象，然后直接调用方法，看是否能够调用UserMapper查询到数据库。然后就不断的包 空指针异常的错误



```java
@SpringBootTest
class BitApplicationTests {

    @Test
    void contextLoads() {
        UserService userService = new UserServiceImpl();
        userService.queryById(13);

        System.out.println(userService);
        System.out.println(userService.queryById(15));
        System.out.println(userService.queryById(13));
    }

}

```

![1656780759585](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656780759585.png)



&emsp;&emsp;我一度以为是mapper接口没有注入到UserServcie中，导致调用UserServcie的方法 就是调用 UserMapper的方法是空的，以为是Mapper接口的问题，各种搜索怎么解决，经过几个小时之后，在他人的博客中找到了答案







&emsp;&emsp;我们的UserMapper 注入到了 UserServiceImpl ,我们不能直接使用 UserServcieIml, 如果在其他的类中进行使用其功能，必须将这个类注入到 当前类中，从容器中拿到这个UserService，才能正确的进行调用，不会发生空指针异常，我一直没有发现，这是也该非常低级的错误。





正确做法： 先装配到当前对象中，再从容器中拿到bean进行使用



```java
@SpringBootTest
class BitApplicationTests {

    @Autowired
    private UserService userService;
    
    @Test
    void contextLoads() {

        System.out.println(userService.queryById(15));
        System.out.println(userService.queryById(13));
    }

}

```



