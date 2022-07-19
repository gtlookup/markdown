# WebAssembly

1）通过 web 执行一种类似于机器码的程序，简称 wasm。相对于js解析执行，性能大大提升，目前主流浏览器都已支持1）

2）wasm 本身是一种字节码标准：一般通过 c/c++、GO、Rust来进行开发，并编译成wasm。其中 Rust 在这块相对更活跃一些

3）场景：如图像处理、视觉效果、3D游戏、其他需要 CPU 高性能计算的场景

## 运行wasm方式

1）Rust 部分

1. 创建项目：cargo new xxx --lib 
2. 修改 Cargo.toml
3. 编写 lib.rs
4. wasm-pack build

2）npm 部分

```toml
# 1. cargo new wasm_demo --lib && cd wasm_demo
# 2. 编辑 Cargo.toml
[lib]
crate-type = ["cdylib"] # c 的动态链接库

[dependencies]
wasm-bindgen = "0.2.71" # 与js交互

[dependencies.web-sys] # 这种写法是指定只引 web-sys 包下的哪几项，全引太大
version = "0.3.48"
features = ["Document", "Element", "HtmlElement", "Node", "Window"]
```

```rust
// 3. 编辑 lib.rs
extern crate wasm_bindgen; // 用于跟js交互
use wasm_bindgen::prelude::*; // 允许js调用rust

#[wasm_bindgen]
extern { pub fn alert(s: &str); } // Rust 要调 js 的方法

#[wasm_bindgen]
pub fn echo() -> String { format!("haha") } // js 要调 Rust 的方法

// 页面一打开给就添加一个<p>rust nice!!!</>
#[wasm_bindgen(start)] // start 表示页面一打开就调这个方法
pub fn init() -> Result<(), JsValue> {
    let window: Window = web_sys::window().unwrap();   // 取得 window
    let document = window.document().unwrap();         // 取得 document
    let body = document.body().unwrap();               // 取得 body
    let p = document.create_element("p").unwrap();     // 创建 p
	alert("haha");                                     // 画面直接弹出 alert
    p.set_inner_html("rust nice!!!");  // 设置p内容
    body.append_child(&p);             // p添加到

    Ok(())
}
// 4. wasm-pack build
// 5. 此时会生成 aswm_demo/pkg 说明已经成功
```

```javascript
// 这几部分可参数 前端 -> webpack.md
// 6. cnpm init
// 7. cnpm i -g webpack --save-dev
// 8. cnpm i -g webpack-cli --save-dev
// 9. cnpm i -g webpack-dev-server --save-dev
// 10. cnpm i -g @wasm-tool/wasm-pack-plugin --save-dev
// 11. cnpm i -g text-encoding --save-dev
// 12. 创建并编辑 webpack.config.js
const path = require('path');
const webpack = require('webpack');
const WasmPackPlugin = require('@wasm-tool/wasm-pack-plugin');

module.exports = {
    entry: './index.js', // 入口文件
    output: {path: path.resolve(__dirname, 'dist'), filename: 'index.js'}, // 出口文件
    // 重要：webpack5+版本必要手动开启，否则就会报
    // Since webpack 5 WebAssembly is not enabled by default and flagged as experimental feature.
	// You need to enable one of the WebAssembly experiments via 'experiments.asyncWebAssembly: true' 
    // (based on async modules) or 'experiments.syncWebAssembly: true' (like webpack 4, deprecated).
    experiments: { asyncWebAssembly: true },
    plugins: [
        new WasmPackPlugin({ crateDirectory: path.resolve(__dirname, '.') }),
        new webpack.ProvidePlugin({
            TextDecoder: ['text-encoding', 'TextDecoder'],
            TextEncoder: ['text-encoding', 'TextEncoder']
        })
    ],
    mode: 'development'
}
```

```javascript
// 13. 创建并编辑 wasm_demo/index.js
const rust = import('./pkg').catch(console.error); // 导入 wasm-pack build 后生成的 pkg 目录
rust.then(x => window.rust = x); // 绑定到画面上，好给 button 用
```

