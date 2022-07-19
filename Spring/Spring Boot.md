# 0. 依赖

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<!-- 或者 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <type>pom</type> <!-- 写在dependency里必须type=pom -->
</dependency>
```



# 1. 填坑

## 1.1 @Autowired

> 当主工程（如 web）@Autowired 其它工程（如 domain）的 bean 时，发现 idea 里提示找不到

```java
// 在app类上加个扫包
@SpringBootApplication(scanBasePackages = "com.cq")
```

> 当 @Configuration 的类里 @Autowired 其它工程的 bean 时，运行后发现是 null

```java
// 先注入 content
@Autowired
ApplicationContext context;
// 再在用到的地方 获取这个 bean
context.getBean(xxx.class);
```

## 1.2 跨域

```java
// 方法一：
// 允许 http://localhost:8080 可以跨域访问自己
@CrossOrigin(origins = "http://localhost:8080")
// 允许所有请求访问自己
@CrossOrigin
// 可以放在 action / controller 上

// 方法二：
@Bean // 全局配置跨域
public CorsFilter corsFilter() {
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    CorsConfiguration config = new CorsConfiguration();
    config.addAllowedOrigin("http://localhost:8001");
    config.addAllowedHeader("*");//("x-requested-with");
    config.addAllowedMethod("*");
    source.registerCorsConfiguration("/**", config);
    return new CorsFilter(source);
}

// 方法三：
@Configuration
public class CORSConfiguration extends WebMvcConfigurerAdapter {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry
                .addMapping("/**")
                .allowedMethods("*")
                .allowedOrigins("*") // .allowedOrigins("http://localhost:8080", "http://localhost:81")
                .allowedHeaders("authorization","Accept");
    }
}
```

==严重注意==：当登陆后发现还是跨域，此时要将 ==HttpSecurity.cors()== 打开。



## 1.3 get传空参数

```java
// 1. 一个带参和一个不带参
@GetMapping(path = {"detail/{code}", "detail"})
// 2. required = false
public String detail(@PathVariable(required = false, name = "code") String code) {}
// 3. 前台请求
//    href = '/detail';
//    href = '/detail/aa';
```

## 1.4 tomcat启动war报错

idea 运行没问题，打成 war 包后 tomcat 启动就报错：

```verilog
Caused by: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Catalina].StandardHost[localhost].StandardContext[/tjj]]
                at org.apache.catalina.util.LifecycleBase.handleSubClassException(LifecycleBase.java:440)
                at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:198)
                at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:717)
```

```java
// 删除了这个就可以启动了
public class ServletInitializer extends SpringBootServletInitializer {
	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(TjjSingleApplication.class);
	}
}
```

相关介绍：https://blog.csdn.net/zq17865815296/article/details/79464403

## 1.5 @Transactional失效

```java
// 原因一：当异常为非 RuntimeException 时，不会回滚，只要 rollbackFor=Exception.class 就可以了
@Transactional(rollbackFor=Exception.class)
// 原因二：同一个类中无事务方法调用了有事务方法，当有事务方法异常时不会回滚（就算加了rollbackFor=Exception.class也不回滚）
//        问题原因就和aop切不到this.方法一样，所以解决的办法也一样
@Service
public class TranService {
    // 关键在这里
    // 1. 先注入一个自己
    @Autowired
    private TranService me;
    
    // 2. 带事务的方法必须是 public
    @Transactional
    public void tranMethod() {}
    
    public void main() {
        // 3. 用注入的自己调用带事务的方法就ok了
        me.tranMethod();
    }
}
```

## 1.6 parent.relativePath警告

> ‘parent.relativePath’ of POM 包名:xxx

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
    <relativePath /> <!-- 加上就好了 -->
</parent>
```

## 1.7 idea报cmd line too long

> 解决Intellij IDEA运行报Command line is too long的问题

```xml
<component name="PropertiesComponent">
	<property name="dynamic.classpath" value="true" /> <!-- 就加这句 -->
</component>
```

