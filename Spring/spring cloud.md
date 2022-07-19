

# 架构示例

> 搭建一个 eureka + config + zuul + 一个微服务的例子

## 1. 父工程

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.4.RELEASE</version>
</parent>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies> <!-- 这两个每个微服务都会用到 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

## 2. eureka服务

这个服务里包含注册中心和配置中心

```xml
<dependencies>
    <dependency> <!-- eureka服务端依赖 -->
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    <dependency> <!-- config服务端依赖 -->
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>
```

```yaml
# application.yml
spring:
  application:
    name: eureka-server
  cloud:
    config: # 配置中心
      server:
        git:
          uri: http://192.168.1.14:8099/guantong/demo.git # 要读取的git地址，demo为git上的项目名
          username: guantong
          password: ps0918@@
        prefix: /config # 配置服务的后缀名，其它服务访问的话就是 http://xxxxx/config
server:
  port: 10000
eureka: # 注册中心
  instance:
    prefer-ip-address: true # 用ip定义注册中心的地址，而不是用机器名
    instance-id: ${spring.cloud.client.ip-address}:${server.port} # eureka页面上显示的地址信息，一般和上一行连用
  client:
    register-with-eureka: false # 不把自己注册到注册中心
    fetch-registry: false       # 不从注册中心获取注册表信息。这两行意思说该服务是 eureka 服务端
    service-url:
      defaultZone: http://${spring.cloud.client.ip-address}:${server.port}/eureka/ # 注册中心地址，有多个就逗号隔开
```

```java
@SpringBootApplication
@EnableEurekaServer // 开启注册中心服务
@EnableConfigServer // 开启配置中心服务
public class EurekaApp {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApp.class, args);
    }
}
```

## 3. gateway服务

这个服务是网关服务

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
    <dependency> <!-- 这个要加，否则转发时报错 -->
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

```yaml
# bootstrap.yml（不是application.yml）
server:
  port: 10001
spring:
  application:
    name: gateway-server
  cloud:
    config:
      discovery:
        service-id: eureka-server # 注册中心的服务名，即eureka端的application.yml.spring.application.name
      label: master # git上配置文件的分支
      profile: local # git上的配置文件名后缀，与下面的name组合用。如：demo-local.yml
      uri: http://localhost:10000/config # 配置中心服务地址
      name: demo # git上的配置文件名前缀，丐上面的profile组合用。如：demo-local.yml
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port} # eureka页面上显示的地址信息
  client:
    service-url:
      defaultZone: http://localhost:10000/eureka/ # 注册中心地址
zuul: # 网关部分
  routes:
    auth: # 网关下的服务名
      path: /auth/** # 当访问网关（http://localhost:10001/auth）时，转发到 auth-server 服务上
      serviceId: auth-server # 转发到哪个服务上
```

```java
@SpringBootApplication
@EnableZuulProxy         // 开启网关（注意：是EnableZuulProxy，不是EnableZuulService）
//@EnableDiscoveryClient // 这个可以省略
public class GatewayApp {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApp.class, args);
    }
}
```

## 4. auth服务

这个是普通的微服务

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

```yaml
# bootstrap.yml（不是application.yml）
server:
  port: 10002
spring:
  application:
    name: auth-server
  cloud:
    config:
      discovery:
        service-id: eureka-server
      label: master
      profile: local
      uri: http://localhost:10000/config
      name: demo
eureka:
  instance:
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://localhost:10000/eureka/
code: ${work.code} # 获取 git 上 demo-local.yml 里的work.code
```

```java
@SpringBootApplication // @EnableDiscoveryClient 可省略
public class AuthApp {
    public static void main(String[] args) {
        SpringApplication.run(AuthApp.class, args);
    }
}
```

```java
@RestController
public class AuthController {
    @Value("${work.name}") // 获取 git 上 demo-local.yml 里的 work.name
    private String workName;
    @Value("${code}")      // 获取 bootstrap.yml 里的 code
    private String code;

    @GetMapping
    public String index() {
        return code + ", " + workName;
    }
}
```

## 5. git上的配置文件

```yaml
# demo-local.yml
work:
  code: work-code
  name: work-name
```

## 6. 测试

```bash
http://localhost:10001/auth # 通过网关访问auth服务，结果：work-code, work-name
http://localhost:10002/     # 直接访问微服务，结果：work-code, work-name
```

# 收藏

```bash
https://www.cnblogs.com/zhixiang-org-cn/p/9846326.html
https://blog.csdn.net/xingbaozhen1210/article/details/80290588  # 配置文件最详细说明
```

