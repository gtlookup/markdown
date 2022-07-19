muuri：多功能风格布局

```html
<head>
    <script src="muuri/muuri.min.js"></script>
    <style>
        /* 不加宽度是横着排，加个宽度就竖着排了 */
        .grid {background-color: azure; width: 220px;}
        .item {
            position: absolute;
            width: 100px;
            height: 100px;
            line-height: 100px;
            margin: 5px;
            z-index: 1;
        }
        .item-content {background-color: #F00;text-align: center;}
    </style>
</head>
<body>
    <div class="grid">
        <div class="item"><div class="item-content">1</div></div>
        <div class="item"><div class="item-content">2</div></div>
        <div class="item"><div class="item-content">3</div></div>
        <div id="noDrop" class="item"><div class="item-content">位置在最后不可改变</div></div>
    </div>
    <script>
        // 必须有3层: 1.grid 2.item 3.item-content 否则报错
        //    参数：被拖拽元素（._id为唯一标识）
        //         grid._id = 1
        //         item 类推：2 3 4
        let drg = new Muuri('.grid', {
            dragEnabled: true, // 开启拖拽
            dragAxis: 'xy', // 拖拽方向，x:横，y：竖，xy：横竖都能
            items: '.item', // 能被拖拽的项目
            // 让最后一项目不可被拖到其它位置
            dragStartPredicate: function (i, e) {
                if (i._element.id == 'noDrop') return false;
                return true;
        	}
        }).on('dragReleaseStart', function(x) { // 拖拽完成后触发
            x._id // 当前被拖拽的id
			drg._items // 拖拽完成后的 item，按拖拽后的顺序排列
            
            // 让最后一项的位置不可被其它项目取代
            let ar = _.filter(drg.getItems(), function(x) {return x.getElement().id != 'noDrop'}), // 取出可变位置的项目
                o = _.find(drg.getItems(), function(x) {return x.getElement().id == 'noDrop'}); // 取出不可变位置的项目
            ar.push(o); // 将不可变的放在最后
            drg.sort(ar); // 排序
        });
    </script>
</body>
```

