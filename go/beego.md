# beego 介绍

一个开源的基于 golang 的 MVC 框架，主要用于 Web 开发。Beego 可以用来快速开发 API、Web 后端服务。

Golang 的 Web 框架有 Beego（star:22.8k）、Buffalo（5.6k）、Echo（17.2k）、Gin（37.9k）、Iris（18.1k）、Revel（11.7k）

个人开发、小项目可用 gin；团队开发用 beego。

官网：https://beego.me/

github：https://github.com/astaxie/beego

# 脚手架 bee

通过 bee 很容易进行 beego 项目的创建、热编译、开发、测试和部署。

```bash
go get -u github.com/beego/bee/v2              # 安装 bee 脚手架
go get -u github.com/beego/beego/v2            # 安装 beego 相关包
```

## 1. 带页面项目

```bash
bee new 项目名称  # 创建项目
# 运行项目，如果改完代码后运行看没发生变化，那就换 go run main.go 来运行
# 不能热编译的原因是：创建了一个叫 bee 的项目，换成别的名就可以了
bee run          # 运行
```

## 2. api接口项目

```bash
bee api 项目名称     # 创建项目
go build -mod=mod   # 如果 bee run 报错，则执行
# 第一次运行时需要执行
# -gendoc=true 表示每次自动化的 build 文档
# -downdoc=true 会自动的下载 swagger
bee run -gendoc=true -downdoc=true
# 上面运行成功后，再运行只需要 bee run 就行，同样能访问 swagger
http://localhost:8080/swagger/  # swagger 页面
```

> 注：安装好后发现cmd里bee不是内部或外部命令，只要将 C:\Users\xxx\go\bin（即gopath） 放到环境变量的 path 下即可。（要重启vscode）

# 代码结构

## conf/app.conf

```bash
appname = beego # 项目名称
httpport = 8080 # 项目端口
runmode = dev # 当前为开发环境
```

## controller/default.go

```go
type MainController struct {
	beego.Controller // 继承自 Controller
}

func (c *MainController) Get() {
    // 给页面赋值
    // .Data：为 map[interface{}]interface{}
	c.Data["Website"] = "beego.me"
	c.Data["Email"] = "astaxie@gmail.com"
    // 渲染 views/index.tpl
    // 也可以是 index.html
	c.TplName = "index.tpl"
    
    // 直接往页面上写文字
    c.ctx.WriteString("文字")
}
```

## views/index.tpl

```html
<!-- 显示 controller 里给赋的值 -->
{{.Website}} <br>
{{.Email}}
```

## routers/router.go

```go
func init() {
    // 配置路由
    beego.Router("/", &controllers.MainController{})
}
```

#  1. MVC

```go
// controller
// 方法名首字母必须大写
func (c *MainController) Post() {
	c.Ctx.WriteString("post")
}
func (c *MainController) Put() {
	c.Ctx.WriteString("put")
}

// router
// 参数1：地址
// 参数2：controller地址
// 参数3：
//     提交方式:方法名（如：post:add, put:update，*:all），不写则默认 get
//             *:all 表示all方法可以用任何提交方式
beego.Router("/", &controllers.MainController{}, "post:Post")
beego.Router("/", &controllers.MainController{}, "put:Put")  // 这两个实现了 RESTFul 风格
```

## 1.1 controller 里取参数

```go
c.GetString(key string) string
c.GetStrings(key string) []string
c.GetInt(key string) (int64, error)
c.GetBool(key string) (bool, error)
c.GetFloat(key string) (float64, error)
```

## 1.2 beego.Info

```go
// 等同于 System.out.println
beego.Info("haha")  // 结果：2020/09/10 22:28:06.036 [I] [default.go:19]  haha
```

## 1.3 结构体接收表单

通过 .ParseForm(&实体类对象)

