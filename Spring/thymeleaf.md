# 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

# 配置

##  ThymeleafProperties.class

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    // html 放在 templates 下
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    // 必须是 .html 文件
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
```

## application.yml

```yaml
spring:
  thymeleaf:
    cache: false # 关闭缓存，默认开启的。不关闭的话改页面浏览器刷不出来
```



# 手册

https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.pdf

https://www.jianshu.com/p/4453368eb716



# 表达式

## ${}

- 用来取变量

```html
<!-- 在元素里要 th:xxx="${...}" 如： -->
<div th:text="${ob.id}"></div>
<!-- 不在元素里就要 [[${...}]] 如： -->
<div>[[${ob.id}]]</div>
```

## *{}

- 用来取对象的属性

```java
@Data
@Accessors(chain = true)
class A {
    private String id;
    private String name;
}

model.addAttribute("ob", new A().setId("id-a").setName("name-a"));
```

```html
<!-- th:object -->
<div th:object="${ob}">
    <p>id：<b th:text="*{id}"></b></p>
    <p>name：<b th:text="*{name}"></b></p>
</div>
```

## #{}

- 与 th:text 一起用，用于国际化

## @{}

- 用于网址或链接

# 应用

## .html 开启 thymeleaf

```html
<html xmlns:th="http://www.thymeleaf.org">
```

## 后台=>画面

```java
@GetMapping("")
public String index(Model model) {
    model.addAttribute("k1", "<h1>haha</h1>");
    // resources/templates/home/index.html
    return "home/index";
}
```

## 转意/不转意

```java
model.addAttribute("k1", "<h1>haha</h1>");
```

```html
<div th:text="${k1}"></div>   <!-- <h1>haha</h1> -->
<div th:utext="${k1}"></div>  <!-- 加粗的haha -->
```

## each 遍历

```java
model.addAttribute("ary", Arrays.asList("a", "b", "c"));
```

```html
<h3 th:each="v:${ary}" th:text="${v}"></h3>
<!-- 或者是一样的 -->
<h3 th:each="v,ct:${ary}">[[${v}]],[[${ct}]]</h3>
<!-- v,ct:豆号左侧是数据元素，右侧则是一个属性对象。结果为： -->
<h3>a,{index = 0, count = 1, size = 3, current = a}</h3>
<h3>b,{index = 1, count = 2, size = 3, current = b}</h3>
<h3>c,{index = 2, count = 3, size = 3, current = c}</h3>
```

## 日期/格式化

```html
<p>[[${#dates.createNow()}]]</p>
<!-- Tue May 19 22:56:42 CST 2020 -->
<p>[[${#dates.createToday()}]]</p>
<!-- Tue May 19 00:00:00 CST 2020 -->
<p>[[${#dates.format(#dates.createNow(), 'yyyy/MM/dd HH:mm:ss')}]]</p>
<!-- 2020/05/19 22:56:42 -->
```

## 字符串

```html
<p>[[${#strings.length('haha')}]]</p>   <!-- 4 -->
<p>[[${#strings.isEmpty('')}]]</p>      <!-- true -->
```

## 数字

```html
<!-- 千分位 -->
<!-- COMMA：, -->
<!-- POINT：. -->
[[${#numbers.formatDecimal(123456789,1,'COMMA',0,'POINT')}]]
<!-- 123,456,789 -->

<!-- count/100结果为整数 -->
<!-- count/100.0结果为4位小数 -->
<!-- formatDecimal(num,2,1)：整数位2位（不足前面补0），小数位1位 -->
[[${#numbers.formatDecimal(count/100.0,1,1)}]]
```

## 比较运算

- \>, <, >=, <= (gt, lt, ge, le)
- == , != ( eq , ne )

```html
<!-- s=abcdefg -->
[[${#strings.length(s) le 5 ? s : #strings.substring(s, 0, 5) + '...'}]]
<!-- abcde... -->
```

## th:block

- 用于执行表达式，并且不会在页面上留下多余标签

## fragment

- 将多个地方出现的元素块用fragment包起来使用

## 局部变量

```html
<!-- th:with -->
<th:block th:with="a=1,b=2,c=3">
    <input type="text" th:value="${a}" />
    <input type="text" th:value="${b}" />
    <input type="text" th:value="${c}" />
</th:block>
```

## if / else

```html
<select class="custom-select" name="parentCode">
    <option value="" selected>---请选择所属菜单---</option>
    <option th:if="${ob.cd}==${rlt.cd}" selected th:each="ob:${list}" th:value="${ob.cd}">[[${ob.nm}]]</option>
    <option th:else th:each="ob:${list}" th:value="${ob.cd}">[[${ob.nm}]]</option>
</select>
```

## if / elseif

```html
<!-- if elseif -->
<div th:if="${o.count>10000}">[[${count/10000}]]万次</div>
<!-- th:else 后接 th:if="..." -->
<div th:else th:if="${o.count<10000 && o.count>1000}">[[${#numbers.formatDecimal(count/10000.0,1,1)}]]万次</div>
<div th:else th:if="${o.count<1000}">[[${count}]]次</div>
```

## switch / case

有时候 th:if / th:else 两个标签的内容都会显示，此时只能用 switch / case

```html
<div class="col-sm-10" th:switch="${#strings.isEmpty(ob.menuCode)}">
    <input th:case="true" type="text" />
    <input th:case="false" type="text" th:value="${ob.menuCode}" />
</div>
```

## th:href

```html
<!-- 首尾拿 | 括起来 -->
<a th:each="v:${list}" th:href="|javascript:OB.setIcon('${v}')|" th:class="${v}">&nbsp;&nbsp;</a>
```

# 整合spring security

```java
// org.springframework.security.core.userdetails.User
user.getAuthorities().add(new SimpleGrantedAuthority("role1"));
```

```html
<!-- 导入 xmlns:sec="http://www.thymeleaf.org/extras/spring-security" -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" xmlns:sec="http://www.thymeleaf.org/extras/spring-security">

    <!-- html元素里用 -->
    <!-- sec:authorize="hasAuthority('role1')" -->
    <button sec:authorize="hasAuthority('role1')" class="btn btn-primary btn-sm">上传表格模型</button>
```

```javascript
// js 里用权限
let has = [[${#authorization.expression('hasAuthority(''role1'')')}]];
if (has) $('.button').hide();
```



