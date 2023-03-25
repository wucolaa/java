

# Spring

[TOC]



## 一，Spring Framework

> 系统架构模块：
>
> - ​			**Core Container** **(核心容器)**, 
> - ​			**Aop（面向切面编程）** , 
> - ​			**Aspects（AOP思想实现）** , 
> - ​			**Data Access/Integration（数据访问/数据集成）** , 
> - ​			web（web开发） ,
> - ​			Test（单元测试与集成测试）.
>

### 	1，IOC/DI

- ​				代码书写现状：**耦合度高**
- ​				解决方法：使用对象时不要主动使用new产生对象，转换为由外部提供对象
- ​				IOC控制反转：对象的创建控制权转移到外部，这种思想被称为控制反转
- ​				Spring技术对IOC思想进行了实现：***1***，Spring提供了一个容器，称为**IOC容器**，用来充当IOC思想中的”外部“      ***2***，IOC容器负责对象的创建，初始化等一系列工作，被创建或被管理的**对象**在IOC容器中统称为**Bean**
- ​                 DI（依赖注入）：对**IOC容器**里面的**bean**进行关系绑定的过程，称为**依赖注入**

#### 		核心概念

目标：充分解耦

- ​		使用IOC容器管理bean（IOC）

- ​		在IOC容器里面将有依赖关系的bean进行关系绑定（DI）

最终效果

- ​		使用对象时不仅可以直接从**IOC容器**中取出，并且获取到的**bean**已经绑定了所有的依赖关系

#### IOC入门案例

1，导入坐标 pom.xml

```
<!--1,导入坐标-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
 </dependency>
```

2，配置文件 applicationContext.xml

​		配置bean

```
<!--2,配置bean-->
<!--bean标签标示配置bean
id属性标示给bean起名字
class属性表示给bean定义类型-->

<bean id="bookDao" class="com.it.dao.impl.BookDaoImpl"/>

<bean id="bookService" class="com.it.service.impl.BookServiceImpl">
      <!--7.配置server与dao的关系-->
      <!--property标签表示配置当前bean的属性
      name属性表示配置哪一个具体的属性
      ref属性表示参照哪一个bean-->
      
      <property name="bookDao" ref="bookDao"/>
</bean>
```

3，运行程序

```
public class App2 {
    public static void main(String[] args) {
        //3.获取IoC容器,创建ApplicationContext容器对象,ApplicationContext是接口，所以要用它的实现类来创建对象
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        //4.获取bean（根据bean配置id获取）
//        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
//        bookDao.save();

        BookService bookService = (BookService) ctx.getBean("bookService");
        bookService.save();

    }
}
```

#### DI入门案例

1，对BookServiceImpI的service进行修改  删除业务层中使用的new方法，通过提供对应的**set方法**进行取代

```
public class BookServiceImpl implements BookService {
    //5.删除业务层中使用new的方式创建的dao对象
    private BookDao bookDao;

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
    //6.提供对应的set方法
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
}
```

2,DI依赖注入 service和dao的关系依赖绑定

```
<bean id="bookService" class="com.it.service.impl.BookServiceImpl">
    <!--7.配置server与dao的关系-->
    <!--property标签表示配置当前bean的属性
    name属性表示配置哪一个具体的属性
    ref属性表示参照哪一个bean-->
    <property name="bookDao" ref="bookDao"/>
</bean>
```

总结：删除了业务层对new形式创建的对象，通过xml配置依赖注入，绑定两个bean之间的关系，从而在运行时，使用bean对象会自动创建于这个bean相关的对象

在Spring中，可以使用XML文件来配置依赖注入。这样，在运行时，使用**bean对象**会自动**创建**于这个bean相关的对象。具体来说，可以通过xml配置依赖注入，绑定两个bean之间的关系。在Spring容器中，首先创建所**依赖Bean实例**，然后传递给类的**构造函数**或调用类的**Setter方法**注入依赖项。

### 2，bean

#### bean取别名

配置bean中，可以通过name属性来取多个别名，使用逗号，分号，空格隔开，如下列例子： bookService = service = service4 = bookEbi

```
<bean id="bookService" name="service service4 bookEbi" class="com.itheima.service.impl.BookServiceImpl">
    <property name="bookDao" ref="bookDao"/>
</bean>
```

#### bean作用范围配置