# 2. lombok

```java
@Accessors(chain = true)     //开启链式写法
@ToString(callSuper = true)  //是否包含父类信息
@Builder //使用创建者模式让被标注的类以实现链式调用风格
```

## 2.1  日志

```java
@Slf4j  // 加上log注解，log方法才能用
@RestController
public class ApiController {
    @GetMapping("")
    public Object index() {
        log.info("---------info insert---------");
        log.debug("---------debug insert---------");
        log.warn("---------warn insert---------");
        log.error("---------error insert---------");       
    }
}
```

# 3. session

```java
// 判断 session 是否有效
HttpServletRequest.isRequestedSessionIdValid();
// 手动使 session 无效
HttpServletRequest.getSession().invalidate();
// 跳转
HttpServletResponse.sendRedirect("/login");
```

## 3.2 拦截ajax请求session过期

参照 springMVC.md -> 6.拦截器

```java
public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) throws Exception {
    // 只拦截 ajax，非 ajax 的 security 都拦了
    // req.getHeader("X-Requested-With") == "XMLHttpRequest" 表示 ajax 请求
    if ("XMLHttpRequest".equals(req.getHeader("X-Requested-With")) && !req.isRequestedSessionIdValid()) {
        res.getWriter().write("session_timeout");
        return false;
    }
    return true;
}
```

```javascript
// 添加一个全局js文件，每个页面都要引
// ajaxSetup 添加全局 ajax 设置
$.ajaxSetup({
    complete: function(req) {
        // 上面 res.getWriter().write("session_timeout");
        if (req.responseText == 'session_timeout') {
            // 跳转到登陆页
            location.href = '/login';
        }
    }
});
```

# 4. 获取 request / response

```java
ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
HttpServletRequest req = attr.getRequest();
HttpServletResponse res = attr.getResponse();
String s = req.getParameter("key");
```

# 5. 获取当前请求URL

```java
// http://127.0.0.1:8080/world/index.jsp?name=lilei&sex=1
HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
request.getScheme() = "http";
request.getServerName() = "127.0.0.1";
request.getServerPort() = "8080";
request.getContextPath() = "/world";
request.getServletPath() = "index.jsp";
request.getQueryString() = "name=lilei&sex=1";
```

# 6. 读取 yml

```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
</dependency>
```

```java
String classPath = ClassUtils.getDefaultClassLoader().getResource("").getPath();
FileInputStream sm = new FileInputStream(new File(classPath + "application.yml"));
Yaml yml = new Yaml();
Properties p = yml.loadAs(sm, Properties.class);
// 取单个值
p.getProperty("key");
// 取一个结点上的
LinkedHashMap map = (LinkedHashMap)p.get("spring");
```

# 7. pom.xml

