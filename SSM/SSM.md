# 1.Spring 配置文件

## 1. 依赖注入的方法有

* set注入

* P命名空间注入

  可以不使用<prpoerty>标签进行注入

* 构造注入

  构造注入的时候如果有多个参数可以利用index

  ```xml
  <bean id="service" class="baoliang.service.imp.serviceimp">
      <constructor-arg index="0" ref="user"/>
      <constructor-arg index="1" value="666"/>
  </bean>
  ```



## 2. 注入的普通数据类型

在spring中中可以注入的三种数据类型：普通数据类型，引用数据类型，集合数据类型

 着重强调 list map  properties的注入，在ApplicationContext中进行注入的时候,下面就是例子

```xml
 <property name="strlist">
            <list>
                <value>朱</value>
                <value>宝</value>
                <value>亮</value>
            </list>
        </property>
        <property name="userMap">
            <map>
                <entry key="u1" value-ref="user1"></entry>
                <entry key="u2" value-ref="user2"></entry>
            </map>
        </property>
        <property name="properties">
            <props>
                <prop key="p1">ppp1</prop>
                <prop key="p2">ppp2</prop>
            </props>
        </property>
    </bean>
    <bean id="user1" class="com.baoliang.dao.User">
        <property name="name" value="张三"/>
        <property name="addr" value="北京"/>
    </bean>
    <bean id="user2" class="com.baoliang.dao.User">
        <property name="name" value="张三"/>
        <property name="addr" value="天津"/>
    </bean>
```

## 3.分模块开发

 ```xml
<import resource="app_test.xml"/>
 ```

## 4. Spring 相关的API

ApplicationContext是个接口

继承体系

![image-20210131170551365](pic/image-20210131170551365.png)    

# 2. Spring配置数据源

## 1. 数据源（连接池）的作用

DBCP，C3P0,BoneCp,Druid

C3P0的演示

```Java
ComboPooledDataSource dataSource=new ComboPooledDataSource();
dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
dataSource.setJdbcUrl("jdbc:mysql://localhost:3306?mydb&serverTimezone=UCT");
dataSource.setPassword("123");
dataSource.setUser("root");
Connection connection=dataSource.getConnection();
System.out.println(connection);
connection.close();
```

## 2.使用properties文件加载数据集

```java
//读取配置文件
ResourceBundle rb=ResourceBundle.getBundle("jdbc");
String driver =rb.getString("jdbc.driver");
String url =rb.getString("jdbc.url");
String password =rb.getString("jdbc.password");
String username =rb.getString("jdbc.username");
ComboPooledDataSource dataSource =new ComboPooledDataSource();
dataSource.setUser(username);
dataSource.setPassword(password);
dataSource.setJdbcUrl(url);
dataSource.setDriverClass(driver);
Connection connection=dataSource.getConnection();
System.out.println(connection);
connection.close();
```

## 3.在配置文件中加载属性文件

1. 首先修改开头的命名空间和网络的地址加入context命名空间

2. 使用

   ```xml
    <context:property-placeholder location="classpath:jdbc.properties"/>
   ```

   加载属性文件

3. 使用${}加载属性文件中的内容

   ```xml
   <context:property-placeholder location="classpath:jdbc.properties"/>
   <bean id="datasource" class="com.mchange.v2.c3p0.ComboPooledDataSource" >
       <property name="jdbcUrl" value="${jdbc.url}"/>
       <property name="driverClass" value="${jdbc.driver}"/>
       <property name="user" value="${jdbc.user}"/>
       <property name="password" value="${jdbc.password}"/>
   </bean>
   ```

   

# 3.Spring的注解开发

## 1.Spring 的原始注解

![image-20210131203757574](pic/image-20210131203757574.png)



## 2.Spring 新注解

![image-20210201103824186](pic/image-20210201103824186.png)

## Spring 配置类

```java
@Configuration
@ComponentScan(basePackages = {"baoliang"})
```

配置类例子