```go
// 实体类
type User struct {
    // form:"id"：key和value之间不能有空格
	Id       int    `form:"id" json:"id"` // 和页面html上name一致
	Name     string `form:"name" json:"name"`
	Password string `form:"password "json:"password"`
}
```

```go
// controller
func (c *UserController) Index() {
	c.TplName = "user/index.html"
}

func (c *UserController) Add() {
	u := models.User{}
    // 接收表单到结构体
	if err := c.ParseForm(&u); err != nil {
		c.Ctx.WriteString("post提交失败")
		return
	}
	fmt.Printf("%#v", u)
	c.Ctx.WriteString("成功")
}
```

## 1.4 返回json

```go
func (c *UserController) GetUser() {
	var u = models.User{Id: 1, Name: "aa", Password: "bb"}
    // 实体类中的 json:"id" 用来防止key的首字母大写
	c.Data["json"] = u
	c.ServeJSON()
}
```

## 1.5 获取post过来的xml数据

> 虽然很少用xml格式的数据，但支付宝和微信支付完成后返回的都是xml格式信息

```bash
# 在 conf/app.conf 加上
copyrequestbody = true
```

```go
// 实体类
type User struct {
    // 要xml反射
	Id    int    `xml:"id"`
	Name  string `xml:"name"`
	Phone string `xml:"phone"`
}
// controller
func (c *MainController) GetXml() {
    // 打印requestBody
	beego.Info(string(c.Ctx.Input.RequestBody))
	u := models.User{}
    // 解析
	if err := xml.Unmarshal(c.Ctx.Input.RequestBody, &u); err == nil {
		c.Data["json"] = u
	} else {
		c.Data["json"] = err.Error()
	}
	c.ServeJSON()
}
// router
beego.Router("/xml", &controllers.MainController{}, "post:GetXml")
```

```xml
<!-- 最后用PostMan传参：body => raw （类型为xml） -->
<?xml version="1.0" encoding="utf-8"?>
<user> <!-- 外层标签随便起名 -->
    <id type="int">1</id>
    <name type="string">gt</name>
    <phone type="string">123456</phone>
</user>
```

# 2. 路由

## 2.1 动态路由

```go
// 完全 RESTFul 风格
beego.Router("/xxx/:id", &controllers.MainController{}, "get:xxx")
// 获取 :id 的值
c.Ctx.Input.Param(":id")
```

## 2.2 正则路由

> 不常用，但可用于==伪静态==

```go
// 将 id([0-9]+).html 看作一个整体
// http://localhost:8080/cms_12.html 就可以调
// 取 :id 和 2.1 一样
beego.Router("/cms_:id([0-9]+).html", &controllers.MainController{}, "get:xxx")
```

## 2.3 路由跳转

- 后台实现

```go
// 301：永久重定向
// 302：临时重定向
c.Redirect("/", 302) // 注意：controller 里的方法不能叫 Redirect
return
```

- 模板实现

```html
<head>
  <!-- 5秒后跳转到百度 -->
  <!-- url=/ 表示跳到首页 -->
  <meta http-equiv="refresh" content="5; url=http://www.baidu.com" />
  <title>Document</title>
</head>
```

## 2.4 api 接口路由

> controller

```go
// 下面每个方法上的 @注解都是有用的，都会在 swagger 上显示
// @router 则是指定方法对应的路由
type LoginController struct {beego.Controller}

// @Title 用户登陆
// @Description 用户登陆
// @Param   userId     formData    string      true      "用户名"
// @Param   password   formData    string      true      "密码"
// @Success 200 {object} string
// @router / [post]
func (c * LoginController) Post() {
    u := models.Login{}
    // 前端访问：axios.post('v1/login', {userId: 'aa', password: 'bb'})
	if err := json.Unmarshal(c.Ctx.Input.RequestBody, &u); err != nil {
		c.Data["json"] = err.Error()
	} else {
		c.Data["json"] = u
	}

	c.ServeJSON()
}
```

