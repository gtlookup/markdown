# 1.画图

https://blog.csdn.net/swinfans/article/details/89393853

```bash
​```mermaid
graph 流程图方向  # TB：上到下，BT：下到上，RL：右到左，LR：左到右，TD：同TB
# A --> B：A 连线 B 带箭头
# A --- B：A 连线 B 不带箭头
# A -.-> B：带箭头虚线
# A -.- B：不带箭头虚线
id[文字] #[]代表方形
id(文字) # ()代表圆角
id((文字)) # (())代表圆
id{文字} # {}代表凌型
id>文字] # 代表右向旗帜状节点
D --> |a=1|F{F} # 双线代表两个图之间的线；|双线中间的代表线上内容|
style id height:100px,widht:100px;  # 设置节点式样
# 子图
subgraph 接入方
    id[文字]
    ...
end
# 创建一个背景红色的样式
classDef red fill:#F00;
# 设置id为A1的节点为红背景
class A1 red
```

# 2.设置 ">" 颜色

```css
blockquote {
    border-left: 3px solid #4FAB24;
    padding: 8px 10px;
    color: #777777;
}
```

# 3.红下划线

- 文件 -> 偏好设置 -> 编辑器 -> 拼写检查：不使用拼写检查

# 3. 双等号 ==== 中间红字

- 文件 -> 偏好设置 -> Markdown -> Markdown 扩展语法 -> 高亮（例：== key ==）

```css
mark {
    background-color: transparent;
	color:#F00;
	font-weight: bold;
}
```



视频教程：https://www.bilibili.com/video/BV12T4y1g7se