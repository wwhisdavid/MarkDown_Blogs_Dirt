# Quartz 2D学习笔记

去年的时候学习了一发Quartz Core，然而除了写几个demo也没有用上。现在想想都忘得差不多了，最近要开始写写博客记录自己的知识体系，就从捡起Quartz2D开始吧。

## 目录
* 什么的Quartz 2D？
* 使用姿势。
* 基本绘图案例。
* UIView的生命周期。
* 绘图进阶。
* 上下文栈。
* 矩阵变换。

## 1.什么的Quartz2D？

### 1.1 官方文档概念：
* Quartz 2D是一个支持iOS和MacOS的二维绘图引擎（C语音API）。
* Quartz 2D在效率上可以充分地利用图像硬件。
* 我们可以使用其进行基于路径的绘制、可变透明度绘制、遮影绘制、平滑渲染、色彩管理等。
* 在iOS系统，Quartz 2D和所有图像渲染相关的技术一起发挥着作用。

### 1.2 几个重要概念

* The Page --> 页面(画布)


	Quartz 2D使用画家的成像模型。在画家的模型中,每个连续的绘图操作一层“油漆”输出到“画布”,通常被称为一个页面。可以修改页面上的油漆覆盖更多的油漆通过额外的绘图操作。一个对象在页面上绘制不能，修改非通过覆盖更多的油漆。这个模型提供强大的原语允许构建非常复杂的图像从小画起。
	
	
	简单的说，Quartz 2D的绘画模型是对实际绘画的抽象。通过不断往画板上绘制、覆盖来达到需要的结果。所以 ，绘制的顺序非常重要。而画布有可能是任何载体。
	
	
	下图描述了这一模型，我们可见，绘图顺序影响了最终的结果。
	![图1.1](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图1.1.png?raw=true)
	
	
* Drawing Destinations: The Graphics Context --> 画布的抽象：图形上下文

	图形上下文(CGContextRef)是一个不透明的数据类型(下文介绍这个概念)，其封装了Quartz 2D使用来画图像到输出设备的信息。在一个图形上下文的信息包括图形的绘图参数和于特定设备的表示页面上的元素。所有Quartz 2D涉及对象在上下文中。我们可以理解为他是你绘制的图像的数据、步骤的存放地，类似一种组件的概念，让程序员不需要考虑设备的兼容性，体现了一种封装的思想。
	
	
	从下图可以看出，绘画元素封装在上下文中，供不同设备取用。	
	![图1.2](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图1.2.png?raw=true)
	
* Quartz 2D Opaque Data Types --> 不透明数据类型

	所谓Opaque，即不透明的意思。个人认为表达了是一种封装的意思，即不为外部可见。Quartz 2D中的各种上下文都被称为不透明的。不透明数据类型指代的是那些上下文元数据类型，包括CGPathRef(路径)、CGImageRef(图片)、CGShadingRef(遮影)等，是绘图的元数据引用，可以看做绘图最基本的上下文元素。
	
	
	下图为不透明数据类型的演示，即各种类型图案都是由不透明数据类型作为元数据构成。
	![图1.3](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图1.3.png?raw=true)
	
* Graphics States --> 图形状态

	Quartz根据当前图形状态参数修改绘图操作的结果。简单来说，图形状态就是图形参数，状态决定了渲染结果。可以将其看做是图形上下文的某一时刻属性的集合。
	
	图形的图形上下文包含堆栈状态。当Quartz创建一个图形上下文,此时栈是空的。当你保存图形状态时, Quartz将当前图形状态的副本压入堆栈。当你恢复图形状态时, Quartz弹出图形状态堆栈的顶部。出现状态成为当前图形状态。
	
	要保存当前的图形状态,使用函数`CGContextSaveGState`来push当前图形状态的副本入堆栈。恢复先前保存的图形状态,使用功能`CGContextRestoreGState`替换当前的图形状态的图形状态堆栈的顶部。
	
	`注意,并不是当前的绘图环境的所有方面是图形的元素状态。`详细被作为状态的参数见下表。
	
	Params  | 。。。
	------------- | -------------
	Current transformation matrix (CTM)  | Content Cell
	Clipping area  | Content Cell
	Line: width, join, cap, dash, miter limit  | Content Cell
	Accuracy of curve estimation (flatness)  | Content Cell
	Anti-aliasing setting  | Content Cell
	Color: fill and stroke settings  | Content Cell
	Alpha value (transparency)  | Content Cell
	Rendering intent  | Content Cell
	Color space: fill and stroke settings  | Content Cell
	Text: font, font size, character spacing, text drawing mode  | Content Cell
	Blend mode  | Content Cell
	
* Quartz 2D Coordinate Systems --> 坐标系
	
	Quartz 2D的坐标系就是平常的几何坐标。坐标系的确定需要做到与设备无关，否则不同设备的像素要求可能会使图像失真。Quartz 2D在处理不同坐标体系和不同设备兼容时，使用了矩阵变换。
	
	下图为Quartz 2D中的坐标系。
	![图1.4](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图1.4.png?raw=true)
	
	需要注意，UIView使用的坐标系与Quartz不同，其以左上角为原点。
	
	
* Memory Management: Object Ownership --> 内存管理
	
	Quartz使用的是Core Foundation的内存管理模型，需要手动释放引用计数（ARC失效）。
	
	注意：如果你调用的函数有`copy`或`create`字眼，就表示你持有了该对象，需要手动去释放，对应的持有函数有CGColorSpaceRetain、CFRetain和释放函数CGColorSpaceRelease、 CFRelease。
		
