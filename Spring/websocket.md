# 简单demo

## 服务端

```xml
<dependency> <!-- 依赖 -->
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

```java
@Bean // 开启websocket
public ServerEndpointExporter serverEndpointExporter() {
    return new ServerEndpointExporter();
}
```

```java
// websocket服务
@ServerEndpoint("/socket/{userId}") // 给前台连接的地址，每个连接都要带个key（就是userId）
@Component
public class WebSocketService {
    private static ConcurrentHashMap<String, Session> map = new ConcurrentHashMap<>(); // 保存客户端的连接
    private String connectionKey; // 当前连接的key

    @OnOpen // 客户端发起连接时触发，即前台new WebSocket后触发
    public void onOpen(Session session, @PathParam("userId") String userId) {
        connectionKey = userId; // 保存当前key

        if (map.containsKey(userId)) map.remove(userId); // 重新连接时先删除原有连接
        map.put(userId, session); // 保存连接
    }
    @OnMessage // 当前连接接收到消息时触发
    public void onMessage(String msg, Session session) {}
    
    @OnClose // 连接关闭时触发
    public void onClose() {
        if (map.containsKey(connectionKey)) map.remove(connectionKey);
    }
    
    @OnError // 发生异常时触发
    public void onError(Session session, Throwable err) {
        err.printStackTrace();
    }

    // 向所有已连接的客户端发送消息
    public void sendAll(String msg) throws IOException {
        for (String s : map.keySet()) {
            map.get(s).getBasicRemote().sendText(msg);
        }
    }
}
```

```java
// 向所有已连接的客户端发送消息
@Autowired
private WebSocketService service;

@GetMapping("send/{msg}")
public void send(@PathVariable String msg) throws IOException {
    service.sendAll("8888:" + msg);
}
```

## 客户端

```html
<button onclick="OB.open()">连接</button>
<div id="dv"></div>
<script>
    let OB = {
        socket: null,
        open: function () {
            // 发起连接
            let sk = new WebSocket("ws://localhost:8888/socket/gt001"); // ws: 开关
            OB.socket = sk;
            sk.onopen = OB.onOpen;
            sk.onclose = OB.onClose;
            sk.onmessage = OB.onMessage;
            sk.onerror = OB.onError;
        },
        onOpen: function () {console.log('打开...')},  // 连接时触发
        onClose: function () {console.log('关闭...')}, // 关闭时触发
        onMessage: function (msg) { // 接收收到消息时触发
            document.getElementById("dv").append(msg.data);
        },
        onError: function () {console.log("Error")} // 异常时触发
    };
</script>
```



# redis发布订阅实现集群websocket

- redis发布订阅的构建参照 redis.ms -> 发布订阅 > 集成springboot

- nginx代理配置参照 nginx.md -> 附录 -> 开启websocket

- websocket服务类

  ```java
  @ServerEndpoint("/socket/{userId}")
  @Component
  public class WebSocketService implements ApplicationContextAware {
      private static ConcurrentHashMap<String, Session> map = new ConcurrentHashMap<>();
      private static RedisTemplate redisTemplate;
      private String connectionKey;
  
      @Value("${server.port}")
      private String port;
  
      @Override
      public void setApplicationContext(ApplicationContext ctx) throws BeansException {
          redisTemplate = ctx.getBean(RedisTemplate.class);
      }
  
      @OnOpen
      public void onOpen(Session session, @PathParam("userId") String userId) {
          connectionKey = userId;
  
          if (redisTemplate.hasKey("key-" + userId)) {
              redisTemplate.delete("key-" + userId);
          }
          redisTemplate.opsForValue().set("key-" + userId, userId);
          if (map.containsKey(userId)) map.remove(userId);
          map.put(userId, session);
      }
      @OnMessage
      public void onMessage(String msg, Session session) { }
      @OnClose
      public void onClose() {
          if (map.containsKey(connectionKey)) map.remove(connectionKey);
          if (redisTemplate.hasKey("key-" + connectionKey)) redisTemplate.delete("key-" + connectionKey);
      }
      @OnError
      public void onError(Session session, Throwable err) {
          err.printStackTrace();
      }
  
      public void sendAll(String msg) throws IOException {
          Iterator itor = redisTemplate.keys("key-*").iterator();
          while (itor.hasNext()) {
              String key = redisTemplate.opsForValue().get(itor.next().toString()).toString();
              if (map.containsKey(key)) {
                  map.get(key).getBasicRemote().sendText(msg + port + "<br>");
              }
          }
      }
  }
  ```

- 监听redis服务

  ```java
  @Component
  public class RedisReceiver implements MessageListener {
      @Autowired
      private WebSocketService webSocketService;
      @Override
      public void onMessage(Message msg, byte[] bytes) {
          try {
              webSocketService.sendAll(new String(msg.getBody()));
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

- 前端连接nginx的代理：ws://192.168.2.36/socket/{userId}

==问题：==连接上一会儿之后就会报 at org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper.fillReadBuffer(NioEndpoint.java:1231)