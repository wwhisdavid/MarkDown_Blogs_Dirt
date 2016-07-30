# Quartz 2D学习笔记

去年的时候学习了一发Quartz Core，然而除了写几个demo也没有用上。现在想想都忘得差不多了，最近要开始写写博客记录自己的知识体系，就从捡起Quartz2D开始吧。

## 目录
* 什么的Quartz 2D？
* 使用姿势。
* 基本绘图案例。
* UIView。
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
		

## 2.使用姿势
**写在前面**：所以Quartz 2D的API均要在UIView的`-(void)drawRect:(CGRect)rect;`方法中调用。这是因为Quartz中使用的上下文只能在该方法获取到，这是与UIView的生命周期相关，这个方法作为生命周期的回调来处理渲染。

### 2.1 使用最原生API（比较繁琐）

* 从画一条线管中窥豹
	
		- (void)drawSimleLine{
    		// 1.获取上下文（画作的封装）
    		CGContextRef ctx = UIGraphicsGetCurrentContext();

    		// 2.描述绘制线条
    		CGMutablePathRef path = CGPathCreateMutable();
    
    		// 3.设置起点 --> 终点 --> 渲染
    
    		// 起点（30，30）
    		CGPathMoveToPoint(path, NULL, 10, 50);
    
    		// 添加一根线到某个终点（100，100）
    		CGPathAddLineToPoint(path, NULL, 100, 100);
    
    		// 4.把路径添加到上下文
    		CGContextAddPath(ctx, path);
    
    		// 5.渲染上下文
    		CGContextStrokePath(ctx);
    
    		// 6.内存释放
    		CGPathRelease(path);
		}
		
* 稍微简洁地画一条线

		- (void)drawSimleLine2{
    		// 1.获取上下文
    		CGContextRef ctx = UIGraphicsGetCurrentContext();
    
    		// 2.描述路径
    		// 设置起点、线、终点
    		CGContextMoveToPoint(ctx, 20, 50);
    
    		CGContextAddLineToPoint(ctx, 100, 100);
    
    		// 3.渲染上下文
    		CGContextStrokePath(ctx);
		}

* 从上面的两条线，我们可以归纳出Quartz 2D画图的步骤：
		
		1.开启上下文 --> 2.画图（点、线） --> 3.渲染 --> 4.内存释放
		
* 第一章中我们提到了上下文有一个重要特性：状态，我们可以理解为就某一时点上下文图像的熟悉。在`渲染上下文之前`我们可以通过设置状态改变最终渲染的样式：

		// 颜色
    	[[UIColor redColor] setStroke];
    
    	// 线宽
    	CGContextSetLineWidth(ctx, 5);
    
    	// 设置连接样式
    	CGContextSetLineJoin(ctx, kCGLineJoinBevel);
    
    	// 设置顶角样式
    	CGContextSetLineCap(ctx, kCGLineCapRound);
    	
* 对于复杂的图像，光靠点和直线肯定是不够的，Quartz 2D支持通过控制点来绘制任意曲线：
		
		@param c   上下文
		@param cpx 控制点x
		@param cpy 控制点y
		@param x   终点x
		@param y   终点y
		void CGContextAddQuadCurveToPoint(CGContextRef __nullable c,CGFloat cpx, CGFloat cpy, CGFloat x, CGFloat y);
		
		关于控制点，可以见下图。
		
![图2.1](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图2.1.png?raw=true)


### 2.2 使用UIKit封装的API

* Apple考虑到Quartz 2D使用的繁琐，和内存管理的问题。使用了`UIBezierPath`对Quartz 2D进行了封装，API十分面向对象。同样是画一条线：

		// 1.创建路径
    	UIBezierPath *path = [UIBezierPath bezierPath];
    
    	// 2.设置起点
    	[path moveToPoint:CGPointMake(30, 50)];
    
    	// 添加一根线到某个点
    	[path addLineToPoint:CGPointMake(100, 100)];
    	// 颜色
    	[[UIColor greenColor] set];
    	// 线宽
		path.lineWidth = 10;
    	
    	// 3.绘制路径
    	[path stroke];
    	
