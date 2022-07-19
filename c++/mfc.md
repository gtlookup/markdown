# 添加窗口

1）添加窗口资源，如资源id：`IDD_WIN_DLG`

2）右键窗口->添加类（比如叫`CDlg`）

3）在主对话框里添加成员

```c++
public: CDlg m_dlg;
```

4）在主对话框的 `OnInitDialog` 中创建新窗口句柄

```c++
m_dlg.Create(IDD_WIN_DLG, this); // IDD_WIN_DLG 资源id
```

5）在主窗口的某个按键事件里显示窗口

```c++
m_dlg.ShowWindow(SW_SHOW);
```



http://c.biancheng.net/cpp/mfc/