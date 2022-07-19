# 标准库

## 命令行用法

### 1. 读取手敲内容

```rust
use std::io;
let mut s = String::new(); // new 一个字符串，用于存手敲内容
io::stdin().read_line(&mut s).expect("无法读取"); // 读取一行手敲内容，如果报错则显示 expect 里的内容
println!("{}", s) // 输入：123 显示结果：123
```

## Range

1）指定一个开始数字和一个结束数字，range 可以生成它们之间的数字（不含结束）

2）rev 方法可以反转 Range

```rust
// (1..4) 是 1 ~ 3（不含结束）
for n in (1..4).rev() { println!("{}", n) } // 结果：3 2 1

let r: RangeFrom<i32> = 0..; // RangeFrom
let r: RangeTo<i32> = ..7;   // RangeTo
```

## OsString => String

```rust
let s = OsString::from("haha").into_string().unwrap();
```

## lambda

### filter

```rust
let ar = vec![1,2,3];
let ls: Vec<&i32> = ar.iter().filter(|x| **x > 2).collect(); // collect 返回给 ls 必须指定类型
let ls: Vec<&i32> = ar.iter().filter(|x| x > &&1).collect(); // 两种写法 **x 和 &&1的区别
println!("{:?}", ls) // [3]
```

### flat_map

这个相当于 linq 里的 SelectMany

```rust
let ar = vec![vec![1,2,3], vec![11,22,33]];
let ls: Vec<&i32> = ar.iter()
	// 根据上个例子(**x > 2)写的 **y.to_string().contains("2") 会报 type `bool` cannot be dereferenced
	// 原因是 ** 解引用的是 contains("2") 的 bool 结果
	// 改成 (**y).to_string().contains("2") 就没问题了
	// 但就算 (&&7).to_string() 也能得到一个 String，所以没必要(**y)
    .flat_map(|x| x.iter().filter(|y| y.to_string().contains("2")))
    .collect();
println!("{:?}", ls) // [2, 22]
```

### any

和 C# 里的 any 一样，判断一个元素在不在集合里

### iter

获取一个**不可变**的

### iter_mut

获取一个**可变**的

### into_iter

获取**所有权**，意味着原来的 vec 就不可用了

### sum

求合。但 sum 后的值必须要落地，且类型要显示指定

```rust
let s: u64 = ls.iter().map(|x| x.size).sum();
```

### chain

连接两个集合，生成新集合

```rust
let vs = (0..=5).chain(6..10).collect::<Vec<i32>>(); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### position

获取下标

```rust
let a = [1, 2, 3];
a.iter().position(|x| x == 2) // Some(1)
```

### 去重复

```rust
let mut ar = vec![1,1,2,2,3,3,4];
ar.sort_by(|a, b| a.cmp(b)); // 一定要先排序，否则去重可能会不起作用
ar.dedup(); // 1,2,3,4。内部实际调的 dedup_by
// dedup_by 用于结构体按某个字段去重复
vs.dedup_by(|a, b| a == b); // 1,2,3,4
```

## Command

```rust
let o = Command::new("D:\\a.exe") // 启动一个进程
    .arg(&s)   // 往 a.exe 里传参数
    .status(); // status 才执行
```

## std::env

### 1. args()

取命令行参数

```rust
// 第一个为可执行文件路径+名
// 如果: a.exe haha
//		则返回：可执行文件路径+名, haha
std::env::args()
let s: String = std::env::args().skip(1).next().unwrap(); // 取上面 haha，即第2个参数（第1个为本程序名）
```

## std::fs

### read_dir

取得指定目录下的所有文件夹

```rust
if let Ok(dirs) = fs::read_dir("D:\\") { // 如果取得成功
    for ob in dirs {
        if let Ok(ob) = ob {
            if let Ok(met) = ob.metadata() { // .metadata().permissions() 返回文件夹权限
                println!("{:?}", ob.path()) // 文件夹路径
            }
        }
    }
}
```

```rust
// 上面例子可用 lambda 简写成
let ls: Vec<PathBuf> = fs::read_dir("D:\\").unwrap().flat_map(|x| x.map(|y| y.path())).collect();
```

### metadata

用来描述文件类型（.file_type)、是不是文件夹（.is_dir）、是不是文件（.is_file）、文件权限（.permissions）、

最后一次修改时间（.modified）、最后一次访问时间（.accessed）、创建时间（.created）

### 是否存在

```rust
pub fn is_exist(p: &str) -> bool {
    match fs::metadata(p).ok() { Some(_) => true, None => false }
}
```

### 创建目录

```rust
std::fs::DirBuilder::new()
	.recursive(true) // 是否递归创建，默认 false，即只创建一级目录
	.create("./a/b/c/d").unwrap();
```

### 删除

```rust
std::fs::remove_dir("tmp").unwrap()    // 删文件夹
std::fs::remove_file("a.txt").unwrap() // 删文件
```

### 写文件

```rust
// File::create 源码
pub fn create<P: AsRef<Path>>(path: P) -> io::Result<File> {
    OpenOptions::new()
//    	.read(true)       // 读模式，read 在 create 里没有
//      .append(true)     // 内容追加模式
    	.write(true)      // 写模式
    	.create(true)     // 创建模式
    	.truncate(true)   // 如果文件存在，则清空文件内容，只为 write 服务
    	.open(path.as_ref())
}
// File::open 源码
pub fn open<P: AsRef<Path>>(path: P) -> io::Result<File> {
    OpenOptions::new().read(true).open(path.as_ref())
}
```

```rust
// 修改文件内容
let mut s = String::new();
File::open(path)?.read_to_string(&mut s)?; // 1. 先以读模式取出文件内容
File::create(path)?.write(s1.as_bytes());  // 2. 再以写模式写入内容
```

### 逐行读

```rust
let f = File::open("a.txt").unwrap();
std::io::BufReader::new(f)
    .lines()
    .map(|x| x.unwrap())
    .for_each(|x| println!("{}", x));
