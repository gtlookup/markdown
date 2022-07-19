老油条说：actix-web 适用 crud，其它的用 warp

# actix_web::App

带路由宏 `#get("/xx")` 的要放到 service 里，不一带的要放到 route 里。

```rust
#[get("/user")]
pub async fn index() -> impl Responder {"/user/index"} // 带宏的放 service 里
pub async fn form() -> impl Responder {"/user/form"} // 不带的放 route 里

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let init_app = || {
        App::new().service(user::index) // 带宏的放 service 里
            .route("/user/form", web::get().to(user::form)) // 不带的放 route 里
    };
    HttpServer::new(init_app).bind("127.0.0.1:8000")?.run().await
}
```

另外，struct impl 里的不能放到 service 里，会报错 `the trait bound `fn() -> impl std::future::Future {UserController::index}: HttpServiceFactory` is not satisfied`

```rust
pub struct UserController;
impl UserController {
    #[get("/user")]
    pub async fn index() -> impl Responder { "/user/index" }
    pub async fn form() -> impl Responder { "/user/form" }
}

let init_app = || {
    App::new()
    .service(UserController::index) // struce impl 里带路由宏的错误：struct is not supported in `trait`s or `impl`s
    .route("/user/form", web::get().to(UserController::form)) // OK
};
```

# 动态编译

监听项目源码，如果有改动，就自动编译；这样就不用每改完一次代码就重新运行了。

这种方式不好的一点是==没法Debug==，虽说可以在 idea 里配置 watch -x，但不建议这么做，还是在 cmd 里运行 watch -x，idea里的负责 debug

```bash
cargo install cargo-watch # 安装
cargo watch -x run # 运行，先得 cd 到项目目录
```

# 交差编译

就是在 windows 下编译，在 linux 上运行

```bash
# 1. 安装
rustup target add x86_64-unknown-linux-musl
# 2. .cargo/config 里添加
[target.x86_64-unknown-linux-musl]
linker = "rust-lld"
# 3. 编译
#	一般项目到这步都能过，但 actix-web 这步死活过不去
cargo build --release --target x86_64-unknown-linux-musl # 上linux了一般都是正式发行了，所以要加 --release，不加也行其实
# 4. 拿到 linux 上执行（先 chmod 777，否则提示没权限）
```

# 部署到阿里

```bash
# 需要注意：
#		在 bind("127.0.0.1:8000") 时候，即不能是 127.0.0.1、localhost 等
#      只能是 bind("0.0.0.0:8000")
#		0.0.0.0 表示127.0.0.1和本机ip都能访问
https://cloud.tencent.com/developer/article/1465705 # 参考
```

# 跨域

```rust
use actix_cors::Cors; // actix-cors = "0.5.4"

App::new()
    .wrap(Cors::default()
        .allowed_origin("http://localhost:8081") // 允许访问的 url
        .allowed_methods(vec!["GET", "POST"])    // 允许访问方式
        .allowed_headers(vec![header::AUTHORIZATION, header::ACCEPT])
        .allowed_header(header::CONTENT_TYPE)
        .max_age(None)
)
// 或者
.wrap(Cors::permissive()) // 各种允许，适用于快速开发，不建议生产环境
```

# auth 认证

```bash
# 其中 actix_web::middleware::identity 已经在新版本的 actix_identity 里了
https://blog.csdn.net/weixin_34364071/article/details/88722231 # 中文
https://gill.net.in/posts/auth-microservice-rust-actix-web-diesel-complete-tutorial-part-1/#read-more # 英文原文
```

> 根据 w3.org：
>
> ```
> 允许用户输入用户名和密码获取其读取资源的令牌（而无需每次都使用用户名和密码）。用户一旦获得令牌，便拥有了一段时间访问特定资源的权限
> ```

## actix_identity

这个很简单，但貌似只针对后台连页面的好用，前后分离的不好用