```java
@Configuration
@ComponentScan(basePackages = {"baoliang"})
@PropertySource(value = "classpath:jdbc.properties")
  public class Config {
    @Bean("userdao")
    public user userdao(){
        return new userimp();
    }

    @Bean("service")
    public service service(){
        serviceimp serimp=new serviceimp();
        return serimp;
    }

    @Value("${jdbc.driver}")
    String DriverClass;
    @Value("${jdbc.url}")
    String jdbcUrl;
    @Value("${jdbc.user}")
    String userName;
    @Value("${jdbc.password}")
    String password;
    @Bean("datasource")
    public DataSource getDataSource() throws Exception{
        ComboPooledDataSource comboPooledDataSource= new ComboPooledDataSource();
        comboPooledDataSource.setDriverClass(DriverClass);
        comboPooledDataSource.setJdbcUrl(jdbcUrl);
        comboPooledDataSource.setUser(userName);
        comboPooledDataSource.setPassword(password);

        return comboPooledDataSource;
    }
}
```

# 4.Spring集成Junit

1. 导入spring集成Junit坐标
2. 使用@Runwith注解替换原来的运行期
3. 使用@ContextConfiguration指定配置文件或配置类
4. 使用@Autowired注入需要测试的对象
5. 创建测试方法进行测试

@Runwith就是一个运行器，用于指定junit运行环境，是junit提供给其他框架测试环境接口扩展，为了便于spring的以来注入spring提供了**SpringJUnit4ClassRunner**作为Junit环境

这样就可以创建一个类用来进行测试，如下例子

```Java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {Configration.class})
public class Test {

    @Autowired //根据类型进行注入
    private DataSource datasource;
    @org.junit.Test //设置测试的方法
    public void SourceTest()throws Exception{
        System.out.println(datasource.getConnection());
    }
}
```

# 5. Spring的Aop简介

Aop为Aspect Oriented Programming的缩写，意思为面向切面编程，是通过编译方式和运行期动态代理实现程序功能的同意维护的一种技术 

动态代理的优点：在不修改源码的情况下对目标方法进行增强，其作用可以完成程序之间的松耦合

**Spring两大核心：AOP，AOC**

AOP的优势：1.在程序运行期间，在不修改源码的情况下对目标方法进行增强

   					2. 优势：减少重复代码，提高开发效率，并且便于维护

AOP常用的动态代理技术

1. JDK代理：基于接口的动态代理技术，必须基于接口，没有接口无法代理

2. cglib代理：基于父类的动态代理技术

   ![image-20210201171308368](pic/image-20210201171308368.png)

   



![image-20210201171417642](pic/image-20210201171417642.png)

## 1. jdk动态代理

```Java
public static void main(String[] args){
    Target target=new Target();
    Advice advice=new Advice();
    //观察上面的第一幅图片就会发现，代理对象跟源对象是兄弟关系
    TargetInterface proxy= (TargetInterface) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    advice.before();
                    Object re=method.invoke(target,args);
                    advice.after();
                    return re;
                }
            });

    proxy.save();
}
```

 

## 2. cglib动态代理

1. 创建增强器

2. 设置父类

3. 设置回调

4. 创建代理对象

   ```Java
   public static void main(String[] args){
       Target target=new Target();
       Advice advice=new Advice();
       Enhancer enhancer=new Enhancer();
       enhancer.setSuperclass(target.getClass());
       enhancer.setCallback(new MethodInterceptor() {
           @Override
           public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
               advice.before();
               Object invoke=method.invoke(target,args);
               advice.after();
               return invoke;
           }
       });
       //上面的第二幅图显示代理对象与原对象是父子关系
       Target proxy=(Target) enhancer.create();
       proxy.save();
   }
   ```

## 3. AOP概念

连接点：可以被增强的方法叫做连接点

切入点：真正被增强的方法叫做切入点，因为有些方法可以增强但不会增强

通知/增强：拦截到切点之后所要做的事情就是通知

切面：是切入点和通知的结合

织入： 它是个动词，就是切点跟通知结合的过程叫做的织入

## 4. 使用哪种代理方式

在spring中，框架会根据目标类是否实现了接口来决定采用哪种代理的方式



## 5. 基于XML的AOP开发

1. 导入AOP相关坐标
2. 创建目标接口和目标类（内部有切点）
3. 创建切面类（内部有增强的方法）
4. 将目标类和切面类的对象创建权交给spring
5. 在applicationContext.xml中配置织入关系
6. 测试代码

