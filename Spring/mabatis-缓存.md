# 概念

一般提到mybatis缓存的时候都是==指二级缓存==，因为一级缓存默认是开启着的。

一级缓存存在于sqlSession的生命周期中，而二级缓存存在于sqlSessionFactory（sqlSessionFactory.openSession取得sqlSession）的生命周期中。



# 一级缓存

存放一级缓存的是一个HashMap。查询时先到一级缓存里找，没有再去db里找。==当遇到 insert / update / delete 时，会清空一级缓存==。

## 缓存结构

> HashMap.key = hashCode + sqlid + sql语句
>
> HashMap.value = sql语句取出来的java对象

## 命中缓存条件

- 必须是相同sql和参数
- 必须是同一个mapper（即namespace）
- 必须是同一个mapper方法
- 必须是相同的会话（sqlSession）
- 不能在查询之前执行sqlSession.clearCache()
- 不能在查询之前执行增删改语句

## 示例

```java
// 前提要先开启控制台输出sql
@RunWith(SpringRunner.class)
@SpringBootTest
public class MybatisCacheTest {
    @Autowired
    private SqlSessionFactory sessionFactory;
    @Resource
    private TrSysHsMapper mapper;  // 这种注入进来的mapper是不会命中缓存的
    
    @Test
    public void cacheTest() {
        SqlSession sqlSession = sessionFactory.openSession(true); // 取得sqlSession
        TrSysHsMapper mapper = sqlSession.getMapper(TrSysHsMapper.class); // 取得mapper
        System.out.println(mapper.selectList(null)); // 执行了sql
        System.out.println(mapper.selectList(null)); // 缓存了，没执行sql
        mapper.updateById(new TrSysHs().setHsCode("1").setHsName("a").setHsNameEn("e")); // 增删改会清空缓存
        System.out.println(mapper.selectList(null)); // 执行了sql
        System.out.println(mapper.selectById("3"));  // 执行了sql
    }
}
```

## 实现原理

### SqlSession结构

用户调用一个代理并传给 sqlSession， sqlSession 通过 Executor 到db里拿数据

**serviceImpl  ->  xxxMapper  ->  sqlSession  ->  Executor  ->  db**

其中 DefaultSqlSession 实现了接口 SqlSession；CachingExecutor 实现了接口 Executor。

CachingExecutor判断缓存不存在，就去db里拿。



# 二级缓存

https://www.cnblogs.com/cxuanBlog/p/11333021.html

应用场景：用于缓存改动较少的表，比如master表

多个sqlSession共享缓存：在一级缓存层和db之间再加一个二级缓存层，**查询时先检索二级层再检索一级层，如果两层都没有再查db**。

需要在mapper上加==@CacheNamespace==手动开启二级缓存。

## 命中缓存条件

- 必须设置@CacheNamespace
- 必须要在一个sqlSession结束（close()或commit）前，才存入二级缓存。（这么做的原因是维护事务的**隔离**性）
- 其它跟一级缓存命中条件一样

## 示例

```java
@Resource
private TrSysHsMapper hsMapper;

@Test
public void cacheTest2() {
    SqlSession sqlSession1 = sessionFactory.openSession(true);
    SqlSession sqlSession2 = sessionFactory.openSession(true);
    System.out.println(sqlSession1.getMapper(TrSysHsMapper.class).selectList(null)); // 执行了sql
    // [1].结束（或commit）一个会话才会存入二级缓存，然后下一个都会从缓存中取
    // 也就是执行完一个controller 再执行就会从缓存中取
    // 但当controller里第一次执行并且连续执行多个相同sql时，并不会从缓存中取，原因是上面[1]
    sqlSession1.close();
    System.out.println(sqlSession2.getMapper(TrSysHsMapper.class).selectList(null)); // 命中缓存，未执行sql
    System.out.println(hsMapper.selectList(null)); // 命中缓存，未执行sql
    System.out.println(new TrSysHs().selectAll()); // 命中缓存，未执行sql
}

@GetMapping("a")
public Object a() {
    return hsMapper.selectAll(); // 只要走一次就会缓存
}

@GetMapping("b")
public Object b() {
    return hsMapper.selectAll(); // 只要走一次就会缓存
}
```

## 缓存到redis里

```java
public class MybatisRedisCache implements Cache {
    private static final Logger log = LoggerFactory.getLogger(MybatisRedisCache.class);
    private String id;
    private static RedisTemplate<String, Object> redisTemplate;

    public MybatisRedisCache(String id) {
        if (id == null) {
            throw new IllegalArgumentException("Cache Instance ID Could Not Be Null");
        } else {
            log.info("Create Redis Cache [{}].", id);
            this.id = id;
        }
    }

    @Override
    public String getId() {
        return this.id;
    }

    @Override
    public int getSize() {
        log.debug("Get Cache [{}] Size.", this.id);
        return getTemplate().keys(this.prefixedKey("*")).size();
    }

    @Override
    public void putObject(Object key, Object value) {
        log.debug("Put Object Key [{}], Value [{}].", key, value);
        getTemplate().opsForValue().set(this.prefixedKey(key), value, 1, TimeUnit.DAYS);
    }

    @Override
    public Object getObject(Object key) {
        Object value = getTemplate().opsForValue().get(prefixedKey(key));
        log.debug("Get Object Key [{}], Value [{}].", key, value);
        return value;
    }

    @Override
    public Object removeObject(Object key) {
        log.debug("Remove Object Key [{}].", key);
        getTemplate().delete(prefixedKey(key));
        return 1;
    }

    @Override
    public void clear() {
        log.debug("Clear Cache Key [{}].", this.id);
        getTemplate().delete(getTemplate().keys(prefixedKey("*")));
    }

    RedisTemplate getTemplate() {
        // SpringBeanUtil参照Spring Boot.md的普通类获取bean
        if (redisTemplate == null) redisTemplate = SpringBeanUtil.getBean(RedisTemplate.class);
        return redisTemplate;
    }

    String prefixedKey(Object key) {
        return this.prefix() + key;
    }

    String prefix() {
        return this.id + ":";
    }
}
```

```java
// mapper上开启二级缓存并指定cache类
@CacheNamespace(implementation = MybatisRedisCache.class)
public interface TrSysHsMapper {}
```

### mybatis-config.xml 配置

如果是依赖 mybatis-config.xml 配置的，==那么 @CacheNamespace(implementation = MybatisRedisCache.class) 将不起作用==，需要在mapper.xml里开启

```yaml
mybatis:
    # 搜索指定包别名
    typeAliasesPackage: com.trace.project.**.domain
    # 配置mapper的扫描，找到所有的mapper.xml映射文件
    mapperLocations: classpath*:mybatis/**/*Mapper.xml
    # 加载全局的配置文件
    configLocation: classpath:mybatis/mybatis-config.xml
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<setting name="cacheEnabled"             value="true" />  <!-- 启用二级缓存 -->
		<setting name="useGeneratedKeys"         value="true" />  <!-- 允许 JDBC 支持自动生成主键 -->
		<setting name="defaultExecutorType"      value="REUSE" /> <!-- 配置默认的执行器 -->
		<setting name="logImpl"                  value="SLF4J" /> <!-- 指定 MyBatis 所用日志的具体实现 -->
		<!-- <setting name="mapUnderscoreToCamelCase" value="true"/>  驼峰式命名 -->
	</settings>
</configuration>
```

```xml
<mapper namespace="com.trace.project.system.mapper.SysConfigMapper">
    <cache type="com.trace.framework.redis.MybatisRedisCache" /> <!-- 开启二级缓存 -->
```