Spring默认给我们创建的bean是**单例对象**，我们可以通过控制bean标签中的scope属性来控制对象是单例还是**非单例**的 scope=”**prototype**“ or ”**singleton**“

> 1. 为什么bean默认为单例的？
>
>    因为单例的bean只有第一次创建新的bean，后面都会复用到该bean，所以不会频繁的创建对象
>
> 2. 适合交给容器进行管理的bean
>
>    Spring中，适合交给容器进行管理的bean有**表现层对象**、**业务层对象**、**数据层对象**、**工具对象**等
>
> 3. 不适合交给容器进行管理的bean
>
>    **封装实体的域对象**
>

#### bean的实例化

bean本质上就是对象，创建bean使用构造方法完成

bean的三种创建方法：

1，通过**构造方法**创建实例化对象，需要用无参的构造方法（常用）

```
public class BookDaoImpl implements BookDao {
//创建一个BookDaoImpl的无参构造方法，用于实例化对象
    public BookDaoImpl() {
        System.out.println("book dao constructor is running ....");
    }

    public void save() {
        System.out.println("book dao save ...");
    }

}
```

```
  <!--方式一：构造方法实例化bean-->
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl"/>
```

2，通过静态工厂创建对象   使用**factory-method**属性指定工厂里面的方法（了解）

```
public class OrderDaoFactory {
    public static OrderDao getOrderDao(){
        System.out.println("factory setup....");
        return new OrderDaoImpl();
    }
}
```

```
<!--方式二：使用静态工厂实例化bean-->
<bean id="orderDao" class="com.itheima.factory.OrderDaoFactory" factory-method="getOrderDao"/>
```

3，实例工厂实例化对象（了解）

```
public class UserDaoFactory {
    public UserDao getUserDao(){
        return new UserDaoImpl();
    }
}
```

```
<!--方式三：使用实例工厂实例化bean-->
<bean id="userFactory" class="com.itheima.factory.UserDaoFactory"/>

<bean id="userDao" factory-method="getUserDao" factory-bean="userFactory"/>
```

4，使用**FactoryBean**实例化bean（实用）

```
public class UserDaoFactoryBean implements FactoryBean<UserDao> {
    //代替原始实例工厂中创建对象的方法
    public UserDao getObject() throws Exception {
        return new UserDaoImpl();
    }
	//返回什么类型的，这里是返回UserDao类型的
    public Class<?> getObjectType() {
        return UserDao.class;
    }
    // 返回是否为单例模式
    public boolean isSingleton() {
        return true;}

}
```

```
<!--方式四：使用FactoryBean实例化bean-->
<bean id="userDao" class="com.itheima.factory.UserDaoFactoryBean"/>
```

#### bean的生命周期

控制bean的生命周期 ---写初始化方法和销毁方法，在xml的bean标签中，使用init-method和destroy-method属性分别指定前面写的初始化方法和销毁方法

```
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
    //表示bean初始化对应的操作
    public void init(){
        System.out.println("init...");
    }
    //表示bean销毁前对应的操作
    public void destory(){
        System.out.println("destory...");
    }

}
```

```
<!--init-method是初始化方法,destroy-method是销毁方法--!>
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl" init-method="init" destroy-method="destory"/>
```

运行程序我们可以发现，**destroy**方法并没有被运行，原因是因为我们的java程序是在java虚拟机中运行，而**destroy**方法是在容器被关闭时候运行的，但是我们并没有写一个关闭容器的方法，使得我们的destroy方法并不能运行。

所以，如果我们希望在程序中正确使用**destroy**方法，则需要我们写一个关闭容器的方法。

如下：

```
public class AppForLifeCycle {
    public static void main( String[] args ) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        bookDao.save();
        //注册关闭钩子函数，在虚拟机退出之前回调此函数，关闭容器
        //ctx.registerShutdownHook();
        //关闭容器
        ctx.close();
    }
}
```

可以使用**ctx.registerShutdownHook()**或者是**ctx.close()**方法，同时我们还要注意，创建ctx对象时，我们需要使用**classPathXmlAppLicationContext**方法，因为作为子类**ApplicationContext**并不具备上述两种关闭容器的方法；

------

同时我们还可以用接口的形式定义初始化方法和销毁方法，从而删除bean属性中对init-method，destroy-method的定义。如下：