```xml
<bean id="target" class="com.baoliang.proxy.aop.Target"/>
<bean id="Aspect" class="com.baoliang.proxy.aop.MyAspect"/>
<aop:config>
    <aop:aspect ref="Aspect">
        <aop:before method="before" pointcut="execution(public void com.baoliang.proxy.aop.Target.save())"/>
    </aop:aspect>
</aop:config> 
```

execution表达式：

1. 访问修饰符可以省略
2. 返回值类型，包名，类名，方法名可以使用*代表任意
3. 包名与类名之间一个点.代表当前包下的类，两个点..代表当前包及其子包下的类
4. 参数列表可以使用两个点..表示任意个数，任意类型参数列表

## 6. xml配置：通知类型



![image-20210203163625802](pic/image-20210203163625802.png)



## 7.切点表达式的抽取

```xml
<aop:config>
        <aop:aspect ref="Aspect">
            <aop:pointcut id="mypoint" expression="execution(public void com.baoliang.proxy.aop.*.*(..))"/>
            <aop:before method="befores" pointcut-ref="mypoint"/>
        </aop:aspect>
    </aop:config>
```



## 8. 使用注解进行配置AOP

1. 导入AOP相关坐标

2. 创建目标接口和目标类（内部有切点）

3. 创建切面类（内部有增强的方法）

4. 在切面中使用注解配置织入关系

5. 在配置文件中开启组件扫描和AOP的自动代理

   下面是自动代理的设置

   ```xml
   <aop:aspectj-autoproxy/>
   ```

   ![image-20210203205111403](pic/image-20210203205111403.png)

上面是注解的通知类型

## 9. 注解切点表达式的抽取

```java
@Component("myaspect")
@Aspect
public class Anno_Aspect {
    @Before("Anno_Aspect.pointcunt()")//也可以写pointcunt()
    public void before(){
        System.out.println("前置增强");
    }
    @Pointcut("execution(* com.baoliang.proxy.anno.Target.*(..) )")//抽取了切点表达式，因为注解必须依存一个东西所以写一个空的方法
    public void pointcunt(){}
}
```

 

# 6.Spring JdbcTemplate的使用

## 1. jdbcTemplate开发步骤

1. 导入spring-jdbc和spring-tx坐标 tx代表transection事务
2. 创建数据库表和实体
3. 创建jdbcTemplate对象
4. 执行数据库操作

## 2. 基本基本步骤示例

```java
ComboPooledDataSource dataSource=new ComboPooledDataSource();
dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
dataSource.setUser("root");
dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydb?serverTimezone=UTC");
dataSource.setPassword("123");
//创建jdbcTemplate然后设置数据源
JdbcTemplate jdbcTemplate=new JdbcTemplate();
jdbcTemplate.setDataSource(dataSource);
int row=jdbcTemplate.update("insert into account values(?,?)","xiaoliang",99999);
System.out.println(row);
```



xml配置

![image-20210215095044735](pic/image-20210215095044735.png)

## 3.jdbc实体封装

```java
@Autowired
private JdbcTemplate jdbcTemplate;
@Test
public void testQuery(){
    List<Account> accountList = jdbcTemplate.query("select * from account",new BeanPropertyRowMapper<Account>(Account.class));
    System.out.println(accountList);
}
```

在类配置文件中直接引用已经存在的bean不用使用AnnotationConfigApplicationContext

```java
@Configuration
@PropertySource("classpath:jdbc.properties")
public class config {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String jdbcURL;
    @Value("${jdbc.user}")
    private String user;
    @Value("${jdbc.password}")
    private String password;
    @Bean("datasource")
    public DataSource comboSource() throws  Exception{
        System.out.println(driver+"=="+jdbcURL+"=="+user+"=="+password);
        ComboPooledDataSource dataSource=new ComboPooledDataSource();
        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(jdbcURL);
        dataSource.setUser(user);
        dataSource.setPassword(password);
        return dataSource;
    }
    @Bean("jdbcTemplate")
    public JdbcTemplate getJdbcTemplate() throws Exception {
        JdbcTemplate jdbcTemplate=new JdbcTemplate();
        jdbcTemplate.setDataSource(comboSource());
        return jdbcTemplate;
    }
```

## 4.实体封装后各种查询