```

### ==换行==

==切记 `\r\n`，不是 `\n`==

```rust
#[derive(Debug)]
enum ABC {A,B,C}

impl ABC {
    fn from_string(s: String) -> Self {
        if s.eq("A") { Self::A }
        else if s.eq("B") { Self::B }
        else { Self::C }
    }
}

fn main() {
    let mut s = String::new();
    /* 假设 a.txt 内容：
    	A
    	A
    	A
    */
    File::open("a.txt").unwrap().read_to_string(&mut s).unwrap();

    for line in s.split("\r\n") { //切记 `\r\n`，不是 `\n`
        // \n   结果就是：C C A
        // \r\n 结果就是：A A A
        println!("{:?}", ABC::from_string(line.to_string()))
    }
}
```

## std::process

```rust
std::process::exit(0); // 退出程序
```



## Option

有值返回自身的有：`or`、`ok_or`、`take`、`unwarp`

有值返回参数的有：`and`、`map`、`filter`

### iter

```rust
// 有就转成迭代器，没有则返回 None
Some(7).iter().next() // Some(7)

let o: Option<i32> = None;    // 不需要 mut，take 需要
o.iter().next()       // None。o 要指定类型，否则 error
```

### or

```rust
// 有返回自身，没有返回参数 Option
Some(1).or(Some(2)) // Some(1)
None.or(Some(2))    // Some(2)
```

### or_else

```rust
// 有返回自身，没有返回闭包参数执行的结果(为Option)
Some(1).or_else(|| Some(2)) // Some(1)
None.or_else(|| Some(2))    // Some(2)
```

### and

```rust
// 有就返回参数里的Option，没有返回 None
Some(1).and(Some(2)) // Some(2)

let o: Option<i32> = None;    // 不需要 mut，take 需要
o.and(Some(2))       // None。o 要指定类型，否则 error
```

### and_then

```rust
// 有就返回参数闭包执行的结果(为Option)，没有返回 None
Some(1).and_then(|x| Some(2)) // Some(2)

let o: Option<i32> = None;    // 不需要 mut，take 需要
o.and_then(|x| Some(2))       // None。o 要指定类型，否则 error
```

### filter

```rust
// 有就返回 filter 闭包里的查询结果(为Option)，没有返回 None
Some(7).filter(|a| a == &6) // None
Some(7).filter(|x| x == &7) //Some(7)
```

### unwrap

```rust
// 有值返回值，没值报错
Some(7).unwrap() // 7

let x: Option<i32> = None;
x.unwrap()       // error: process didn't exit successfully...
```

### unwrap_or_default

```rust
// 有值返回值，没值返回默认值
// 数值：0，字符：""，bool：false
Some(7).unwrap_or_default() // 7

let x: Option<i32> = None;
x.unwrap_or_default()       // 0
```

### unwrap_or

```rust
// 有值返回值，没值返回参数里的值
Some(7).unwrap_or(0) // 7
None.unwrap_or(0)    // 0
```

### unwrap_or_else

```rust
// 有值返回值，无值返回执行闭包的值
Some()7.unwrap_or_else(|| 0) // 7
None.unwrap_or_else(|| 0)    // 0
```

### ok_or

```rust
// 有则转成 Result 返回，无则返回自定义错误
Some(7).ok_or(1) // Ok(7)

let x: Option<i32> = None;
x.ok_or(1)       // Err(1)
```

### ok_or_else

```rust
// 有则转成 Result 返回，无则返回参数里闭包返回的自定义错误
Some(7).ok_or_else(|| 1) // Ok(7)

let x: Option<i32> = None;
x.ok_or_else(|| 1)       // Err(1)
```

### map

```rust
// 有值返回闭包里的结果(是Option)，无值返回 None
Some(7).map(|x| 0) // Some(7)

let x: Option<i32> = None;
x.map(|x| 7)       // None
```

### map_or

```rust
// 有则返回闭包执行结果（参数2），没有则返回参数1
// 参数1和参数2的类型必须一致
// 最后返回`值`类型，非 Option。可以当 ？号表达式取值用
Some(7).map_or("0".to_string(), |x| x.to_string()) // "7"

let x: Option<i32> = None;
x.map_or("0".to_string(), |x| x.to_string())       // "0"
```

### map_or_else

```rust
// 有值则执行闭包2，没值则执行闭包1
Some(7).map_or_else(|| 0, |x| x) // 7

let x: Option<i32> = None;
x.map_or_else(|| 0, |x| x)       // 0
```

### xor

```rust
// 自身及参数都有值时返回 None；自身或参数有一个 None 时，返回有值那个
Some(7).xor(Some(1)) // None
None.xor(Some(1))    // Some(1)
```

### take

```rust
// 有返回本身，没有返回None
Some(1).take() // Some(1)

let mut x: Option<i32> = None;
x.take()       // None。x 要 mut 且指定类型，否则 error
```

### flatten

```rust
// 消去一层 Option
Some(Some(Some(7))).flatten() // Some(Some(7))
```

### zip

```rust
let k = Some("a");
let v = Some(7);
let x: (&str, i32) = k.zip(v).unwrap(); // 结果：("a", 7)
```

### 解决多层

```rust
// 假设数据结构
#[derive(Debug, Deserialize, Clone)]
pub struct GraphqlErrorItem {
    pub extensions: Option<Value>,
    pub message: String,
    pub locations: Option<Vec<Value>>,
    pub path: Vec<String>
}

#[derive(Debug, Deserialize, Clone)]
pub struct GraphqlError { pub errors: Vec<GraphqlErrorItem> }
```

```rust
// 假设类的实现：
struct SE {save_error: Option<GraphqlError>}

