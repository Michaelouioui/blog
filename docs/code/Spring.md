# Maven

 



# Spring 基础

创建一个 Maven 项目

在src里面添加java包和resource包

添加org.csu.spring.demo.ioc和aop

在ioc中添加persistence

在persistence中添加Dao和DaoImpl

在 pom.xml 中添加和spring相关的4个依赖包

- spring-core
- spring-beans
- spring-expression
- spring-context

然后在resource中添加applicationContext.xml 用spring的config

下面以accountDao为例

```java
applicationContext.xml

<bean id="accountDao" class="org.csu.spring.demo.ioc.persistence.AccountDaoImpl"/>
```

在使用时不需要创建新的对象

只需要

```Java
//初始化Spring环境

ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

//让工厂来创建对象

        AccountDao accountDao = (AccountDao) context.getBean("accountDao");
```

之后对象就成功被创建

在配置中也可以直接给对象创建时就先配置好一些属性

```
//applicationContext.xml

<bean id="account" class="org.csu.spring.demo.ioc.domain.Account">
    <property name="username" value="michael"/>
    <property name="password" value="123456" />
    <property name="age" value="20"/>
</bean>
```

```
//test

Account temp = (Account) context.getBean("account");

System.out.println(temp.getUsername());
System.out.println(temp.getPassword());
System.out.println(temp.getAge());
```

即可输出

如果要在别的bean中使用已经创建好的对象

```
<bean id="accountService" class="org.csu.spring.demo.ioc.service.AccountService">
        <property name="accountDao" ref="accountDao"/>
    </bean>
```

  当使用构造注入时

```

```



```
<constructor-arg name="username" value="Michael"/>
        <constructor-arg name="password" value="123456"/>
        <constructor-arg name="age" value="20"/>
```



## 使用注解的方式使用

在pom.xml中引入aop依赖

```
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>5.2.12.RELEASE</version>
    </dependency>
```

开启注解的包扫描

```
xmlns:context="http://www.springframework.org/schema/context"
http://www.springframework.org/schema/context
https://www.springframework.org/schema/context/spring-context.xsd
```

### 注解的使用

组件，类似于bean的定义

```
@Component("account")
```

为变量输入值

```
@Value("Michael")
private String username;

@Value("123456")
private String password;

@Value("20")
private int age;
```

对于类的注解

```
@Controller

@Repository

@Service
```

主要用法，在service中

```
//    按类型去搜索
    @Autowired
//    按名称去搜索
    @Resource
```

## 其他

导入依赖

```
spring-test
```

在 test 中，自动启动

```
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

相当于

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
```

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class demo {

    @Autowired
    private AccountService service;


    @Test
    public void test(){


//        初始化Spring环境
//        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

//        AccountService service = (AccountService) context.getBean("accountService");
        service.login();
```

# Spring boot