```
public class BookServiceImpl implements BookService, InitializingBean, DisposableBean {
    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        System.out.println("set .....");
        this.bookDao = bookDao;
    }

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
//复写下面两种方法
    public void destroy() throws Exception {
        System.out.println("service destroy");
    }

    public void afterPropertiesSet() throws Exception {
        System.out.println("service init");
    }
}
```

此处是通过实现了`InitializingBean`和`DisposableBean`接口，在Spring初始化bean时，如果该bean实现了`InitializingBean`接口，则系统会先调用`afterPropertiesSet()`方法，然后再调用指定的init-method方法。

在Spring中，有两种初始化bean的方式：实现`InitializingBean`接口并实现其`afterPropertiesSet()`方法，或者在配置文件中通过init-method指定。两种方式可以同时使用。

### 3，依赖注入方式

#### 依赖注入的方式

1. ##### Setter注入

- 简单类型

  在bean中定义引用类型属性并提供可访问的set方法

  ```
  public class BookDaoImpl implements BookDao {
  
      private String databaseName;
      private int connectionNum;
      //setter注入需要提供要注入对象的set方法
      public void setConnectionNum(int connectionNum) {
          this.connectionNum = connectionNum;
      }
      //setter注入需要提供要注入对象的set方法
      public void setDatabaseName(String databaseName) {
          this.databaseName = databaseName;
      }
  
      public void save() {
          System.out.println("book dao save ..."+databaseName+","+connectionNum);
      }
  }
  ```

  配置xml中使用**property标签**，**value属性**注入简单类型的数据；

  ```
  <!--注入简单类型-->
  <bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
      <!--property标签：设置注入属性-->
      <!--name属性：设置注入的属性名，实际是set方法对应的名称-->
      <!--value属性：设置注入简单类型数据值-->
      <property name="connectionNum" value="100"/>
      <property name="databaseName" value="mysql"/>
  </bean>
  ```
  
- 引用类型

  在bean中定义引用类型属性并提供可访问的set方法，配置xml中使用**property标签** ，**ref属性**注入引用类型对象，详细可看前面的DI注入案例。

2. ##### 构造器注入：

- 简单类型

  在bean中定义类型属性并提供可访问的构造方法

  配置中使用**constructor-arg**标签**value**属性注入引用类型对象

- 引用类型（了解）

  在bean中定义类型属性并提供可访问的构造方法

  配置中使用**constructor-arg**标签**ref**属性注入引用类型对象

------

#### 构造器注入的参数适配问题

配置中使用constructor-arg标签**name属性**设置按形参名称注入（缺点：耦合度高）

```
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
    根据构造方法参数名称注入
    <constructor-arg name="connectionNum" value="10"/>
    <constructor-arg name="databaseName" value="mysql"/>
</bean>
```

配置中使用constructor-arg标签**type属性**设置按形参类型注入（缺点：可能有类型重复问题）

```
<!--
    解决形参名称的问题，与形参名不耦合
    <bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
        根据构造方法参数类型注入
        <constructor-arg type="int" value="10"/>
        <constructor-arg type="java.lang.String" value="mysql"/>
    </bean>
```

配置中使用constructor-arg标签**index属性**设置按形参位置注入（解决参数类型重复问题，使用位置解决参数匹配）

```
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
    <!--根据构造方法参数位置注入-->
    <constructor-arg index="0" value="mysql"/>
    <constructor-arg index="1" value="100"/>
</bean>
```

------

#### 依赖注入的方式选择

> - 强制依赖使用构造器，使用setter注入有概率不进行注入导致null对象出现
> - 可选依赖使用setter注入进行，灵活性强
> - Spring框架倡导使用构造器，第三方框架大多数采用构造器注入的形式进行数据初始化，相对严谨
> - 如果有必要可以两者同时使用，使用构造器注入完成强制依赖的注入，使用setter注入完成可选依赖的注入
> - 实际开发过程中还要根据实际情况分析，如果受控对象没有提供setter方法就必须使用构造器注入
> - **自己开发的模块推荐使用setter注入**

### 4，依赖自动装配

**概念**：IOC容器根据bean所依赖的资源在容器中自动查找并注入到bean中的过程称为自动装配

#### 自动装配的方法

