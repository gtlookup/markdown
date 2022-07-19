```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>RELEASE</version>
    <scope>test</scope>
</dependency>
```

**@Before, @BeforeClass, @BeforeEach 和 @BeforeAll之间的不同**

| 特性                                                         | junit4       | junit5      |
| ------------------------------------------------------------ | ------------ | ----------- |
| - 在当前类的所有测试方法之前执行。 <br>- 注解在静态方法上。 <br>- 此方法可以包含一些初始化代码。 | @BeforeClass | @BeforeAll  |
| - 在当前类中的所有测试方法之后执行。<br>- 注解在静态方法上。<br>- 此方法可以包含一些清理代码。 | @AfterClass  | @AfterAll   |
| - 在每个测试方法之前执行。<br>- 注解在非静态方法上。<br>- 可以重新初始化测试方法所需要使用的类的某些属性。 | @Before      | @BeforeEach |
| - 在每个测试方法之后执行。<br>- 注解在非静态方法上。<br>- 可以回滚测试方法引起的数据库修改。 | @After       | @AfterEach  |

