# 概述

取代 Flaash 做动画广告、游戏等。canvas 是一个轻量级的画布，可使用js编程，不需额外插件，性能也好，在手机端也很流畅

```react
// 只有宽、高两个属性。如果低版本浏览器不支持，则显示标签中间的内容
// 注意：不要用css设置宽高，否则画布会失真
<canvas id="cv" width="100" height="100">当前浏览器不支持 canvas</canvas>

let cv = document.getElementById('cv');
let ctx = cv.getContext('2d');    // 获取2d上下文
let ctx = cv.getContext('webgl'); // 获取3d上下文。注意：只能获取一种上下文，要么2d，要么3d
ctx.fillStyle = 'red';            // 要先设置颜色、再画矩形
ctx.fillRect(100,100,200,50);     // 画矩形
```

## 1.1 像素化

一旦绘制图形成功，canvas就像素化了，再无能力修改画布上的内容。

## 1.2 动画思想

如果想让其动起来，必须清屏后重绘

```js
let n = 0;
ctx.fillStyle = 'red';
setInterval(() => {
    ctx.clearRect(n - 1, 100, 50, 50); // 清屏区域
    n++;
    ctx.fillRect(n,100,50,50);  // 重绘
}, 10);
```

# 绘制功能

## 1. 矩形

```js
ctx.fillStyle = 'xx'     // 设置填充色
ctx.fillRect(x,y,w,h)    // 绘制矩形并填充
ctx.strokeStyle = 'xx'   // 设置边框色
ctx.strokeRect(x,y,w,h)  // 绘制矩形不填充
```

## 2. 路径

为了设置一个不规则的多边形状态。

```js
ctx.beginPath();         // 创建一个路径
ctx.moveTo(100, 100);    // 移动绘制点（比如画笔的落比点，即从哪个点开始画）
ctx.lineTo(200,200);     // 画线
ctx.lineTo(120, 230);
ctx.lineTo(150,280);
ctx.closePath();          // 封闭路径
ctx.strokeStyle = 'red';  // 路径（线）颜色
ctx.stroke();             // 开始绘制
ctx.fillStyle = 'yellow'; // 填充色
ctx.fill();               // 填充
```

## 3. 圆弧

```js
// 画了一个笑脸
ctx.beginPath();
// x, y, 半径, 起始点(x轴开始计算)，Math.PI半圆（*2整圆）, true逆时针false顺时针
ctx.arc(75, 75, 50, 0, Math.PI * 2, true); // 绘制
ctx.moveTo(110, 75);
ctx.arc(75, 75, 35, 0, Math.PI, false);   // 口(顺时针)
ctx.moveTo(65, 65);
ctx.arc(60, 65, 5, 0, Math.PI * 2, true);  // 左眼
ctx.moveTo(95, 65);
ctx.arc(90, 65, 5, 0, Math.PI * 2, true);  // 右眼
ctx.stroke();
```

## 4. 图片

```js
// 绘制图像
//    image：new Image 对象
//	  sx,sy,sw,sh：起始点、宽高。切图片的位置
//    dx,dy,dw,dh：起始点、宽高。切出来图片后，在画布上显示的位置
ctx.drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight);
```

```js
// 例一：
var img = new Image();
img.src = 'res/rust_abstract.png';
img.onload = () => ctx.drawImage(img, 0,0); // 必须放在 onload 里
```

## 5. 状态

```js
ctx.save();    // 保存ctx的状态(如颜色等)，相当于压栈
ctx.restore(); // 恢复上一次save的状态，相当于出栈
```



# API

```js
ctx.globalAlpha = 0.2;             // 设置透明度(0到1)
ctx.lineWidth = 10;                // 设置线宽度
ctx.lineCap =                      // 线端点的样子。butt，round 和 square。默认是 butt
ctx.lineJoin =                     // 两条线连接时的样子。round, bevel 和 miter。默认是 miter。
ctx.measureText("foo haha").width  // 测文本宽度
```



# 收藏

```bash
https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API # 中文API
```



https://www.bilibili.com/video/BV1KV411q7yw?from=search&seid=12528327578012764780