```xml
<!--project是pom.xml的根元素，包含了一些约束的信息，比如xlms,xsi-->
<project>
    <!--指定了当前pom的版本-->
    <modelVersion>4.0.0</modelVersion>
    <!--maven2.0必须是这样写，现在是maven2唯一支持的版本-->
    <!-- 基础设置 -->
    <groupId>反写公司的网址+项目名称</groupId>
    <artifactId>项目名称+模块名</artifactId>
    <version>当前项目版本号</version>
    <!-- 设置项目jdk版本 -->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>
        UTF-8
    </project.reporting.outputEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <!--各种版本管理-->
    <spring.version>5.2.2.RELEASE</spring.version>
    <java.version>1.8</java.version>
  </properties>
    <!-- 默认是jar，还可以打包成war/zip/pom-->
    <packaging>...</packaging>
    <!-- 项目描述名，一般是写项目文档的时候才使用 -->
    <name>...</name>
    <!-- 项目的地址-->
    <url>...</url>
    <!-- 项目的描述 -->
    <description>...</description>
    <!-- 开发人员的列表 -->
    <developers>...</developers>
    <!-- 许可证的信息 -->
    <licenses>...</licenses>
    <!-- 组织信息 -->
    <organization>...</organization>
    <!-- 依赖列表，下面可以包含多个依赖项dependency-->
    <dependencies>
        <dependency>
            <!-- 指定坐标确定依赖项的位置 -->
            <groupId></groupId>
            <artifactId></artifactId>
            <version></version>
            <type></type>
            <!-- 依赖包的依赖范围-->
            <scope></scope>
            <!-- 这个标签只有true和false两个值，是用来设置依赖是否可选 -->
            <optional></optional>
            <!-- 排除依赖传递列表 -->
            <exclusions> 
                <exclusion>
                </exclusion>  
            </exclusions>
        </dependency>
    </dependencies>
    <!-- 依赖管理，里面包含多个依赖，但是它并不会被运行，-->
    <!-- 即不会被引用到实际的依赖中 -->
    <!-- 这个标签主要是用来定义在父模块中，供子模块继承用 -->
    <dependencyManagement>                              
        <dependencies>
        <dependency>
        </dependency>
        </dependencies>
    </dependencyManagement>
    <!-- 常用于给构件的行为提供相应的支持 -->
    <build>
        <!-- 插件列表 -->
        <plugins>
            <plugin>
                <!--指定jdk1.8-->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <!--<version>3.6.0</version>-->
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <!-- 用于在子模块中对父模块的pom的继承 -->
    <parent>...</parent>
    <!-- 用来聚合多个模块，让多个模块进行编译，不用一个一个来 -->
    <modules>
        <module>
        </module>
    </modules>
</project>
```

scope：一个maven项目中，如果存在编译需要而发布不需要的jar包，可以用scope标签，值设为provided

**scope其它参数：**

compile：默认scope，表示 dependency 都可以在生命周期中使用。且这些dependencies会传递到依赖的项目中。适用于所有阶段，会随着项目一起发布

provided：跟compile相似，但表明了dependency由JDK或容器提供，如Servlet AP和一些Java EE APIs。这个scope只能作用在编译和测试时，同时没有传递性。

runtime：表示dependency不作用在编译时，但会作用在运行和测试时，如JDBC驱动，适用运行和测试阶段。

test：表示dependency作用在测试时，不作用在运行时。 只在测试时使用，用于编译和运行测试代码。不会随项目发布。

system：跟provided 相似，不过被依赖项不会从maven仓库抓，而是从本地文件系统拿，一定需要配合systemPath属性使用。

import：https://blog.csdn.net/sunzhenhua0608/article/details/80767419



# 8. maven

## 8.1 国内镜像

```xml
<!-- maven/conf/settings.xml -->
<mirrors>
    <mirror> 
        <id>nexus-aliyun</id> 
        <mirrorOf>central</mirrorOf>   
        <name>Nexus aliyun</name> 
        <url>http://maven.aliyun.com/nexus/content/groups/public</url> 
    </mirror>
</mirrors>
```

## 8.2 本地仓库

```xml
<!-- maven/conf/settings.xml -->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- 指定本地库路径 -->
  <localRepository>C:/Program Files (x86)/apache-maven-3.6.0/repository</localRepository>
```

## 8.3 springboot用jar包启动

```xml
<!-- 步骤一：pom.xml -->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

```yaml
# 步骤二：application.yml里指定端口和前缀
server:
  port: 8081
  servlet:
    context-path: /xxx
```

```bash
# 步骤三：cmd到项目根路径下执行
mvn clean package -Dmaven.test.skip=true
```

```bash
# 步骤四：cmd到target下执行jar包
java -jar xxx.jar # 双击也可
#        后台执行
nohup java -jar xxx.jar &
```

# 9. bean

## 普通类获取bean

```java
@Component
public class SpringBeanUtil implements ApplicationContextAware {
    protected static ApplicationContext ctx;