```html
<!-- 14. 创建并编辑 wasm_demo/index.js -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="index.js"></script> <!-- 引入上一步的 index.js -->
    <script>
        function clk() {
            alert(rust.echo()) // js 调用 Rust 里的方法
        }
    </script>
</head>
<body><button onclick="clk()">Button</button></body>
</html>
```

```json
// 15. 编辑 package.json
{
  "name": "wasm_demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js", // 13. 创建的 index.js
  "scripts": {
    "build": "webpack",       // cnpm build 打包
    "serve": "webpack serve"  // cnpm run serve 运行
  },
  "author": "",
  "license": "ISC",
  // 这些都是 cnpm i xxx --save-dev 后自动生成的
  "devDependencies": {
    "@wasm-tool/wasm-pack-plugin": "^1.3.3",
    "text-encoding": "^0.7.0",
    "webpack": "^5.26.3",
    "webpack-cli": "^4.5.0",
    "webpack-dev-server": "^3.11.2"
  }
}
```

```bash
# 16. cnpm run serve
# 20. 浏览器打开 localhost:8080 就会显示：rust nice!!!
```

# yew介绍

目前贡献者最多的 Rust 前端框架，基于 WebAssembly 创建多线程的前端 web 应用

- 高性能：将前端工作分流致后端，减少 DOM API 调用
- 支持与 js 交互：允许使用 NPM 包，并与现有的 js 应用程序结合

# Hello world

通过`#收藏`里的`一个不错的例子`发现，还有比 webpack 更简便的构建方式

```rust
// 1. cargo install wasm-pack
// 2. cargo install cargo-make
// 3. cargo install simple-http-server
//		这个在安装时总超时，可以从 https://github.com/TheWaWaR/simple-http-server/releases 下载一个.exe的，
//		然后放到.cargo\bin下，然后重命名为 simple-http-server.exe 就可以了

// 4. Cargo.toml 编辑
[lib]
crate-type = ["cdylib"] # c 的动态链接库
[dependencies]
yew = "0.17.4"
yew-router = "0.14.0"   # 路由
wasm-bindgen = "0.2.71" # 与js交互
web-sys = "0.3.48"
log = "0.4"
wasm-logger = "0.2.0"

// 5. 写 hello world 组件，切记：主方法上一定一定要加 start 启动宏
#[wasm_bindgen(start)]
pub fn run() {
    wasm_logger::init(wasm_logger::Config::new(log::Level::Trace)); // 开启日志
    yew::start_app::<component::app::App>();
}

// 6. 创建 static/index.html
<html lang="en">
<head>
    <meta charset="utf-8" />
    <title>RustMart</title>
    <script type="module">
        import init from "/wasm.js";
        init();
    </script>
</head>
<body></body>
</html>

// 7. 创建 Makefile.toml
[tasks.build]
command = "wasm-pack"
args = ["build", "--dev", "--target", "web", "--out-name", "wasm", "--out-dir", "./static"]
watch = { ignore_pattern = "static/*" }
[tasks.serve]
command = "simple-http-server"
args = ["-i", "./static/", "-p", "8080", "--nocache", "--try-file", "./static/index.html"]

// 8. cargo make build 会一直牌监听状态，一旦有代码改动，便会自动build
// 9. cargo make serve
// 具体在链接里了
// 注意：需要开两个cmd窗口，一个用来build，另一个用来 run
//	   这样就能和 webpack 一样边改代码边实时运行了
```

# 组件

用于管理页面状态，并渲染成DOM；每个组件都必须实现 Components 接口（trait），任何实现了 `Component` trait 的类型都可被用在 `html!` 宏中。

如果组件的 `Properties` 中有 `children` 字段，则可以被传递子组件

```rust
pub trait Component: Sized + 'static {
    // 事件传递的枚举，简单的组件可以声明为`()`，即单元类型的值，`;`返回`()`
    // 复杂点儿的就定义个枚举来表示多种消息类型
    // 然后在 update 方法里 match 这个枚举，以标识画面上是点了按钮还是<select>的change事件
    type Message: 'static;
    // 组件属性，理解为一个函数的入参，即外界想让本组件做什么，不应该在组件内发生变化（父组件到子组件的通信）
    // html! {
    //		<Model prop="value" />
	// }
    // 在写这个属性类时，要加上 #[derive(Properties)]
    type Properties: Properties;
}
```

