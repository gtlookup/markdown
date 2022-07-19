# 1. 环境搭建

1.下载安装nodejs,因为新版nodejs自带npm

2.安装angular-cli

  npm install -g @angular/cli

3.创建项目

  ng new 项目名

4.启动服务

  cd 项目文件夹

  ng server

5.打开网页

  http://localhost:4200/

**停止服务**

在命令行里ctrl+c然后y

**安装typescript**

npm install -g typescript typings

**编译**

1.ng build //生成的dist里非压缩文件

  ng build --prod //生成的压缩后的文件

2.把index.html里的

```html
 <base href="/">改成<base href="./">
```

  就可以直接运行了

3.双击打开也好,还是iis/apache/tomcat都可以

**升级npm**

npm -g install npm@5.3.0 (可以根据已发布的版本随意升级或降级)

**升级ng**

npm uninstall -g angular-cli    --旧版卸载

npm uninstall -g @angular/cli  --新版卸载



npm cache clean --清除缓存

npm cache verify --5.3以上版本

npm install -g @angular/cli



> **当5.4版本一安装库就出现**
>
> **Please try running this command again as root/Administrator**
>
> **EPERM: operation not permitted, unlink '......\node_modules\fsevents\node_modules\abbrev\package.json'**
>
> **错误的解决办法**

用 npm -g install npm@5.3.0 倒回到5.3.0版本



**@NgModule**是一个装饰器函数，它接收一个用来描述模块属性的元数据对象。其中最重要的属性是：

- declarations - 声明本模块中拥有的视图类。Angular有三种视图类：组件、指令和管道。

- exports - declarations 的子集，可用于其它模块的组件模板。

- imports - 将js引入的素材模块或者组件模块等，进行angular模块化引用。
- providers - 服务的创建者，并加入到全局服务列表中，可用于应用任何部分。

- bootstrap - 指定应用的主视图(称为根组件)，它是所有其它视图的宿主。只有根模块才能设置bootstrap属性。



**让别的机器通过IP访问**

1.在package.json里加上"start": "ng serve --host 0.0.0.0",

2.npm start启动

3.浏览器上访问http://xxx.xxx.xxx.xxx:4200



# 2. 数据绑定

**在HTML标签中显示组件值**

```html
<div>{{value}}</div>
```

**把组件值赋给dom属性，[]：表示数据从Model流向View**

```html
<img [src]="xxx" />
```

**事件绑定，()：表示数据从View流向Model**

```html
<button (click)="xxx()" />
```

**双向绑定**

```typescript
// 1.在.module.ts里导入FormsModule
import { FormsModule } from '@angular/forms';
// 2.还在这个文件里@NgModule.imports里添加
imports: [xxx,FormsModule]
// 3.就可以用双向绑定了
<input type="text" [(ngModel)]="title" />
{{title}} //title是组件的成员变量
```

**往按钮事件里传参数**

```html
<input type="text" #ipt />   <!-- #ipt是对入力框的引用 -->
<button (click)="clk(ipt.value)">button</button>
```

**绑定函有 html 元素的字符**

```typescript
private vv = 'aaa<br>bbb';
{{vv}} // <br>会直接显示出来
<span [innerHTML]="vv"></span> // 换行了
// 绑定.json里的属性
<span [innerHTML]="'home.title' | i18n"></span>
```

# 3. 组件 - component

## 3.1 创建组件

1.cd到项目文件夹

2.ng generate component 组件名

  ng g c 组件名 //简写

　--inline-template --inline-style　//把html和css放一个文件里

  -it -is //简写

## 3.2 引用组件

1.找到被引用的.component.ts里的selector属性值

2.在主引用的.html里加上<属性值></属性值>

**想要==import==必须==export==**

## 3.3 动态添加组件

```typescript
// 1.在要加载组件的组件中
import {
    ViewChild, ViewContainerRef, ComponentFactoryResolver
} from '@angular/core';
//在类内方法外 成员变量位置
//tab为前台容器 如：<div #tab></div>
@ViewChild('tab', {read: ViewContainerRef}) tab: ViewContainerRef;
//cfr工厂实例
constructor(private cfr: ComponentFactoryResolver) { }
//动态加载组件的事件
menuClick() {
    const c = this.cfr.resolveComponentFactory(UserComponent);
    this.tab.createComponent(c);
}
// 2.声明要动态加载的组件
import { UserComponent } from './main/user/user.component';
//在@NgModule里
entryComponents: [UserComponent]
 
// 把类型当参数传递
public x(typ: any) {
  const c = this.cfr.resolveComponentFactory(typ);
  const o: ComponentRef<typeof typ> = this.temp.createComponent(c);
}
//调用
x(RoleComponent)
```

## 3.4 通信组件

父组件向子组件传 @Input

子组件通知父组件数据已经处理完 @Output、EventEmitter

```typescript
// 1.子组件
import { Component, Output, EventEmitter } from '@angular/core';
@Output() finishDetail = new EventEmitter<参数类型>();
//触发事件通知父组件
this.finishDetail.emit(参数);
// 2.父组件
<app-detail (finishDetail)="search($event)" #detail></app-detail>
search(参数) {...}
// 3.注意 坑
//   当子组件的Module里面配置了Routing时,Input和Output是不起作用的
```

## 3.5 EventEmitter

```typescript
let numberEmitter: EventEmitter<number> = new EventEmitter<number>();
numberEmitter.subscribe((value: number) => console.log(value));
numberEmitter.emit(10); // 输出：10
```

## 3.6 离开页面事件

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
export class xxxComponent implements OnInit, OnDestroy {
    ngOnDestroy() {
        $('body').parent().css('overflow-y', 'auto');
    }
}
```

# 4. 指令 - directive

```typescript
import { Directive, HostBinding, Input, HostListener, Attribute } from '@angular/core';
 
@Directive({selector: '[test]'})  // 一定要[]
export class TestDirective {
  @HostBinding()   // 绑定调用指令者的属性
  innerText = 'Hello, Everyone!';
 
  // 必须等于selector的值
  @Input() test: string;
  // 同一个HostBinding属性只能写一次
  // 当innerText和innerHTML同时出现时 谁在下面谁会被执行
  @HostBinding() get innerHTML() {
    return `str: ${this.test}`;
  }
 
  // 指令事件
  @HostListener('click', ['$e'])
  clk(e) {
    this.test = 'Clicked!'; // str: Clicked!
  }
 
  // 获取dom的属性
  // <div [test]="'heihei'" a="aaaaa"></div>
  constructor(@Attribute('a') public aa: string) {
    console.log(aa);   // aaaaa
  }
}
// 在module里导入
import { TestDirective } from './test.directive'
@NgModule({
  imports: [...],
  declarations: [xxxComponent, TestDirective]
})
// 调用
<div test></div>
// "'双引套单引号表明这里不是变量'"
<div [test]="'heihei'"></div>
 
结构指令要加* 如：
<div *test></div>
```