    @Override
    public void setApplicationContext(ApplicationContext ctx) throws BeansException {
        SpringBeanUtil.ctx = ctx;
    }

    public static <T> T getBean(Class<T> cls) {
        return ctx.getBean(cls);
    }
}
```

## 快速获取bean

```java
ContextLoader.getCurrentWebApplicationContext().getBean(xxx.class);
```

# 10. @Value

```yaml
cors:
  origins: htto://localhost:80,htto://localhost:8080 # ,号前后不能有空格
  map: "{k1:'v1',k2:2}"
```

```java
@Value("#{'${cors.origins}'.split(',')}")
private String[] ary;            // 数组
@Value("#{'${cors.origins}'.split(',')}")
private List<String> list;       // list
@Value("#{${cors.map}}")
private Map<String, Object> map; // map
```

# 11. @AutoConfigureBefore/After

```java
// A将会在B和C之前加载
@AutoConfigureBefore({B.class,C.class})
public class A {}
// A将会在B和C之后加载
@AutoConfigureAfter({B.class,C.class})
public class A {}
```

# 12. yml语法

```yaml
port: ${server.port:8888} # 冒号:表示如果该文件里没配置server.port，则默认值是8888
```

# 13. @Transactional

https://www.cnblogs.com/seasail/p/12179363.html

https://blog.csdn.net/jiangyu1013/article/details/84397366

## 13.1 隔离级别

```java
// 参考 `mysql.md` -> 9，锁
@Transactional(isolation = Isolation.READ_UNCOMMITTED) // 读取未提交数据(会出现脏读,不可重复读) 基本不使用
@Transactional(isolation = Isolation.READ_COMMITTED)   // 读取已提交数据(会出现不可重复读和幻读)
@Transactional(isolation = Isolation.REPEATABLE_READ)  // 可重复读(会出现幻读) 
@Transactional(isolation = Isolation.SERIALIZABLE)     // 串行化
```

## 13.2 传播行为

```java
@Transactional(propagation = Propagation.REQUIRED)      // 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务(默认)
@Transactional(propagation = Propagation.SUPPORTS)      // 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行
@Transactional(propagation = Propagation.MANDATORY)     // 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常
@Transactional(propagation = Propagation.REQUIRES_NEW)  // 创建一个新的事务，如果当前存在事务，则把当前事务停掉
@Transactional(propagation = Propagation.NOT_SUPPORTED) // 如果当前存在事务，则把当前事务停掉
@Transactional(propagation = Propagation.NEVER)         // 如果当前存在事务，则抛出异常
@Transactional(propagation = Propagation.NESTED)        // 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行
```



# 附录

## 获取目录

```java
// 项目路径
System.getProperty("user.dir");
// classes 根目录
// /D:/work/git/tjj/tjj-single/target/classes/
String s = ClassUtils.getDefaultClassLoader().getResource("").getPath();
String s = ResourceUtils.getURL("classpath:").getPath();
// 坑点：
// 如果获取的路径是 D:\Program Files\apache-tomcat-9.0.27\webapps\
// 则会把 Program Files 变成 Program%20Files
// 所以取得的路径要转下码
s = URLDecoder.decode(s, "UTF-8");
```

## 下载文件

```java
@GetMapping("download/{file}")
public void download(@PathVariable String file, HttpServletResponse res) throws IOException {
    // tomcat/webapps/uploads
    String path = new File(System.getProperty("user.dir")).getParent() + "\\webapps\\uploads\\" + file;
    res.setContentType("application/force-download");
    res.setHeader("Content-Disposition", "attachment;filename=" + file);
    // util 为 hutool
    res.getOutputStream().write(FileUtil.readBytes(path));
    IoUtil.close(res.getOutputStream());
}
```

## 发布两个相同工程

tomcat 下发布两个相同的 springboot 工程其中一个404

```properties
# 每个工程指定一个不同的 domain
spring.jmx.default-domain=project1
spring.jmx.default-domain=project2
```