## 属性

```rust
/// 一个 type Properties: Properties; 的例子

pub struct LinkColor { Blue, Red, Green, Black, Purple }

impl Default for LinkColor {
    fn default() -> Self {
        // 除非另有说明，否则链接的颜色将为蓝色
        LinkColor::Blue
    }
}
fn default_back_color -> LinkColor { // #[prop_or_else(default_back_color)]
    LinkColor::Red
}

#[derive(Properties, PartialEq)] // PartialEq 一般都要加，项目少行，多了一项项比太麻烦；还可避免重复渲染
pub struct LinkProps {
    /// 所有字段默认都是必填的，不填会报错
    href: String,
    /// 每个字段都是有所有权的，每当重新渲染都会有拷贝行为
    /// 如果文本太大，产生了效率问题，可以考虑用 Rc
    text: Rc<String>,
    /// 字段默认是必填的，如果想有默认值要加这个宏
    /// 还要实现 impl Default 这个 trait
    #[prop_or_default]
    color: LinkColor,
    /// 如果为 None，则 view 函数将不指定大小
    #[prop_or_default]
    size: Option<u32>
    /// 另一种指定默认值的宏
    #[prop_or_else(true)]
    active: bool,
    #[prop_or_else(default_back_color)] // 还可以指定方法去取默认值
    back_color: LinkColor
}
```

## create

创建组件时调用

```rust
// props 是外界传进来的参数，link 用于注册回调
fn create(props: Self::Properties, link: ComponentLink<Self>) -> Self;
```

## update

事件发生时调用

```rust
// 接收并处理异步消息，并根据消息内容决定是否重新渲染
// ShouldRender：就是一个 bool 型，返回 true 就重新渲染，false 就不渲染（即重新执行 view）
// 		pub type ShouldRender = bool;
fn update(&mut self, msg: Self::Message) -> ShouldRender;
```

## change

组件属性（type Properties: Properties;）改变时调用

```rust
// _props 有变化就返回true，没变化就返回false
//		  如果没有_props，就返回false
// 返回值和 update 一样，决定本组件是否重新渲染（即重新执行 view）
fn change(&mut self, _props: Self::Properties) -> ShouldRender;
```

## view

==注意：html! { 这里只能有一个根节点 }==，否则报：`only one root html element is allowed (hint: you can wrap multiple html elements in a fragment `<></>`)`，==但最外层可以包一个 `<></>`==

```rust
// 画组件的 html
fn view(&self) -> Html;
```

## rendered

```rust
// 每次组件创建完成后，浏览器渲染之前，会走这个方法
fn rendered(&mut self, _first_render: bool) {}
```

## destroy

```rust
// 组件销毁前会走
fn destroy(&mut self) {}
```

## 子组件

```rust
use yew::prelude::*;

#[derive(Properties, Clone)]       // 1. 实现 Properties 接口
pub struct ModalBody {
    pub id: String,
    #[prop_or_default]
    pub children: Children         // 2. 有了 Children 才能在  <Modal>这里加东西</Modal>
}

pub struct Modal {
    props: ModalBody,              // 3. Modal 的子组件
    link: ComponentLink<Self>
}

impl Component for Modal {
    type Message = ();
    type Properties = ModalBody;   // 4. 绑定子组件到 Modal
    ...
    fn view(&self) -> Html {
        html! {
            <div class="list">     // 5. 渲染子组件
                { for self.props.children.iter() }
            </div>
        }
    }
```

```rust
<Modal id="dlg">{"haha"}</Modal>    // 5. 使用组件
```



# 回调

```rust
// 注册了一个按钮事件，当点击事件发生后会走 update
<button onclick=self.link.callback(|_| Msg::AddOne)>{ "Button" }</button>
```

## emit

通知父组件（比如重新加载数据），例子在下面 reform 里

