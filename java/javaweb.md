# 1. 基本概念

在 java 中，动态 web 资源开发的技术统称为 javaweb

## 1.1 jsp / servlet

- sun 公司主推的 B/S 架构
- 基于 java 语言
- 可以承载三高问题（高并发，高可用，高性能）
- 语法像ASP（比 ASP 出来的晚）

# 2. servlet

## 2.1 简介

- sun 公司开发动态 web 的一门技术
- 开发动态 web 所用到的 api 中提供了一个叫 servlet 的接口
- 开发 servlet 程序需要两步：
  - 编写一个类，实现 servlet 接口
  - 把开发好的 java 类部署到 web 服务器上
- **实现了 servlet 接口的 java 程序叫 servlet 程序**
- 两个默认实现类：==GenericServlet== 和 ==HttpServlet==，HttpServlet 继承了 GenericServlet

## 2.2 HelloServlet

### 2.2.0 添加依赖

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.3</version>
</dependency>
<!-- 因为要依赖本地 tomcat，所以不能是pom -->
<!-- 变成 war 后一定要重刷下 maven，否则配tomcat时没法配置Deployment项 -->
<packaging>war</packaging>
```

### 2.2.1 创建 module

new  -->  model  -->  选中 Create from archetype  -->  maven-archetype-webapp  -->  next

### 2.2.2 实现 HttpServlet

右键 main 文件夹  -->  添加 java 文件夹  -->  右键 java 文件夹 -->  mak directory as  -->  Sources root

再在 java 下添加 gt.com.servlet.HelloServlet.java

```java
// 可以想象成 Controller
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().print("Hello Servlet Get");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().print("Hello Servlet Post");
    }
}
```

### 2.2.3 编写 servlet 映射

> 为什么要映射？
>
> 需要通过浏览器访问，而浏览器需要连接 web 服务器，所以需要在 web 服务器中注册上面写的 servlet 程序。
>
> 还需要给它一个浏览器访问的路径。

### 2.2.4 添加 web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 注册 servlet -->
    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>gt.com.servlet.HelloServlet</servlet-class>
    </servlet>
    <!-- Servlet的请求路径 -->
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

### 2.2.5 添加tomcat

Add Configuration...  -->  左上角 +  -->  Tomcat Server  -->  Local

**tab标签：Server** 

Application server：选本地 tomcat 的路径

JRE：选本地 jdk 版本==（选了这项下面的 JMX port: 才会显示1099）==

**tab标签：Deployment**

点 +  -->  Artifact...  -->  选 xxxx:war

Application context：/s1  ==//启动后访问localhost:8080/s1==

### 2.2.6 启动测试

运行后访问： localhost:8080/s1/hello

## 2.3 Servlet 原理

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;
    ServletConfig getServletConfig();
	// 浏览器发起 http 请求会调这个方法
    // 从 var1 里取前台传过来的参数
    // 处理完后将结果返回给 var2
    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
    String getServletInfo();
    void destroy();
}
```

## 2.4 Mapping

- 一个 servlet 指定一个路径

  ```xml
  <servlet-mapping>
      <servlet-name>hello</servlet-name>
      <url-pattern>/hello</url-pattern>
  </servlet-mapping>
  ```

- 一个 servlet 指定多个路径

  ```xml
  <servlet-mapping>
      <servlet-name>hello</servlet-name>
      <url-pattern>/hello1</url-pattern>
  </servlet-mapping>
  <servlet-mapping>
      <servlet-name>hello</servlet-name>
      <url-pattern>/hello2</url-pattern>
  </servlet-mapping>
  ```

- 一个 servlet 指定通配符路径

  ```xml
  <servlet-mapping>
      <servlet-name>hello</servlet-name>
      <url-pattern>/hello/*</url-pattern>
  </servlet-mapping>
  ```

- 自定义后缀

  ```xml
  <servlet-mapping>
      <servlet-name>hello</servlet-name>
      <!-- * 前不能加路径 -->
      <url-pattern>*.do</url-pattern>
  </servlet-mapping>
  ```

## 2.5 ServletContext

web 容器在启动时会为每个 web 程序都创建一个对应的 ServletContext 对象

### 2.5.1 数据共享

在一个 servlet 里存数据，另一个 servlet 中可以拿到

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // Servlet 上下文
        this.getServletContext().setAttribute("id", 111);
    }
}
public class GetServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().print(getServletContext().getAttribute("id"));
    }
}
// 先访问 HelloServlet 再访问 GetServlet
// 结果：111
```

### 2.5.2 获取初始化参数

```xml
<context-param>
    <param-name>url</param-name>
    <param-value>jdbc:mysql://localhost:3306/mybatis</param-value>
