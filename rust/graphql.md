# Graphql 介绍

1）facebook 开发的一种数据查询语言，是 RESTFul API 的替代品

2）即是一种 API 查询语言，也是满足你数据查询的运行时。对你 API 的数据提供了一套易于理解的完整描述；使客户端能准确获取需要的数据，且没有冗余

3）官网：https://graphql.cn/

## 特点

1）请求需要的数据，不多不少。如：一个表中只取某几个想要的字段

2）获取多个资源只用一个请求

3）描述所有可能类型的系统。易于维护，根据需要平滑演进，添加或隐藏字段

## 与 restful 对比

1）restful：Representational State Transfer 属性状态转移。本质上就是定义用 Uri，通过 api 接口来获取资源。通用系统架构，不受语言限制

2）restful 一个接口只能返回一个资源，而 graphql 一次能获得多个资源

3）restful 用不同的 url 来区分资源，graphql 用类型来区分资源

## 参数

### 类型

1）上面说到 graphql 是用 `类型` 来区分资源，这个类型由 shema 和 query 由两部分组成

2）基本类型：String、Int、Float、Array、Bool 和 ID（代表数据库里的主键），可以在 shema 声明的时候使用

### 传参

和 js 基本一样，但需要指定参数及返回值类型，`!` 代表不为空，

```javascript
type query { data(x: Int!, y: Int): [Int] } // x 不能为空，y 可以，返回 Int 数组
```

## node

https://www.bilibili.com/video/BV1Ab411H7Yv?p=5

### 1. 示例

```javascript
// 1. cnpm init
// 2. cnpm i express express-graphql graphql -D
//      -D = --save-dev
//      -S = --save

const express = require('express');
const build = require('graphql').buildSchema;
const http = require('express-graphql').graphqlHTTP;

// 定义查询 schema（及类型）
const schema = build(`
	type User {                  # 一个复合类型
        id: Int,
        name: String,
        age: Int,
        fimaly(id: Int!): [User] # 家庭成员方法
    },
    type Query {
        hello: String,
        user: User,
		fun1(a: Int!, b: Int!): [String] # 传两个int型，返回一个字符串数组
    }
`);
// 定义查询处理器
const root = {
    hello: () => 'hello',
    user: () => {
        return {id: 1, name: 'gt', age: 27, sex: '男', fimaly({id}) {
            return [
                {id: 2, name: 'n2', age: 2, sex: '女'},
                {id: 3, name: 'n2', age: 3, sex: '女'}
            ]
        }}
    },
    fun1({a, b}) { // 参数要解构出来
        return [(a + b).toString()]
    }
};

const app = express();

app.use('/graphql', http({
    schema: schema,  // schema
    rootValue: root, // 查询处理器
    graphiql: true   // true：显示查询工具页面（开发），false：默认，页面只显示json字符串格式的查询数据（正式）
}));

app.listen(3000);
// 3. 浏览器打开 http://localhost:3000/graphql
// 4. 测试：
//		1.左边输入：{ hello }，右边输出：{"data": {"hello": "hello"}}
//		2.输入：{fun1(a: 1, b: 2)}，输出：{"fun1": ["3"]}
//		3.输入：{user {id,name,age,sex,fimaly(id: 2) {id,name}}}
//		  输出：{"data": {"user": {"id": 1,"name": "gt","age": 27,"sex": "男","fimaly": [{"id": 2,"name": "n2"},{"id": 3,"name": "n2"}]}}}
```

### 2. 客户端查询

