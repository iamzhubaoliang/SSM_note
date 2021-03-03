# 1. 快速入门

## 1. SpringBoot的特点

![image-20210302182655888](pic/image-20210302182655888.png)

* springBoot是什么，有什么作用

  Spring boot是一个便捷的脚手架；作用是帮助开发任意快速的搭建大型spring项目。简化工程配置，以来管理；实现开发人员把时间都集中在业务开发上。

![image-20210302184500234](pic/image-20210302184500234.png)

## 2.java配置应用

![image-20210302184630434](pic/image-20210302184630434.png)

![image-20210302184829112](pic/image-20210302184829112.png) 

## 3.Spring boot属性注入方式

![image-20210302190904318](pic/image-20210302190904318.png)

在以前我们使用

```java
@Value("${jdbc.driver}")
String driver;
@Value("${jdbc.url}")
String url;
@Value("${jdbc.user}")
String user;
@Value("${jdbc.password}")
String password;
```

这种方式将properties文件中的内容进行注入，但是这样的注入式非常的繁琐的。所以

我们将以前的jdbc.properties改名为application.properties内容是不变的

在spring配置类上加上注解@EnableConfigurationProperties(JdbcProperties.class) 其中JdbcProperties.class是我们编写的类

```java
//作用是从配置文件application配置文件中读取配置项
//其中prefix是前缀如jdbc.url中的jdbc
//配置项类中的类变量名必须要与前缀之后的配置项名称保持松散绑定（相同）
@ConfigurationProperties(prefix = "jdbc")
public class JdbcProperties {
    private String driver;
    private String url;
    private String user;
    private String password;
    //省略get和set方法
}
```

```java
@Configuration
@EnableConfigurationProperties(JdbcProperties.class)
public class JdbcConfig {
    @Bean
    public DataSource dataSource(JdbcProperties jdbcProperties) throws PropertyVetoException {
        ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
        comboPooledDataSource.setDriverClass(jdbcProperties.getDriver());
        comboPooledDataSource.setJdbcUrl(jdbcProperties.getUrl());
        comboPooledDataSource.setUser(jdbcProperties.getUser());
        comboPooledDataSource.setPassword(jdbcProperties.getPassword());
        return comboPooledDataSource;
    }

}
```

上面是我们的配置类

## 4.更优雅的注入方式

```java
@Bean
@ConfigurationProperties(prefix = "jdbc")
public DataSource dataSource() throws PropertyVetoException {
    ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();

    return comboPooledDataSource;
}
```

只写这个跟application.properties就可以，不再需要编写类

## 5.多Yml文件配置

![image-20210302194210546](pic/image-20210302194210546.png)

![image-20210302194225599](pic/image-20210302194225599.png)

![image-20210302194812627](pic/image-20210302194812627.png)

注意后面的key.def是个集合

![image-20210302195156390](pic/image-20210302195156390.png)

多个properties文件

主properties文件
application.yml

```yml
jdbc:
  driver: com.mysql.cj.jdbc.Driver
  url: jdbc:mysql://localhost:3306/mydb?serverTimezone=UTC
  user: root
  password: 123

#激活配置文件，指定其他配置文件名称
spring:
  profiles:
    active: abc,def//只写文件名-后面的就行
```

application-abc.yml

```yml
baoliang:
  url: http://www.baoliang.com
```

application-def.yml

```yml
xiaozhong:
  url: http://www.xiaozhong.com
```

注意文件名称都是以application为开头的

还有就是，在这里读取配置文件没有任何的注解，因为写文件名称为application.yml的时候，springboot就会认为是配置文件。