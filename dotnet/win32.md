# WndProc

用于拦截系统消息和自定义消息，消息列表：https://www.cnblogs.com/idben/p/3783997.html

| 消息代码 | 消息名       | 描述                 |
| -------- | ------------ | -------------------- |
| 0x2      | WM_DESTORY   | 窗口关闭             |
| 0x82     | WM_NCDESTORY | 发生在WM_DESTORY之后 |

## 1. 禁止拖动窗口

```c#
protected override void WndProc(ref Message m)
{
    if (m.Msg == 0xA1 && m.WParam.ToInt32() == 2) return;
    base.WndProc(ref m);
}
```

