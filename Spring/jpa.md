# 写sql 文

```java
// 传参方式1：?1/2/3.../n
@Query("select o from Employee o where o.name=?1 and o.age=?2")
public List<Employee> queryParams1(String name,Integer age);
// 传参方式2：@Param("name") String name => name:name
@Query("select o from Employee o where o.name=:name and o.age=:age")
public List<Employee> queryParams2(@Param("name") String name, @Param("age") Integer age);
```

# @Modifying

update / delete 时必须加

```java
@Modifying
@Query("update Employee o set o.age=:age where o.id=:id")
public void update(@Param("id") Integer id,@Param("age") Integer age);
```

# @Query 自定义vo查询

```java
// vo 必须要是 interface
public interface UserVO {
    // 字段：uid
    public Integer getUid();
    // 字段：name
    public String getName();
}
```

```java
public interface UserRepository extends JpaRepository<User, Integer> {
    // 1. 列要as
    // 2. table名：#{#entityName}，代表 User 的 @Table(name = "system_user")
    // 3. nativeQuery=true 就必须直接写表名了，#{#entityName}会报错
	@Query(value="SELECT uid as uid, name as name FROM #{#entityName}", nativeQuery=true)
	List<UserVO> uidAndNameList();
}
```

```java
// Map<String, Object> 更好，避免转成 json 串后多了好多信息
List<Map<String, Object>> uidAndNameList();
```

# JpaSpecificationExecutor

https://blog.csdn.net/liangweihua123/article/details/82147109

# @DynamicInsert / Update

默认为 false，若为 true ，则字段为 null 时不会加到 sql 文中

# 联合主键

```java
// 1. 添加联合主键类，并且实现 Serializable
//    成员就是联合主键列
// 2. 添加 @IdClass

@Entity
@Table(name = "target_label")
@IdClass(TargetLabelPK.class)
@Data
public class TargetLabel {
    @Id
    @Column(name="template_id")
    private Integer templateId;
    @Id
    @Column(name="item_id")
    private Integer itemId;
}
@Data
class TargetLabelPK implements Serializable {
    private Integer templateId;
    private Integer itemId;
}
```

# repository.save

```java
// 当方法上加了事务 @Transactional
repository.save(ob);
// 则 save 下面再对 ob 进行赋值仍会反映到数据库里，并且日期类型的数据也不正确（当前日期2020年，db里则是2009年）
ob.setName("aaa");
// 建议去年事务，调再次 save
```