## reform

给 callback 里再加一个闭包方法，但原来那个会在新加这个之后执行

```rust
// 原码
pub fn reform<F, T>(&self, func: F) -> Callback<T>
    where F: Fn(T) -> IN + 'static,
{
    let this = self.clone(); // 先将原来的闭包方法克隆一份
    let func = move |input| { // this 所有权交给闭包
        let output = func(input); // 调用reform最新的闭包取得结果
        this.emit(output); // 新结果通知原闭包
    };
    Callback::from(func) // 新闭包返回
}
```

```rust
// 例子，app.rs
pub struct App {
    link: ComponentLink<Self>,
    b: &'static str,
    b_e: Callback<&'static str>
}
fn create(props: Self::Properties, link: ComponentLink<Self>) -> Self {
    Self {
        link,
        b: "BBBB",
        b_e: Callback::from(|x| x_log("22bb")) // create里必须初始化一下，可以给个空 from(|_| ())
    }
}
fn view(&self) -> Html {
    let link = self.link.clone(); // 克隆个 link
    let be = self.b_e.reform(move |x| { // 所有权给到闭包里
        x_log("app.view");
        link.send_message(AppAction::B); // 通知 update 方法
        x // 返回
    });
    html! {
        <B name=self.b event=be />
    }
}
////////////////////// b.rs
fn update(&mut self, msg: Self::Message) -> bool {
    x_log("b.update");
    self.props.event.emit("B.update"); // 通知父组件
    false
}
fn view(&self) -> Html {
    let e = self.link.callback(|_| BAction::Click); // b.rs 里的按钮点击事件，后会到 update
    html! {<button onclick=e>{self.props.name}</button>}
}

// 执行顺序：
// 1. b.rs.view 里的 self.link.callback 的闭包
// 2. b.rs.update
// 3. app.rs.view 里的 be
// 4. app.rs.create.b_e 的闭包
// 5. app.rs.update
```

## 事件传参

```rust
let ps = self.get_display_pages().map(|x| { // 一个循环
    let active = if x == self.props.pageNo { "page-item active" } else {"page-item"};
    html! {
        <li class={active}> // 直接在link上创建，再把 x move 进来
        	<a onclick=self.link.callback(move |_| Action::Click(x)) class="page-link" href="#">{x}</a>
        </li>
    }
});
```



# 路由

```toml
# 1. 添加依赖
yew-router = "0.14.0"
```

```rust
// 2. 添加路由枚举
use yew_router::prelude::*;
#[derive(Switch, Debug, Clone)]
pub enum Route {
    #[to = "/a"]
    A,
    #[to = "/b"]
    B,
    #[to = "/"]
    Home
}
```

```rust
// 3. 添加程序入口组件
use yew::prelude::*;
use yew_router::prelude::*;

use crate::pages::{Home, A, B};
use crate::route::Route;

pub struct App;

impl Component for App {
    type Message = ();    // 给自定义的消息类型重命名为 Message，没有就()
    type Properties = (); // 给自定义的属性类重命名为 Properties，没有就()
	fn create(props: Self::Properties, link: ComponentLink<Self>) -> Self { Self }
    fn update(&mut self, msg: Self::Message) -> ShouldRender { false }
    fn change(&mut self, prop: Self::Properties) -> ShouldRender { false }

    fn view(&self) -> Html {
        // 将第2步定义的路由枚举关联到组件
        let render = Router::render(|switch: Route| match switch {
            Route::A => html! {<A />},
            Route::B => html! {<B />},
            Route::Home => html! {<Home/>}
        });

        html! {
            <Router<Route, ()> render=render /> // 渲染
        }
    }
}
```

```rust
// 4. Route::A 对应的组件，Route::B 也同样写法
use yew::prelude::*;
use yew_router::components::RouterAnchor;
use crate::route::Route;

pub struct A;

impl Component for A {
    ...

    fn view(&self) -> Html {
        type Anchor = RouterAnchor<Route>; // 类型重命名。重点：不能有-或_
        html! { <Anchor route=Route::B>{"B"}</Anchor> } // 会生成 <a href="/b">B</a>
    }
}
```

