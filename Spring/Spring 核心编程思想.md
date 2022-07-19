# 1. 基础

## 1.1 概念

**Aware：**契约接口的回调，也称**Aware接口回调**。每当Bean实现这样的接口都会回调一个对象给我。如ApplicationContextAware，BeanFactoryAware

**BeanPostProcessor：**bean生命周期的后置处理。用于给bean进行扩展



**观察者模式：**通过事件的方式，让监听器进行状态回调（或事件处理）。

**组合模式：**implements多个接口

**模板模式：**public < T > T execute(ConnectionCallback< T > action)



**面向切面编程：**动态代理，字节码提升

**面向元编程：**注解，配置元信息

**面向模块编程：**maven，Spring @Eanble*注解

**面向函数编程：**lambda，reactive



**BeanFactory：**是IoC的底层容器，用来获取Bean

**FactoryBean：**是创建Bean的一种方式，帮助实现复杂的初始化逻辑

## 1.2 三种注入方式

### 1.2.1 setter 注入

#### 1.2.1.1 XML 方式

```xml
<bean id="user" class="org.ioc.comm.model.Person">
    <property name="id" value="id-1" />
    <property name="name" value="nm-1" />
</bean>
<bean id="demo" class="org.dependency.lookup.SetterInjectDemo">
    <!-- ref引用上面的 -->
    <property name="user" ref="user" />
</bean>
```

```java
public class SetterInjectDemo {
    private Person user;
    // set方法
    public void setUser(Person user) {
        this.user = user;
    }

    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("inject-demo-context.xml");
        System.out.println(ctx.getBean(SetterInjectDemo.class).user);
        //结果：Person(id=id-1, name=nm-1)
        ctx.close();
    }
}
```

#### 1.2.1.2 注解方式

```java
@Bean
public Person user() {
    return new Person().setId("id-2").setName("nm-2");
}

private Person user;
@Autowired // 修饰set方法
public void setUser(Person user) {
    this.user = user;
}

public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(SetterInjectDemo.class);
    ctx.refresh();
    System.out.println(ctx.getBean(SetterInjectDemo.class).user);
    //结果：Person(id=id-2, name=nm-2)
    ctx.close();
}
```

### 1.2.2 构造方法注入

#### 1.2.2.1 XML 方式

```xml
<bean id="constructor-demo" class="org.dependency.lookup.ConstructorInjectDemo">
    <!-- constructor-arg表示构造器注入 -->
    <constructor-arg name="user" ref="user" />
</bean>
```

#### 1.2.2.2 注解方式

```java
// 因为ConstructorInjectDemo是启动类 所以bean要在别的类里配置
class Config {
    @Bean
    public Person user() {
        return new Person().setId("id-2").setName("nm-2");
    }
}
```

```java
public class ConstructorInjectDemo {
    private Person user;
    @Autowired // 标注构造函数 达成构造方法注入
    public ConstructorInjectDemo(Person user) {
        this.user = user;
    }

    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
        // 注册一个bean配置类和本类
        // 不配置本类没法取得user 因为main是静态方法
        ctx.register(Config.class, ConstructorInjectDemo.class);
        ctx.refresh();
        System.out.println(ctx.getBean(ConstructorInjectDemo.class).user);
        // 结果：Person(id=id-2, name=nm-2)
        ctx.close();
    }
}
```

### 1.2.3 自动注入（@Autowired 不用多说）

# 2. java beans

**特性：**依赖查找，生命周期管理，配置元信息，事件，自定义，资源管理，持久化

## 2.1 元信息

```java
static class ToIntPropertyEditor extends PropertyEditorSupport {
    public void setAsText(String text) throws java.lang.IllegalArgumentException {
        setValue(Integer.valueOf(text));
    }
}

public static void main(String[] args) throws IntrospectionException {
    // bean的元信息
    BeanInfo info = Introspector.getBeanInfo(Person.class, Object.class);
    Stream.of(info.getPropertyDescriptors())
          .forEach(x -> {
              // PropertyDescriptor 允许添加属性编辑器 - PropertyEditor
              // GUI程序的text(String)转换成PropertyType
              // name:String,age->int
              Class<?> type = x.getPropertyType();
              // 为字段/属性增加 PropertyEditor
              if ("age".equals(x.getName())) {
                  // String -> int
                  x.setPropertyEditorClass(ToIntPropertyEditor.class);
              }
          });
}
```

# 3. 依赖查找

```java
// User.java
@Data
@Accessors(chain = true)
public class User {
    String id;
    String name;
}
```