impl SE {
    /// 假设有这种多层option
    fn foo(&self, s: &str) -> String {
        if let Some(o) = &self.save_error {        // save_error 为 Option
            if let Some(x) = o.errors.get(0) {     // get 结果也是 Option
                if let Some(v) = &x.extensions {   // extensions 也是 Option
                    if let Some(z) = v.get(s) {    // Value.get 也是 Option
                        return z.as_str().unwrap().to_string();
                    }
                }
            }
        }
        String::new()
    }
}
```

```rust
// 最终可改成：
fn err_content(&self, s: &str) -> String {
    self.save_error.as_ref()                    // as_ref 等价上面第7行 -> &self.save_error
        .and_then(|x| x.errors.get(0))
        .and_then(|x| x.extensions.as_ref())    // as_ref 等价上面第9行 -> &x.extensions
        .and_then(|x| x.get(s))
        .and_then(|x| x.as_str())
        .unwrap_or(&String::new()).to_string()
}
```

### 判空

```rust
struct Cls { url: Option<String> }

let c = Cls { url: Some("1".to_string()) };
// 1. as_ref 保证不丢所有权
// 2. filter 选不为空的
// 3. is_none 最后看是不是为空
let b = c.url.as_ref().filter(|x| !x.is_empty()).is_none();
// c.url: None 或 Some("".to_string()) 结果 true 表示空
```

## Result

### ok

```rust
// 正确返回 Option，错误返回 None
let x: Result<i32, &str> = Ok(7);
x.ok() // Some(7)

let x: Result<i32, &str> = Err("haha");
x.ok() // None
```

### map 、map_err

都是正确返回 Ok、错误返回 Err

不同点：map闭包里参数是 ok 的值，map_err闭包里的参数是 err 的值

```rust
let o: Result<i32, &str> = Ok(7);
let e: Result<i32, &str> = Err("a");

// x 为 Err 的 &str
println!("{:?}", o.map_err(|x| "haha")); // Ok(7)
println!("{:?}", e.map_err(|x| "haha")); // Err("haha")
// x 为 Ok 的 i32
println!("{:?}", o.map(|x| x + 1)); // Ok(8)
println!("{:?}", e.map(|x| x + 1)); // Err("a")
```

### and_then

与 map 区别在于，闭包要返回 Ok

```rust
let o1 = v.and_then(|x| Ok(x + 1));
let o2 = e.and_then(|x| Ok(x + 1));
println!("{:?}, {:?}", o1, o2) // Ok(8) Err("a")
```

### and

与 and_then 区别在于，没有闭包，参数只要Ok

```rust
let o1 = v.and(Ok(2));
let o2 = e.and(Ok(2));
println!("{:?}, {:?}", o1, o2) //  Ok(2) Err("a")
```

### map_or

```rust
pub fn get_field(&self) -> Option<FieldRule> {
    if let Some(v) = self.field_rule.as_ref() {
        return serde_json::to_string(v)
            .and_then(|x| serde_json::from_str::<FieldRule>(&x))
            .map_or(None, |x| Some(x))
    }

    None
}
```



# 外部库

## toml

```toml
# Cargo.toml 引入需要的依赖
[dependencies]
toml = "0.5.8"
serde_derive = "1.0.124"
serde = "1.0.124"
```

```toml
# 在项目根目录下创建 conf.toml
path = "C:\\Users\\gtlookup"
```

```rust
// 定义一个用于表示 conf.toml 的实体类
use serde_derive::Deserialize;

#[derive(Debug, Deserialize)]
pub struct AppToml{ path: String }
```

```rust
use std::fs::File;
use std::io::Read;