```Java
@Autowired
private JdbcTemplate jdbcTemplate;
@Test

public void testQuery(){
    //查询全部
    List<Account> accountList = jdbcTemplate.query("select * from account",new BeanPropertyRowMapper<Account>(Account.class));
    System.out.println(accountList);
//查询单个
    Account accountq=jdbcTemplate.queryForObject("select * from account where name=?",new BeanPropertyRowMapper<Account>(Account.class),"sam");
    System.out.println(accountq);
//聚合查询 
    Long count=jdbcTemplate.queryForObject("select count(*) from account",Long.class);
    System.out.println(count);

}
```

## 5. jdbc总结

![image-20210215110553027](pic/image-20210215110553027.png)

# 7.Spring事务控制

## 1.编程式事务控制

PlatformTransactionManager是Spring的事务管理器接口，它提供了我们常用的操作事务的方法，注意它只是规范了有哪些方法，具体的实现有很多种，这就要看Dao层的技术是什么，就用哪一种实现。

需要通过配置的方法告诉spring使用的什么技术。

## 2.编程式事务控制相关对象

**TransactionDefinition 是事务的定义信息对象，它封装了一些相关的参数，里面有如下的方法**

| 方法                         | 说明               |
| ---------------------------- | ------------------ |
| int getIsolationLevel()      | 获得事务的隔离级别 |
| int getPropogationBehavior() | 获得事务的传播行为 |
| int getTimeout()             | 获得超时时间       |
| boolean isReadOnly()         | 是否只读           |

### 1.事务的隔离级别

设置隔离级别，可以解决事务并发产生的问题，如**脏读，不可重复读，和虚读**（**幻读**）

一共有四种事务的隔离级别，能设置5种

1. ISOLATION_DEFAULT 数据库 默认的
2. ISOLATION_READ_UNCOMMITTED 哪一种都不能解决
3. ISOLATION_READ_COMMITTED 解决脏读
4. ISOLATION_REPEATABLE_READ 解决不可重复读
5. ISOLATION_SERIALIZABLE 全能解决，但性能较低

### 2. 事务的传播行为

传播行为主要解决业务方法在调用业务方法的时**候事务同一性的问题**，事务当中有重复的行为

举例子：A业务方法调用B业务方法，A和B中可能存在相同的方法，这时候就出现事务的同一性问题。

 

1. REQUIRED ：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入这个事务中。一般选择（默认值）---解释一下前面,举个例子A调用B，B看A当前没有事务，就新建一个事务，如果B看A有事务，就加入到这个事务中
2. SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式进行（没有事务）--解释一下，A调用B，如果B看A有事务则一块用这个事务运行，如果B看A没有事务，那么就以非事务方式运行
3. MANDATORY：使用当前的事务，如果当前没有事务，就抛出异常--解释一下A调用B，如果B看A有事务那么使用当前A的事务一块运行，如果B看A没有事务那么就抛出异常。
4. REQUERS_NEW:新建事务,如果当前在事务中,把当前事务挂起--解释一下B调用A,如果B事务在A事务当中,新建事务,把A中当前事务挂起
5. NOT_SUPPORTED:以非事务方式执行操作,如果当前存在事务,就把当前事务挂起--解释一下,A调用B,B看A如果A存在事务，就把当前事务挂起，B以非事务方式执行
6. NEVER:以非事务方式运行，如果当前存在事务，抛出异常--解释一下,B调用A如果B看A当前存在事务，就抛出异常，B以非事务方式运行
7. NESTED：如果当前存在事务,则在嵌套事务内执行,如果当前没有事务,则执行REQUIRED类似的操作
8. 超时时间:默认值是-1,没有时间限制,如果有,以秒为单位进行设置
9. 是否只读:建议查询时设置为只读

### 3.TransactionStatus

TransactionStatus接口提供的是事务具体的运行状态,方法介绍如下

| 方法                       | 说明           |
| -------------------------- | -------------- |
| boolean hasSavepoint()     | 是否存储回滚点 |
| boolean isCompleted()      | 事务是否完成   |
| boolean isNewTransaction() | 是否是新事务   |
| boolean isRollbackOnly()   | 事务是否回滚   |

### 4.小结

![image-20210215165108628](pic/image-20210215165108628.png)



上面第一个,第二个需要进行配置

## 3.基于XML声明式事务控制

**什么是声明式事务控制?**

