https://poem.rs/zh-CN/   # 中文文档

# 解决问题

## 1. 响应 json

```rust
#[handler]
async fn index(req: &Request, body: Body) -> poem::Response {
    let s = r#"{"code":"not found","message":"database \"PI1\" not found"}"#;
    let v: serde_json::Value = serde_json::from_str(s).unwrap();
    poem::web::Json(v).into_response()
}
```

## 2. 返回错误 json

```rust
#[derive(Debug)]
pub enum HaError {
    JsonError(String),  // json型错误
    StringError(String)
}

impl poem::error::ResponseError for HaError {
    fn status(&self) -> StatusCode {
        match &self.inner {
            Self::JsonError(_) => StatusCode::BAD_REQUEST,
            _ => StatusCode::INTERNAL_SERVER_ERROR
        }
    }

    fn as_response(&self) -> Response where Self: std::error::Error + Send + Sync + 'static {
        let build = poem::Response::builder().status(self.status());
        match self {
            // IntoResponse 时判断是否是 json
            // 是的话给 header 加 application/json
            // 最后把 json 字符串给 body
            Self::JsonError(s) => build
                .header(poem::http::header::CONTENT_TYPE, "application/json")
                .body(s.to_string()),
            _ => build.body(self.to_string())
        }
    }
}
```