## 3.1 根据 bean 名称查找

### 3.1.1 实时查找

```xml
<!-- resources/META-INFO/lookup-context.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="ioc.dependency.domain.User">
        <property name="id" value="1" />
        <property name="name" value="nm" />
    </bean>
    <!-- 给user起个user-1别名 -->
    <alias name="user" alias="user-1" />
</beans>
```

```java
// 启动spring应用上下文
BeanFactory fy = new ClassPathXmlApplicationContext("classpath:/META-INFO/lookup-context.xml");
User usr = (User)fy.getBean("user"); // 或用别名user-1取
```

### 3.1.2 延时查找

```xml
<bean id="obfy" class="org.springframework.beans.factory.config.ObjectFactoryCreatingFactoryBean">
    <property name="targetBeanName" value="user" />
</bean>
```

```java
ObjectFactory<User> obfy = (ObjectFactory<User>)fy.getBean("obfy");
User usr = obfy.getObject();
```

## 3.2 根据 bean 类型查找

### 3.2.1 单个 bean 对象

```java
User usr = fy.getBean(User.class);
```

### 3.2.2 集合 bean 对象

```java
if (fy instanceof ListableBeanFactory) {
    ListableBeanFactory lstFy = (ListableBeanFactory) fy;
    Map<String, User> map = lstFy.getBeansOfType(User.class);
}
```

## 3.3 根据注解查找

```java
@Data
@ToString(callSuper = true)
@Accessors(chain = true)
@Super // 根据这个注解查找
public class SuperUser extends User {
    String auth;
}
```

```xml
<bean id="super" class="ioc.dependency.domain.SuperUser" 
         <!--继承user里的内容-->
         parent="user"
         <!--如果报错加这个（视频里报了 本地没报）-->
         primary="true">
    <property name="auth" value="admin" />
</bean>
```

```java
if (fy instanceof ListableBeanFactory) {
    ListableBeanFactory lstFy = (ListableBeanFactory) fy;
    Map<String, SuperUser> map = (Map) lstFy.getBeansWithAnnotation(Super.class);
}
```

# 4. 依赖注入

## 4.1 根据bean名称注入

## 4.2 根据bean类型注入

### 4.2.1 单个bean对象

### 4.2.2 集合bean对象

```java
@Data
public class UserImpl {
    Collection<User> list;
    // obf.getObject() == ClassPathXmlApplicationContext
    ObjectFactory<ApplicationContext> obf;
}
```

```xml
<!--继承-->
<import resource="lookup-context.xml" />
<!--autowire 自动绑定-->
<bean id="userImpl" class="ioc.dependency.impl.UserImpl" autowire="byType"></bean>
```

```java
BeanFactory bfy = new ClassPathXmlApplicationContext("classpath:/META-INFO/injection-context.xml");
bfy.getBean(UserImpl.class).getList();
us.getObf().getObject() == bfy;   // true
```

## 4.3 注入容器内建bean对象

## 4.4 注入非bean对象

## 4.5 注入类型

### 4.5.1 实时注入

### 4.5.2 延时注入

## 4.6 方法注入

```java
@Autowired //或@Resource
public void init1(User usr) {
    this usr = usr
}
```

# 5. ioc 容器上下文

- 依赖来源
  - 自定义bean
  - 容器内建bean对象
  - 容器内建依赖

ApplicationContext 是 BeanFactory 的超集，BeanFactory 有的 ApplicationContext 全都具备，并且还更多

**ApplicationContext 除了IoC容器角色还提供：**

- AOP
- 配置元信息（Configuration Metadata）
- 资源管理（resources）
- 事件（events）
- 国际化（i18n）
- 注解（annotations）
- environment 抽象

## 5.1 xml 方式

```java
// 创建BeanFactory容器
DefaultListableBeanFactory bf = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(bf);
// XML 配置文件 ClassPath 路径
String xmlPath = "classpath:/META-INFO/injection-context.xml";
// 加载配置 返回bean数量
int count = reader.loadBeanDefinitions(xmlPath);
```

## 5.2 注解方式

```java
@Bean
public static User getUser() {
    return new User().setId("id1").setName("nm1");
}

public static void main(String[] args) {
    // 创建BeanFactory容器
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    // 当前类作为配置类
    ctx.register(AppContextAsContainerDemo.class);
    // 加载bean
    ctx.refresh();
    // 取得
    User usr = ctx.getBean(User.class);
    // 停止
    ctx.close();
}
```

