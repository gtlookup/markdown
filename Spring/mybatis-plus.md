# 概念

租户：表示一个db的用户租用了db的部分资源。

# 填坑

## @SqlParser

- filter
  - true: 表示==过滤SQL解析==，即不会进入ISqlParser解析链
  - false：会进解析链并

```java
public interface CommonMapper {
    // 拷贝表结构
    @SqlParser(filter=true) // 过滤sql解析，这个注解将来会被删，新的@InterceptorIgnore
    @Update("CREATE TABLE IF NOT EXISTS ${newTb} LIKE ${oldTb}")
    void copyTableSchema(String newTb, String oldTb);
}
```

## 日期型字段总有默认值

```java
// 一般 insert 会插入空，但一 update 新就有奇怪的日期值
// 加上 updateStrategy = FieldStrategy.IGNORED
@TableField(value = "begin", updateStrategy = FieldStrategy.IGNORED)
private Date begin;
```



# 注解sql文

## 1. 插入

```java
@SqlParser(filter = true)
// 直接写 ob 里的属性
@Insert("insert into ${tableName} (template_id,ym,value1) value(#{id},#{ym},#{col1})")
void insTargetData(TargetTemplate ob);
```

## 2. 查询

```java
// 几点坑：
// 1. 想要用 mybatis 的标签就要加 <script></script>
// 2. 最好用 {} 括起来sql文，这样就不用注意前后空格了。{"select", "*", "from ..."}
// 3. 判断参数不等于空格为："<if test=\"createTo!=null and createTo!=''\">"
// 4. < 小于号要用 "&lt;"，否则项目都无法启动
@Select({"<script>",
         "SELECT",
         "T1.id",
         "FROM",
         "target_import T1",
         "WHERE",
         "1 = 1",
         "<if test=\"createTo!=null and createTo!=''\">",
         "and T1.create_date &lt; #{createTo}",
         "</if>",
         "</script>"})
List<TargetImport> search(ImportSearchVO vo);
```



# mysql 5 / 8

- 驱动不同，8 需要配时区
- mysql5 驱动：com.mysql.jdbc.driver
- mysql8 驱动：com.mysql.cj.jdbc.driver。兼容 mysql 5



# 让 mybatis plus 转起来

```java
class Tb01 extends Model<Tb01> // 1
interface Tb01Mapper extends BaseMapper<Tb01> // 2
@MapperScan("com.gt.mapper") //3
```



# service / impl

```java
// service
public interface xxxService extends IService<model类> // 如上面 Tb01
// impl
public class xxxServiceImpl extends ServiceImpl<Mapper接口, model类> implements xxxService
```



# 控制台输出sql

sql 默认是不可见的，开发阶段一定要看，上线后给关掉。

```yaml
# application.yml
mybatis-plus: # 换成 mybatis: 纯mybatis
  configuration:
  	# 控制台输出
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```



# 主键生成策略

## 雪花算法

- **当主键是 Long 型且不是自增，insert 时主键为null，就会触发雪花算法**

- snowflake是twitter开源的分布式id生成算法，结果是 long 型 id

- http://github.com/twitter/snowflake

> 核心思想：前41bit为毫秒数，10bit为随机id（5个数据中心5个机器id），12bit毫秒内的流水号（意味着每个结点在每毫秒可以生产4096个id），最后还有一个符号位，永远是0。

```java
public enum IdType {
    /**
     * 数据库ID自增
     */
    AUTO(0),
    /**
     * 默认 - 该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT)
     */
    NONE(1),
    /**
     * 用户输入ID
     * <p>该类型可以通过自己注册自动填充插件进行填充</p>
     */
    INPUT(2),

    /* 以下3种类型、只有当插入对象ID 为空，才自动填充。 */
    /**
     * 分配ID (主键类型为number或string）
     *
     * @since 3.3.0
     */
    ASSIGN_ID(3),
    /**
     * 分配UUID (主键类型为 string)
     */
    ASSIGN_UUID(4)
```

