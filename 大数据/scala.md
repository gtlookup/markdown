# 概述

## 1. 为什么要学 scala

主要用于 spark（大数据计算） 框架，spark 也是用 scala 写的。kafka 由 java 和 scala 编写

## 2. 编译原理

- java：（先编译再解释）.java源文件    -->  编译器（javac）-->  .class字节码文件   -->   jvm（不同平台）-->   机器指令
- scala： (先编译再解释)  .scala源文件   -->  编译器(scalac)   -->  .class字节码文件    -->   jvm（不同平台）-->   机器指令

# 环境搭建

## 1. scala 安装

1. 安装 jdk8+

2. 下载 scala.zip 解压后在环境变量的 path 里添加（D:\Program Files\scala-2.13.3\bin）

3. 命令行里 scala -version，显示出版本代表安装成功

4. 在 cmd 中简单写写代码

   ```bash
   # 进入代码环境
   scala
   # 进入后写代码
   scala> var a = 10
   # 结果
   var a: Int = 10
   ```

   

## 2. idea 安装 scala 插件

setting -> plugins 

在线安装：-> marketplace 搜 scala，点安装

离线安装：-> marketplace 右边的齿轮 -> install plugin from disk...

​                        到 idea 官网  -> tools -> scala -> versions  -> 找到和idea对应的版本下载

创建 maven 工程 -> 右键工程 -> add frameworks support -> 勾上 scala -> create 按钮 -> 选scala -> ok

==注：每创建一个 module 都要 add frameworks support 一次==



https://www.bilibili.com/video/BV1ZK411J7Qz?p=13