let mut s = String::new();
File::open("conf.toml").unwrap().read_to_string(&mut s).unwrap(); // 读取文件
println!("{}", s); // conf.toml 文件内容
let cnf: AppToml = toml::from_str(&s).unwrap(); // 返回值类型必须显示指定
println!("{:?}", cnf) // 结果：AppToml { path: "C:\\Users\\gtlookup" }
```

```rust
// 基本应用
use toml::Value;
// 例一
let v = "foo = 'bar'".parse::<Value>().unwrap();
println!("{:?}, {}", v, v["foo"]) // 结果：Table({"foo": String("bar")}), "bar"
// 例二
let v: Value = toml::from_str(r#"
        ip = '127.0.0.1'

        [keys]
        github = 'xxxxxxxxxxxxxxxxx'
        travis = 'yyyyyyyyyyyyyyyyy'
    "#).unwrap();
println!("{:?}", v)
```



## itertools

标准库里 lambda 的扩展增强

```toml
[dependencies]
itertools = "0.10.0"
```

### group_by

```rust
use itertools::Itertools;

let data = vec![1, 3, -2, -2, 1, 0, 1, 2];
for (key, group) in &data.into_iter().group_by(|elt| *elt >= 0) {}
```

### minmax

```rust
if let Some((min, max)) = vec![1,2,3,4,5]
    .iter()
    .minmax()
    .into_option() {

    println!("{}, {}", min, max) // 1, 5
}
```

### minmax_by

```rust
if let Some((min, max)) = vec![1,2,3,4,5]
    .iter()
    .minmax_by(|x, y| x.cmp(y))
    .into_option() {

    println!("{}, {}", min, max) // 1, 5
}
```

### sorted

```rust
use std::collections::HashMap;
use itertools::Itertools;

let mut map = HashMap::new();
map.insert(1, "a");
map.insert(2, "b");
map.insert(3, "c");

let ks = map
    .iter()
    .sorted()          // 标准库里是没有的，只有加了 itertools 库才能排序
    .map(|(k, _)| *k)  // 正常情况下，map 出来的 key 是无序的
    .collect::<Vec<u8>>();
```

## htmlescape

html 字符串编 / 解码

```toml
[dependencies]
htmlescape = "0.3.1"
```

```rust
<div>{htmlescape::decode_html("&nbsp;").unwrap()}</div>
```

## cc-rs

从 `gcc-rs` 改名过来的

A library to compile C/C++/assembly into a Rust library/application.

https://github.com/alexcrichton/cc-rs#compile-time-requirements

> 实例：

```toml
# 1. 添加依赖
[build-dependencies]
cc = "1.0.73"
```

```c++
// 2. 添加 `hello.c`（与src平级目录下）
#include <stdio.h>

int test() { return 7; }
```

```rust
// 3. 添加 build.rs（与src平级目录下）
fn main() {
    cc::Build::new().file("hello.c").compile("hello");
}
```

```rust
// 4. main.rs
#[link(name = "hello", kind = "static")]
extern "C" { pub fn test() -> i32; }
fn main() { unsafe { println!("{}", test());} } // 结果：7
```



## lazy_static

动态产生全局变量，lazy 表示变量在第一次使用时才初始化

### 1. 基本用法

```toml
[dependencies]
lazy_static = "1.4.0" # 依赖
```

```rust
#[macro_use] extern crate lazy_static;  // 用法

lazy_static! {
    static ref db: Rbatis = Rbatis::new();
}
```

### 2. 修改全局变量

```rust
 use std::sync::Mutex;

lazy_static! {
    // 需要 Mutex 加锁
    pub static ref S: Mutex<String> = Mutex::new(String::new());
    // 需求：1. 这里放一个从.toml解析出来的对象
    //	    2. 会根据不同参数从不同的.toml解析出对象
    // 一般想到的实现方法是给对象加 Mutex，其实完全没必要
    // 只需要加一个 file_name = Mutex<String>
    // 而在解析时就取该 file_name 找.toml即可
}

fn main() {
    S.lock().unwrap().push_str("haha");  // 添加内容
    println!("{:?}", S.lock().unwrap()); // haha
    
    let s = String::from("hoho");
    // 要实现 s = "haha"; s = "hoho"; 这种多次赋值
    S.lock().unwrap().clear();           // 先 clear
    S.lock().unwrap().push_str(&s);      // 再 push
    // 或
    let mut lock = S.lock().unwrap();
    *lock = s;
    println!("{:?}", S.lock().unwrap())  // hoho
}
```



## thiserror

为 `std::error::Error` 提供方便的宏 ==(写库用)==

```rust
use std::fmt::Error;

fn main() -> Result<(), Error> { // 正常情况下，返回类型的 Error 不认识下面的错误
    std::env::var("DIR_PATH")?; // 报：the trait `From<VarError>` is not implemented for `std::fmt::Error`
}
```

```rust
#[derive(Debug, thiserror::Error)]
enum ExError { // 包装一个枚举
    #[error(transparent)]
    ExVarError(#[from] std::env::VarError) // 将 ExVarError 的原指向 std::env::VarError
}

fn main() -> Result<(), ExError> { // 规换成上面包装的枚举
    std::env::var("DIR_PATH")?; // 成功捕获异常：Error: ExVarError(NotPresent)
    Ok(())
}
```

## anyhow

提供 `anyhow::Error`，可以大大简化 Rust 中错误处理 ==(写业务用)==

```rust
fn main() -> anyhow::Result<()> { // 会自动处理 Result
    std::env::var("DIR_PATH")?; // 结果：Error: environment variable not found
    Ok(())
}
```

### 返回 Err

```rust
fn main() -> anyhow::Result<(), String> {
    Err(anyhow!("haha").to_string())
}
```

### 不要尝试

> 不要尝试 `Result<(), Box< dyn std::error::Error >)` ，会直接报错并终止程序
>

## serde

https://serde.rs/ 官方教程

```toml
[dependencies]
serde = "1"
serde_json = "1"

# 一定要加 features = ["derive"] 否则：
# #[derive(Debug, serde::Deserialize)]
# pub struct Pi;
# 时会报：could not find `Deserialize` in `serde`
serde = { version = "1", features = ["derive"] }
```

### 1. json!

```rust
use serde_json::json;

let value = json!({ // json! 宏，很方便
    "code": code,
    "success": code == 200,
    "payload": {
        features[0]: features[1]
    }
});
```

### 2. to map

```rust
// 字符串转成 map
let v = r#"{
    "data": {
        "int": 1,
        "str": "string",
        "decimal": 10.99,
        "bool": false
    }
}"#;
// 这个 Value 类型可以将上面的字符串转回来
// HashMap 是不行的，如果上面json串只有一种类型的value倒是可以，因为 HashMap<key, value> 限制住了
let map: Value = serde_json::from_str(v).unwrap();
let val = &map["data"];
println!("{},{},{},{}", val["int"], val["str"], val["decimal"], val["bool"]); // 结果：1,"string",10.99,false
```

```rust
// 字符串转map
use std::collections::HashMap;
use serde_json::json;
let j = json!({"a": 1, "b": 2});            // Value
let s = serde_json::to_string(&j).unwrap(); // string
let m: HashMap<&str, u8> = serde_json::from_str(&s).unwrap(); // hashmap
```

### 3. from_str

```rust
// from_str 定义了入参和返回是同一生命周期
pub fn from_str<'a, T>(s: &'a str) -> Result<T> where T: de::Deserialize<'a>
// 注意：如果返回值里含有引用，那么
//	1. 返回值不能比入参活的长
//		1.1 根据省略生命周期原则，省略了入参和返回值的 'a
//		1.2 from_str 必须使用入参，否则就不是同一生命周期了
fn der(s: &str) -> Vec<&str> { serde_json::from_str(s).unwrap() }
der("[\"1\",\"2\",\"3\"]")
//  2. 如果没有入参，那么返回值必须是 'static，如：
fn der() -> Vec<&str> { serde_json::from_str("[\"1\",\"2\",\"3\"]").unwrap() } // NG
fn der() -> Vec<&'static str> { serde_json::from_str("[\"1\",\"2\",\"3\"]").unwrap() } // OK
```

### 4. rename

```rust
pub struct Database {
    pub id: String,
    // 由于 type 是关键字，所以不能用做结构体的字段
    #[serde(rename = "type")] // 此时 rename 一下就可以
    pub x_type: String
}
```

### 5. struct< T > 'de 限制

```rust
#[derive(Debug, serde::Deserialize)]
// 1. 需要 serde bound
// 2. 不能叫'de，必须叫其它名，因为原码里叫'de
//	  否则会报：lifetime `'de` already in scope
#[serde(bound = "T: Debug, for<'des> T: serde::Deserialize<'des>")]
pub struct PiResult<T>
	// 3. 需要加个 where，跟上面 bound 保持一致
    where T: Debug, for<'des> T: serde::Deserialize<'des> {

    #[serde(rename = "Items")]pub items: Vec<T>
}
```

### 6. DeserializeOwned

```rust
// 读取文件并返回泛型
fn t_from_file<P, T>(p: P) -> anyhow::Result<T>
    where T: serde::de::DeserializeOwned,
          P: AsRef<Path> {

    let f = File::open(p)?;
    let r = BufReader::new(f);
    Ok(serde_json::from_reader(r)?)
}
```

### 7. #[serde(skip)]

标识该成员从不序列化和反序列化

```rust
#[derive(Debug, serde::Serialize, serde::Deserialize)]
struct A { id: usize, #[serde(skip)] name: String }

let o = A { id: 1, name: "gt".to_string() };
let s = serde_json::to_string(&o)?;      // {"id":1}
let s = "{\"id\":1, \"name\": \"gt\"}";
let o = serde_json::from_str::<Ob>(s)?;  // Ob { id: 1, name: "" }
```

### 8. #[serde(skip_serializing_if)]

默认情况下，`struct` 的成员为 `Option = None` 时，`to_string` 后会显示为 `null`，而加上该修饰后就会删除该成员

```rust
#[derive(Debug, serde::Serialize, serde::Deserialize)]
struct A {
    #[serde(skip_serializing_if = "Option::is_none")]
    id: Option<i32>,
    name: Option<String>
}
let a = A {  id: None, name: None }; // 两个成员都为 None
let s = serde_json::to_string(&a)?;  // 结果：{"name":null}
```

### 9. Value & T 互转

```rust
#[derive(Debug, serde::Serialize, serde::Deserialize)]
struct User { name: String }

let val = serde_json::json!({ "name": "gt" } );
let t = serde_json::from_value::<User>(val)?; // Value to T
println!("{:?}", t); // 结果：User { name: "gt" }
let v = serde_json::to_value(t)?;             // T to Value
println!("{:?}", v); // 结果：Object({"name": String("gt")})
```

### 10. serde_repr

```rust
use serde_repr::{Serialize_repr, Deserialize_repr};

#[derive(Deserialize_repr, PartialEq, Debug)]
#[repr(u8)]
enum SmallPrime { Two = 2, Three = 3 }

let p: SmallPrime = serde_json::from_str("2")?;
```



## dotenv

用来读项目根目录下的 `.env` 文件。==注意：不能当作配置文件用，改了 .env 文件内容后，重启服务不好用，需要重新编译才行，当配置用参考上面 toml==

```toml
[dependencies]
dotenv = "0.15.0"
```

```bash
# .evn
key=haha
```

```rust
fn main() {
    dotenv::dotenv().ok();
    println!("{}", std::env::var("key").unwrap()); // haha
}
```

### dotenv_codegen

这个比上面的用法更简单

```toml
[dependencies]
dotenv_codegen = "0.15.0"
```

```rust
#[macro_use]
extern crate dotenv_codegen;

println!("{}", dotenv!("key"));  // haha
```

## validator

用于后台验证前台表单

```toml
[dependencies]
validator = { version = "0.13", features = ["derive"] }
```

```rust
use validator::{Validate, ValidationError};

#[derive(Debug, Validate)] // 实现这全责 trait
pub struct Vo {
    // required：字段必须是 Option<T>，字段 None 会返回 false
    #[validate(required, length(max = 5, min = 1))] // 不能是空""，不能大于5个字符
    acc: Option<String>, // 如果跟前端 form 绑定的话，一般不会有 required（除非None，但表单一般都是空字符串），最多就是 length = 0，所以不用加 Option
    #[validate(custom = "custom_fn")] // 字定义验证
    pwd: String
}

fn custom_fn(s: &str) -> Result<(), ValidationError> { // 自定义验证函数
    if s == "abc" {
        Err(ValidationError::new("eq abc"))
    } else { Ok(()) }
}

fn main() {
    let vo = Vo { acc: Some("".to_string()), pwd: "abc".to_string() };

    println!("{:?}", vo.validate().is_ok())
}
```

```rust
// 错误处理
#[test]
fn can_fail_custom_fn_validation() {
    #[derive(Debug, Validate)]
    struct TestStruct {
        #[validate(custom = "invalid_custom_fn")]
        val: String,
    }

    let s = TestStruct { val: String::new() };
    let res = s.validate();
    assert!(res.is_err()); // 是否是错误
    let err = res.unwrap_err(); // 返回err
    let errs = err.field_errors(); // 所有err
    assert!(errs.contains_key("val")); // 哪个字段出错
    assert_eq!(errs["val"].len(), 1);
    assert_eq!(errs["val"][0].code, "meh");
    assert_eq!(errs["val"][0].params["value"], "");
}
```

```rust
// 扩展一个验证
use validator::{Validate, HasLen};
use std::collections::HashMap;

pub trait Valid {
    fn is_valid(&self) -> bool;
    fn errors(&self) -> Option<HashMap<&str, String>>;
}

impl<T> Valid for T where T: Validate {
    fn is_valid(&self) -> bool {
        self.validate().is_ok()
    }