Spring 的声明式事务顾名思义就是采用声明的方式来处理事务,这里所说的声明,就是指在配置文件中声明,用在Spring配置文件中声明式的处理事务来代替代码的处理事务

**声明式事务处理的作用**

1. 事务管理不侵入开发的组件(解耦),具体来说,业务逻辑对象就不会意识到正在事务管理之中,事实上也应该如此,因为事务管理是属于系统层面的服务,而不是业务逻辑的一部分,如果想要改变事务管理策划的话,也只需要在定义文件中重新配置即可:上面这个就是AOP思想,在业务执行的时候进行了事务控制,切点就是业务,通知就是事务的控制

2.  在不需要事务管理的时候,只要在设定文件上修改一下,即可移除事务管理服务,无需改变代码重新编译

   注意:Spring声明式事务控制底层就是AOP

### 1. 声明式事务控制实现

明确的事项：

1. 谁是切点
2. 谁是通知
3. 配置切面

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:property-placeholder location="classpath:jdbc.properties"/>
    <bean id="datasource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="datasource"/>
    </bean>
    <bean id="Account" class="com.baoliang.dao.imp.AccountDaoImp">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>
<!--    目标对象，内部方法就是切点-->
    <bean id="AccountService" class="com.baoliang.service.imp.AccountServiceImpl">
        <property name="accountDao" ref="Account"/>
    </bean>
<!--    配置平台事务管理器-->
    <bean id="transactionmanager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="datasource"/>为什么配置datasource是因为平台事务管理器要获取DataSource来进行事务的控制
    </bean>
<!--    通知，事务增强-->
    <tx:advice id="txAdvice" transaction-manager="transactionmanager">平台事务管理器是加在通知，事务增强上的。
        设置事务的属性信息
        <tx:attributes>transaction
            <tx:method name="transfer" isolation="DEFAULT" propagation="REQUIRED" timeout="-1" read-only="false"/>可以对某个方法单独的进行事务的配置
            <tx:method name="*" isolation="DEFAULT" propagation="REQUIRED" timeout="-1" read-only="false"/>表示哪个方法被增强，*代表所有的方法
        </tx:attributes>
    </tx:advice>

<!--    事务织入-->
    <aop:config>普通的是<aop:advice></aop:advice>标签
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.baoliang.service.imp.*.*(..))"></aop:advisor>
    </aop:config>
</beans>
```



其中<tx:method>代表切点方法的事务参数的配置，例如：

```xml
<tx:method name="transfer" isolation="DEFAULT" propagation="REQUIRED" timeout="-1" read-only="false"/>
```

name:切点方法名称

isolation:事务的隔离级别

propogation:事务的传播行为

timeout:超时时间

read-only:是否只读

![image-20210216094643511](pic/image-20210216094643511.png)

配置织入可以将切点进行提取

## 4.基于注解的声明式事务控制

在事务上加注解控制

```java
@Service("service")
public class AccountServiceImpl implements AccountService {
    @Autowired
    private AccountDao accountDao;
    @Transactional(isolation = Isolation.READ_COMMITTED,propagation = Propagation.REQUIRED)
    @Override
    public void transfer(String outMan, String inMan, double money) {
        accountDao.out(outMan,money);
        int a=1/0;
        accountDao.in(inMan,money);
    }
}
```

   @Transactional(isolation = Isolation.READ_COMMITTED,propagation = Propagation.REQUIRED)这一行也可以加在类上面，这样整个类的方法事务控制属性都是这个，如果类和方法上同时有，则根据就近原则选择方法上面。

最重要的一点要开启注解事务的驱动以及包的扫描

```xml
<tx:annotation-driven transaction-manager="transactionmanager"/>
<context:component-scan base-package="com.baoliang"/>
```

# 8. Spring 集成web环境

## 1.基本环境搭建

web.xml进行文件配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>userServerlet</servlet-name>
        <servlet-class>com.baoliang.web.userServerlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>userServerlet</servlet-name>
        <url-pattern>/userServletss</url-pattern>
    </servlet-mapping>
</web-app>
```

serverlet配置

```java
public class userServerlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        super.doGet(req, resp);
        ApplicationContext app=new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService=app.getBean(UserService.class);
        userService.serviceFunc();
    }
}
```

## 2. ContextLoaderListener

ApplicationContext引用上下文获取方式