```java
public class Test extends Model<Test> {
    // 当主键是 Long 时，并且 insert 时是 null
    // 就会以雪花算法生成主键
    @TableId(value = "id", type = IdType.ASSIGN_ID)
    private Long id; // 不能是 long，否则不给值就是0，就不能产生雪花算法
}

// 没给设id 因此就是 null（long不给是0，Long不给是null）
new Test().setName("gt2").insert();
// id: 1263492092945047553
```



# 自动填充

## 1. 给 model 属性加注解

```java
// FieldFill.INSERT：insert 时填充
@TableField(value = "create_date", fill = FieldFill.INSERT)
private Date createDate;
// FieldFill.INSERT_UPDATE：update 时填充
@TableField(value = "update_date", fill = FieldFill.INSERT_UPDATE)
private Date updateDate;
```

## 2. 实现 handler

```java
@Component // 丢到 bean 容器里
public class AutoFillHandler implements MetaObjectHandler {
    // 插入策略
    @Override
    public void insertFill(MetaObject metaObject) {
        setFieldValByName("createDate", new Date(), metaObject);
        setFieldValByName("updateDate", new Date(), metaObject);
    }
	// 更新策略
    @Override
    public void updateFill(MetaObject metaObject) {
        setFieldValByName("updateDate", new Date(), metaObject);
    }
}
```

## 3. 调用 insert 或 update

- 乐观锁：采用版本控制方式，数据提交前判断版本，OK 了才做处理。
- 悲观锁：锁定当前事务，操作完成后才允许其它操作。

### 乐观锁插件

```java
// 1. 添加 @Version
@Version
@TableField("version")
private Integer version;
// 2. 添加插件
@MapperScan("com.gt.mapper")
@Configuration
public class OtherConfig {
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
}
// 3. 正常测试
new Test().selectById(0).setName("gt1").updateById();
// 4. 异常测试
Test t1 = new Test().selectById(0).setName("gt1");
System.out.println(new Test().selectById(0).setName("gt2").updateById()); // true
System.out.println(t1.updateById()); // false 但抛异常
```



# 逻辑删除

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted # 全局逻辑删除字段 3.3.0 开始支持
      logic-delete-value: 1       # 逻辑已删除值
      logic-not-delete-value: 0   # 逻辑未删除值
```

```java
mapper.deleteById(0); // 结果 deleted 变成 1
```



# 查询

```java
@Resource
private TestMapper mapper;
// 查 0 1 2 三条记录
mapper.selectBatchIds(Arrays.asList(0,1,2));
// 根据 map 查询
mapper.selectByMap(map);
```

## 分页查询

```java
// 1. 配置插件（旧版）
@Bean
// 该插件过时了，替换插件为 PaginationInnerInterceptor
public PaginationInterceptor paginationInterceptor() {
    return new PaginationInterceptor();
}

@Bean // 新版
public MybatisPlusInterceptor paginationInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
    return interceptor;
}
```

### 1. 单表分布

```java
// 2. 查询：每页10行 第1页
mapper.selectPage(new Page<>(1, 10), new LambdaQueryWrapper<>());
```

### 2. 多表联合分页

```java
// 坑点：
// 当 search 方法只传进来vo一个参数时：<if test="id!=null and id!=''">，不能写 vo.id，只能写 id
// 当 search 方法传进来多个参数时：<if test="vo.id!=null and vo.id!=''">，只能写 vo.id
@Select({"<script>",
         "SELECT * FROM target_import",
         "WHERE 1 = 1",
         "<if test=\"vo.id!=null and vo.id!=''\">",
         "and T1.template_id = #{vo.id}",
         "</if>",
         "</script>"})
Page<TargetImport> search(Page page, ImportSearchVO vo);
```

```java
public Page<TargetImport> list(ImportSearchVO vo) {
    return importMapper.search(new Page<>(vo.getOffset(), vo.getLimit()), vo);
}
```

```javascript
// 返回数据结构：
{rows: [{...},{...}], total: n};
```

# 性能分析插件

一个性能分析拦截器，用于输出每条 sql 语句及其执行时间==（3.2以上版本已经移除）==



# 代码生成器

**导入依赖**

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.3.1.tmp</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.19</version>
</dependency>
<dependency>
    <groupId>com.ibeetl</groupId>
    <artifactId>beetl</artifactId>
    <version>3.1.6.RELEASE</version>
</dependency>
```