    fn errors(&self) -> Option<HashMap<&str, String>> {
        if self.is_valid() { return None; }

        let map = self.validate()
            .unwrap_err()
            .field_errors()
            .iter()
            .map(|(x, v)| {
                let z = v.first().unwrap();
                let s = if z.code == "length" {
                    let min = z.params["min"].as_u64().unwrap_or(0);
                    if min > z.params["value"].as_str().unwrap_or("").length() {
                        format!("输入内容不能小于{}", min)
                    } else {
                        format!("输入内容不能大于{}", z.params["max"].as_u64().unwrap())
                    }
                } else { "".to_string() };
                (*x, s)
            }).collect::<HashMap<_, _>>();

        Some(map)
    }
}
```



## rand

一个用来产生随机数的库

```toml
[dependencies]
rand = "^0.8.3"    # ^表示与这个版本兼容的最新版本
```

```rust
use rand::Rng;

// 返回一个位于本地线程空间的随机数生成器，并通过OS获得随机数种子
let mut r = rand::thread_rng();
let n = r.gen_range(1..100); // 在 1 和 100 之间生成随机数
```

```rust
use rand::{thread_rng, Rng};
use rand::distributions::Alphanumeric;

let mut rng = thread_rng();
let s: String = std::iter::repeat(())
    .map(|()| rng.sample(Alphanumeric))
    .map(char::from)
    .take(16)
    .collect();
println!("{:?}", s) // 结果：LJS5l2qsgP98Lqn2
```

## bcrypt

用于密码加密、验证的库

```toml
[dependencies]
bcrypt = "0.9"
```

### 不带salt

```rust
use bcrypt::{DEFAULT_COST, hash, verify};

// 第2个参数是 4（MIN_COST），如果用 DEFAULT_COST（12）感觉很慢
let hashed = hash("hunter2", 4).unwrap();        // 密码加密
let valid = verify("hunter2", &hashed).unwrap(); // 验证密码
```

### 带 salt

```rust
use bcrypt::{verify, hash_with_salt};

let pwd = "lookup";
let salt = "0123456789012345".as_bytes();  // 盐必须16位
let hashed = hash_with_salt(pwd, 4, salt);
let p = hashed.unwrap().to_string(); // to_string 后再传给 verify
let valid = verify(pwd, &p).unwrap();

println!("{}, {}, {}", p, p.len(), valid)
// 结果：$2y$04$KBCwKxOzLha2MR.vKhKyLOBY5ot0NWKRP34bkn5oH9/tCfh7pYfZW, 60, true
```

## chrono

一套日期时间库，默认时区敏感。

chrono::DateTime：表示带时区的日期和时间

std::time::SystemTime：表示当前系统时间，不带时区

Utc：UTC 时区。Utc::now() 获取UTC时区的当前时间

Local：系统本地时区。Local::now() 获取本地当前时间

```toml
[dependencies]
chrono = "0.4.19"
```

> 当报错：the trait `Deserialize<'_>` is not implemented for `NaiveDateTime` 时
>
> ```toml
> chrono = { version = "0.4", features = ["serde"]}
> ```

```rust
use chrono::*;
use std::time::{Duration, UNIX_EPOCH};

let s = "%Y/%m/%d %H:%M:%S";
let u = Utc::now();   // 带时区时间
let l = Local::now(); // 本地（不带时区）时间
println!("{}, {}", u.format(s).to_string(), l.format(s).to_string());
// 结果：2021/04/26 09:43:41, 2021/04/26 17:43:41
```

### 1. parse

```rust
use chrono::{Utc, DateTime};
"2021-10-20T09:56:40.312045799Z"
	.parse::<DateTime<Utc>>(); // Ok(2021-10-20T09:56:40.312045799Z)
```

### 2. 转字符串

```rust
let d = "2021-10-20T09:25:00.000000Z".parse::<DateTime<Utc>>().unwrap();
serde_json::to_string(&d).unwrap(); // 2021-10-20T09:25:00Z
```

### 3. 计算

```rust
Utc::now() + Duration::minutes(10) // 加10分钟
```

### 4. timestamp

```rust
// to timestamp
let n = Utc::now().timestamp_nanos(); // 1634733999318955800

// timestamp to string
use std::time::{Duration, UNIX_EPOCH, SystemTime};
let t: SystemTime = UNIX_EPOCH + Duration::from_nanos(u as u64);
let d = chrono::DateTime::<chrono::Utc>::from(t); // 2022-03-08 05:12:04.401155 UTC
let s = serde_json::to_string(&d)?; // "2022-03-08T05:12:04.401155Z"
```

### 5. T & Z

比如时间字符串： 2021-10-20T09:56:40.312045799Z

```rust
// T：代表后面跟着的是时间
// Z：表示 0 时区，即相关北京 8 小时

serde_json::to_string(&chrono::Utc::now().naive_local())? // 不带Z
serde_json::to_string(&chrono::Utc::now())? // 带Z
```

### 6. 时区转换

```rust
// 将 timestamp 转成带时区的 datetime
let mp = Local::now().timestamp();
let n = NaiveDateTime::from_timestamp(mp, 0);
n.format(s).to_string(); // 2021/04/26 10:06:05
let utc: DateTime<chrono::Utc> = DateTime::from_utc(n, Utc); // to Utc

// 将 timestamp 转成不带时区的本地 datetime
let d = UNIX_EPOCH + Duration::from_secs(mp as u64);
let v = DateTime::<Local>::from(d).format(s).to_string();  // 2021/04/26 18:06:05
```

## jwt

https://blog.csdn.net/wowotuo/article/details/84453785

用户信息可以不用保存在 session / redis 里，因为可以直接通过 token 解析成用户信息

==这是无状态的，想做 logout 无法在服务端废掉 token，只能依靠 redis 另想办法==

由三部分组成：

- 头部（header）：放 token 类型、算法类型等等
- 载荷（payload）：放用户信息的部分（不要加用户密码及真实姓名等敏感信息）
- 签证（sign）：放加密用的key

```toml
[dependencies]
jwt = {version = "7", package = "jsonwebtoken"} # 重命名写法
```

```rust
use serde::{Deserialize, Serialize};
use jwt::{decode, encode, Header, Validation, EncodingKey, DecodingKey};
use crate::entity::sys_user::SysUser;
use chrono::Utc;

#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    pub user: SysUser, // 当前登陆的用户
    exp: usize,        // token 到什么时候过期
}

/// 根据用户创建 token
/// 根据用户创建 token
pub fn create_token(mut usr: SysUser) -> String {
    usr.password = None; // 防止密码泄露
    // exp 是一个 utc timestamp，创建时在当前时间基础上往后延长多少时间
    let claim = Claims { user: usr, exp: (Utc::now().timestamp() + crate::CONFIG.token_exp) as usize };
    let secret = &crate::CONFIG.secure;

    encode(
        &Header::default(),
        &claim,
        &EncodingKey::from_secret(secret.as_bytes())
    ).unwrap_or(String::new())
}
/// 根据 token 验证，并返回用户
pub fn valid_token(token: &String) -> Option<SysUser> {
    let secret = &crate::CONFIG.secure;
    
    decode::<Claims>(
        token,
        &DecodingKey::from_secret(secret.as_bytes()),
        &Validation::default()
    ).map_or(None, |x| Some(x.claims.user))
}
```
## maplit

一个 hashmap! 宏，用着很方便

```toml
maplit = "1.0.2" # 依赖
```

```rust
let m = hashmap! { "a" => "aa" };
```