引用上下文对象是通过new ClasspathXmlAppliationContext(spring配置文件)方式获取的，但是每次从容器中获得Bean都要编写new ClaspathXmlApplicationContext配置文件，这样的弊端是配置文件加载多次，应用上下文创建多次。

在Web项目中，可以使用ServletContextListener监听Web应用的启动，我们可以在Web应用启动时，就加载Spring配置文件，创建，应用上下文对象ApplicationContext,在将其存储到最大的域servletContext域中，这样就可以在任意位置从域中获得应用上下文Application对象了。

步骤：

step1:创建监听器,必须实现ServletContextListener两个方法

```java
public class listener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        ApplicationContext app=new ClassPathXmlApplicationContext("applicationContext.xml");
        ServletContext servletContext=servletContextEvent.getServletContext();
        servletContext.setAttribute("app",app);
        System.out.println("Spring容器创建完毕");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {

    }
}
```

step2:在web.xml中配置监听器，这样就不用一次次的创建ApplicationContext了。我们创建了监听器但是没有Spring并不知道，所以要进行配置

```xml
<listener>
    <listener-class>com.baoliang.listener.listener</listener-class>
</listener>
```

使用方法

```java
ServletContext servletContext=this.getServletContext();获取上下文
ApplicationContext app= (ApplicationContext) servletContext.getAttribute("app");获取上面存储的app
UserService userService=app.getBean(UserService.class);
userService.serviceFunc();
```

## 3.进一步优化，使用初始化参数

配置文件的名字可以是任何名称，如果写在程序中就会耦合死，那么需要解耦合，设置初始化参数

web.xml配置

```xml
<context-param>
    <param-name>contextConfiguration</param-name>
    <param-value>applicationContext.xml</param-value>
</context-param>
```

使用

```java
ServletContext servletContext=servletContextEvent.getServletContext();
String contextConfigLoation=servletContext.getInitParameter("contextConfiguration");
ApplicationContext app=new ClassPathXmlApplicationContext(contextConfigLoation);
```

## 4.使用工具类

因为我们设置了属性，就会产生一个问题，会产生大量的String 类型的(类似"app")的上下文属性名称，这个时候很容易混淆，就需要创建一个工具类获得这些属性

```java
public class WebApplicationContextUtils {

    public static ApplicationContext getApplicationContext(ServletContext servletContext){
        return (ApplicationContext) servletContext.getAttribute("app");
   
   }
}
```

使用

```java
ServletContext servletContext=this.getServletContext();
        ApplicationContext app= WebApplicationContextUtils.getApplicationContext(servletContext);
        UserService userService=app.getBean(UserService.class);
        userService.serviceFunc();
```

## 5.Spring提供的工具类

上面的实现是模拟spring工具类，现在写的是spring封装的工具类

上面的分析不用手动实现，spring提供了一个监听器ContextLoaderListener就是对上述功能的封装，**该监听器内部加载spring配置文件，创建应用上下文，并存储到ServletContext域中**，提供了一个客户端工具WebAppliationContextUtils供使用者获得应用上下文对象。

步骤

1. 在web.xml中配置ContextLoaderListener(导入spring-web坐标)
2. 使用WebApplicationContextUtils获得应用上下文对象ApplicationContext

在web.xml中配置applicationContext.xml路径的时候context-param中的name一定要写

```xml
contextConfigLocation
```

否则找不到文件

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>这个一定要固定
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

# 9.SpringMVC

## 1.Springmvc概述

SpringMVC是一种基于Java实现MVC设计模型的请求驱动类型的轻量级Web框架，属于SpringFrameWork的后续产品，已经融合在SpringWebFlow中

它通过一套注解，让一个简单的Java类成为处理请求的控制器，而无需实现任何接口。同时它还支持RESTful编程风格的请求 

![image-20210218093413675](pic/image-20210218093413675.png)

Tomcat服务器接收客户端的请求，封装代表请求的req,代表响应的resp，然后去Web应用中调用相应的资源，其中在Servelet中有一部分是共有的行为，有一部分是特有的行为，共有行为使用前端控制器SpringMVC来完成，特有的行为自己编写（普通javabean自己写）。

## 2.springmvc开发步骤

1. 导入springMVC的包，导入坐标