## 重定向

==切记：==一定不能写 `<a href="/home">` 这种跳转，这样会走一趟服务端（运行前端的服务）刷新，

​            而下面这种只会在内（通过hash状态）刷新。验证办法也很简单，浏览器F12看 NetWork 里请求文件的数量就能知道。

### 1. 标签方式

```rust
use yew_router::components::{RouterButton, RouterAnchor};

fn view(&self) -> Html {
    type Btn = RouterButton<WebRouter>; // 生成一个 <button></button> 标签
    type A = RouterAnchor<WebRouter>;   // 生成一个 <a></a> 标签
    html! {
        <>
            // 属性 route=路由枚举，即想跳的那个页
            <Btn route=WebRouter::Home classes="btn btn-primary">{"button"}</Btn>
            <A route=WebRouter::Home classes="btn btn-primary">{"a"}</A>
        </>
    }
}
```

### 2. 代码方式

看原码后重新包了一个

```rust
use yew_router::RouterState;
use yew_router::agent::{RouteAgentDispatcher, RouteRequest};
use crate::router::WebRouter;

#[derive(Debug)]
pub struct Redirect<STATE: RouterState = ()> { router: RouteAgentDispatcher<STATE> }
impl Redirect {
    pub fn to(to: WebRouter) {
        let mut me = Self { router: RouteAgentDispatcher::new() };
        let r = yew_router::route::Route::from(to);
        me.router.send(RouteRequest::ChangeRoute(r));
    }
}

// 用法：
Redirect::to(WebRouter::Home);
```

## 当前 url

```rust
let s = RouteService::<WebRouter>::new().get_path(); // WebRouter 为项目定义的全局路由
```

# Service

## ConsoleService

全称 `yew::services::ConsoleService`，用这个下面那俩库都可以不用加了

```toml
log = "0.4"
wasm-logger = "0.2.0"
```

# 表单

## 双向绑定

这两个事件都在 `yew::html::listener` 下

### 1. InputData

对应的不是 onkeydown / onkeyup，只要 < input > 里字符一变立刻触发事件

```rust
/****************/
// 1. 添加组件属性
pub struct xxx { name: String, link: ComponentLink<Self> }
// 2. 创建一个组件内事件
pub enum Msg { InputValue(String) }
// 3. 添加输入事件，并绑定到输入框上
fn view(&self) -> Html {
    let on_input = self.link.callback(|e: InputData| Msg::InputValue(e.value)); // InputData
    html! { <input oninput=on_input type="text" /> } // 下面例子是 onchange
}
// 4. 通过事件给组件属性赋值
fn update(&mut self, msg: Self::Message) -> bool {
    match msg {
        Msg::InputValue(v) => {
            self.name = v; // 每次事件触发需要在这里把内容存信
            false // 不用重新渲染也能达到效果
        }
    }
}
```

### 2. ChangeData

对应 html 里的 onchange 事件，输入完后光标离开才触发事件

```rust
#[derive(Debug)] // 原码
pub enum ChangeData {
    /// Value of the element in cases of `<input>`, `<textarea>`
    Value(String),
    /// SelectElement in case of `<select>` element. You can use one of methods of SelectElement
    /// to collect your required data such as: `value`, `selected_index`, `selected_indices` or
    /// `selected_values`. You can also iterate throught `selected_options` yourself.
    Select(SelectElement),
    /// Files
    Files(FileList) // <input type="file" />
}
```

```rust
// 添加输入事件，并绑定到输入框上，这里和 InputData 稍微有点不一样
fn view(&self) -> Html {
    let on_input = self.link.callback(|e: ChangeData| {
        match e { // 因为 ChangeData 是个枚举，所以这里要 match
            ChangeData::Value(v) => Msg::Name(v),
            _ => Msg::Name("".to_string()) // Select 和 Files 不关心
        }
    });
    html! { <input onchange=on_input type="text" /> } // 上面例子是 oninput
}
```