 ## plotters

画图库，支持 WASM 和 本地应用

Crossbeam

## reqwest

```toml
reqwest = { version = "0.11.6", features = ["json"]}
tokio = { version = "1", features = ["full"] }
```

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let http = reqwest::Client::new();
    let res = http.get("http://192.168.3.157:8713/agilorapi/v6/database")
        .header("Authorization", "token xxx")
        .send()
        .await?;
    println!("{:#?}", res.json::<serde_json::Value>().await?);
    
    Ok(())
}
```

## 高亮库

```toml
colored = "2.0.0" # 700w 下载
syntect = "4.6.0" # 70w  下载
```

## clap

用于编写命令行功能的库

```toml
clap = "2.33.3"
```

```rust
use clap::{Arg, App};
// 检查是否是整数
fn is_int(v: String) -> anyhow::Result<(), String> {
    match v.parse::<u32>() {
        Ok(_) => Ok(()),
        Err(x) => Err(x.to_string())
    }
}

let cmd = App::new("配置文件检查工具")
        .version("0.0.1")
        .author("gt")
        .about("检查 toml 格式的配置文件是否正确")
        .arg(Arg::with_name("lines") // 添加命令参数
            .short("l")      // 缩写 -l
            .long("lines")   // 全称 --lines
            .multiple(true)  // 是否多参数，如：-l 1 2 3
            .required_unless("range") // 如果没有 range 参数，那 lines 必填
            .help("检查指定某几行，如：2、4、4行") // -h 后的帮助内容
            .validator(is_int)  // 验证是否是正整数
            .takes_value(true)) // 参数是否要带值，如果：-l 7
        .arg(Arg::with_name("range")
            .short("r")
            .long("range")
            .required_unless("lines") // 如果没有 lines 参数，那 range 必填
            .multiple(true)
            .help("检查某一区间，如：20 到 70 行")
            .validator(is_int)    // 验。证是否是正整数
            // .max_values(2))    // 最多2个值
            .number_of_values(2)) // 限定死为2个值 
        .arg(Arg::with_name("file")
            .short("f")
            .long("file")
            .help("指定要检查的文件")
            .required(true)       // 必选参数
            .takes_value(true))
        .get_matches();