```rust
use actix_web::*;
use actix_identity::{Identity, CookieIdentityPolicy, IdentityService};
use actix_cors::Cors;

async fn index(id: Identity) -> String {
    // access request identity
    if let Some(id) = id.identity() {
        format!("Welcome! {}", id) // 登陆了
    } else {
        "Welcome Anonymous!".to_owned() // 未登陆
    }
}

async fn login(id: Identity) -> HttpResponse {
    id.remember("User1".to_owned()); // 登陆
    HttpResponse::Ok().finish()
}

async fn logout(id: Identity) -> HttpResponse {
    id.forget();                     // 登出
    HttpResponse::Ok().finish()
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| App::new()
        .wrap(Cors::permissive())
        .wrap(IdentityService::new(CookieIdentityPolicy::new(&[0; 32])
            .name("auth-cookie") // 存 token 的 key
            .max_age(86400) // cookie 有效期1天
            .secure(false))) // true 表示只适用于 https
        .route("/login", web::get().to(login))    // 1. 浏览器先访问login后会发现 auth-cookie
        .route("/index", web::get().to(index))    // 2. 再访问 index 也会有 auth-cookie
        .route("/logout", web::get().to(logout))) // 3. logout 后再访问 index，auth-cookie 就空了
        .bind("0.0.0.0:8080")?.run().await        // 说明：只有页面访问可以，ajax 访问就不行
}
```

# 拦截token有效性

参考 actix-web 文档`中间件`一节

```rust
use actix_web::dev::{Transform, Service, ServiceRequest, ServiceResponse};
use actix_web::{Error, error};
use async_graphql::futures_util::future::{Ready, ok, Future};
use std::pin::Pin;
use std::task::{Context, Poll};
use actix_web::http::HeaderValue;

// There are two steps in middleware processing.
// 1. Middleware initialization, middleware factory gets called with
//    next service in chain as parameter.
// 2. Middleware's call method gets called with normal request.
pub struct AuthFilter;

// Middleware factory is `Transform` trait from actix-service crate
// `S` - type of the next service
// `B` - type of response's body
impl<S, B> Transform<S> for AuthFilter
    where
        S: Service<Request = ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
        S::Future: 'static,
        B: 'static,
{
    type Request = ServiceRequest;
    type Response = ServiceResponse<B>;
    type Error = Error;
    type Transform = AuthHiMiddleware<S>;
    type InitError = ();
    type Future = Ready<Result<Self::Transform, Self::InitError>>;

    fn new_transform(&self, service: S) -> Self::Future {
        ok(AuthHiMiddleware { service })
    }
}

pub struct AuthHiMiddleware<S> { service: S }

impl<S, B> Service for AuthHiMiddleware<S>
    where
        S: Service<Request = ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
        S::Future: 'static,
        B: 'static,
{
    type Request = ServiceRequest;
    type Response = ServiceResponse<B>;
    type Error = Error;
    type Future = Pin<Box<dyn Future<Output = Result<Self::Response, Self::Error>>>>;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.service.poll_ready(cx)
    }
	// 从这往上都是拷贝过来的，call 方法是关键
    fn call(&mut self, req: ServiceRequest) -> Self::Future {
        let is_login = req.path().eq("/login"); // 判断请求是否允许不带 token，一般 /login 允许不带
        let token = req.headers() // 从 header 里获取 token
            .get("Authorization")
            .unwrap_or(&HeaderValue::from_str("").unwrap())
            .to_str().unwrap_or("").to_string();
        let fut = self.service.call(req); // 继续请求处理，但 await 才往下走

        Box::pin(async move {
            // 是 /login 或带了 token 并且 token 有效
            if is_login || !token.is_empty() && valid_token(&token).is_some() {
                Ok(fut.await?) // 继续请求处理
            } else {
                Err(error::ErrorUnauthorized("Unauthorized")) // 否则报没有访问权限，请重新登陆
            }
        })
    }
}
```

```rust
App::new().wrap(server::utils::token::AuthFilter) // 然后在 main 方法里注册中间件
// 客户端只要在 header 里加上 .header("Authorization", "xxx token xx") 就行
```



# 收藏

```bash
https://actix.rs/ # 官网
https://actix-web.budshome.com/ # 中文教程
https://www.bilibili.com/video/BV1zK4y1j7gE?from=search&seid=16778758797706032836 # B站视频教程
https://github.com/dslchd/actix-web3-CN-doc # actix-web3.0 文档
```