```rust
// 添加 <select> 的 change 事件
let on_change = self.link.callback(|e: ChangeData| match e {
    ChangeData::Select(el) => Action::Position(el.value()), // 要从 select 里取值
    _ => Action::Position(String::new())
});
html! { <select onchange=on_change><option value=""></option><option value="0">{"aaa"}</option></select> }
```

## 事件

### 1. keyup

```rust
// 13：回车
let keyup = Callback::from(move |e: KeyboardEvent| if e.key_code() == 13 {link.send_message(Action::Login)});
<input onkeyup=keyup /> // 绑定
```





# ==注意==

## html! 报错

```rust
#![recursion_limit="1024"] // 这个要加到 lib.rs 的第1行，否则 view 里的 html 写多了会报错
```

## 一览编辑按钮

```rust
pub enum Action {
    Add,
    // 当编辑事件时，不要传整个结构体了，传一个id，然后再去结果集里 find，否则画表格时没法绑按钮事件
    Edit(u32),
    // 但是，如果 id 是 String 类型的话还是没法绑事件
    // 所以把上面的传 id 改成传索引，如：
    // list.iter().enumerate().map(|(i, o)| {
    //		let edit = self.link.callback(move |_| Action::Edit(i));
	// });
    // <button onclick=edit type="button">{"编辑"}</button>
    Edit(usize)
    // 最后在事件里通过索引直接取就可以了，如：
    // Action::Edit(i) => { list.get(i) }
}
```



# Log

```toml
log = "0.4"
wasm-logger = "0.2.0"
```

```rust
#[wasm_bindgen(start)]
pub fn run() {
    wasm_logger::init(wasm_logger::Config::new(log::Level::Trace)); // 初始化一下
    yew::start_app::<home::Home>();
}

log::log!(log::Level::Info, "Dirs"); // 画面上就显示 Dirs 了
```

# http请求

## 返回字符串例子

```rust
// actix-web 的服务端，地址是 localhost:8080
App::new().route("/u", web::get().to(|| HttpResponse::Ok().body("haha")))
```

```toml
# 额外需要的依赖
[dependencies]
log = "0.4"           # log 用
wasm-logger = "0.2.0" # log 用
anyhow = "1"          # Error 用
serde = "1"           # Json 用
```

```rust
use yew::services::FetchService;
use yew::services::fetch::FetchTask;

type FetchResult<T> = Response<Result<T, anyhow::Error>>; // 请求返回类型
pub enum Msg { Done } // 请求返回后告诉通知组件的事件
pub struct View { link: ComponentLink<Self>, task: Option<FetchTask> } // task 必须放到结构体里，否则被 drop 后就无法请求了，保证它生命周期够长
```

```rust
fn update(&mut self, msg: Self::Message) -> bool {
    let req = Request::get("http://localhost:8080/u").body(Nothing).unwrap();
    let cb = self.link.callback(|res: FetchResult<String>| {
        let dt: String = res.into_body().unwrap();
        info(dt); // 打印结果：haha
        LoginMsg::Done
    });
    self.task = Some(FetchService::fetch(req, cb).expect("req error")); // task 保存到组件里，否则短命导致请求失败
}
```

## 返回json 例子

```rust
// actix-web 的服务端，地址是 localhost:8080

#[derive(serde::Serialize)] // 序列化
struct User { id: u32, name: String } // 返回的 json 实体类

App::new().route("/u", web::post().to(|| HttpResponse::Ok()
    .content_type("application/json") // 设置返回 json
    .body(serde_json::to_string(&User{id: 1, name: "小小".to_string()}).unwrap())))
```

```rust
// yew 客户端，地址 localhost:8081
#[derive(serde::Deserialize, Debug)] // 反序列化
pub struct User { id: u32, name: String } // 返回的 json 实体类
type FetchResult<T> = Response<Json<Result<T, anyhow::Error>>>; // 请求返回类型，比上个例子多了个 Json<T>

let req = Request::post("http://localhost:8080/u").body(Nothing).unwrap(); // post 提交
let cb = self.link.callback(|res: FetchResult<User>| {
    let (_, Json(dt)) = res.into_parts();
    info(dt.unwrap()); // 打印结果：User { id: 1, name: "小小" }
    LoginMsg::Done
});
self.task = Some(FetchService::fetch(req, cb).expect("req error")); // task 保存到组件里，否则短命导致请求失败
```