```go
type RegisterController struct {beego.Controller}

// @Title 用户注册
// @Description 用户注册
// @Param   userId            formData    string      true      "用户名"
// @Param   password          formData    string      true      "密码"
// @Param   passwordConfirm   formData    string      true      "确认密码"
// @Success 200 {object} string
// @router / [post]
func (c * RegisterController) Register() {}
```

> router

```go
// @APIVersion 1.0.0
// @Title 数采一体机管理平台API
// @Description 数采一体机管理平台API
// @Contact
// @TermsOfServiceUrl
// @License agilor
// @LicenseUrl
package routers

import (
	"api/controllers"
	"github.com/beego/beego/v2/server/web"
)

func init() {
	ns := web.NewNamespace("/v1",
		web.NSNamespace("/login", web.NSInclude(&controllers.LoginController{})),
		web.NSNamespace("/register", web.NSInclude(&controllers.RegisterController{})))
	web.AddNamespace(ns)
}

// 坑一：
// 想把 login 和 register 写在一个 controller 里
// 但是 不能用 NSNamespace("/", ...) 这种写法
// bee run 后会报 "prefix should has path" 错，但 bee run -gendoc=true 不会报
// 不报错但每次访问都是 404

// 坑二：
// 如果启动起来后发现访问是404，就用 bee run 跑一下看有没有错
// 如果没有错，基本上第二次 run 后就可以访问了（无论带不带 -gendoc=true ）

// 注意：
// 每次改动路由时，都要删除 routers/commentsRouter_controllers.go
// 然后再 bee run -gendoc=true，不删的话不会重新生成
```

# 3. 视图

```go
func (c *MainController) Get() {
	c.Data["key"] = "value"
	c.TplName = "index.html"
}
// 前台
// <div>{{.key}}</div>
```

## 3.1 修改模板标签

```bash
# app.conf
TemplateLeft=<<
TemplateRight=>>
```

```html
<!-- 此时模板标签就由 {{.key}} 变成了 <<.key>> -->
<div><<.key>></div>
```

## 3.2 模板中的自定义变量

```html
<!-- $变量名 := 值 -->
{{$val := "haha"}}
<!-- 显示变量 -->
{{$val}}
```

## 3.3 循环遍历切片

==前端循环map也和切片的写法一样==

```go
c.Data["list"] = []string {"a","b","c"}
```

```html
{{range $k,$v := .list}}
<a href="#">{{$v}}</a>
{{end}}
```

## 3.4 条件判断

=={{with}}和{{if}}基本一样==

```html
{{if .isTrue}}
<div>true</div>
{{else}}
<div>false</div>
{{end}}
```

- eq ==，ne !=，lt <，le <=，gt >，ge >=

```html
{{if gt .n1 .n2}}
<div>n1 gt n2</div>
{{else}}
<div>n1 le n2</div>
{{end}}
```

## 3.5 自定义模板

```html
<!-- 定义模板 -->
{{define "tplName"}}
<h1>自定义模板</h1>
{{end}}

<!-- 引用模板 -->
{{template "tplName" .}}  <!-- .代表自定义模板里能引用c.Data里的数据 -->
```

## 3.6 模板引入

```html
<!-- views/_footer.html -->
<h1>footer.html</h1>
<p>Email：{{.Email}}</p>   <!-- c.Data["Email"] = "astaxie@gmail.com" -->

<!-- .代表自定义模板里能引用c.Data里的数据 -->
{{template "/_footer.html" .}}
```

# 4. 内置模板函数

## 4.1 date

```go
c.Data["now"] = time.Now()
{{date .now "Y/m/d H:i:s"}} // 结果：2020/09/25 17:28:06
```

## 4.2 substr

```go
c.Data["str"] = "a一b二c三d四"
{{substr .str 0 4}} // 结果：a一b二
```

## 4.3 html2str

去掉字符串里的html标签