```javascript
// 1.  开放一个 html 文件夹
app.use(express.static('public'));
app.listen(3000);
// 2. 创建 public/index.html
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script>
        function get() {
            let q = `
            query fun1($a: Int!, $b: Int!) {
                fun1(a: $a, b: $b)
            }`, vars = {a: 100, b: 200};

            fetch('/graphql', { // fetch 是浏览器自带的
                method: 'post',
                headers: {
                    'Content-Type': 'application/json', // 以 json 格式传给后台
                    'Accept': 'application/json'        // 以 json 格式接收后台的返回
                },
                body: JSON.stringify({query: q, variables: vars})
            })
            // res.json()必须带()
            .then(res => res.json()).then(res => console.log(JSON.stringify(res)));
            
            // 用 jquery 请求
            $.ajax({
                url: '/graphql',
                type: 'post',
                contentType: 'application/json',
                data: JSON.stringify({query: q, variables: vars})
            }).done(rlt => console.log(rlt));
            // 实质上请求体就是：
            // {"query":"上面的q","variables":null}
        }
    </script>
</head>
<body>
    <button onclick="get()">Button</button>
</body>
</html>
```

```javascript
// 3. 测试
//		1. 输出结果：{"data":{"fun1":["300"]}}
//		2. 输入：
let q = `
query xx($id: Int!) { # xx相当于一个方法名，可以随便起；方法肯定要有参数了，$参数名: 类型，!是否填
	hello,
	user {
        id,
        name,
        fimaly(id: $id) { id, name }
    }
}`, vars = {id: 200};
//		   输出：{"data":{"hello":"hello","user":{"id":1,"name":"gt","fimaly":[{"id":2,"name":"n2"},{"id":3,"name":"n2"}]}}}
```

### 3. 修改数据

```javascript
// 后端
const schema = build(`
    input UserInput {                           # 输入参数必须是 input
        name: String,
        age: Int,
        sex: String
    }
    type User {                                 # 查询类型用 type 开关
        name: String,
        age: Int,
        sex: String
    }
    type Query {                                # 如果没有 Query，则页面右侧doc里没有接口
        users: [User]
    }
    type Mutation {                             # 修改的 type 是 Mutation，查询的是 Query
        create(input: UserInput): User,
        update(id: ID!, input: UserInput): User # ID类型表示唯一主键
    }
`);

const db = []; // 模拟数据库

const root = { // schema 的实现
    create({input}) {
        db.push(input);
        return db[db.length - 1];
    },
    update({id, input}) {
        let o = Object.assign({}, db[id], input);
        db[id] = o;
        return o;
    },
    users: () => db
};
```

```javascript
// 前端
function req(p) {
    fetch('/graphql', {
        method: 'post',
        headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        },
        body: JSON.stringify(p)
    }).then(res => res.json()).then(res => console.log(JSON.stringify(res)));
}
function mut() {
    let q = `
        mutation req1($in: UserInput!) {
            create(input: $in) { name, age, sex } // 返回name/age/sex
        }
    `;
    req({
        query: q, // mutation 也必须用 query
        variables: {
            in: {name: 'gt2', age: 19, sex: '男'}
        }
    });
}
```

# async-graphql

## 案例一

用 yew 连 actix-web 服务的例子

### 服务端

```toml
[dependencies]
actix-web = "3"
actix-cors = "0.5.4"     # 跨域
async-graphql = "2.7"
async-graphql-actix-web = "2.7"

serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
// graphql.rs
use async_graphql::{Schema, EmptyMutation, EmptySubscription, Object};
use actix_web::{web, post};

pub struct PMsysQuery; // 查询对象

#[Object]
impl PMsysQuery {
    async fn user(&self) -> &str { "haha" } // 查询字段
}

pub type QuerySchema = Schema<PMsysQuery, EmptyMutation, EmptySubscription>; // schema

#[post("/graphql")]
pub async fn index(
    schema: web::Data<QuerySchema>,
    req: async_graphql_actix_web::Request
) -> async_graphql_actix_web::Response {
    schema.execute("{ user }").await.into()
}
```

```rust
// main.rs
use async_graphql::{Schema, EmptyMutation, EmptySubscription};
use actix_cors::Cors;
use actix_web::http::header;
use actix_web::{HttpServer, App};
use server::service::graphql::{PMsysQuery, index};

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let schema = Schema::new(PMsysQuery, EmptyMutation, EmptySubscription); // 创建 schema
    let init_app = move || {
        App::new()
            .data(schema.clone()) // 放到全局里
            .wrap(Cors::permissive()) // 允许跨域
        	.service(index)
    };
    HttpServer::new(init_app).bind("0.0.0.0:8080")?.run().await
}
```

