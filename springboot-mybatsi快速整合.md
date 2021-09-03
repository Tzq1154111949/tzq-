# 一、创建一个SpringBoot项目

![img](C:\Users\tzq\Desktop\Java整理\springboot-mybatis快速整合\886)

![img](C:\Users\tzq\Desktop\Java整理\springboot-mybatis快速整合\1)

![img](C:\Users\tzq\Desktop\Java整理\springboot-mybatis快速整合\2)

![img](C:\Users\tzq\Desktop\Java整理\springboot-mybatis快速整合\3)

# 二、引入相关依赖

```xml
        <!--web核心依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--mysql数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

```

# 三、创建如下结构文件

![img](https://upload-images.jianshu.io/upload_images/15111093-873184f9a1a6f44b.png?imageMogr2/auto-orient/strip|imageView2/2/w/476)

- 编写实体类com.zhg.demo.mybatis.entity.User

```java
public class User implements Serializable {
    private Long id;//编号
    private String username;//用户名
    private String password;//密码
    //省略get set方法
}

```

编写接口com.zhg.demo.mybatis.mapper.UserMapper
 **注意**：需要使用@Mapper注解，不然SpringBoot无法扫描

```java
@Mapper//指定这是一个操作数据库的mapper
public interface UserMapper {
    List<User> findAll();
}

```

编写在resources文件中创建 mapper/UserMapper.xml文件
注意：
1.namespace中需要与使用@Mapper的接口对应
2.UserMapper.xml文件名称必须与使用@Mapper的接口一致
3.标签中的id必须与@Mapper的接口中的方法名一致，且参数一致

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.zhg.demo.mybatis.mapper.UserMapper">
    <select id="findAll" resultType="User">
        SELECT * FROM tb_user
    </select>
</mapper>

```

编写接口com.zhg.demo.mybatis.service

```java
public interface UserService {
    List<User> findAll();
}
```



- 编写实现类com.zhg.emo.mybatis.service.impl.UserServiceimpl
   **注意**：需要在接口实现类中使用@Service注解，才能被SpringBoot扫描，在Controller中使用@Authwired注入

```java
@Service("userService")
public class UserServiceimpl implements UserService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public List<User> findAll() {
        return userMapper.findAll();
    }
}
```

编写api接口com.zhg.demo.mybatis.controller.UserController

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/findAll")
    public List<User> findAll(){
        return userService.findAll();
    }

}
```

```java
@SpringBootApplication
@MapperScan("com.zhg.demo.mybatis.mapper")//使用MapperScan批量扫描所有的Mapper接口；
public class MybatisApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisApplication.class, args);
    }

}
```

# 四、配置文件

**注意**：
 1.mybatis中的mapper-locations是mapper的xml文件位置
 2.mybatis中的type-aliases-package是为了配置xml文件中resultType返回值的包位置，如果未配置请使用全包名如下：

```xml
<select id="findAll" resultType="com.zhg.demo.mybatis.entity.User">
        SELECT * FROM tb_user
</select>
```

在resources中创建application.yml文件，并编写配置

```yaml
server:
  port: 8081
spring:
  #数据库连接配置
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://47.107.105.158:3306/test?characterEncoding=utf-8&useSSL=false
    username: root
    password: 123456

#mybatis的相关配置
mybatis:
  #mapper配置文件
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.zhg.demo.mybatis.entity
  #开启驼峰命名
  configuration:
    map-underscore-to-camel-case: true
```