```go
c.Data["s"] = "<a href=\"#\">link</a>"
{{.s | html2str}} // 方法一
{{html2str .s}}   // 方法二，两种写法效果一样
```

## 4.4 str2html

把html字符串画到页面上

```go
{{.s | str2html}} // 方法一
{{str2html .s}}   // 方法二，两种写法效果一样
```

## 4.5 htmlquote

html转义

```go
{{htmlquote .s}} // 结果：&lt;a&nbsp;href=&#34;#&#34;&gt;link&lt;/a&gt;
```

## 4.6 htmlunquote

将4.5的字符串转回来

```go
c.Data["s"] = "&lt;a&nbsp;href=&#34;#&#34;&gt;link&lt;/a&gt;"
{{htmlunquote .s}} // 结果：<a href="#">link</a>
```

## 4.7 assets_js

引入.js文件

```go
{{assets_js "/static/js/a.js"}} // <script src="/static/js/a.js"></script>
```

## 4.8 assets_css

引入.css文件

```go
{{assets_css "/static/css/a.css"}} // <link href="/static/css/a.js" rel="stylesheet" />
```

## 4.9 config

获取 conf/app.conf 里的值。

{{config configType configKey defaultValue}}

configType为可选参数：有 String Bool Int Int64 Float DIY ==(首字母大写)==

```go
{{config "String" "httpport" ""}} // 结果：8080
```

## 4.10 map_get

获取map的值

```go
c.Data["map"] = map[string]interface{}{
    "a": "aaa",
    "m": map[int]int{0: 100},
}

{{map_get .map "a"}}    // 结果：aaa
{{map_get .map "m" 0}}  // 结果：100
```

# 5. 自定义模板函数

```go
// 定义一个方法
func hello(in string) string {
	return in + " world!"
}

func main() {
    // 加到模板里
	beego.AddFuncMap("hi", hello)
	beego.Run()
}

// 也是两种方法调用，参数给常量就"值"，参数给c.Data就 .变量
{{hi "AA"}}
{{.s | hi}}
```

# 6. 配置静态目录

beego默认的静态目录是static，此时如果想加个下载目录download的话

```go
func main() {
	// 先新建一个download文件夹，然后里没添加个文件
    // 要配置多个静态目录就再建几个文件夹，再写几行 SetStaticPath
	beego.SetStaticPath("dw", "download")
    // 当访问  http://localhost:8080/dw/文件名  时就能够下载文件了
	beego.Run()
}
```

# 7. model（模块）

如果某一功能需要在多个控制器里复用，那么抽出来作为一个独立的模块（model）。也可以在 model 里实现和数据库打交道的功能。

# 8. app.conf 参数配置

```bash
key = value # =号左右有空格，重启后才能生效
```

## 8.1 beego.AppConfig

| 方法                               | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| Set(key, val string) error         | 设置全局 config。**html里可通过 {{config "String" "key" ""}} 获取** |
| String(key string) string          | 获取config里的string值                                       |
| Strings(key string) []string       | 获取config里的string切片<br>如：==ar = "a","b","c"==<br>beego.AppConfig.Strings("ar") // 结果：["a","b","c"] |
| Int(key string) (int, error)       | 获取config里的int值                                          |
| Int64(key string) (int64, error)   | 获取config里的int64值                                        |
| Bool(key string) (bool, error)     | 获取config里的bool值                                         |
| Float(key string) (float64, error) | 获取config里的float值                                        |

## 8.2 定义不同模式

```bash
# 配置文件里指定mode：runmode = test
# 配置文件里的runmode别和代码里的beego.BConfig.RunMode重复
[dev]
httpport = 8080
# 代码里指定mode：beego.BConfig.RunMode = "dev"
[prod]
httpport = 80
# 代码里指定mode：beego.BConfig.RunMode = "prod"
[test]
httpport = 8888
# 代码里指定mode：beego.BConfig.RunMode = "test"
```