// 如果控制台输入了 -l 或 --lines 参数
if let Some(vs) = cmd.value_of("lines") {}

// 命令行> toml-check.exe -f d:/a.txt -l 1 2 3 -r 1 100
```

## num

```bash
num = "0.4.0"
```

```rust
use num::Integer;

let v = 10.div_ceil(&3); // 向下取整，结果：4
```

## flate2

一个压缩 / 解压缩的库，支持格式：deflate、zlib、gzip

```bash
flate2 = "1.0.22"
```

```rust
use flate2::Compression;
use flate2::read::GzDecoder;
use flate2::write::GzEncoder;

// 压缩
let mut e = GzEncoder::new(Vec::new(), Compression::default());
e.write_all(b"haha");
let vs = e.finish()?;

// 解压
let mut d = GzDecoder::new(vs.as_slice());
let mut s = String::new();
d.read_to_string(&mut s)?;
println!("{}", s); // 结果：haha
```

## async-trait

用于创建、实现异步 trait

```bash
async-trait = "0.1.51"
```

```rust
// 创建异步 trait 时
trait A { async fn seek(); }
// 报错：
error[E0706]: functions in traits cannot be declared `async`
  --> src\main.rs:26:5
   |
26 |     async fn seek();
   |     -----^^^^^^^^^^^
```

```rust
use async_trait::async_trait;

#[async_trait] // 加上注解就 OK 了
trait A { async fn seek(); }
```

```rust
// 实现异步 trait
struct B;
#[async_trait] // impl 上也加解决就 OK
impl A for B {
    async fn seek() {
        println!("haha")
    }
}
```

## tracing-subscriber

tokio 出的 log 库，可以不依赖于 tokio 运行时

```toml
tracing = "0.1.29"
tracing-subscriber = "0.3.1"
```

### 1. 基本应用

```rust
tracing_subscriber::fmt::init(); // 默认初始化
tracing::debug!("debug"); // debug 和 release 跑都不显示
tracing::trace!("trace"); // debug 和 release 跑都不显示
// 可以只写第2个参数
tracing::warn!(b = 1 == 2, "warn");   // 2021-11-15T12:28:03.878983Z  WARN agilor_pi: warn b=false
tracing::info!(b = 1 == 1, "info");   // 2021-11-15T12:28:03.879210Z  INFO agilor_pi: info b=true
tracing::error!(b = 1 == 2, "error"); // 2021-11-15T12:28:03.879328Z ERROR agilor_pi: error b=false