- **按类型**（常用）--使用autowire="byType"属性

  ```
  <bean class="com.itheima.dao.impl.BookDaoImpl"/>
  <!--autowire属性：开启自动装配，通常使用按类型装配-->
  <bean id="bookService" class="com.itheima.service.impl.BookServiceImpl" autowire="byType"/>
  ```

- 按名称 --使用autowire="byName"属性

  ```
  <bean class="com.itheima.dao.impl.BookDaoImpl"/>
  <!--autowire属性：开启自动装配，通常使用按名称装配-->
  <bean id="bookService" class="com.itheima.service.impl.BookServiceImpl" autowire="byName"/>
  ```

- 按构造方法

- 不启用自动装配

------

#### 依赖自动装配特征

> - 自动装配用于引用类型注入，不能对简单类型进行操作
> - 使用按类型装配时必须保障容器中相同类型的bean唯一，推荐使用
> - 使用按名称装配时必须保障容器中具有指定名称的bean，因变量配置耦合，**不推荐使用**
> - 自动装配优先级低于setter注入与构造器注入，同时出现自动装配配置失效

### 5，集合注入

- 数组注入

  ```
  <!--数组注入-->
  <property name="array">
      <array>
          <value>100</value>
          <value>200</value>
          <value>300</value>
      </array>
  </property>
  ```

- list集合注入

  ```
  <!--list集合注入-->
  <property name="list">
      <list>
          <value>itcast</value>
          <value>itheima</value>
          <value>boxuegu</value>
          <value>chuanzhihui</value>
      </list>
  </property>
  ```

- set集合注入

  ```
  <!--set集合注入-->
  <property name="set">
      <set>
          <value>itcast</value>
          <value>itheima</value>
          <value>boxuegu</value>
          <value>boxuegu</value>
      </set>
  </property>
  ```

- map集合注入

  ```
  <!--map集合注入-->
  <property name="map">
      <map>
          <entry key="country" value="china"/>
          <entry key="province" value="henan"/>
          <entry key="city" value="kaifeng"/>
      </map>
  </property>
  ```

- Properties注入

  ```
  <!--Properties注入-->
  <property name="properties">
      <props>
          <prop key="country">china</prop>
          <prop key="province">henan</prop>
          <prop key="city">kaifeng</prop>
      </props>
  </property>
  ```

### 6，案例：数据源对象管理--Spring管理第三方资源

#### 数据库连接池

> 数据库连接池是一个存放着数据库连接的容器，它可以在程序启动时建立足够的数据库连接，并将这些连接组成一个连接池，由程序动态地对池中的连接进行申请，使用，释放。
>
> 数据库连接池的作用是**节约资源，提高效率，避免频繁地创建和关闭数据库连接。**

#### druid连接池

> Druid是阿里巴巴开源的一款数据库连接池，它可以很好地监控**数据库连接和SQL**的执行情况，**提高数据库访问性能和稳定性，****防止SQL注入攻击**，**支持数据库密码加密等功能。**



##### 导druid坐标 

```
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
```

##### 配置bean

```
<!--    管理DruidDataSource对象-->
<!--    <bean id=dataSource class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring_db"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>
```

##### 使用bean

```
public class App {
    public static void main(String[] args) {
    	//创建一个IOC容器对象
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        //使用创建的dataSource的bean
        DataSource dataSource = (DataSource) ctx.getBean("dataSource");
        //打印输出的dataSource这个bean
        System.out.println(dataSource);
    }
}
```

------

#### c3p0连接池

> C3P0连接池是一种开源的JDBC连接池，它可以预先创建一定数量的数据库连接，当线程需要时直接取走，缩短了创建连接的时间，当使用完毕后，释放连接后放回连接池，以此类推。C3P0连接池的数据库配置是写在c3p0-config.xml的xml文件中。使用C3P0连接池，主要是创建出ComboPooledDataSource数据源对象，然后调用其getConnection方法获取数据库连接对象。**使用连接池的优势有资源的高效利用、更快的系统反应速度、减少了资源独占的风险、统一的连接管理等**。

##### 导坐标

```
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
```

##### 创建bean 

```
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/spring_db"/>
    <property name="user" value="root"/>
    <property name="password" value="root"/>
    <property name="maxPoolSize" value="1000"/>
</bean>-->
```

##### 使用bean