# storage

sessionStorage为临时保存，而localStorage为永久保存

```rust
use yew::services::StorageService;
use yew::services::storage::Area;

// localStorage：是 Area::Local，sessionStorage：Area::Session
fn area(e: Area) -> StorageService {
    StorageService::new(e).ok().unwrap()
}
pub fn set_local(k: &str, s: String) { // 设值
    let mut local = area(Area::Local);
    local.store(k, Ok(s));
}
pub fn get_local(k: &str) -> String { // 取值
    let local = area(Area::Local);
    match local.restore(k) {
        Ok(v) => v,
        Err(_) => "".to_string()
    }
}
pub fn remove_local(k: &str) { // 删除
    let mut local = area(Area::Local);
    local.remove(k);
}
```

# 项目配置

用 `.env` 有一点有好，当改了配置内容后重启还不好用，属性要重新编译；.toml 的话前端还不能访问 `std::fs::io`，所以只能用 `.json` 了，其实也非常简单

```bash
# 1. 在 ./static 下创建 app.json
# 2. 然后利用普通的 `get` 请求 '/app.json' 就可以了
```

# 封装request

1）先创建一个 `Error` 全局错误枚举

```rust
use thiserror::Error as ThisError; // thiserror = "1.0.24"

#[derive(ThisError, Clone, Debug)]
pub enum Error {
    /// 401
    #[error("Unauthorized")]
    Unauthorized,
    /// 403
    #[error("Forbidden")]
    Forbidden,
    /// 404
    #[error("Not Found")]
    NotFound,
    /// 422
    #[error("Unprocessable Entity")]
    UnprocessableEntity,
    /// 500
    #[error("Internal Server Error")]
    InternalServerError,
    /// serde deserialize error
    #[error("Deserialize Error")]
    DeserializeError,
    /// request error
    #[error("Http Request Error")]
    RequestError
}
```

2）封装 `request`

