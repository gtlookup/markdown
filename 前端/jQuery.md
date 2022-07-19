```javascript
// 判断是否有事件，只绑完只会执行一次，就不用判断有没有了
$('#btn').one('click', fun);
// 监听 class 是 layui-table-edit 的 focus 事件
$(document).on('focus', '.layui-table-edit', function() {$(this).prop('maxLength', 9)});
```

