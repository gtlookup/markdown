```javascript
// 前台请求
let o = {
    i: 123,
    s: 'abcd',
    iAry: [1, 2, 3],
    sAry: ['a', 'b', 'c'],
    o: { a: 'aa', b: 'bb', c: 'cc'},
    oAry: [{ a: 'aa', b: 'bb', c: 'cc'}, { a: 'aa1', b: 'bb1', c: 'cc1'}]
};
$('#map').val(JSON.stringify(o));
$('#fmNext').submit();
```

```java
// 后台接收
// 取json对象
JSONObject json = JSON.parseObject(map);
// 取整数
Integer i = json.getInteger("i");
// 取字符
String s = json.getString("s");
// 取int数组
List<Integer> iList = json.getJSONArray("iAry").toJavaList(Integer.class);
// 取String数组
List<String> sList = json.getJSONArray("sAry").toJavaList(String.class);
// 取map
Map<String, Object> o = json.getObject("o", Map.class);
// 取List<map>
List<Map<String, Object>> oList = JSON.parseObject(str, List.class);
List<Map<String, Object>> oList = (List<Map<String, Object>>)
        json.parse(json.get("oAry").toString());
// 取List<ClassXXX>
List<ClassXXX> list = (List<ClassXXX>) JSON.parseObject(ds, List.class)
        .stream()
        .map(x -> JSON.parseObject(JSON.toJSONString(x), ClassXXX.class))
        .collect(Collectors.toList());
```

# JSON.toJSONString

```java
// 序列化某些字段（id 和 name）
JSON.toJSONString(list, new SimplePropertyPreFilter(user.class, new String[] {"id", "name"}));
// 排除 age 字段
// addFilter().addIncludes() 实现和上面相同功能
JSON.toJSONString(list, new PropertyPreFilters().addFilter().addExcludes(new String[] {"age"}))
```