```rust
use yew::services::fetch::{FetchTask, Request, Response, FetchService};
use yew::Callback;
use yew::format::{Text, Nothing, Json};
use serde::{Deserialize, Serialize};
use crate::utils::state::get_api;
use crate::error::Error;
use serde_json::Value;

// data：用来接普通的json
// value: 用来接graphql的返回数据。serde_json::Value 类似于 hashmap，可以和字符串来回转换（标准库里的HashMap不支持多类型）
// 坑：强烈不建议这么干，不是客户端创建 json 结构体的好，不然光解析 Value 都会怀疑人生！！！
#[derive(Debug)]
pub struct ResHql<T> { pub data: Option<T>, pub value: Option<Value> }

#[derive(Default, Debug)]
pub struct Http;

impl Http {
    pub fn new() -> Self { Self } // new 一个自己
    pub fn builder<T, B>( // T：http返回后的json实体类，B：传给 body 的类型
        &mut self,
        method: &str, // 这可以是个字符串
        url: &str,
        body: B,
        done: Callback<Result<ResHql<T>, Error>>
    ) -> FetchTask
    where
    	// T 怎么限制，根据编译时错误提示去限就行
    	for<'de> T: Deserialize<'de> + 'static + std::fmt::Debug, 
    	B: Into<Text>
    {
        let url = if url == "/app.json" { url.to_string() } // 取 app.json 里的 api 地址
        else { format!("{}{}", get_api().unwrap(), url) };  // 其它情况就是 api地址 + 具体请求

        let handler = Callback::from(move |res: Response<Text>| { // 这里先滤过一下，用 Response<Text> 接
            if let (head, Ok(rlt)) = res.into_parts() { // 把 Response 里的 head / body 解构出来
                if head.status.is_success() { // 请求是否成功
                    let data: Result<T, _> = serde_json::from_str(&rlt); // 反序列化
                    if let Ok(data) = data { // 反序列化成功，则返回数据
                        done.emit(Ok(ResHql{data: Some(dt), value: None})); // 返回 data
                    } else { // 反序列化不成功，看看转成 Value 是否成功
                        let val: Result<Value, _> = serde_json::from_str(&rlt);
                        match val {
                            Ok(dt) => done.emit(Ok(ResHql{data: None, value: Some(dt)})), // 成功返回 Value
                            _ => done.emit(Err(Error::DeserializeError)) // 否则返回错误信息
                        }
                    }
                } else { // 请求不成功
                    match head.status.as_u16() { // 根据错误代码返回错误信息
                        401 => done.emit(Err(Error::Unauthorized)),
                        403 => done.emit(Err(Error::Forbidden)),
                        404 => done.emit(Err(Error::NotFound)),
                        500 => done.emit(Err(Error::InternalServerError)),
                        422 => done.emit(Err(Error::DeserializeError)),
                        _ => done.emit(Err(Error::RequestError)),
                    }
                }
            } else {
                done.emit(Err(Error::RequestError))
            }
        });

        let req = Request::builder()
            .uri(url)
            .method(method)
            .header("Content-Type", "application/json")
            .body(body).unwrap();

        FetchService::fetch(req, handler).unwrap() // 返回 FetchTask，这个要保存到组件属性里，否则命不够长导致请求失败
    }

    pub fn get<T>(
        &mut self, url: &str,
        done: Callback<Result<ResHql<T>, Error>>
    ) -> FetchTask where for<'de> T: Deserialize<'de> + 'static + std::fmt::Debug {
        self.builder("GET", url, Nothing, done)
    }

    pub fn delete<T>(
        &mut self, url: &str,
        done: Callback<Result<ResHql<T>, Error>>
    ) -> FetchTask where for<'de> T: Deserialize<'de> + 'static + std::fmt::Debug {
        self.builder("DELETE", url, Nothing, done)
    }

    pub fn post<T, B>(
        &mut self, url: &str, body: B,
        done: Callback<Result<ResHql<T>, Error>>
    ) -> FetchTask
    where for<'de> T: Deserialize<'de> + 'static + std::fmt::Debug, B: Serialize {
        let body = Json(&body); // 传进来 json!({"key": "value"})
        self.builder("POST", url, body, done)
    }

    pub fn put<T, B>(
        &mut self, url: &str, body: B,
        done: Callback<Result<ResHql<T>, Error>>
    ) -> FetchTask
        where for<'de> T: Deserialize<'de> + 'static + std::fmt::Debug, B: Serialize {
        let body = Json(&body); // 传进来 json!({"key": "value"})
        self.builder("PUT", url, body, done)
    }
}
```

3）再封一层 service

```rust
type _Result = Callback<Result<ResHql<AppConfig>, Error>>; // 返回类型太长，给个短名

pub struct AppService { http: Http }

impl AppService {
    pub fn new() -> Self { Self { http: Http::new() } } // new 一个 http
    pub fn load_token(&mut self, done: _Result) -> FetchTask {
        self.http.get("/app.json", done)
    }
}
```

4）组件里调用

```rust
pub struct App {
    task: Option<FetchTask>,
    service: crate::service::app::AppService
}

impl Component for App {
    fn create(_: Self::Properties, link: ComponentLink<Self>) -> Self {
        let mut me = Self { task: None, service: AppService::new() };
        
        me.task = Some( // 调用
            me.service.load_token(Callback::from(move |res: Result<ResHql<AppConfig>, Error>| {
                if let Ok(rlt) = res { set_api(rlt.api) }
            }))
        );
        
        me
    }
}
```

# 收藏

```bash
https://yew.rs/docs/zh-CN/next/ # 中文文档
https://yew.rs/docs/en/next/ # 英文的版本新
http://www.jtthink.com/course/114#2528 # rust + wasm 视频
http://www.sheshbabu.com/posts/rust-wasm-yew-single-page-application/ # 一个不错的例子
https://juejin.cn/post/6907898610084478989 # 上一行的翻译
https://github.com/jetli/awesome-yew  # 例子大全
```