tracing::warn!("{:#?}", o) // 可以当 println! 用
```

==linux 通过 nohup 后台运行并指定个log文件就行==

### 2. log to 文件

```rust
let file = tracing_appender::rolling::hourly("./", "log");
// 注意：返回的第2个参数不能直接'_'，否则文件里不会有内容
let (file_blocking, _g) = tracing_appender::non_blocking(file);
tracing_subscriber::fmt().with_writer(file_blocking).init();
```

### 3. 控制台+文件

```toml
tracing-appender = "0.2.0"
```

```rust
use tracing_appender::non_blocking::WorkerGuard;
use tracing_subscriber::fmt;
use tracing_subscriber::layer::SubscriberExt;

pub fn init_log() -> WorkerGuard {
    let file_appender = tracing_appender::rolling::hourly("./", "log");
    let (file_writer, guard) = tracing_appender::non_blocking(file_appender);
    tracing::subscriber::set_global_default(
        fmt::Subscriber::builder()
            .with_max_level(tracing::Level::DEBUG)
            .finish()
            .with(fmt::Layer::default().with_writer(file_writer))
    ).unwrap();
    
    guard
}

// 用法
let _g = agilor_pi::init_log(); // 主线程里必须拿变量接一个 guard
```

## criterion

性能测试的库，可生成漂亮的报告。需要用到 `gnuplot`：

```bash
# 下载 gnuplot 后安装
https://sourceforge.net/projects/gnuplot/files/gnuplot/5.2.8/gp528-win64-mingw.exe/download
```

```toml
[dev-dependencies]
criterion = "0.3"

[[bench]]
name = "bench"   # 例如：$project/benches/bench.rs
harness = false
```

## num_cpus

用于取得本机 cpu 核数

```toml
num_cpus = "1.13.1"
```

```rust
let num: usize = num_cpus::get();
```

## rayon

一个轻量级的并行计算库

```toml
rayon = "1.5.1"
```

## libloading

用于和 `C` 动/静态库交互的库，例子用法参考 `db->pi.md->PIAPI`

### 1. 64调32位dll

```bash
Error: LoadLibraryExW failed

Caused by:
    %1 不是有效的 Win32 应用程序。 (os error 193)
error: process didn't exit successfully: `target\debug\piapic.exe` (exit code: 1)
# 当64位调32位时，会报上面的错
```

> **解决办法**

```
1. 添加一个交叉编译环境：rustup target add i686-pc-windows-msvc
2. cargo run --target i686-pc-windows-msvc
```

## autocxx

https://google.github.io/autocxx/ ==需要安装llvm==

## sdl2

跨平台多媒体开发库，多用于游戏、模拟器、播放器等领域

## dioxus

`rust` 版 `react`，中文教程强大，虽然 `yew` 也是这种，但 `陈天` 说yew设计的不好，到处都是 clone

## encoding_rs

一套编/解码工具

```toml
encoding_rs = "0.8.30"
```

```rust
let mut name = [0u8;512]; // 调了某dll方法返回了字符串
// b = false 表示没有错误
let (cw, _encoding, b) = encoding_rs::GBK.decode(name.as_ref());
let s = cs.as_ref(); // 最终值
```

## smol

https://zhuanlan.zhihu.com/p/137353103 # 源码分析（以及异步trait的源码）

### 1. Semaphore

```rust
fn main() {
    smol::block_on(s_main());
}
// 1. new一个static的sph
static SPH: smol::lock::Semaphore = smol::lock::Semaphore::new(3);
async fn s_main() {
    let mut vh = vec![];

    for i in 0..9 {
        // 2. acquire
        let g = SPH.acquire().await;
        vh.push(smol::spawn(async move {
            println!("{}", i);
            smol::Timer::after(std::time::Duration::from_secs(1)).await;
            drop(g); // 3. drop
        }));
    }

    for h in vh {
        h.detach();
    } // 结果：每秒打印3个
}
```

## v8

谷歌浏览器的 `v8` 引擎，可用于运行 `js` 代码

## simple_excel_writer

一个==写 `excel`==的库

```toml
simple_excel_writer = "0.2.0"
```

```rust
use simple_excel_writer::*;
    
let mut wb = Workbook::create("a.xlsx");
let mut sheet = wb.create_sheet("sheet1");
sheet.add_column(Column { width: 100.0 });
wb.write_sheet(&mut sheet, |sw| {
    sw.append_row(row!["a"]); // 添加一行一个值
    
    // 添加一条多个值
    let s = "a,b,c";
    let mut row = Row::new();
    s.split(",").for_each(|x| row.add_cell(x));
    sw.append_row(row)
});
wb.close();
```

```rust
// 如果写入内容过大，可用create_in_memory
let mut wb = Workbook::create_in_memory();
......
if let Ok(Some(buf)) = wb.close() { // 返回流后保存
    OpenOptions::new()
    .write(true).create(true).truncate(true).open(p)?
    .write(buf.as_slice())?;
}
```

## calamine

读 `excel`

## libc

https://blog.csdn.net/u012067469/article/details/105721613/  一套封装了 `C` 对系统调用的库

## piston_window

https://github.com/PistonDevelopers/piston  该游戏引擎文档

# 收藏

```bash
Amadeus # 大数据处理
https://github.com/twitter/rustcommon         # twitter 内部用的公共库
https://rust-cookbook.budshome.com/intro.html # 介绍 crates.io 里的常用库
```