* 画一些基本图形：
		
		// 1. 扇形
		// 半径
		CGFloat radius = rect.size.width * 0.5;
		// 圆心
    	CGPoint center = CGPointMake(radius, radius);
    	// 圆弧终点角度
    	CGFloat endA = M_PI_2;
    	// 设置圆弧路径（其中clockwise:YES表示顺时针渲染）
    	UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center radius:radius - 2 startAngle:- M_PI_2 endAngle:endA clockwise:YES];
    	// 从弧线上拉直线到圆心
    	[path addLineToPoint:center];
    	// 闭合扇形
    	[path closePath];
    	// 以填充方式渲染
    	[path fill];
    	
    	// 2. 圆角矩形
    	// cornerRadius表示圆角半径
    	UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:CGRectMake(0, 10, 100, 100) cornerRadius:30];
    	[path stroke];
    	
* 当然，同样可以画复杂曲线（有两个极点）：


    	UIBezierPath *path = [UIBezierPath bezierPath];
    	// 设定起点
    	[path moveToPoint:CGPointMake(0,0)];
    	// 添加两个控制点
    	[path addCurveToPoint:CGPointMake(100, 100)
             controlPoint1:CGPointMake(50, 0)   
             controlPoint2:CGPointMake(0, 50)];            
    	// 添加终点
    	[path addLineToPoint:CGPointMake(0, 100)];
    	// 闭合
    	[path closePath];
    	
    	大概结果见下图。
		
![图2.2](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图2.2.png?raw=true)

## 3.基本绘图案例

### 3.1 进度圈绘制

* 这里举一个使用UIBezierPath绘制一个类似ProgressHUB的进度圈。这里的思路来自于上文绘制圆弧，在`0°--> 360°`范围内通过调用UIView的重绘回调（关于UIView的各种绘制时机，下一章介绍），不断在圆心向圆弧画线，且以fill方式渲染。具体代码如下：

		- (void)drawRect:(CGRect)rect
		{
    		CGFloat radius = rect.size.width * 0.5;
    		CGPoint center = CGPointMake(radius, radius);
    		// 此处的_progress是一个CGFloat变量（0，1），控制当前的进度
    		CGFloat endA = - M_PI_2 + _progress * M_PI * 2;
    
    		UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center radius:radius - 2 startAngle:-M_PI_2 endAngle:endA clockwise:YES];
    		[path addLineToPoint:center];
    		[path closePath];
    		[path fill];
		}

		// 上述代码仅仅描述了一次扇形的微分绘制，外部通过设置_progress属性来重绘，代码如下：
		- (void)setProgress:(CGFloat)progress
		{
    		_progress = progress;
    		// 系统重绘
    		[self setNeedsDisplay];
		}

![图3.1](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图3.1.png?raw=true)
		
### 3.2 饼状图

* 本节说明如何使用UIBezierPath绘制饼状图。绘制饼状图的思路与绘制进度圈的思路基本一致，根据数据比值在360°范围内绘制各自扇形形成饼状图。代码如下：

		- (void)drawRect:(CGRect)rect{
			// 设置饼状数据，分别占10%、20%、70%
    		NSArray *arr = [NSArray arrayWithObjects:@10, @20, @70, nil];
    		CGFloat radius = rect.size.width * 0.5;
    		CGPoint center = CGPointMake(radius, radius);
    
    		__block CGFloat startA = 0;
    		__block CGFloat angle = 0;
    		__block CGFloat endA = 0;
    		// 依次画数组中的扇形
    		for (int i = 0; i < arr.count; i++) {
        		startA = endA;
        		angle = [arr[i] integerValue] / 100.0 * M_PI * 2;
        		endA = startA + angle;
        
        		UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center radius:radius startAngle:startA endAngle:endA clockwise:YES];
        
        		[path addLineToPoint:center];
        		// 设置颜色这里选择设置随机颜色，代码就不贴了。
        		[[self colorRandom] set];
        		[path fill];
    		}
		}

![图3.2](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图3.2.png?raw=true)
### 3.3 柱状图

