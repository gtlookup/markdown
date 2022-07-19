```bash
# 安装
cnpm i -g webpack
cnpm i -g webpack-cli
```

# webpack-dev-server

用它运行项目后，每次修改不用再重新 build 和 run

```bash
npm install webpack-dev-server --save-dev # 安装
# –save-dev：将webpack-dev-server保存配置信息到pacjage.json的devDependencies(开发环境依赖)节点中。
#			 这样做是因为webpack-dev-server仅仅在本地开发时才会用到，在生产环境中并不需要它 。
#			 项目上线的时候，要进行依赖安装，就可以通过npm install–production过滤掉devDependencies中的冗余模块，从而加快安装和发布的速度。
```

## 示例

```bash
# 1. 创建 deom 文件夹，并 cmd 进去
# 2. cnpm init                            初始化项目
# 3. cnpm i webpack --save-dev            本地安装 webpack
# 4. cnpm i webpack-cli --save-dev        本地安装 webpack-cli
# 5. cnpm i webpack-dev-server --save-dev 本地安装 webpack-dev-server
```

```javascript
// 6. 创建并编辑 index.js
document.write('haha hoho')
```

```html
<!-- 7. 创建并编辑 index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="index.js"></script> <!-- 导入第6步的 index.js -->
</head>
<body></body>
</html>
```

```javascript
// 8. 新建并编辑 webpack.config.js
const path = require('path');

module.exports = {
    entry: './index.js',                       // 入口文件
    output: {
        path: path.resolve(__dirname, 'dist'), // 输出到./dist
        filename: 'index.js'                   // 输入到./dist/index.js
    },
    devServer: { // 启动地址 localhost:8081
        host: 'localhost',
        port: 8081
    },
    // 默认 production
    // 当 development 时，修改代码后会直接刷新页面
    mode: 'development'
};
```

```json
// 9. 编辑 package.josn
{
  "name": "demo1",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack",       // 用命令 cnpm build 打包
    "start": "webpack serve"  // 用命令 cnpm run start 运行
  },
  "author": "",
  "license": "ISC",
  // 这几项都是通过 cnpm i webpack --save-dev 自动添加进来的
  // 也可以事先写好后，通过 cnpm i 自动安装
  "devDependencies": {
    "webpack": "^5.26.3",
    "webpack-cli": "^4.5.0",
    "webpack-dev-server": "^3.11.2"
  }
}

// 10. cnpm run start 运行
// 11. 打开浏览器 localhost:8081
```

## bootstrap4

```bash
# 在上面基础上还要安装：
cnpm i bootstrap jquery popper.js css-loader style-loader --save-dev
```

```javascript
// webpack.config.js 里额外添加：
module: {
    rules: [
        {
            test: /\.css$/,
            use: ['style-loader', 'css-loader']
        }
    ]
}
```

```javascript
// index.js 里加上：
import 'bootstrap';
import 'bootstrap/dist/css/bootstrap.min.css';
```



# 收藏

```bash
https://webpack.docschina.org/concepts/ # 官方文档
```

