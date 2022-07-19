**线性地址**

- 64位机

  - 8字节 = 64位，只有 48 位在用，16位保留位

  - 支持最大内存大小为2的48次方**（不是64次方）** = 256T

- 32位机
  - 32 位（4字节）

**指针压缩**

把线性地址用4字节来表示，不开指针压缩就是8个字节



# 1. 计算对象大小

## 1.1 对象的内存结构

- 对象头区域（Header）
  - Mark Word
  - 类型指针，基于哪个类生成的对象
  - 数组长度

- 实例数据区域

- 对齐填充区域
  - jdk 里所有的数据都是 8 字节对齐
  - 如果一个数据是 12B，则会再给其 + 4，凑成 16（必须是8的倍数）

```xml
<!-- 查看对象内存信息 -->
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

```bash
# 开启指针压缩
VM options: -XX: +UseCompressedOops
```



> 空对象占多少字节？





https://www.bilibili.com/video/BV1ek4y1r7vJ?p=3