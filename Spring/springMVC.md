# 1. Servlet 回顾

## 1.1 创建工程

1，创建一个普通的 maven module

2，右键刚创建的 module ---> Add Framework Support  ---> 勾上 Web Application 点 OK

# 2. 初识 SpringMVC

围绕 DispatcherServlet 设计。DispatcherServlet 将所有的请求分发到不同的处理器（controller），可以采用 @Controller 注解来声明。

DispatcherServlet 是 SpringMVC 的核心，任何请求都会被它拦截

```java
// 继承关系图
DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
// 所以 DispatcherServlet 本质上也是一个 Servlet
```

# 3. HelloSpringMVC

1，创建 module

2，添加 web 支持

3，web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 注册 DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 关联一个 springmvc 的配置文件：[servlet-name]-servlet.xml -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- 在 src/main/resources 下创建 -->
            <!-- 右键 resources 》 new  》 XML Configuration File 》 Spring Config -->
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <!-- 启动级别：1 -->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!-- / 匹配所有请求（不包含.jsp） -->
    <!-- /* 匹配所有请求（包含.jsp） -->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

4，springmvc-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 添加处理映射器 -->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
    <!-- 添加处理器适配器 -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />
    <!-- 添加视图解析器，还有：Tyhmeleaf/beetl/Freemarker -->
    <!-- DispatcherServlet 提供的 ModelAndView -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!-- 后缀 -->
        <property name="suffix" value=".jsp" />
    </bean>

    <!-- 因为上面的映射器是 BeanNameUrlHandlerMapping -->
    <!-- 所以id的值相当于 mapping，访问：localhost:8080/hello -->
    <!-- 否则不写id访问不了 -->
    <bean id="/hello" class="com.gt.controller.HelloController" />
</beans>
```

5，com.gt.controller.HelloController.java

```java
public class HelloController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse res) throws Exception {
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg", "haha");
        mv.setViewName("hello"); // /WEB-INF/jsp/hello.jsp
        return mv;
    }
}
```

6，WEB-INF/jsp/hello.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${msg}
</body>
</html>
```

7，添加 tomcat 启动运行

8，如果报 404错

```java
File -> Project Structure... -> Artifacts -> Output root -> 
发现 WEB-INF 下没有 lib 目录 -> 右键 WEB-INF 添加 lib 目录 -> 选中 lib 点 + -> Library Files ->
选中所有 jar 包后点 OK -> 重启运行
```

9，http://localhost:8080/hello  画面显示：haha

# 4. 注解版 SpringMVC

1，web.xml 和上面一样

2，main/resources/springmvc-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 自动扫描包，让指定包下的注解生效，由IOC容器统一管理 -->
    <context:component-scan base-package="com.gt.controller" />
    <!-- 让 SpringMVC 不处理静态资源：.css .js .html -->
    <mvc:default-servlet-handler />
    <!-- 支持注解驱动 -->
    <mvc:annotation-driven />

    <!-- 添加视图解析器，还有：Tyhmeleaf/beetl/Freemarker -->
    <!-- DispatcherServlet 提供的 ModelAndView -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!-- 后缀 -->
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

3，com.gt.controller.HomeController.java

```java
@Controller
@RequestMapping("home")
public class HomeController {
    @GetMapping("")
    public String index(Model model) {
        model.addAttribute("msg", "haha");
        return "home";  // 会被视图解析器处理
    }
}
```

4，WEB-INF/jsp/home.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head><title>Title</title></head>
<body>${msg}</body>
</html>
```

5，启动运行 http://localhost:8080/home 显示结果：haha



https://www.bilibili.com/video/BV1aE41167Tu?p=8



# 5. 过滤器

和 servlet 里的过滤器一样，不多说。

```xml
<!-- springMVC 内置的一个解决乱码的过滤器 -->
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encoding</filter-name>
    <!-- 一定要 /* -->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

# 6. 拦截器

## 6.1 与过滤器的区别

在于拦截器是 AOP 思想的具体应用

**必须实现 HandlerInterceptor 接口**

- 过滤器
  - servlet 规范中的一部分，任何 java web 工程都可使用
  - 配置 url-pattern 为 /* 后，对所有要访问的资源进行拦截
- 拦截器
  - springMVC 框架自己的，只有使用了 springMVC 框架的工程才能使用
  - 只拦截控制器方法，==静态资源不拦截==

```java
// 用 HandlerInterceptorAdapter 也行，继承自 HandlerInterceptor
public class InterceptorDemo implements HandlerInterceptor {
    // return true：放行，执行下一个拦截器
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("处理前");
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("处理后");
    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("完成");
    }
}
```

```xml
<!-- 在 xxx-servlet.xml 里 -->
<!-- 配置拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 拦截所有请求 -->
        <mvc:mapping path="/a/**"/>
        <bean class="com.gt.interceptor.InterceptorDemo"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

```java
// 注解配置
@Configuration
public class TjjWebMvcConfigurer implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new InterceptorDemo())
        .addPathPatterns("/**") //配置要拦截的路径
        .excludePathPatterns("/login"); //排除要拦截的路径
    }
}
```



https://www.bilibili.com/video/BV1aE41167Tu?p=27