</context-param>
```

```java
resp.getWriter().print(getServletContext().getInitParameter("url"));
// 结果：jdbc:mysql://localhost:3306/mybatis
```

### 2.5.3 请求转发

```java
getServletContext().getRequestDispatcher("/srv01").forward(req, resp);
```

### 2.5.4 读取资源文件

```xml
<!-- pom.xml -->
<!-- 当发现在java文件夹下创建的 .properties 文件没有包含在 target 里的解决办法 -->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
<!-- 然后重刷下maven，clean 下工程，再启动；就会发现放在 java 下的 .properties 文件包含在 target下了 -->
```

**读取文件**

```properties
# main/resources/db.properties
username=root
password=123456
```

```java
InputStream is = getServletContext().getResourceAsStream("/WEB-INF/classes/db.properties");
Properties p = new Properties();
p.load(is);
resp.getWriter().print(p.getProperty("password")); // 结果：123456
```

## 2.6 HttpServletResponse

### 2.6.1 方法分类

**向浏览器发送数据的方法**

```java
ServletOutputStream getOutputStream() throws IOException;
PrintWriter getWriter() throws IOException;
```

**向浏览器发送响应头的方法**

```java
void setCharacterEncoding(String var1);
void setContentLength(int var1);
void setContentLengthLong(long var1);
void setContentType(String var1);
void setDateHeader(String var1, long var2);
void addDateHeader(String var1, long var2);
void setHeader(String var1, String var2);
void addHeader(String var1, String var2);
void setIntHeader(String var1, int var2);
void addIntHeader(String var1, int var2);
```

## 2.7 HttpServletRequest

## 2.8 @WebServlet

每个 servlet 都要在 web.xml 中配置才能使用，比较麻烦。所以 Servlet 3.0 之后提供了这个注解

等价于在 web.xml 中配置 <servlet-mapping> 的 <url-pattern>

### 2.8.1 注解属性

| 属性               | 类型           | 是否必须 | 说明                                                         |
| ------------------ | -------------- | -------- | ------------------------------------------------------------ |
| asyncSupported     | boolean        | 否       | 指定 servlet 是否支持异步操作形式                            |
| displayName        | String         | 否       | 指定 servlet 显示名称                                        |
| initParams         | WebInitParam[] | 否       | 配置初始化参数                                               |
| loadOnStartup      | int            | 否       | 标记容器是否在应用启动时就加载这个Servlet<br>默认或负数：客户端第一次请求时再加载<br>0或正数：应用启动就加载；值越小，加载该 servlet 的优先级越高 |
| name               | String         | 否       | 指定 servlet 名称<br>如果指定：通过 getServletName 获取<br>如果没指定：getServletName 获取到的是 servlet 完整类名 |
| urlPattems / value | String[]       | 否       | 这两个属性作用相同，指定 servlet 处理的 url<br>可配置多个映射，如：urlPattems={"/api/test1", "/api/test2"}<br>- /或/\*：拦截所有<br>- \*.do：拦截是 .do 的请求<br>- /user/*.do，/\*.do，test\*.do 都是非法的，启动会报错 |

```java
// 不需要在 web.xml 里配任何配置
@WebServlet("/ann/servlet")
public class AnnotationDemoServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().print("AnnotationDemoServlet");
    }
}
```

https://www.bilibili.com/video/BV12J411M7Sj?p=12



# 4. JSP

## 4.1 什么是 JSP

java server pages：java 服务器端页面，和 servlet 一样，用于动态 web 技术

jsp 中可以嵌入 java 代码

## 4.2 JSP 原理

### 4.2.1 JSP 如何执行的？

- 服务器内部工作

  - tomcat 中有一个 work 目录

  - 浏览器发送请求不管访问什么资源，其实都是在访问 Servlet

  - JSP 最终也会被转换成一个 java 类

  - JSP 本质上就是一个 Servlet

    ```java
    // HttpJspBase 继承 HttpServlet
    public final class index_jsp extends HttpJspBase {
      public void _jspInit() {}
    
      public void _jspDestroy() {}
    
      public void _jspService(final HttpServletRequest request, final HttpServletResponse response)
            throws IOException, ServletException
    }
    ```

  - 内置对象

    ```java
    final javax.servlet.jsp.PageContext pageContext;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
    ```

  - 输出页面前增加的代码

    ```java
    // 设置页面响应类型
    response.setContentType("text/html; charset=UTF-8");
    pageContext = _jspxFactory.getPageContext(this, request, response, null, false, 8192, true);
    _jspx_page_context = pageContext;
    application = pageContext.getServletContext();
    config = pageContext.getServletConfig();
    out = pageContext.getOut();
    _jspx_out = out;
    ```

  - 以上对象都可在 JSP 页面中直接使用

https://www.bilibili.com/video/BV12J411M7Sj?p=19

# 5. Filter

对请求进行过滤，==发生在 servlet 之前==（跟 servlet 一模一样，只不过发生在 servlet 之前）

```java
// @Component 坑点：如果在 Spring 框架里必须加 否则不起作用
// 拦截所有请求
@WebFilter("/*") // 注解方式
public class FilterDemo implements Filter {
    @Override
    // tomcat 启动时走
    public void init(FilterConfig filterConfig) throws ServletException {}
    
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
        		throws IOException, ServletException {
        // 放行请求，如果不写就不再走 servlet
        chain.doFilter(req, res);
    }
    
    @Override
    // tomcat 停止进走
    public void destroy() { }
}
```

```xml
<!-- web.xml 非注解方式 -->
<filter>
    <filter-name>filter</filter-name>
    <filter-class>com.gt.filter.FilterDemo</filter-class>
</filter>
<filter-mapping>
    <filter-name>filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

# 6. 监听器

1. 实现监听器接口

```java
// 统计网站在线人数：即 session 个数
@WebListener
public class OnlineCountListener implements HttpSessionListener {
    //每次创建 session 都会触发
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        // session 的 id
        System.out.println(se.getSession().getId());
        // 手动销毁 session
        req.getSession().invalidate();
        ServletContext ctx = se.getSession().getServletContext();
        Integer ct = (Integer) ctx.getAttribute("OnlineCount");
        ct = ct == null ?  1 : ct + 1;
        ctx.setAttribute("OnlineCount", ct);
    }

    // 每次销毁 session 都会触发
    @Override
    public void sessionDestroyed(HttpSessionEvent se) {}
}
```

2. 配置监听器（web.xml）

```xml
<listener>
    <listener-class>com.gt.listener.OnlineCountListener</listener-class>
</listener>
<!-- 或 @WebListener -->
```



https://www.bilibili.com/video/BV12J411M7Sj?p=26