```java
public class GeneratorCode {
    public static void main(String[] args) {
        // 构建代码生成器
        AutoGenerator auto = new AutoGenerator();

        // 配置策略
        //   1. 全局配置
        GlobalConfig cf = new GlobalConfig();
        //      生成到哪
        cf.setOutputDir(System.getProperty("user.dir") + "/src");
        //      生成作者（即生成的代码注释中的作者）
        cf.setAuthor("gt");
        //      生成后是否打开目录
        cf.setOpen(false);
        //      是否覆盖已有文件
        cf.setFileOverride(false);
        //      去 Service 的 I 前缀
        cf.setServiceName("%sService");
        //      全局唯一Id
        cf.setIdType(IdType.ASSIGN_ID);

        auto.setGlobalConfig(cf);

        //   2. 设置数据源
        DataSourceConfig cnn = new DataSourceConfig();
        cnn.setUrl("jdbc:mysql://localhost:3306/gt?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=UTC");
        cnn.setDriverName("com.mysql.cj.jdbc.Driver");
        cnn.setUsername("root");
        cnn.setPassword("123456");
        cnn.setDbType(DbType.MYSQL);
        auto.setDataSource(cnn);

        //   3. 包的配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName("generator");
        pc.setParent("com.gt");
        pc.setEntity("entity");
        pc.setMapper("mapper");
        pc.setService("service");
        pc.setController("controller");
        auto.setPackageInfo(pc);

        //   4. 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setInclude("person"); // 最常改的地方 映射的表名
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true); // 自动生成 lombok
        strategy.setRestControllerStyle(true);
        auto.setStrategy(strategy);
        // 必须要指定一个前端模板
        auto.setTemplateEngine(new BeetlTemplateEngine());
        // 执行
        auto.execute();
    }
}
```

# 动态表名

```java
@Component
public class MySqlTableNameHandler implements ITableNameHandler {
    @Override
    public String dynamicTableName(MetaObject ob, String sql, String tb) {
        // 只当 insert 时才换表名
        if (sql.toLowerCase().startsWith("insert")) {
            return '新表名';
        }
        // 原始表名
        return tb;
    }
 
    @Override
    public String process(MetaObject ob, String sql, String tb) {
        String dynamic = dynamicTableName(ob, sql, tb);
        if (null != dynamic && !dynamic.equalsIgnoreCase(tb)) {
            sql = sql.replaceAll("INTO " + tb, "INTO " + dynamic);
        }
        return sql;
    }
}
```

# 查询想要/不想要的字段

```java
List<String> unField = Arrays.asList("ID", "NAME", "DATE");
List<SysModule> list = baseMapper
     .selectList(new LambdaQueryWrapper<SysModule>()
     // == -1 不包含，> -1 包含
     .select(SysModule.class, x -> unField.indexOf(x.getColumn()) == -1));
```

# 不重启换DB

```java
@Bean
public DriverManagerDataSource dataSource() {
	String drv = "com.microsoft.sqlserver.jdbc.SQLServerDriver";
    // 默认的数据源比较高级 里面有连接沲 
    // 当第一次被访问便会被锁住 一旦尝试改变就会报错
    // 而DriverManagerDataSource是最基本的数据源 
    // 不存在在连接沲 可以随时改变
    DriverManagerDataSource hsrc = new DriverManagerDataSource();
    hsrc.setDriverClassName(drv);
   	hsrc.setUrl(url);
   	hsrc.setUsername(userName);
   	hsrc.setPassword(pwd);
   	return hsrc;
}
```

```java
@Autowired
private DriverManagerDataSource src;
//修改完target/classes/application_prod.yml后访问下refresh
@GetMapping("refresh")
public String refresh() throws FileNotFoundException {
    // target/classes/application_prod.yml
    String path = this.getClass().getClassLoader().getResource("").getPath() + "/application-prod.yml";
    FileInputStream sm = new FileInputStream(new File(path));
    Yaml yml = new Yaml();
    LinkedHashMap<String, Object> p = (LinkedHashMap) ((LinkedHashMap) yml.loadAs(sm, Properties.class).get("spring")).get("datasource");

   	src.setUrl(p.get("url").toString());
   	src.setUsername(p.get("username").toString());
   	src.setPassword(p.get("password").toString());
   	return "redirect:/";
}
```

