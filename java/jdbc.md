# 1. 存储过程

## 1.1 返回值

```sql
ALTER proc testproc @i int, @str varchar(255) output as
set nocount on;
begin
    set @str = 'haha'
    return
end
```

```java
Connection cnn = DriverManager.getConnection(url + ";username=" + user + ";password=" + pwd);
CallableStatement pro = cnn.prepareCall("{ call testproc(?, ?) }");
pro.setObject(1, 10);
//重要：第二个参数为返回的(参数下标从1开始)
pro.registerOutParameter(2, java.sql.Types.VARCHAR);
pro.execute();
String s = pro.getString(2);
ResultSet rs=(ResultSet)pro.getObject(n);   //返回一个游标
```

## 1.2 返回多个结果集

```sql
ALTER PROCEDURE [dbo].[testpro] @tp NVARCHAR(100) = null AS
BEGIN
  　SET NOCOUNT ON;
    SELECT top 1 * FROM sys_dict;
    SELECT top 2 * FROM sys_dict;
    SELECT top 3 * FROM sys_dict;
END
```

```java
Connection cnn = DriverManager.getConnection(url + ";username=" + user + ";password=" + pwd);
CallableStatement pro = cnn.prepareCall("{ call testpro(?) }");
pro.setObject(1, "haha");
boolean b = pro.execute();
while (b) {
    ResultSet rs = pro.getResultSet();
    int i = 0;
    while(rs.next()) i++;
    System.out.println(i);
    b = pro.getMoreResults();
}
```

# 2. JdbcTemplate

```java
// 注入
@Autowired
private JdbcTemplate jdbc;
```

## 2.1 queryForList

```java
List<T> queryForList(String sql, Class<T> elementType)
// 乍一看下似乎这样就可行
List<User> list = jdbc.queryForList("select id,name from user", User.class);
// 但是，报错了；这个 T 只支持 Integer 和 String
// 改为：
List<User> list = jdbc.query("select id,name from user", new BeanPropertyRowMapper(LabelVO.class));
// 就 OK 了
// queryForList => query
// User.class => new BeanPropertyRowMapper(LabelVO.class)
```

