# 安装

> 如果遇到npm install 任何插件都报 Cannot read property 'resolve' of undefined那就需要.zip安装

1. **下载.zip文件并解压**

2. **配置环境变量**
- NODE_PATH：C:\Program Files\node-v12.16.1-win-x64
  
- path：%NODE_PATH%

## cnpm 插件安装

```bash
# cnpm 
# 淘宝团队做的国内镜像，因为npm的服务器位于国外可能会影响安装。
# 淘宝镜像与官方同步频率目前为 10分钟一次以保证尽量与官方服务同步。
npm install cnpm -g --registry=https://registry.npm.taobao.org # 安装失败的话 --registry 的 --有问题
# 解决 cnpm -v / install 无响应
# 1. 先卸载
npm uninstall -g cnpm --registry=https://registry.npm.taobao.org
# 2. 注册模块镜像
npm set registry https://registry.npm.taobao.org
# 3. node-gyp 编译依赖的 node 源码镜像
npm set disturl https://npm.taobao.org/dist
# 4. 清空缓存
npm cache clean --force
# 5. 执行上面的安装
```

## centos下安装

- **1.解压后cd到bin目录下**
- **2.pwd 查看当前目录**
- **3.vi /etc/profile 写上**
  - export PATH=$PATH:/root/node-v12.16.2-linux-x64/bin

- **5.source /etc/profile**
- **最后查看版本：node -v；npm -v；成功！！！**

# npm

```bash
npm config list # 查看配置
npm install --registry=https://registry.npm.taobao.org # 单次使用淘宝镜像
npm cache clean --force # 清除失败的缓存
```

# 代码

## 创建http服务

1. **在d盘根目录下创建service.js并加入代码：**

```javascript
var http = require("http");
http.createServer(function (request, response) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.write("Hello World");
  response.end();
}).listen(8888);
```

2. **cd到d盘下输入命令：node service.js**

3. **在浏览器地址栏里输入 http://localhost:8888**
   - 显示结果：Hello World

## 创建模块

- **hello.js**

```javascript
exports.aaa = 'haha';
```

- **main.js**

```javascript
let aaa = require('./hello');
console.log(aaa.aaa);
```

## 封装到模块中

- **hello.js**

```javascript
function Hello() {
  this.fun = () => 'haha';
}
module.exports = Hello;
var ss = "dd"
```

- **main.js**

```javascript
let aa = require('./hello');
console.log(new aa().fun());
```

# 启动web服务

```bash
npm install http-server -g # 全局安装 http-server
http-server # 在指定静态页面文件夹下执行
```



# http

```javascript
// get请求
app.get('/', (req, res) => {
  let http = require('http');
  http.get('http://localhost:3357/api/test', (request, response) => {
    var html='';
    request.on('data', (data) => html += data);
    request.on('end', () => res.send(html));
  });
});
 
// http进行Post请求
app.get('/', (req, response) => {
  let http = require('http');
  let data = '';
  let options = {
    hostname:'localhost',
    port: 3357,
    path:'/api/Menu/Index',
    method:'POST',
    headers:{'Content-Type':'application/x-www-form-urlencoded;charset=UTF-8'}
  };
  http.request(options, res => {
    res.setEncoding('utf-8');
    res.on('data', dt => data += dt);
    res.on('end', () => {
      let os = JSON.parse(data);
      // 返回数据不能用send会报错
      response.end(data);
    });
  }).on('error',err => console.log(err)).write('要传的参数').end();
});
```

# Express 4.x

## 安装

```bash
npm install express --save
// 中间件，用于处理 JSON, Raw, Text 和 URL 编码的数据
npm install body-parser --save
// 一个解析Cookie的工具。通过req.cookies可以取到传过来的cookie，并把它们转成对象
npm install cookie-parser --save
// 用于处理 enctype="multipart/form-data"（设置表单的MIME编码）的表单数据
npm install multer --save
// 查看版本号
npm list express
```

## 例子

```javascript
let express = require('express');
let app = express();
// 解决跨域访问
// 一定要放在这个最开始的位置
app.all("/*", function(req, res, next) {
    // 跨域处理
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "X-Requested-With");
    res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
    res.header("X-Powered-By", ' 3.2.1');
    //res.header("Content-Type", "application/json;charset=utf-8");
    next(); // 执行下一个路由
});
// get请求
app.get('/', (req, res) => {
  res.send('get:hello world');
});
// post请求
app.post('/', (req, res) => {
  res.send('post:hello world');
});
// 自定义路游http://localhost:8888/test
app.get('/test', (req, res) => {
  res.send('test');
});
// 启动服务
let server = app.listen(8888, () => {
  let host = server.address().address;
  let port = server.address().port;
  console.log(`http://%s:%s`, host, port);
});
```

```javascript
// app.locals对象是一个javascript对象，它的属性就是程序本地的变量
// post参数
let body_parser = require('body-parser'); // npm install body-parser --save
app.use(body_parser.json()); // application/json
// application/x-www-form-urlencoded
app.use(body_parser.urlencoded({ extended: true }));
app.use(multer()); // multipart/form-data
 
app.post('/db/menu/save', (req, res) => {
    res.end(JSON.stringify(req.body));
});
```

# 命令

```bash
# cnpm view 名 version/versions
cnpm view webpack-dev-server version  # 查看某包当前版本
cnpm view webpack-dev-server versions # 查看某包所有版本
```

# nodemon

修改代码后不用重启，实时反应到服务上

```bash
cnpm i nodemon -g # 全局安装
nodemon xxx.js    # 启动服务
```

# 坑

## 1. node-sass安装失败

```bash
# 当提示说需要用到python时
# 1. 在应用商店里安装 python3.9
# 2. node install
npm install node-sass --registry=https://registry.npm.taobao.org
```

```bash
# 当上一步完成后发现项目还是跑不起来，说node-sass版本太高，4.14.1合适
# 1. 卸掉 python3.9
# 2. 以`管理员身份`进cmd，cd到项目目录，执行：
cnpm install
# 3. 发现还是报错，一看是把 package.json 里的某几个库版本改了（非手动，可能是哪个命令）
#	 直接再取一个原始的 package.json，再 cnpm install，OK 好用！
```