# ==多数据源==

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.2.0</version>
</dependency>
```

这是baomidou生态中另一个产品，其它的还有kisso（单点登陆）lock4j（分布式锁）jobs（分布式调度平台）

```yaml
spring:
  # 项目即有的数据库链接  下
  # 在二次开发的时候，需要用到多数据源，并且原项目中用的非mybatisplus（如jpa）
  # 此时添加多数据源时不能影响到原项目
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.2.210:3306/tjj?serverTimezone=Asia/Shanghai
    username: root
    password: root
  # ----------------  上
    dynamic: # baomidou-dynamic 自带的
      datasource:
        # master 代表默认的主连接
        primary: zsk # 如果需要让zsk成为主连接则加上primary
        master: # 上面即有项目的连接原封不动拷过来
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://192.168.2.210:3306/tjj?serverTimezone=Asia/Shanghai
          username: root
          password: root
        zsk: # 二次开发新加的连接
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://localhost:3306/tjj?serverTimezone=Asia/Shanghai
          username: root
          password: 123
```

```java
// 二次开发嘛，当然是两套级别想同的目录
// io.jee.tjj是原项目的代码
// kw.le.lib是二次开发的项目代码
@ComponentScan({"io.jee.tjj", "kw.le.lib"})
// 同样mybatisplus也要指定好
@MapperScan({"io.jee.tjj.mybatis.mapper", "kw.le.lib.mybatis.mapper"})
```

```java
@DS("zsk") // 可以在mapper上添加来指定数据源，但官方不推荐。官方推荐放到serviceImpl上
public interface PersonMapper extends BaseMapper<Person> {}
```

# 前 n 条数据

```java
// last：在sql文最后加上一段sql文
new LambdaQueryWrapper<Cls>().orderByDesc(Cls::getCreateDate).last("limit 10");
```

# mybatis 查询注意项

> 发现查询出来的结果集中，某些列值为null，总结一下

```java
// 实体类
@Data
public class SysUser {
    private String userId;
    private String userName;
    private String nickName;
    /* 表外字段 */
    // 当需要在实体类里另外加一个属性时，就只能用selectList2这种写法了
    // 因为companyName不属于sys_user表，所以不能加到<resultMap>里
    private String companyName;
}
```

```xml
<!-- mapper.xml -->
<resultMap type="SysUser" id="SysUserResult">
    <id     property="userId"    column="user_id"    />
    <result property="userName"  column="user_name"  />
    <result property="nickName"  column="nick_name"  />
</resultMap>

<!-- 因为映射了上面的resultMap，所以在 select 时不用 as 别名 -->
<select id="selectList1" parameterType="SysUser" resultMap="SysUserResult">
    select user_id,user_name,nick_name from sys_user
</select>
<!-- 因为resultType指定的返回类型SysUser，所以在 select 时要 as 别名和实体类的属性保持一致 -->
<select id="selectList2" parameterType="SysUser" resultType="SysUser">
    select user_id userId,user_name userName,nick_name nickName from sys_user
</select>
```

# 流式查询

https://segmentfault.com/a/1190000022478915

指查询成功后不是返回一个集合而是返回一个迭代器，应用每次从迭代器取一条查询结果。流式查询的好处是能够降低内存使用。

```java
@Select("select * from user_0 limit #{limit},10")
Cursor<User> selectPageList(@Param("limit") int limit); // Cursor
```

```java
@Resource
private UserMapper mapper;

@GetMapping("{limit}")
@Transactional // 必须加，防止db连接自动关闭
public Object index(@PathVariable int limit) throws IOException {
    List<User> list = new ArrayList<>();

    // try () {} 这种语法参照：java -> 基础&技巧.md -> 8，try-with-resources
    try (Cursor<User> cursor1 = mapper.selectPageList(limit);
         Cursor<User> cursor2 = mapper.selectPageList(90)) {
 
        cursor1.forEach(x -> list.add(x)); // 假如limit=10，则取10~19的数据
        cursor2.forEach(x -> list.add(x)); // 90~99的数据
    }

    return list;
}
```