* 本节说明如何使用UIBezierPath绘制柱状图，通过排布矩形实现：

		- (void)drawRect:(CGRect)rect
		{
    		NSArray *arr = @[@10, @20, @30, @15];

    		CGFloat x = 0;
    		CGFloat y = 0;
    		CGFloat w = 0;
    		CGFloat h = 0;
    
    		for (int i = 0; i < arr.count; i++) {
        		w = rect.size.width / (2 * arr.count - 1);
        		x = 2 * w * i;
        		h = [arr[i] floatValue] / 100.0 * rect.size.height;
        		y = rect.size.height - h;
        
        		UIBezierPath *path = [UIBezierPath bezierPathWithRect:CGRectMake(x, y, w, h)];
        
        		[[self colorRandom] set];
        		[path fill];
        
        
    		}
		} 

![图3.3](https://github.com/wwhisdavid/MD_Pictures/blob/master/Quartz2D/图3.3.png?raw=true)
## 4.UIView的生命周期
这篇文章抽空来谈谈UIView。UIView是我们UI编程的基础与核心，也是Quartz 2D的渲染载体。

### 4.1 不常用属性介绍


* @property(nonatomic,getter=isExclusiveTouch) BOOL exclusiveTouch 

		// “专一触摸”，默认NO，设置为YES时，对应的View在一个时间点不会和其他的View同时响应，保证了多点触碰同时响应时的数据、逻辑安全。

* @property(nonatomic) CGAffineTransform transform
		
		// 描述视图的“仿射变换”，通过这个属性隐式改变视图的大小、旋转、位移、矩阵变形。
	[参考链接](http://www.jianshu.com/p/32ac4d71cac7)
	
* @property(nonatomic) CGFloat contentScaleFactor

		// “内容缩放因子”，其决定了视图的内容是怎样从点坐标系映射到设备的像素坐标系。例如,如果比例因子是2.0和视图frame尺寸是50 x 50,位图用于呈现内容的大小是100 x 100像素。一般情况下，不需要修改这个属性的价值。然而,如果应用程序将使用OpenGL ES,可能想要改变比例因子提升图像质量和渲染性能。
		
* @property (nonatomic) UIEdgeInsets layoutMargins

		// 在iOS 8中，可以使用layoutMargins去定义view之间的间距,该属性只对AutoLayout布局生效。
		
* @property (nonatomic) BOOL clearsContextBeforeDrawing

		// 决定是否在绘制（-drawRect:）前清除原上下文内容。
		
* @property(nonatomic, strong) UIView *maskView

		// 视图的掩饰层视图（蒙版）。iOS 8可用，该视图的alpha决定了又多少视图的内容和背景可以展示。完全或不完全的不透明像素允许底层内容展示，但完全不透明将遮挡内容。（没有使用过系统蒙层，等完全抛弃iOS7后可以试一试。）
		
* @property(nonatomic, strong) UIColor *tintColor

		// 在iOS7中，UIView新增了一个属性tintColor. 这是一个UIColor，被使用在UIView中改变应用程序的外观的。默认tintColor的值为nil，这表示它将会运用父视图层次的颜色来进行着色。部分诸如UILabel这样的控件不受tintColor控制，而子控件如果设置tintColor，则子控件的子控件的tintColor由最近的父控件传导，这样tintColor传导链就断开了。
		// 如果要给自定义控件着色且使用tintColor链条，apple提供了对应回调，需要手动在自定义控件中实现`-(void)tintColorDidChange`，并写对应的设置颜色代码。
		// 此外，tintColor支持改变图片颜色。
	[链接](http://www.cnblogs.com/wfwenchao/p/4884833.html)
	
* @property(nonatomic) UIViewTintAdjustmentMode tintAdjustmentMode

		// 当这个属性的值是UIViewTintAdjustmentModeDimmed,tintColor属性的值被修改提供一个黯淡的外观。(又是一个类似实现蒙版的属性)
		// 当这个属性值改变时(通过视图的对应值变化或改变它的父视图的一个值),系统调用-tintColorDidChange方法允许自定义渲染。
		
* @property(readonly,strong) UIView *viewForFirstBaselineLayout NS_AVAILABLE_IOS(9_0)

		// 对于多行文本视图,第一个基线是最上面一行字的视图基准。（待测试）
		// 当你给视图的做NSLayoutAttributeFirstBaseline属性约束,Auto Layout使用此视图的基线作为方法返回。如果这一视图没有基线约束,Auto Layout使用视图的顶层边缘返回。

* 约束属性，类似mansory。解决了诸如不同方向上约束编译不报错但约束失败的问题，做了约束泛型处理。

		@property(readonly, strong) NSLayoutXAxisAnchor *leadingAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutXAxisAnchor *trailingAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutXAxisAnchor *leftAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutXAxisAnchor *rightAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutYAxisAnchor *topAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutYAxisAnchor *bottomAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutDimension *widthAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutDimension *heightAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutXAxisAnchor *centerXAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutYAxisAnchor *centerYAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutYAxisAnchor *firstBaselineAnchor NS_AVAILABLE_IOS(9_0);
		@property(readonly, strong) NSLayoutYAxisAnchor *lastBaselineAnchor NS_AVAILABLE_IOS(9_0);
		
		示例代码：
			//以下代码编译不通过。
			NSLayoutConstraint *constraint =  [view1.leadingAnchor constraintEqualToAnchor:view2.topAnchor];

* @property (nullable, nonatomic, copy) NSString *restorationIdentifier NS_AVAILABLE_IOS(6_0);

		// 这个属性指出是否视图状态信息应该被保存；它也用来在恢复状态的过程中辨识视图。它的值默认nil，默认视图状态不被保存。一旦它被设置，对应的拥有的ViewController便知晓需要保存视图相关信息。
		// 这个属性只有在自定义控件中且实现了-encodeRestorableStateWithCoder:和- decodeRestorableStateWithCoder:方法来保存和解析视图信息。

### 4.2 不常用API介绍
* -(BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event;

		/**
 		*  判断一个点是否在该响应接收者（View）内
 		*
 		*  @param point 点坐标
 		*  @param event 通过什么事件进行传导
 		*
 		*  @return 是就YES
 		*/
		- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event;

* -(nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event;

		/**
 		*  递归调用-pointInside:withEvent:，找到子类中最后一个包含这个点的且响应事件的子类对象。
 		*
 		*  @param point 点坐标
 		*  @param event 事件
 		*
 		*  @return 对应的对象
 		*/
		- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event;
	
* -(CGPoint)convertPoint:(CGPoint)point toView:(nullable UIView *)view;
	
		/**
 		*  将一个点的坐标从当前View转换到目标View
 		*
 		*  @param point 当前View的点
 		*  @param view  目标View
 		*
 		*  @return 目标View上转换后的坐标
 		*/
		- (CGPoint)convertPoint:(CGPoint)point toView:(nullable UIView *)view;
		
		类似的API还有：
		- (CGPoint)convertPoint:(CGPoint)point fromView:(nullable UIView *)view;
		- (CGRect)convertRect:(CGRect)rect toView:(nullable UIView *)view;
		- (CGRect)convertRect:(CGRect)rect fromView:(nullable UIView *)view;

* -(CGSize)sizeThatFits:(CGSize)size;
		
		/**
 		*  根据传入尺寸和当前View尺寸给出合适尺寸。
 		*
 		*  @param size
 		*
 		*  @return 返回系统认为合适的尺寸
 		*/
		- (CGSize)sizeThatFits:(CGSize)size;
		
		这个API可以使用诸如在UIView子类UILabel里面，计算出文字所需要的尺寸；在UIImageView里面计算image的大小；或者是UIView内部的其他元素所占边界。
		
* -(void)sizeToFit; 

		/**
 		*  调用-sizeThatFits:，并把对应尺寸赋给UIView
 		*/
		- (void)sizeToFit;
	
* -(void)didAddSubview:(UIView *)subview

		/**
 		*  在子视图被添加的时候调用，默认没有实现。
 		*
 		*  @param subview 子视图
 		*/
		- (void)didAddSubview:(UIView *)subview;

		类似的API有：
		
		// 将要移除子视图调用
		- (void)willRemoveSubview:(UIView *)subview;
		
		// 调用者将要被加到父视图时调用
		- (void)willMoveToSuperview:(nullable UIView *)newSuperview;
		
		// 恰好被加到父视图时调用
		- (void)didMoveToSuperview;
		
		// 与当前所在window对象改变相关 
		- (void)willMoveToWindow:(nullable UIWindow *)newWindow;
		- (void)didMoveToWindow;

* -(BOOL)isDescendantOfView:(UIView *)view;
		
		/**
		 *  判断当前view是否传入View的子视图
 		 *
 		 *  @param view 判断的父视图
 		 *
 		 *  @return 是否是传入视图的子视图
 		 */
		 -(BOOL)isDescendantOfView:(UIView *)view;

* -(void)setNeedsLayout;

		/**
 		*  当想调整视图的子视图的布局在应用程序的主线程调用这个方法。这个方法会做一个标记并立刻返回。因为这个方法不强行立即更新,而是等待下一个更新周期,可以失效之前多个视图的布局并进行视图更新。这种方式加固了新布局更新在更新周期。
 		*  简言之，失效当前布局并标记，在下一个布局周期更新布局。
 		*/
		- (void)setNeedsLayout;

* -(void)layoutIfNeeded;

		/**
 		*  以当前视图为根视图，立即更新子视图布局
		*/
		- (void)layoutIfNeeded;

* -(void)layoutSubviews;

		/**
 		*  子类可以重写此方法,以进行更精确的子视图的布局。
 		*  只有在autoresizing和基于约束布局的子视图不能提供想要的行为时，应该重写此方法。
 		*  你不应该直接调用这个方法。如果你想强行更新布局，就调用setNeedsLayout方法，在下一个布局周期进行。如果你想立即更新视图的布局,调用layoutIfNeeded方法。 
 		*/
		- (void)layoutSubviews;

* -(UIView *)snapshotViewAfterScreenUpdates:(BOOL)afterUpdates;

		/**
 		*  为当前视图创建一个快照（截图）
 		*
 		*  @param afterUpdates 表示是否在所有效果应用在视图上了以后再获取快照
 		*
 		*  @return 快照
 		*/
		- (UIView *)snapshotViewAfterScreenUpdates:(BOOL)afterUpdates;

		类似API有：
		- (UIView *)resizableSnapshotViewFromRect:(CGRect)rect afterScreenUpdates:(BOOL)afterUpdates withCapInsets:(UIEdgeInsets)capInsets;
		- (BOOL)drawViewHierarchyInRect:(CGRect)rect afterScreenUpdates:(BOOL)afterUpdates;

* -(void)exerciseAmbiguityInLayout;

		/**
 		*  随机改变布局，一般用于基于约束的视图调试。
 		*/
		- (void)exerciseAmbiguityInLayout;
		 
* -(void)drawRect:(CGRect)rect;

		/**
 		*  执行重绘操作的回调。这个方法会在一个view第一次出现或当有一个事件触发使当前可见部分失效时触发。不应该主动直接调用它。如果要重新绘制，则调用setNeedsDisplay或setNeedsDisplayInRect:方法，这两个方法会异步在合适的时机回调该方法。
 		*
 		*  @param rect UIView自己的frame
 		*/
		- (void)drawRect:(CGRect)rect;

* -(void)setNeedsDisplay;

		/**
 		*  标记UIView为需要重绘，在下一个绘制循环调用对应绘制方法。
 		*  可以使用此方法或setNeedsDisplayInRect:通知系统视图的内容需要重绘。这是一个异步方法。
 		*  如果你的view是基于CAEAGLLayer对象，这个方法将不会有作用，它只适用于native drawing technologies（原生绘制）。 
 		*/
		- (void)setNeedsDisplay;

		类似方法：
		- (void)setNeedsDisplayInRect:(CGRect)invalidRect;

* -(BOOL)canBecomeFocused;

		/**
 		*  [可以看看这个，我不会描述。](http://stackoverflow.com/questions/32753109/how-do-you-set-canbecomefocused-for-a-custom-view)
 		*
 		*  @return 是否View可以被重点突出。
 		*/
		- (BOOL)canBecomeFocused; 

* 还有很多关于布局的不常用方法，这里暂不介绍，有兴趣可以去看[文档](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html)。

## 5.