### 客户端

```toml
[dependencies]
yew = "0.17.4"
yew-router = "0.14.0"

wasm-bindgen = "0.2.71" # 与js交互
web-sys = "0.3.48"
log = "0.4"
wasm-logger = "0.2.0"
anyhow = "1"
serde = "1"
serde_json = "1"
```

```rust
use serde_json::json; // json!

let s = json!({"query": "{ user }", "variables": null});
let req = Request::post("http://localhost:8080/graphql")
    .header("Content-Type", "application/json")
    .body(Json(&s)).unwrap();
let cb = self.link.callback(|res: FetchResult<String>| {
    let dt: String = res.into_body().unwrap();
    info(dt); // 打印结果：{\"data\":{\"user\":\"haha\"}}
});
self.task = Some(FetchService::fetch(req, cb).expect("req error"));
```

```rust
// 传给 body 内容如果复杂的话，可以用 format! 宏
let acc = "admin";
let pwd = "123456";
let q = "{ login($account: String!, $password: String!) }";
let vars = format!("{{account: {}, password: {}}}", acc, pwd); // 最外面再套一层{}
let data = format!("{{query: {}, variables: {}}}", q, vars);
// 结果："{query: { login($account: String!, $password: String!) }, variables: {account: admin, password: 123456}}"
```

## 案例二

上个例子只返回一个字符串，代表前后端连上了。本例是将多个表实体类映射到 graphql 的 schema 中，供前端 yew 调用

### 后端

```rust
use async_graphql::{Schema, EmptyMutation, EmptySubscription, Object, SimpleObject, MergedObject};
use actix_web::{web, post, HttpResponse};
use crate::entity::sys_user::SysUser;

#[derive(Default)] // 好比数据库表实体类 user，要实现 Default
struct Usr { id: u32, name: String, age: u8 }
#[Object] // impl 要实现 Object
impl Usr {
    // 每一个方法名都要对应 struct 里的字段，否则前端查不到 impl 里没定的字段
    // 可以写结构体里没有的字段，如：表里是 owner_id，可以写个方法 owner_name，在这个方法里根据 owner_id 取 owner_name
    async fn id(&self) -> u32 { self.id }
    async fn name(&self) -> String { self.name.clone() }
    async fn age(&self) -> u8 { self.age }
    // 最后还要加个取表所有数据的方法，就像 select * from 表，上面的方法名对应字段名相当于 select 列1，列2
    async fn find_usr(&self, acc: String, pwd: String) -> Vec<Usr> {
        vec![Usr { id: 1, name: acc, age: 11 }, Usr { id: 2, name: pwd, age: 12 }] // 好比 select * from xx 返回 2 行数据
    }
}

#[derive(Default)] // 好比数据库实体类 dept
struct Dept { dept_id: u32, dept_name: String }
#[Object]
impl Dept {
    async fn dept_id(&self) -> u32 { self.dept_id }
    async fn dept_name(&self) -> String { self.dept_name.clone() }
    async fn find_dept(&self) -> Vec<Dept> {
        vec![
            Dept { dept_id: 1, dept_name: "dept名称1".to_string() },
            Dept { dept_id: 2, dept_name: "dept名称2".to_string() }
        ]
    }
}

#[derive(MergedObject, Default)] // 合并就要加 MergedObject, Default
pub struct PMsysQuery(Usr, Dept); // 将上面表实体类加进来

pub type QuerySchema = Schema<PMsysQuery, EmptyMutation, EmptySubscription>;

#[post("/graphql")]
pub async fn index(
    schema: web::Data<QuerySchema>,
    req: async_graphql_actix_web::Request
) -> async_graphql_actix_web::Response {
    schema.execute(req.into_inner()).await.into() // 执行查询
}
```