2. 配置Servlet,一般配置/表示所有请求都过这个处理（共有行为），也就是配置SpringMVC核心控制器DispathcerServlet，这个玩意就是前端控制器
3. 编写POJO(Controller)，也就是创建Controller和视图页面
4. 把POJO（Conroller）使用注解配置放到容器当中 （@Component或者@Controller）,也就是使用注解配置Controller类中业务方法的映射地址
5. 开启组件扫描，在spring_mvc.xml中配置组件扫描

导入spring-webmvc才有DispathcerServlet前端控制器



示例演示

1. 

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.12.RELEASE</version>
</dependency>
```

2. ```xml
   <!--    配置前端控制器-->
       <servlet>
           <servlet-name>DispathcerServlet</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:spring_mvc.xml</param-value>
           </init-param>
           <load-on-startup>1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>DispathcerServlet</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   ```

3. 

```java
@Controller
public class UserController {
    @RequestMapping("/quick")
    public String save(){
        System.out.println("Controller save running");
        return "success.jsp";
    }
}
```

同时还要创建success.jsp

5. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
">
<context:component-scan base-package="com.baoliang.controller"/>
</beans>
```

## 3.SpringMVC组件解析

DispathcerServlet**前端控制器只负责调度**，

![image-20210218104049418](pic/image-20210218104049418.png)

1. 首先客户端进行请求
2. 前端控制器查询Handler，帮你取请求HandlerMapping组件,由处理器映射器HandlerMapping处理，它负责帮你解析，知道最终要找谁
3. HandlerMapping返回处理器执行链HandlerExecutionChain，为什么是链而不是一个，是因为在调用资源前要通过一系列的拦截器进行处理,返回给前端控制器
4. 前端控制器执行Handler，帮你取请求处理器适配器HandlerAdaptor
5. 由HandlerAdaptor请求自己写的那个controller也就是处理器Handler
6. controller响应
7. 处理器适配器HandlerAdaptor返回ModelAndView给前端控制器
8. 前端控制器再取请求视图解析器ViewResolver
9. 视图解析器返回view对象
10. 由前端控制器进行解析渲染视图页面 
11. 最终返回给客户端

## 4. SpringMVC注解解析

**@RequestMapping:**请求映射，用于建立请求URL和处理请求方法之间的对应关系

位置：

* 类上，请求URL的第一级访问目录。此处不写的话，就相当于应用的根目录

* 方法上，请求URL的第二级访问目录，与类上的使用@RequestMapping标注的一级目录一起组成虚拟路径

  此处要注意，返回值例如"success.jsp"这样写就会找不到资源，所以我们要使用相对路径返回"/success.jsp"

  此处请求的应该是http:localhost:8080/xxxProject/user/quick

```java
@RequestMapping("/user")
public class UserController {
    @RequestMapping("/quick")
    public String save(){
        System.out.println("Controller save running");
        return "/success.jsp";
    }
}
```

@RequestMapping的参数

1. value:用于指定请求的URL。它和path 属性的作用是一致的
2. method:用于指定请求的方式 get,post,枚举方式 RequestMethod.GET........
3. params:用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的key和value必须和配置的一模一样

举例：params={"accountName"},表示请求参数必须有accountName

params={"money!100"}，表示请求参数中money不能是100

## 5.SpringMVC组件扫描

```xml
<context:component-scan base-package="com.baoliang">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

上面的这个是只扫Controller注解下的

```xml
<context:exclude-filter type="" expression=""/>
```

这个的意思是排除这个下面的其他全部扫。

## 6.SpringMVC的XML配置解析

forward:是转发，当我们输入地址页面变化后地址没有变化，并没有变成资源地址

redirect:是重定向，当我们输入地址页面变化后出现的是资源的地址。

在springMVC包下有一个DispatcherServlet.properties文件，文件中有前端控制器的一些默认的配置，包括HandlerMapping，HandlerAdapter，ViewResolver等，其中ViewResolver是视图解析器，这个解析器为InternalResourceViewResolver，它的父类UrlBasedViewResolver中包括之前说的forward和redirect以及URL前后缀

prefix为前缀

suffix为后缀

设置URL的前后缀

```xml
<bean id="ViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

这样我们在Controller中只返回success就会拼接成/jsp/success.jsp

## 7.springMVC知识要点

SpringMVC的相关组件