# 6. BeanDefinition

## 6.1 概念

**BeanDefinition：**是 spring 中定义bean的配置元信息接口，包含

- bean 的类名（包 + 名）
- bean 行为配置元素，如作用域，自动绑定的模式，生命周期回调等
- 与其它 bean 的引用关系，又称合作者或依赖者
- 配置设置，比如bean属性

**BeanDefinition 元信息**

- class：bean的全类名，必须是具体类
- name：bean的名称或者id
- scope：bean的作用域（singletion 单例模式、prototype 原型模式）
- properties：依赖注入属性
- autowiring：自动绑定模式
- lazy：延迟初始化模式
- initialization：bean初始化回调方法名
- destruction：销毁回调方法名

## 6.2 构建

### 6.2.1 BeanDefinitionBuilder

```java
BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
// 通过属性设置
builder.addPropertyValue("id", "id1");
builder.addPropertyValue("name", "name1");
// 获取BeanDefinition实例
BeanDefinition def = builder.getBeanDefinition();
```

### 6.2.2 AbstractBeanDefinition 派生类

```java
GenericBeanDefinition bean = new GenericBeanDefinition();
bean.setBeanClass(User.class);
bean.setPropertyValues(new MutablePropertyValues(){
    {addPropertyValue("id", "id2");}
    {addPropertyValue("name","name1");}
});
// 或
bean.setPropertyValues(new MutablePropertyValues().add("id", "id3").add("name", "name1"));
```

## 6.3 注册

### 6.3.1 xml 注册

```xml
<bean id="..." class="xxx.xxx.xxx"></bean>
```

### 6.3.2 注解注册

@Bean，@Component，@Import等（参照"ioc容器-上下文"）

### 6.3.3 java API 注册（命名方式 / 非命名方式）

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    //命名方式
    //definitionRegister(ctx, User.class, "user");
    //非命名方式
    definitionRegister(ctx, User.class, null);
    ctx.refresh();
    System.out.println(ctx.getBean(User.class));
}