## 8.3 加载其它配置文件

```go
// 添加文件 conf/cf.conf
beego.LoadAppConfig("ini", "conf/cf.conf")
// 参数1：
// 目前支持：INI、XML、JSON、YAML
```

## 8.4 BConfig 系统默认参数

```go
beego.BConfig.AppName // 应用名称
beego.BConfig.RunMode // 运行模式
beego.BConfig.RouterCaseSensitive // 路由是否忽略大小写匹配，默认true，区分大小写
beego.BConfig.EnableGzip // 开启gzip压缩
beego.BConfig.MaxMemory // 文件上传默认缓存大小，默认64M
beego.BConfig.EnableErrorsShow // 是否显示系统错误信息，默认true
beego.BConfig.EnableErrorsRender // 是否渲染错误信息，默认true；api接口的应用应设置false
```

# 9. cookies

## 9.1 设置获取

```go
// 设置
// 参数3：为过期时间，单位秒，默认3600秒
// 参数4：设置访问权限，/detail只有 http://localhost:8080/detail 能获得cookie值；/ 则所有路径都能获得cookie值
// 参数5：表示 a.gt.com，b.gt.com，c.gt.com 都能获取cookie值
// 参数6：secure
// 参数7：httpOnly
c.Ctx.SetCookie("key", "value", 10, "/detail", ".gt.com")
c.Ctx.GetCookie("key") // 获取
// 注意：GetCookie没法设置中文
```

## 9.2 加密/设置中文

```go
// 比GetCookie多了第一个参数 =》 加密字符串
c.Ctx.SetSecureCookie("123", "aaa", "a一b二c三d") // 设置
v, _ := c.Ctx.GetSecureCookie("123", "aaa") // 获取
```

## 9.3 删除

```go
// 时间设置为0
c.Ctx.SetCookie("key", "value", 0)
c.Ctx.SetSecureCookie("123", "aaa", "", 0)
```

# 10. session

## 10.1 开启session

```go
// app.conf
sessionon = true
// 代码里
beego.BConfig.WebConfig.Session.SessionOn = true
```

## 10.2 设置/获取

```go
c.SetSession("key", "value") // 设置
c.GetSession("key") // 获取方法一
c.Ctx.Input.Session("key") // 获取方法二
```

## 10.3 配置

```go
// 默认是memory，还支持 file，mysql，redis 等
// app.conf：sessionprovider
beego.BConfig.WebConfig.Session.SessionProvider
// 也是cookie的name，默认beegosessionID
beego.BConfig.WebConfig.Session.SessionName
// session过期时间，默认3600秒；每次请求都会强行设置过期时间
// 假如10秒过期，在第9秒刷新了，此时过期时间又变10秒了
beego.BConfig.WebConfig.Session.SessionGCMaxLifetime
// session的加密key，默认beegoserversessionkey，建议修改一下再用session
beego.BConfig.WebConfig.Session.SessionHashKey
```

## 10.4 分布式session解决方案

通过 SessionProvider 设置，将session保存到redis里，这时候每个服务器都能够访问同一个session源了

# 11. 错误拦截与处理

beego 只支持 401、403、404、500、503 这几种错误：

| code | cn             | en                    |
| ---- | -------------- | --------------------- |
| 401  | 身份认证未通过 | Unauthorized          |
| 403  | 拒绝访问       | Forbidden             |
| 404  | 资源不存在     | Not Found             |
| 500  | 服务器内部错误 | Internal Server Error |
| 503  | 服务不可用     | Service Unavailable   |

## 11.1 代码介绍