1. 前端控制器:DispatcherServlet

2. 处理器映射器:HandlerMapping

3. 处理器适配器:HandlerAdapter

4. 处理器:Handler

5. 视图解析器:ViewResolver

6. 视图View

   SpringMVC的注解和配置

   1. 请求映射注解：@RequestMapping

   2. 视图解析器配置

      REDIRECT_URL_PREFIX="redirect:"

      FORWARD_URL_PREFIX="forward"

      prefix="";

      suffix="";

## 8. .SpringMVC的数据响应

SpringMVC的数据响应方式

1. 页面跳转

   直接返回字符串

   通过ModelAndView对象返回

2. 回写数据

   直接返回字符串

   返回对象或集合 

### 1. 页面跳转-直接返回字符串形式

直接返回字符串：此种方式会将返回的字符串与视图解析器的前后缀拼接后跳转

前面演示过此种方式：forward和redirect以及字符串的拼接

### 2. 页面跳转-返回ModelAndView

```java
@RequestMapping("/quick2")
public ModelAndView save2(){
    ModelAndView modelAndView=new ModelAndView();
    System.out.println("Controller save running");
    modelAndView.addObject("username","itcast");//这里设置后，可以在页面中通过${username的方式取出来}
    modelAndView.setViewName("success");
    return modelAndView;
}
```

```html
<body>
success!${username}
</body>
```

自动注入ModelAndView

```java
@RequestMapping("/quick3")
public ModelAndView save3(ModelAndView modelAndView){
    modelAndView.addObject("username","itcast");
    modelAndView.setViewName("success");
    return modelAndView;
}
```

上面这个没有new ModelAndView而是 SpringMVC自动注入的

分离式的Model和view

下面的这个其实是将modelAndView进行了拆解

```java
@RequestMapping("/quick4")
public String save4(Model model){//这里也是自动注入的
    model.addAttribute("username","博学谷");//这里使用的是addAttribute而上面使用是addObject
    return "success";//注意看这个是返回的字符串model
}
```

其实上面这些都差不多只是形式不同而已



下面这个使用原生的HttpServletRequest

```java
@RequestMapping("/quick5")
public String save5(HttpServletRequest request){
    request.setAttribute("username","博学谷");
    return "success";
}
```

### 3.回写数据

1. 直接返回字符串

```java
@RequestMapping("/quick6")
public void save6(HttpServletResponse response) throws Exception{
    response.getWriter().println("success!!");
}
```

2. 直接返回字符串

   如果直接返回字符串就是寻找资源，但是我们要回写数据到页面上，所以要告诉框架，这个返回的字符串不是页面而是回写的数据，用到注解@ResponseBody

   

```java
@RequestMapping("quick7")
@ResponseBody
public String save7(){
    return "hello func7";
}
```

3. 返回json格式字符串

   必须导入包

   ```xml
   <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-core</artifactId>
       <version>2.9.8</version>
   </dependency>
   <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-databind</artifactId>
       <version>2.9.8</version>
   </dependency>
   <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-annotations</artifactId>
       <version>2.9.8</version>
   </dependency>
   ```

   

   ```java
   @RequestMapping("quick8")
   @ResponseBody
   public String save8() throws Exception{
       user us=new user();
       us.setUsername("lisi");
       us.setAge(18);
       ObjectMapper objectMapper=new ObjectMapper();
       String json=objectMapper.writeValueAsString(us);
       return json;
   }
   ```

### 4. 返回对象或者集合

首先要配置处理器适配器

```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
    </property>
</bean>
```

```java
@RequestMapping("quick9")
@ResponseBody
public user save9() throws Exception{
    user us=new user();
    us.setUsername("lisi");
    us.setAge(18);
    return us;
}
```

上面xml配置太麻烦，可以使用mvc注解驱动，这样就不用上面那样去配置了

```xml
<mvc:annotation-driven/>
```

但前提是引入mvc命名空间

### 5. SpringMVC获得请求参数

客户端请求参数的格式是：name=value&name=value...

服务器端要活得请求的参数，有时还需要进行数据的封装，springMVC可以接收如下类型的参数：

1. 基本类型参数
2. POJO类型参数
3. 数组类型参数
4. 集合类型参数

### 6.SpringMVC获得基本类型参数

Controller中的业务方法的 