public static void definitionRegister(BeanDefinitionRegistry r, Class<?> cls, String beanName) {
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(cls);
    builder.addPropertyValue("id", "id-n").addPropertyValue("name", "name-n");
    // 关联两个bean; 相当于bean1=bean2
    builder.addPropertyReference("bean1", "bean2");
    if (StringUtils.isEmpty(beanName)) {
        // 非命名方式
        BeanDefinitionReaderUtils.registerWithGeneratedName(builder.getBeanDefinition(), r);
    } else {
        // 命名方式
        r.registerBeanDefinition(beanName, builder.getBeanDefinition());
    }
}
```

# 7. 实例化 spring bean

## 7.1 常规方式

- 通过构造器（xml,注解和 java api）

- 通过**静态工厂**方法（xml和java api）

  ```java
  @Data
  @Accessors(chain = true)
  public class User {
      String id;
      String name;
  
      public static User create() {
          return new User();
      }
  }
  ```

  ```xml
  <bean id="byStaticMethod" factory-method="create" <!-- 指向User类的create静态方法 -->
        class="ioc.dependency.domain.User">
      <property name="id" value="id-n"/>
      <property name="name" value="name-n"/>
  </bean>
  ```

  ```java
  BeanFactory fy = new ClassPathXmlApplicationContext("classpath:META-INFO/bean-creation-context.xml");
  User usr = fy.getBean(User.class);
  ```

- 通过 bean 工厂方法（ xml 和 java api ）

  ```java
  public class UserBeanFactory {
      public User create() {
          return new User().setId("id-aa").setName("name-aa");
      }
  }
  ```

  ```xml
  <bean id="userFactory" class="ioc.bean.factory.UserBeanFactory"></bean>
  <bean id="byFactory" factory-bean="userFactory" factory-method="create"></bean>
  ```

  ```java
  BeanFactory fy = new ClassPathXmlApplicationContext("classpath:META-INFO/bean-creation-context.xml");
  User usrFy = fy.getBean("byFactory", User.class);
  ```

- 通过FactoryBean（ xml,注解和 java api ）

  ```java
  public class UserBean implements FactoryBean {
      @Override
      public Object getObject() throws Exception {
          return User.create().setId("id-bean").setName("name-bean");
      }
      @Override
      public Class<?> getObjectType() {
          return User.class;
      }
  }
  ```

  ```xml
  <bean id="userBean" class="ioc.bean.factory.UserBean"></bean>
  ```

  ```java
  User usr = fy.getBean("userBean", User.class);
  ```

## 7.2 特殊方式

- ServiceLoader 实例

  ```java
  public interface IUserFactory {
      default User create() {
          return User.create().setId("id-I").setName("name-I");
      }
  }
  public class UserFactory implements IUserFactory {}
  ```

  ```bash
  #无后缀文件：META-INF/services/ioc.bean.factory.IUserFactory  内容如下：
  ioc.bean.factory.UserFactory
  ```

  ```java
  ServiceLoader<IUserFactory> loader = ServiceLoader
      .load(IUserFactory.class, Thread.currentThread().getContextClassLoader());
  Iterator<IUserFactory> itor = loader.iterator();
  while (itor.hasNext()) System.out.println(itor.next().create());
  ```

- 通过 ServiceLoaderFactoryBean（xml,注解和java api）

  ```xml
  <bean id="loadBean" class="org.springframework.beans.factory.serviceloader.ServiceLoaderFactoryBean">
      <property name="serviceType" value="ioc.bean.factory.IUserFactory" />
  </bean>
  ```

  ```java
  BeanFactory fy = new ClassPathXmlApplicationContext("classpath:META-INF/service-load-bean-factory.xml");
  ServiceLoader<IUserFactory> loader = fy.getBean("loadBean", ServiceLoader.class);
  Iterator<IUserFactory> itor = loader.iterator();
  while (itor.hasNext()) System.out.println(itor.next().create());
  ```

- 通过AutowireCapableBeanFactory#createBean(Class, int, boolean)

  ```java
  ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:META-INF/service-load-bean-factory.xml");
  AutowireCapableBeanFactory auto = ctx.getAutowireCapableBeanFactory();
  IUserFactory i = auto.createBean(UserFactory.class);
  User usr = i.create();
  ```

- 通过BeanDefinitionRegistry#registerBeanDefinition(String, BeanDefinition)

# 8. 初始化 spring bean

- @PostConstruct

- 实现 InitializingBean 接口的 afterPropertiesSet() 方法

  ```java
  public class UserFactory implements IUserFactory, InitializingBean {
      @PostConstruct
      public void init() { //初始化1
          System.out.println("IUserFactory -> UserFactory -> @PostConstruct -> init");
      }
      public void init_2() { //初始化2
          System.out.println("@Bean(initMethod = init_2)");
      }
      @Override  //初始化3
      public void afterPropertiesSet() throws Exception {
          System.out.println("InitializingBean.afterPropertiesSet");
      }
  }
  ```

  ```java
  @Configuration
  public class InitBeanDemo {
      @Bean(initMethod = "init_2") //初始化2
      public UserFactory ufc() { return new UserFactory(); }
  
      public static void main(String[] args) {
          AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
          ctx.register(InitBeanDemo.class);
          ctx.refresh();
          UserFactory ufc = ctx.getBean(UserFactory.class);
          ctx.close();
      }
  }
  ```

  > 结果：IUserFactory -> UserFactory -> @PostConstruct -> init
  >
  > ​      InitializingBean.afterPropertiesSet
  >
  > ​      @Bean(initMethod = init_2)

- 自定义初始化方法
- xml配置：<bean init-method="init" ... />
- java注解：@Bean(initMethod="init")
- java api：AbstractBeanDefinition#setInitMethodName(String)

# Spring 配置元信息

## Spring Bean 配置元信息 - BeanDefinition

- GenericBeanDefinition：通用型 BeanDefinition
- RootBeanDefinition：无 Parent 的 BeanDefinition 或合并后的 BeanDefinition
- AnnotatedBeanDefinition：注解标注的 BeanDefinition

## Spring Bean 属性元信息 - PropertyValues

1. 可修改实现 - MutablePropertyValues

2. 元素成员 - PropertyValue

3. Bean 属性上下文存储 - AttributeAccessor

4. Bean 元信息元素 - BeanMetadataElement

  ```java
BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
// 设置 bean 属性
builder.addPropertyValue("name", "builder");
// 获取
AbstractBeanDefinition abs = builder.getBeanDefinition();
// 设置附加属性（不影响 Bean 实例化、属性赋值、初始化 [即：populate initialize]的结果）
abs.setAttribute("name", "GT");
// 4. Bean 元信息元素 - BeanMetadataElement
// 当前 Bean 来自于哪
abs.setSource(BeanConfigMetadataDemo.class);

DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// *1 若想让附加属性影响结果
factory.addBeanPostProcessor(new BeanPostProcessor() {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        BeanDefinition bd = factory.getBeanDefinition(beanName);

        // 4. Bean 元信息元素 - BeanMetadataElement
        // 通过 source 来判断
        if (BeanConfigMetadataDemo.class.equals(bd.getSource())) {
            User usr = User.class.cast(bean);
            // getAttribute 属性存储上下文
            usr.setName(bd.getAttribute("name").toString());
        }

        // 返回 bean 或 null 不影响结果是GT
        return bean;
    }
});
// 注册 user 的 beanDefinition
factory.registerBeanDefinition("user", abs);

System.out.println(factory.getBean(User.class));
// 没有*1结果：User(id=null, name=builder)
// 有*1结果：User(id=null, name=GT)
  ```

## Spring 容器配置元信息

## Spring 外部化配置元信息 - PropertySource

## Spring Profile 元信息 - @Profile



# Spring 容器配置元信息

- **Spring XML 配置元信息 - Beans 元素相关（底层实现 - BeanDefinitionParserDelegate）**

| beans 元素属性              | 默认值          | 使用场景                                                     |
| --------------------------- | --------------- | ------------------------------------------------------------ |
| profile                     | null            | Spring Profiles 配置值                                       |
| default-lazy-init           | default（继承） | 当 outer beans "default-lazy-init" 属<br>性存在时，继承该值，否则为false |
| default-merge               | default（继承） | 当 outer beans "default-merge" 属<br/>性存在时，继承该值，否则为false |
| default-autowire            | default（继承） | 当 outer beans "default-autowire" 属<br/>性存在时，继承该值，否则为no |
| default-autowire-candidates | null            | 默认 Spring Bean 名称 pattern                                |
| default-init-method         | null            | 默认 Spring Bean 自定义初始化方法                            |
| default-destroy-method      | null            | 默认 Spring Bean 自定义销毁方法                              |

> **outer beans 含义：**
>
> ```xml
> <beans xmlns="http://......">  <!--这个就是 outer beans -->
> 	<import resource="classpath:/..." />  <!-- inner beans -->
> </beans>
> ```

- **Spring XML 配置元信息 - 应用上下文**

| XML元素                          | 使用场景                               |
| -------------------------------- | -------------------------------------- |
| <context:annotation-config />    | 激活 Spring 注解驱动                   |
| <context:component-scan />       | Spring @Component 以及自定义注解扫描   |
| <context:load-time-weaver />     | 激活 Spring LoadTimeWeaver             |
| <context:mbean-export />         | 暴露 Spring Beans 作为 JMS Beans       |
| <context:mbean-server />         | 将当前平台作为 MBeanServer             |
| <context:property-placeholder /> | 加载外部化配置资源作为 Spring 属性配置 |
| <context:property-override />    | 利用外部化配置资源覆盖 Spring 属性值   |



# 基于 XML 资源装载 Spring Bean 配置元信息

- **Spring Bean 配置元信息（底层实现 - XmlBeanDefinitionReader）**

| XML 元素         | 使用场景                                      |
| ---------------- | --------------------------------------------- |
| <beans:beans />  | 单 XML 资源下的多个 Spring Beans 配置         |
| <beans:bean />   | 单个 Spring Bean 定义（BeanDefinition）配置   |
| <beans:alias />  | 为 Spring Bean 定义（BeanDefinition）映射别名 |
| <beans:import /> | 加载外部 Spring XML 配置资源                  |



# 基于 Properties 资源装载 Bean 配置元信息

- **Spring Bean 配置元信息**  (注：.properties文件里指定类名要加括号。如：User.(class)=org.xxx.xxx.User)
- **底层实现 - PropertiesBeanDefinitionReader**

| Properties 属性名 | 使用场景                        |
| ----------------- | ------------------------------- |
| (class)           | Bean 类全称限定名               |
| (abstract)        | 是否为抽象的 BeanDefinition     |
| (parent)          | 指定 parent BeanDefinition 名称 |
| (lazy-init)       | 是否为延迟初始化                |
| (ref)             | 引用其它 Bean 名称              |
| (scope)           | 设置 Bean 的 scope 属性         |
| ${n}              | n 表示第 n+1 个构造器参数       |



# 基于 java 注解装载 Spring Bean 配置元信息

| Spring 注解    | 场景说明                            | 起始版本 |
| -------------- | ----------------------------------- | -------- |
| @Repository    | 数据仓储模式注解                    | 2.0      |
| @Component     | 通用组件模式注解                    | 2.5      |
| @Service       | 服务模式注解                        | 2.5      |
| @Controller    | Web 控制器模式注解                  | 2.5      |
| @Configuration | 配置类模式注解                      | 3.0      |
| @Autowired     | Bean 依赖注入，支持多种依赖查找方式 | 2.5      |
| @Qualifier     | 细粒度的 @Autowired 依赖查找        | 2.5      |

| Java 注解 | 场景说明          | 起始版本 |
| --------- | ----------------- | -------- |
| @Resource | 类似于 @Autowired | 2.5      |
| @Inject   | 类似于 @Autowired | 2.5      |

| Spring 条件注解 | 场景说明                        | 起始版本 |
| --------------- | ------------------------------- | -------- |
| @Profile        | 配置条件化装配（dev,prod,test） | 3.1      |
| @Conditional    | 编程条件装配                    | 4.0      |



# 基于 Java 注解装载 Spring Bean 配置元信息

- **Spring Bean 生命周期回调注解**

| Spring 注解    | 场景说明                                                     | 起始版本 |
| -------------- | ------------------------------------------------------------ | -------- |
| @PostConstruct | 替换 XML 元素 <bean init-method="..." /> 或 InitializingBean | 2.5      |
| @PerDestory    | 替换 XML 元素 <bean destroy-method="..." /> 或 DisposableBean | 2.5      |



# Spring Bean 配置元信息底层实现

- **Spring BeanDefinition 解析与注册**

  | 实现场景        | 实现类                         | 起始版本 |
  | --------------- | ------------------------------ | -------- |
  | XML 资源        | XmlBeanDefinitionReader        | 1.0      |
  | Properties 资源 | PropertiesBeanDefinitionReader | 1.0      |
  | Java 注解       | AnnotatedBeanDefinitionReader  | 3.0      |

- **Spring XML 资源 BeanDefinition 解析注册**
  - 核心 API - XmlBeanDefinitionReader
    - 资源 - Resource
    - 底层 - BeanDefinitionDocumentReader
      - XML 解析 - Java DOM Level 3 API
      - BeanDefinition 解析 - BeanDefinitionParserDelegate
      - BeanDefinition 注册 - BeanDefinitionRegistry

- **Spring Properties 资源 BeanDefinition 解析与注册**

  - 核心 API - PropertiesBeanDefinitionReader

    - 资源
      - 字节流 - Resource
      - 字符流 - EncodedResource

    - 底层
      - 存储 - java.util.Properties
      - BeanDefinition 解析 - API 内部实现
      - BeanDefinition 注册 - BeanDefinitionRegistry

- **Java 注册 BeanDefinition 解析注册**
  - 核心 API - AnnotatedBeanDefinitionReader
    - 资源
      - 类对象 java.lang.Class
    - 底层
      - 条件评估 - ConditionEvaluator
      - Bean 范围解析 - ScopeMetadataResolver
      - BeanDefinition 解析 - API 内部实现
      - BeanDefinition 处理 - AnnotationConfigUtils.processCommonDefinitionAnnotations
      - BeanDefinition 注册 - BeanDefinitionRegistry



# 基于 XML 资源装载 Spring IoC 窗口配置元信息

- **Spring IoC 容器相关 XML 配置**

| 命名空间 | 所属模块       | Schema 资源 URL                                              |
| -------- | -------------- | ------------------------------------------------------------ |
| beans    | spring-beans   | https://www.springframework.org/schema/beans/spring-beans.xsd |
| context  | spring-context | https://www.springframework.org/schema/context/spring-context.xsd |
| aop      | spring-aop     | https://www.springframework.org/schema/aop/spring-aop.xsd    |
| tx       | spring-tx      | https://www.springframework.org/schema/tx/spring-tx.xsd      |
| util     | spring-beans   | https://www.springframework.org/schema/util/spring-util.xsd  |
| tool     | spring-beans   | https://www.springframework.org/schema/tool/spring-tool.xsd  |



# 基于 Java 注解装载 Spring IoC 容器配置元信息

- **Spring IoC 容器装载注解**

| Spring 注解     | 场景说明                                    | 起始版本 |
| --------------- | ------------------------------------------- | -------- |
| @ImportResource | 替换 XML 元素 <import />                    | 3.0      |
| @Import         | 导入 Configuration Class                    | 3.0      |
| @ComponentScan  | 扫描指定 package 下标注 Spring 模式注解的类 | 3.1      |

```xml
<bean id="user" class="ioc.dependency.domain.User" primary="true">
    <property name="id" value="1" />
    <property name="name" value="nm" />