```
public class App {
    public static void main(String[] args) {
       //创建一个IOC容器对象
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        //使用创建的dataSource的bean
        DataSource dataSource = (DataSource) ctx.getBean("dataSource");
        //打印输出的dataSource这个bean
        System.out.println(dataSource);
    }
}
```

------

##### Spring管理第三方资源总结：

Spring可以通过配置文件或注解的方式管理第三方数据池资源，例如上述的**Druid数据源**和**c3p0数据源**。

1. 首先要管理第三方数据池资源，需要先在pom.xml中添加相应的依赖坐标 ----**导坐标**
2. 然后在applicationContext.xml配置文件中导入bean，指定数据库连接的四要素——Driver、url、userName、password2，或者使用注解的方式进行配置。  ----**配置bean**
3. 最后，可以通过IoC容器获取bean，并使用数据池资源。   ----**通过Ioc使用bean**

### 7，加载properties文件

> Java加载properties文件的作用是**为了读取配置文件中的变量，方便用户修改相关的设置，而不用修改程序本身。**
>
> properties文件是一种持久的属性集，其中每个键和其对应值都是**一个字符串。**

#### 开启context命名空间

**填加context空间配置** **----applicationContext.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            ">
```

#### 使用context空间加载properties文件

<!--    system-properties-mode属性：是否加载系统属性-->

<!--    classpath:*.properties：   设置加载当前工程类路径中的所有properties文件-->**（标准写法）**

<!--    classpath*:*.properties：  设置加载当前工程类路径和当前工程所依赖的所有jar包中的所有properties文件-->

```
<!--    2.使用context空间加载properties文件-->
<!--    <context:property-placeholder location="jdbc.properties" system-properties-mode="NEVER"/>-->
<!--    <context:property-placeholder location="jdbc.properties,jdbc2.properties" system-properties-mode="NEVER"/>-->

<!--    <context:property-placeholder location="*.properties" system-properties-mode="NEVER"/>-->

    
<context:property-placeholder location="classpath*:*.properties" system-properties-mode="NEVER"/>
```

#### 使用属性占位符&{}读取properties文件中的属性

```
<!--    3.使用属性占位符${}读取properties文件中的属性-->
<!--    说明：idea自动识别${}加载的属性值，需要手工点击才可以查阅原始书写格式-->
    <bean class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```

### 8，容器

#### 创建容器

```
//1.加载类路径下的配置文件 一般选择第一种
 ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
//2.从文件系统下加载配置文件
// ApplicationContextctx=newFileSystemXmlApplicationContext("D:\\workspace\\spring\\spring_10_container\\src\\main\\resources\\applicationContext.xml");
```

#### 获取bean

第一种是获取bean再转类型

第二种是获取bean的同时，指定bean的类型

第三种是通过指定容器的类型来获取bean（缺点是，当容器有多个类型则会报错，类型不唯一）

```
//       BookDao bookDao = (BookDao) ctx.getBean("bookDao");
//       BookDao bookDao = ctx.getBean("bookDao",BookDao.class);
//       BookDao bookDao = ctx.getBean(BookDao.class);
```

#### 容器类层次结构

- BeanFactory ----顶层接口
  - HierarchicalBeanFactory
    - ApplicationContext ----开发常用的接口
      - ConfigurableApplicationContext ---提供关闭容器的功能，.close函数
        - AbstractApplicationContext
          - AbstractXmlApplicationContext
            - ClassPathXmlApplicationContext ----常用的实现类

#### BeanFactory

**BeanFactory和ApplicationContext两个接口的不同点主要有以下几个方面：**

- BeanFactory是Spring框架最核心的接口，它提供了基本的IoC配置机制，如获取bean和管理bean的生命周期。ApplicationContext是BeanFactory的子接口，它提供了更多的高级功能，如事件发布、国际化和资源访问。
- **BeanFactory按需加载bean，即只有在需要使用bean时才会创建和初始化bean（延迟加载bean）。ApplicationContext则在容器创建时就已经将所有的bean对象都实例化和装配好了，提供了更好的性能和可扩展性（立即加载bean）。**
- BeanFactory需要手动注册后置处理器（PostProcessor），而ApplicationContext则会自动检测并注册后置处理器。
- BeanFactory不支持注解（Annotation），而ApplicationContext则支持注解，并且可以与其他Spring框架（如Spring MVC）集成。



