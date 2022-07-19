# 概念

AOP实现框架有SpringAOP还有就是AspectJ(还有另外几种)

**两者区别：**

AspectJ：可以对类的成员变量，方法进行拦截。由于AspectJ是Java语言语法和语义的扩展，所以它提供了自己的一套处理方面的关键字。

SpringAOP：Spring的AOP技术只能是对方法进行拦截。使用J2SE动态代理来代理接口，对象使用CGLIB代理。

SpringAOP和AscpectJ之间的关系：Spring使用了和aspectj一样的注解，并使用Aspectj来做切入点解析和匹配。

​                                                            但springAOP运行时仍旧是纯的springAOP,并不依赖于Aspectj的编译器或者织入器



#  execution表达式

```java
//?前的访问修饰符为可选
execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)
//public *返回 *方法(..)任意参数
execution(public * *(..))
//*返回 包名不含子包 *方法(..)任意参数
execution(* com.imooc.dao.*(..))
//*返回 包名 ..以及子包 *方法名 (..)任意参数
execution(* com.imooc.dao..*(..))
//*返回 包及类 *方法(..)任意参数
execution(* com.imooc.dao.UserService.*(..))
//+以及子类 *方法 (..)任意
execution(* com.imooc.dao.UserService+.*(..))
//*返回 save*开头的方法 (..)任意参数
execution(* save*(..))
//*返回 ..包以及子包 *.*所有类及方法 (..)任意参数
execution(public * com.didispace.web..*.*(..))
```



# 示例

## 导入依赖

spring-Aop够用，AspectJ更全更强大

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <!-- 第一次运行没写版本号 死活儿不能切 也不报错，写过一次import后 再删掉 就好用了 -->
    <version>1.9.4</version>
</dependency>
<!-- 或 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 切面代码

```java
//声明这是一个组件
@Component
//声明这是一个切面Bean
@Aspect
public class ServiceAspect {
    //配置切入点,该方法无方法体,主要为方便同类中其他方法使用此处配置的切入点
    //@Pointcut("@annotation(com.xxx.xxx.xx注解)") 配置注解为切入点
    @Pointcut("execution(* com.gt.controller..*(..))")
    public void aspect() {}
    /*
     * 前置通知,使用在方法aspect()上注册的切入点
     * 同时接受JoinPoint切入点对象,可以没有该参数
     */
    @Before("aspect()")
    public void before(JoinPoint joinPoint) {
        log.info("before " + joinPoint);
    }
    //后置通知,使用在方法aspect()上注册的切入点
    @After("aspect()")
    public void after(JoinPoint joinPoint) {
        log.info("after " + joinPoint);
    }
    //环绕通知,使用在方法aspect()上注册的切入点
    //还可以修改原方法的参数和返回值 阻止原方法执行
    // 也可以把注解作为参数传给around方法：
    //     @Around("@annotation(em)")  -> em 要和around方法里的第二个参数同名
    //     public Object around(ProceedingJoinPoint point, ExceptionMethod em) throws Throwable {
    @Around("aspect()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        //前置操作 ...
        //获取方法参数
        Object[] args = point.getArgs();
        //调用原方法
        Object rlt = point.proceed(args);
        //后置操作 …
        Method method = ((MethodSignature) point.getSignature()).getMethod();
        //获取注解
        ExceptionMethod em = method.getAnnotation(ExceptionMethod.class);
        //当前切入点=> execution(返回类型 包+方法(参数))
        point.getSignature();
        //方法名
        point.getSignature().getName();
        //包+类名
        point.getSignature().getDeclaringTypeName();
        //重要：如果代理的方法有返回值 这里必须返回
        //因为需要显式的调用原方法
        return rlt;
    }
    //后置返回通知,使用在方法aspect()上注册的切入点
    @AfterReturning("aspect()")
    public void afterReturn(JoinPoint joinPoint) {
        log.info("afterReturn " + joinPoint);
    }
    //抛出异常后通知,使用在方法aspect()上注册的切入点
    @AfterThrowing(value = "aspect()", throwing = "ex")
    public void afterThrow(JoinPoint joinPoint, Exception ex) {
        log.info("afterThrow " + joinPoint + "\t" + ex.getMessage());
    }
}
```



## 切面执行顺序

- 正常终了：@Around -> @Before -> @After -> @AfterReturning
- 异常终了：@Around -> @Before -> @After -> @AfterThrowing
- point.proceed未执行：@Around -> @After



# 坑

==private代理不了==

## 解决this调方法代表不了

```java
@Service
public class AServiceImpl implements IAService {
    @Autowired
    private AServiceImpl me;
    @Autowired
    private ApplicationContext ctx;
    
    @Override
    public void test() {
        //由于自己内部调用 非代理调用 无法触发aop
        test1();
        //方法一：因为注入后是个代理实例 所以能够触发aop
        me.test1();
        //方法二：需要在app.java加上
        //       @EnableAspectJAutoProxy(exposeProxy = true)
        ((AServiceImpl)AopContext.currentProxy()).test1();
        //方法三：bean name 一般都是类名首字母小写
        ((AServiceImpl)ctx.getBean("aServiceImpl ")).test1();
    }
    @AopAnno  //标记后被代理
    public void test1() {}
}
```

