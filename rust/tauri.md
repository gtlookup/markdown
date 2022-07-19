# 开始

```bash
# 1. 下载安装 webview2
https://developer.microsoft.com/en-us/microsoft-edge/webview2/#download-section
# 2. 创建 react 项目
create-react-app 项目名
cargo install tauri-cli --version 1.0.0-beta.7 # 4. 安装脚手架，到 crates.io 上查最新版本
cargo install tauri-cli --git https://github.com/tauri-apps/tauri # 4. 如果报错，可从git上安装
cargo tauri init         # 5. 创建rust项目
		 # 设置打包后生成的路径 ../build，因为 react 默认打包也是生成 ../build
         Where are your web assets (HTML/CSS/JS) located...
         What is the url of your dev server?: http://localhost:3000 # react 默认启动
yarn add @tauri-apps/api # 6. 前端安装调用 rust 的 api
cargo tauri info         # 7. 查看环境还缺什么
cnpm start               # 8. 启动 react
cargo tauri dev          # 9. 启动 exe

######## 其它命令 ##########
cargo tauri -h # 帮助
            -V # 查看版本
```

# 打包

```bash
cargo tauri build # 默认 --release，可指定 --debug
# 此时要下载
# https://github.com/wixtoolset/wix3/releases/download/wix3112rtm/wix311-binaries.zip
# 这个工具，可能会等好久。可以先下载好后解压到 src-tauri\WixTools 下
# 最后重新 build，成功后会在
# src-tauri\target\release\bundle\msi 下会生成一个 .msi 安装文件
# 安装后启动
```

```toml
# 发布版打包性能优化（Cargo.toml）
[profile.release]
panic = "abort"
codegen-units = 1
lto = true
incremental = false
opt-level = "s"
```

# tauri.conf.json

```json
"build": {
    "distDir": "../build",                 // 前端打包后生成的目录
    "devPath": "http://localhost:3000",    // npm start 的 url
    "beforeDevCommand": "npm start",       // cargo tauri dev 之前先 npm start
    "beforeBuildCommand": "npm run build", // cargo tarui build 之前先 npm run build
    "withGlobalTauri": true                // 支持 window.__TAURI__.invoke
}
```

# command

用于前端调用 rust

## 1. 传参 + 返回值

> rust 端

```rust
#[tauri::command] // 1. 方法上添加 command
fn cmd1(s: &str) -> String {
    // 返回可以是任何类型，只要实现了 Serde::Serialize
    format!("{} cmd1", s).into()
}
// 2. 添加 invoke_handler
tauri::Builder::default().invoke_handler(tauri::generate_handler!(cmd1))
```

> 前端

```react
// 1. tauri.conf.json 添加
"build": { "withGlobalTauri": true } // 使其支持 window.__TAURI__.invoke

// 2. 引用全局 invoke
const invoke = window.__TAURI__.invoke;
// 或者把 "withGlobalTauri": true 及上一行删掉，直接导入 api
import { invoke } from '@tauri-apps/api/tauri';
// 3. 调用
//   rust方法名  参数               返回值
invoke('cmd1', { s: 'haha' })
    .then(m => console.log(m)) // 结果：haha cmd1
	.catch(err => console.log(err))
```

## 2. async 命令

```rust
#[tauri::command]
async fn async_cmd() -> u8 {
  let f = async {
    println!("async block")
  };

  println!("async_cmd, {:?}", f.await);
  7
}
```

```react
invoke('async_cmd').then(x => console.log(x)); // 结果：7
```

# menu

```toml
# Cargo.toml
tauri = { version = "1.0.0-beta.5", features = ["api-all", "menu"] } # 添加menu
```

```rust
// main.rs
let quit = CustomMenuItem::new("quit".to_string(), "Quit");    // Quit 二级菜单
let close = CustomMenuItem::new("close".to_string(), "Close"); // Close 二级菜单
let submenu = Submenu::new("File", Menu::new().add_item(quit).add_item(close)); // File 一级菜单
let menu = Menu::new().add_submenu(submenu);

tauri::Builder::default()
      .menu(menu)          // 加入menu
      .on_menu_event(|e| { // menu 事件
        match e.menu_item_id() {
          "quit" => println!("quit"),
          "close" => println!("close"),
          _ => println!("other")
        }
      })
```

# event

## 1. emit web

后台通知前台

```react
// 前端监听
import { listen } from '@tauri-apps/api/event';

async function fn() { // 必须放在 async 函数里才能 await
    await listen('web_listen', e => console.log(e));
}
fn();
```

```rust
// 后台推送
use tauri::Manager;

#[derive(Clone, serde::Serialize)]
struct Payload { message: String }

fn main() {
    tauri::Builder::default()
        .setup(|app| {
            let w = app.get_window("main").unwrap(); // 窗口实例
            tauri::async_runtime::spawn(async move {
                loop {
                    tokio::time::sleep(tokio::time::Duration::from_secs(3)).await;
                    // 推送
                    w.emit("web_listen", Payload { message: "Tauri is awesome!".into() }).unwrap();
                }
            });
            
            Ok(())
        })...
}
```



## 2. emit rust

前台通知后台

```react
import { emit } from '@tauri-apps/api/event';

emit('rust_listen', 'haha hoho'); // 推送给后台
```

```rust
use tauri::Manager;

fn main() {
    tauri::Builder::default()
        .setup(|app| {
            let id = app.listen_global("rust_listen", |e| {
                println!("app.listen_global {:?}", e.payload());
            });
            app.unlisten(id); // 只接收一次
            Ok(())
        })...
}
```



# 收藏

https://tauri.studio/   # 官方文档
https://www.bilibili.com/video/BV1tf4y1b7m3?from=search&seid=6528057198387086472&spm_id_from=333.337.0.0  24:25 # 英文视频

