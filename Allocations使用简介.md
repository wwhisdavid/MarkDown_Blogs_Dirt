---
title: Allocations使用简介
---

### [官方文档指导](http//:developer.apple.com/library/etc/redirect/xcode/devtools/1157/recipes/Instruments_help_articles/Articles/FindingAbandonedMemory.html)

### 简要说明

####  启动Allocations
	打开Instruments后，选择Allocations。
	进入后左上角选择模拟器和对应要检查的项目。
	点击红色圆圈开始检测。
#### 总体检测
	启动后整个界面分为上下两块。
	下部为App在当前运行的内存使用情况。上部为对应勾选指标的使用情况图示。
	指标含义说明：
	1. Graph:复选框，勾上展示对应指标图示在上部。
	2. Category:检测对象，列出了所有占用内存的持有者。主要关注All Heap Allocations(堆内存分配)和All Anonymous VM（匿名虚拟内存）选项。
	3. Persistent Bytes:常驻字节，表示现在内存的占用量。
	4. #Persistent:常驻对象数。
	5. #Transient:已释放对象数。
	6. Total Bytes:App启动分析开始总使用内存。
#### 标记检测
	总体检测功能强大，但无法细致看出每个指标的具体变化，这时就可以用标记检测。
	在右侧菜单栏选择设置按钮，下有Mark Generation按钮，点击即可进行一次标记检测。
	标记检测指标含义说明：
	1. Snapshot:快照。表示一次标记的结果，通过小箭头可以查看这次快照下的内存占用的对象分布，并可以找到对应的对象相关代码。
	2. timestamp:时间戳，快照时间。
	3. Growth:内存相对增长，表示内存相对于上次快照的增长情况。
	PS：如果要测试一个页面的内存泄漏情况，在进入界面前Mark一次，pop界面后Mark一次，观察第二次Mark的Growth。