```rust
// main.rs
let schema = Schema::new(
    PMsysQuery::default(), // 这里要 ::default()
    EmptyMutation,
    EmptySubscription
);
let init_app = move || {
    App::new()
    .data(schema.clone()) // 加到全局
    .wrap(Cors::permissive()) // 不限制跨域访问，任何地址都能访问
    .service(index)
    .service(login)
};
```

### 前端

```rust
pub struct LoginService(Http);

impl LoginService {
    pub fn new() -> Self { Self(Http::new()) } // Http 是封装了 fetch，参考【yew.md -> 封装request】
    pub fn login(&mut self, acc: &String, pwd: &String, done: _LoginResult) -> FetchTask {
        let q = r#"
            query xx($acc: String!, $pwd: String!) {            # query 随便起个方法名，目地用来定义参数
                findUsr(acc: $acc, pwd: $pwd) { id, name, age}, # 对应上面例子第 15 行
                findDept { deptId, deptName }                   # 对应上面例子第 26 行
            }"#;
        let vrs = json!({"acc": acc, "pwd": pwd});
        let data = json!({"query": q, "variables": vrs});
        self.0.post("/graphql", data, done)
    }
}
```

## 分页

配合 `rbatis` 的分页

### 后端

```rust
use async_graphql::*;
use crate::entity::sys_user::SysUser;

#[derive(SimpleObject, Debug)] // 要 SimpleObject 特性
#[graphql(concrete(name="SysUser", params(SysUser)))] // 指定 T 可以是哪些类型
// #[graphql(concrete(name="xx", params(xx)))] 再有别的类型再这么往上加
pub struct Page<T: OutputType> { // 和 rbatis::plugin::page::Page 一样，拷过来用。因为孤独原则，不能直接用
    pub record: Vec<T>,
    pub total: u64,
    pub pages: u64,
    pub page_no: u64,
    pub page_size: u64,
    pub search_count: bool
}
// rbatis::plugin::page::Page 转换成上面的 Page
impl<T: OutputType> From<rbatis::plugin::page::Page<T>> for Page<T> {
    fn from(p: rbatis::plugin::page::Page<T>) -> Self {
        Self {
            record: p.records,
            total: p.total,
            pages: p.pages,
            page_no: p.page_no,
            page_size: p.page_size,
            search_count: p.search_count
        }
    }
}
```

```rust
#[Object]
impl SysUser {
    // ... 省略
    pub async fn find_user_list_page(&self) -> crate::entity::page::Page<SysUser> {
        Self::list_page().await.unwrap().into() // into 转换成 crate::entity::page::Page
    }
}

impl SysUser {
	// rbatis 里的取分页
    pub async fn list_page() -> rbatis::Result<rbatis::plugin::page::Page<Self>> {
        let req = PageRequest::new(1, 20);
        let wrp = crate::DB.new_wrapper().eq("is_delete", 0);
        crate::DB
            .fetch_page_by_wrapper("", &wrp, &req).await
    }
}
```

### 前端

```rust
let data = json!({"query": "{ findUserListPage { record { id, name }, total } }", "variables": Null});
self.0.post("/graphql", data, done)
```

# 取token

```rust
#[post("/graphql")]
pub async fn index(
    schema: web::Data<QuerySchema>,
    req: HttpRequest,                        // actix-web 的 request
    g_req: async_graphql_actix_web::Request  // async_graphql 的 request
) -> async_graphql_actix_web::Response {
    let token = req.headers() // 从header里取token
        .get("Authorization")
        .unwrap_or(&HeaderValue::from_str("").unwrap())
        .to_str().unwrap_or("").to_string();
    schema.execute(g_req.into_inner().data(token)).await.into() // token 放到 graphql.request 里
}
```

```rust
pub async fn update(&self, ctx: &Context<'_>, ob: InputUpdateUser) -> Result<bool> {
    let token = ctx.data::<String>().unwrap_or(&"".to_string()).to_string(); // 从 ctx 里取 token
    UserService::update(ob, token).await
}
```



# 收藏

```bash
https://async-graphql.github.io/async-graphql/zh-CN/index.html  # async-graphql
```

