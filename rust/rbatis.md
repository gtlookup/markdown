```rust
// 不管允许空与否，字段最好都加 Option，这样 update 时，就不用给不想变的字段赋值了
#[derive(Debug, Clone, CRUDTable, Serialize, Deserialize)]
pub struct SysUser {
    pub id: Option<u32>,
    pub account: Option<String>,
    pub name: Option<String>,
    pub is_delete: Option<u8>, // 默认的 CRUDTable，字段名要和表里一样
    pub update_time: Option<NaiveDateTime>
}

// update 是想更哪个字段就给哪个字段值，其它 None 就行
let mut ob = SysUser { id: Some(1), version: Some(100), is_delete: None, create_by: None ... };
crate::db.update_by_id("", &mut ob).await.unwrap();
```



# 收藏

```bash
https://rbatis.github.io/rbatis.io/#/ # 官方中文文档
```