```go
package lib

import (
	"github.com/beego/beego/v2/core/logs"
	"github.com/beego/beego/v2/server/web"
	"github.com/beego/beego/v2/server/web/context" // 第一个参数 ctx 的类型
    "runtime"
)

func ErrorIntercept(ctx *context.Context, cf *web.Config) {
	if err := recover(); err != nil {
        for i := 0;; i++ {
			_, file, line, ok := runtime.Caller(i) // 取错误调用路径
			if !ok {
				break
			}
            fmt.Println("==>", file, line, ok)
            // 设 ... = C:/Users/11487/go/pkg/mod
            // ==> C:/GT/work/Agilor-Hub-Web/api/lib/error.go 13 true
			// ==> C:/Program Files/Go/src/runtime/panic.go 965 true
			// ==> .../github.com/beego/beego/v2@v2.0.1/server/web/controller.go 354 true
			// ==> .../github.com/beego/beego/v2@v2.0.1/server/web/controller.go 346 true
			// ==> C:/GT/work/Agilor-Hub-Web/api/controllers/register.go 27 true
			// ==> C:/Program Files/Go/src/reflect/value.go 476 true
			// ==> C:/Program Files/Go/src/reflect/value.go 337 true
			// ==> .../github.com/beego/beego/v2@v2.0.1/server/web/router.go 883 true
			// ==> .../github.com/beego/beego/v2@v2.0.1/server/web/filter.go 81 true
			// ==> .../github.com/beego/beego/v2@v2.0.1/server/web/router.go 664 true
			// ==> C:/Program Files/Go/src/net/http/server.go 2867 true
			// ==> C:/Program Files/Go/src/net/http/server.go 1932 true
			// ==> C:/Program Files/Go/src/runtime/asm_amd64.s 1371 true
		}
        
        // panic("haha")，则 err = haha
        // controller.Abort("401"), 则 err = 401
        logs.Error("err", err)
        // panic，则 ctx.Output.Status = 0
        // controller.Abort("401")，则 ctx.Output.Status = 401
		logs.Error("status", ctx.Output.Status) 
		logs.Error("method", ctx.Input.Method()) // POST、GET、PUT等
		logs.Error("url", ctx.Input.URL())       // 如：/v1/login，/v1/register
		logs.Error("uri", ctx.Input.URI())       // 同上
		logs.Error("ip", ctx.Input.IP())         // 客户端 ip
	}
}
```

```go
func main() {
	web.BConfig.RecoverFunc = lib.ErrorIntercept // 使用全局错误拦截
	web.Run()
}
```

## 11.2 完整功能

```go
package models
// 统一返回给前端的数据结构
type Result struct {
	Success bool        `json:"success"`
	Data    interface{} `json:"data"`
	Message string      `json:"message"`
	Stack   []string    `json:"stack"`
}
// 成功
func Success(data interface{}, msg string) Result {
	return Result{Success: true, Data: data, Message: msg}
}
// 失败
func Failed(msg string, stack []string) Result {
	return Result{Success: false, Message: msg, Stack: stack}
}
```

```go
func ErrorIntercept(ctx *context.Context, cf *web.Config) {
	if err := recover(); err != nil {
		var stack []string
		for i := 0;; i++ {
			_, file, line, ok := runtime.Caller(i)
			if !ok {
				break
			}
			s := fmt.Sprintf("%v:%v", file, line)
			logs.Critical(s)
			stack = append(stack, s)
		}
		// 失败的 Result
		result := models.Failed(fmt.Sprintf("%v", err), stack)
		ctx.Output.JSON(result, true, true)

		if ctx.Output.Status == 0 {
            // panic 触发的
            // axios.interceptors.response 走正确的回调（Status Code = 200）
            // 前端有 Result 结构，用于处理业务 error
			ctx.ResponseWriter.WriteHeader(500)
		} else {
            // controller.Abort 触发的
            // axios.interceptors.response 会走 err 回调（Status Code != 200）
            // 前端无 Result 结构，用于处理 401、403、404 等错误
			ctx.ResponseWriter.WriteHeader(ctx.Output.Status)
		}
	}
}
```





https://www.bilibili.com/video/BV16441137ut?from=search&seid=2754981446573609310