</bean>
<bean id="super" class="ioc.dependency.domain.SuperUser" parent="user">
    <property name="auth" value="admin" />
</bean>
```

```java
// 结果：
// user: User(id=1, name=nm)
// super: SuperUser(super=User(id=1, name=nm), auth=admin)
@ImportResource("classpath:/META-INFO/lookup-context.xml")
// 结果：
// ioc.dependency.domain.User: User(id=null, name=null)
@Import(User.class)
public class SpringIoCContainerDemo {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
        ctx.register(SpringIoCContainerDemo.class);
        ctx.refresh();
        Map<String, User> map = ctx.getBeansOfType(User.class);
        map.entrySet()
            .stream()
            .forEach(x -> System.out.println(x.getKey() + ": " + x.getValue()));
        ctx.close();
    }
}
```

- **Spring IoC 配置属性注解**

| Spring 注解      | 场景说明                         | 起始版本 |
| ---------------- | -------------------------------- | -------- |
| @PropertySource  | 配置属性抽象 PropertySource 注解 | 3.1      |
| @PropertySources | @PropertySource 集合注解         | 4.0      |

```pro
# a.properties
usr.(class) =ioc.dependency.domain.User
usr.id = id-a.properties
usr.name = nm-a.properties
```

```java
// 要和 @Bean 组合使用
@PropertySource("classpath:/META-INF/a.properties")
// java 1.8+ 支持 @Repeatable 表示可以 @PropertySource 多个
//@PropertySource("classpath:/META-INF/b.properties")
//@PropertySource("classpath:/META-INF/c.properties")
//...
@Bean
// @Value： 从 a.properties 里取值
public User user(@Value("${usr.id}") String id, @Value("${usr.name}") String name) {
    return new User().setId(id).setName(name);
}
// forEach(x -> System.out.println(x.getKey() + ": " + x.getValue()) 结果：
// user: User(id=id-a.properties, name=nm-a.properties)
```



# 基于 Extensible XML authoring 扩展 Spring XML 元素

- **Spring XML 扩展**

  - 编写 XML Schema 文件：定义 XML 结构

  - 自定义 NamespaceHandler 实现：命名空间绑定

  - 自定义 BeanDefinitionParser 实现：XML 元素与 BeanDefinition 解析

  - 注册 XML 扩展：命名空间与 XML Schema 映射

```java
@Data
@Accessors(chain = true)
public class User {
    String id;
    String name;
    City city;
}
```

```xml
<!-- 文件路径：resource/org/config/metadata/user.xsd => 跟main方法的类package相同 -->
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<xsd:schema xmlns="http://gt.lookup.org/schema/users"            <!-- 自定义url -->
            xmlns:xsd="http://www.w3.org/2001/XMLSchema"
            targetNamespace="http://gt.lookup.org/schema/users"> <!-- 自定义url -->

    <xsd:import namespace="http://www.w3.org/XML/1998/namespace"/>

    <!-- 定义 User 类型（复杂类型） -->
    <xsd:complexType name="User">
        <!-- 字段名：id,类型：string,必须项 -->
        <xsd:attribute name="id" type="xsd:string" use="required" />
        <xsd:attribute name="name" type="xsd:string" use="required" />
        <xsd:attribute name="city" type="City" use="required" />
    </xsd:complexType>

    <!-- 定义 枚举（简单类型） -->
    <xsd:simpleType name="City">
        <!-- base="xsd:string" 关键，不写报错 -->
        <xsd:restriction base="xsd:string">
            <xsd:enumeration value="Beijing" />
            <xsd:enumeration value="Shanghai" />
            <xsd:enumeration value="Shengzhen" />
        </xsd:restriction>
    </xsd:simpleType>

    <!-- 定义 user 元素 -->
    <xsd:element name="user" type="User" />
