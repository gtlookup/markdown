# IE问题

- **解决不支持 FormData.forEach 问题**

```javascript
// 引入
<script src="https://unpkg.com/formdata-polyfill"></script>

let formData = new FormData($('#form')[0]), ob = {};
formData.forEach(function (v, k) {ob[k] = v});
```

- **解决不支持 Promise** 

```javascript
<script type="text/javascript" src="https://cdn.polyfill.io/v2/polyfill.min.js?features=es6"></script>
```

# call/apply/bind

## 1.call

```javascript
// 第一个参数：this指向如果要传参，后面依次是参数
function fn(x,y) {console.log(this)}
var obj = {name:"zs"}
fn.call(obj,1,2); // 结果：{name:"zs"}
// 让方法的this指向按钮
window.OB = { click: function() {console.log(this.id)} };
OB.click.call($('#btn')[0]);
```

## 2.apply

```javascript
// 与第一种方法不同的是，用数组的形式表示参数
function fn(x,y) {console.log(this)}
var obj = {name:"zs"}
fn.apply(obj,[1,2]); // 结果：{name:"zs"}
```

## 3.bind

```javascript
function fn(x,y) {console.log(this)}
var obj = {name:"zs"}
fn.bind(obj,1,2)(); // ()表示立即调用
```

# script 模板

```html
<script id="element_toolbar" type="text/html">
    <a class="layui-btn layui-btn-danger" lay-event="edit">[btnEdit]</a>
    <a class="layui-btn layui-btn-danger" lay-event="delete">[btnDelete]</a>
</script>
```

```javascript
let html = $('#element_toolbar').html(),
    reg = new RegExp("\\[([^\\[\\]]*?)\\]", 'igm');
$('#element_toolbar').html(html.replace(reg, function (node, key) {
    return {
        btnEdit: elements.ELEMENT_INDEX.btnEdit,     // 编辑
        btnDelete: elements.ELEMENT_INDEX.btnDelete  // 删除
    }[key];
}));

```

# 数字操作

```javascript
/******** 千分位 ************/
let n = 123456789.56789;
n.toLocaleString();   //123,456,789.568
/******** 保留两位小数 ************/
// toLocaleString最多保留3位
parseFloat(n.toFixed(2)).toLocaleString();  //123,456,789.57
```

