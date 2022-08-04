是一款跨平台GUI框架，下载：

https://download.qt.io/archive/qt/  # 安装包安装

https://download.qt.io/archive/online_installers/4.4/  # 在线安装

# 创建项目

Qt Widgets Application：桌面平台的图形用户界面（GUI）应用程序

Qt Console Application：控制台应用程序

Qt Quick Application：可部署的 Quick2 应用程序。是Qt支持的一套GUI开发框架，采用QML设计界面

## 基类

QMainWindow：主窗口类，具有主菜单、工具栏和状态栏，类似一般程序主窗口

QWidget：是所有可视化界面类的基类，各种界面组件都支持

QDialog：对话框类，基于对话框的界面

# Qt Creator

写Qt代码的IDE，相当于vs

| 快捷键           | 说明                                               |
| ---------------- | -------------------------------------------------- |
| F4               | .h和.cpp来回切换                                   |
| F2               | 成员变量：跳到定义处。成员函数：定义和实现来回切换 |
| ctrl + shift + r | 内容替换                                           |
| ctrl + i         | 整理选中的代码格式                                 |
| ctrl + /         | 注释、取消注释                                     |
| F1               | 显示帮助信息                                       |
| ctrl + shift + s | 保存所有修改过的文件                               |
| ctrl + f         | 查找、替换                                         |
| F3               | 查找下一个                                         |
| ctrl + b         | 编译                                               |
| F5               | 开始调试                                           |
| F10              | 单步调试，不进入                                   |
| F11              | 单步调试，进入                                     |
| F9               | 设置、取消断点                                     |

# Qt核心

## Qt核心特点

- 对标准C++进行了扩展，引入了一些新的概念和功能

- 元对象编译器（Meta-Object Compiler，MOC）是一个预处理器

- 先将Qt程序转换为标准C++程序，再由标准C++编译器进行编译

> 使用信号与槽机制，只有添加 `Q_OBJECT` 宏，moc 才能对类里的信号与槽进行预处理

> Qt 为 C++ 语言增加的特性在 Qt Core 模块里实现，由 Qt 的元对象系统实现。包括：信号与槽机制、属性系统、动态类型转换等

## 元对象系统

- QObject 类是所有使用元对象系统类的基类
- 在一个类的 private 部分声明 Q_OBJECT 宏

- MOC（元对象编译器）为每个 QObject 的子类提供必要的代码

  ```c++
  QObject* ob = new QPushButton;
  ob->metaObject()->className(); // metaObject 返回类关联的元对象，className返回QPushButton
  
  QTimer* t = new QTimer;
  // inherits：是否继承某类型
  t->inherits("QTimer");         // true
  t->inherits("QObject");        // true
  t->inherits("QAbstrctButton"); // false
  ```

## 属性系统

`Q_PROPERTY` 宏定义一个返回类型为 `type`，名称为 `name` 的属性

```c++
QLabel lbl;
lbl.setProperty("a", "haha"); // 动态添加属性
qDebug() << lbl.property("a"); // QVariant(QString, "haha")
qDebug(lbl.property("a").toString().toStdString().c_str()); // haha
```

```c++
// 遍历动态属性
const QMetaObject* meta = metaObject();

for (int i = 0; i < meta->propertyCount(); i++) {
    QMetaProperty prop = meta->property(i);
    qDebug() << prop.name() << ": " << property(prop.name());
}
```

## 附加信息

```c++
class Widget : public QWidget {
    Q_OBJECT
    Q_CLASSINFO("a", "aaa") // 附加信息
    ...
}

const QMetaObject* meta = metaObject();
// 遍历附加信息
for (int i = meta->classInfoOffset(); i < meta->classInfoCount(); i++) {
    QMetaClassInfo info = meta->classInfo(i);
    qDebug() << info.name() << ": " << info.value();
} // 结果：
// a :  aaa
// b :  bbb
```



## 信号与槽

用于绑定控件事件和响应方法

```c++
// arg1：事件发送方，arg2：信号（事件），arg3：接收方，arg4：槽（响应函数）
connect(ui->btnOK, &QPushButton::clicked, this, &Widget::on_btnOK);
// 最后一个参数：Qt::ConnectionType 表示信号（事件）与槽（响应方法）之间的关联方式
// Qt::AutoConnection（缺省值）：自动确定关联方式
// Qt::DirectConnection：信号被发送时，槽立即执行，槽函数与信号在同一线程
// Qt::QueuedConnection：事件循环回到接收者线程后执行，槽函数与信号在不同线程
// Qt::BlockingConnection：与Queued相似，信号线程会被阻塞，直到槽函数执行完毕。当槽与信号在同一线程会死锁
```

```c++
// 槽函数里获取信号发送者指针
QPushButton* btn = qobject_cast<QPushButton*>(sender());
```



# SDK

## qDebug

可以取代 std::cout

```c++
qDebug() << "haha";
```

## qobject_cast

元对象间类型转换

```c++
class A : public QWidget {}; // QWidget 内部包含 Q_OBJECT
// qobject_cast 支持的类型内部必须要有 Q_OBJECT 宏
QObject* p = new A;
QWidget* qw = qobject_cast<QWidget*>(p); // 通过 qobject_cast 转成父类指针
std::cout << (qw == NULL) << std::endl;  // 0
QLabel* lbl = qobject_cast<QLabel*>(qw); // 但转成“非本人”就会返回空
std::cout << (lbl == NULL) << std::endl; // 1
```

## QString

```C++
QString s = QString::asprintf("name: %s", "haha"); // name: haha
QString s = QString("a: %1, b: %2, c: %3").arg("1").arg("2").arg("3"); // a: 1, b: 2, c: 3
```







https://www.bilibili.com/video/BV1AX4y1w7Nt?p=10&spm_id_from=pageDriver&vd_source=e611dc7ed99505a0ef548dd66f0a11dd # v5.9

https://www.bilibili.com/video/BV1G94y1Q7h6?spm_id_from=333.337.search-card.all.click  # v6.3.1