</xsd:schema>
```

```xml
<!-- 文件路径：resource/META-INF/user-context.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:users="http://gt.lookup.org/schema/users"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://gt.lookup.org/schema/users http://gt.lookup.org/schema/users.xsd">

    <!-- 要成对出现 -->
    <!-- xmlns:users="http://gt.lookup.org/schema/users" -->
    <!-- http://gt.lookup.org/schema/users http://gt.lookup.org/schema/users.xsd -->
    <users:user id="id-gt" name="name-gt" city="Shanghai" />
</beans>
```

```java
// 自定义 BeanDefinitionParser 实现：XML 元素与 BeanDefinition 解析
public class UserBeanDefinitionParser extends AbstractSingleBeanDefinitionParser {
    @Override // 返回 User类型
    protected Class<?> getBeanClass(Element element) { return User.class; }

    @Override
    protected void doParse(Element element, BeanDefinitionBuilder builder) {
        // element 值来自 user-context.xml
        // 设置 BeanDefinition 属性值
        builder.addPropertyValue("id", element.getAttribute("id"));
        builder.addPropertyValue("name", element.getAttribute("name"));
        builder.addPropertyValue("city", element.getAttribute("city"));
    }
}
```

```java
// 自定义 NamespaceHandler 实现：命名空间绑定
public class UserNamespaceHandler extends NamespaceHandlerSupport {
    @Override
    public void init() {
        // 注册 user 元素对应的 BeanDefinitionParser 实现
        registerBeanDefinitionParser("user", new UserBeanDefinitionParser());
    }
}
```

```properties
# 文件路径：resource/META-INF/spring.handlers
# 定义 namespace 与 NamespaceHandler 映射
http\://gt.lookup.org/schema/users=org.config.metadata.UserNamespaceHandler
```

```properties
# 文件路径：resource/META-INF/spring.schemas
http\://gt.lookup.org/schema/users.xsd = org/config/metadata/user.xsd
```

```java
DefaultListableBeanFactory fy = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(fy);
reader.loadBeanDefinitions("META-INF/user-context.xml");
System.out.println(fy.getBean(User.class));
// 结果：
// User(id=id-gt, name=name-gt, city=Shanghai)
```

## Extensible XML authoring 扩展原理

- **触发时机**
  1. AbstractApplicationContext#obtainFreshBeanFactory
  2. AbstractRefreshableApplicationContext#refreshBeanFactory
  3. AbstractXmlApplicationContext#loadBeanDefinitions
  4. 。。。
  5. XmlBeanDefinitionReader#doLoadBeanDefinitions
  6. 。。。
  7. BeanDefinitionParserDelegate#parseCustomElement

- **核心流程**
  - BeanDefinitionParserDelegate#parseCustomElement(Element, BeanDefinition)
    - 获取 namespace
    - 通过 namespace 解析 NamespaceHandler
    - 构造 ParserContext
    - 解析元素，获取 BeanDefinition



# 基于 Properties 资源装载外部化配置

- **注解驱动**
  - org.springframework.context.annotation.PropertySource
  - org.springframework.context.annotation.PropertySources

- **API 编程**
  - org.springframework.core.env.PropertySource
  - org.springframework.core.env.PropertySources

```java
@PropertySource("classpath:/META-INF/a.properties")
public class SpringIoCContainerDemo {
    @Bean
    public User user(@Value("${usr.id}") String id,
                     @Value("${usr.name}") String name) {
        return new User().setId(id).setName(name);
    }
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = 
            			new AnnotationConfigApplicationContext();
        Map<String, Object> m = new HashMap<>();
        m.put("usr.name", "haha");
        // 扩展 Environment 中的 PropertySource
        // 添加 PropertySource 操作必须在 refresh 之前完成
        // * 这里 new 的是 org.springframework.core.env.PropertySource
        ctx.getEnvironment()
            .getPropertySources()
            .addFirst(new MapPropertySource("first", m));
        ctx.register(SpringIoCContainerDemo.class);
        ctx.refresh();
        Map<String, User> map = ctx.getBeansOfType(User.class);
        map.entrySet().stream()
            .forEach(x -> System.out.println(x.getKey() + ": " + x.getValue()));

        ctx.close();
    }
}
// 结果（name=haha）：
// user: User(id=id-a.properties, name=haha, city=null)
```



# 基于 YAML 资源装载外部化配置

- **API 编程**
  - YamlProcessor
    - YamlMapFactoryBean
    - YamlPropertiesFactoryBean

```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.18</version>
</dependency>
```

```yam
# META-INF/user.yml
user:
  id: id-yaml
  name: nm-yaml
```

```java
@Bean
public YamlPropertiesFactoryBean yaml() {
    YamlPropertiesFactoryBean bean = new YamlPropertiesFactoryBean();
    bean.setResources(new ClassPathResource("META-INF/user.yml"));
    return bean;
}
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = 
        new AnnotationConfigApplicationContext();
    ctx.register(YamlPropertyDemo.class);
    ctx.refresh();
    System.out.println(ctx.getBean("yaml", Map.class));
    ctx.close();
}
// 结果：
// {user.id=id-yaml, user.name=nm-yaml}
```

