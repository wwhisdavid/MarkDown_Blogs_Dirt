# Core Animation学习笔记

## 目录
1. 核心动画概述
2. 核心：图层（Layer）概述
3. 图层设置
4. 图层动画


## 1. 核心动画概述

![图1.1](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图1.1.png?raw=true)

* 从上图可以看出，Core Animation处于Quartz 2D（Core Graphics）的上层，UIKit的下层。按照Apple的说法，Core Animation不是一个绘图系统，而是一个可以在硬件基础上构造内容的基础框架。这个基础框架的核心是layer对象。layer对象可以很简单地调用GPU来捕获我们的内容到位图中。

* Core Animation提供了一个通用的系统来使视图和其他可见元素动起来。但其并不是视图的替代者，相反，这种技术整合了视图并为视图动画提供更好的性能。它达到这样的功能是通过缓存视图的内容到位图，而这个位图是可以直接被GPU操作的。在一些情况下，这个缓存行为可能需要程序员去制定展示策略，但是大多数情况我们使用Core Animation框架是不需要考虑缓存的问题的。

## 2. 核心：图层（Layer）

### 2.1 图层提供了绘制和动画基础

`图层对象是三维空间的二维映射，是Core Animation所有行为的核心。和UIView类似，图层管理几何图形、内容和表面的可视化属性。与UIView不同的是，图层不能定义自己的外观。图层仅仅管理了位图（就是给GPU操作的那个）的状态信息。因此，大多数我们使用的图层被抽象为model，正是由于它主要是管理数据。`

#### 2.1.1 基于图层（硬件渲染）的绘制模型

* 如上所述，大多数图层不操作绘制行为。图层只是捕获app提供的内容并缓存到位图。这个过程有时候被称为`backing store`。当我们连续改变图层的某一属性，且这个改变触发了动画，Core Animation传递图层位图和状态信息给`graphics hardware`来渲染这些信息。基于图层（硬件渲染）的绘制模型如下图，使用硬件来渲染比软件快得多（至于为什么，后面会解释）。
![图2.1](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图2.1.png?raw=true)

* 由于基于图层的绘制操作的是一个静态位图（static bitmap），这是与基于view的绘制最大的不同。在基于UIView的绘制中，改变view会导致调用drawRect:方法来使用新参数重画内容。但是这种绘制方式是很昂贵的，因为这个过程是使用CPU在主线程中操作。Core Animation避免了这种CPU占用，通过缓存硬件位图来达到相同的目的。

#### 2.1.2 基于图层的动画

* 图层的状态信息与图层在屏幕上展示的内容是不挂钩的。这种不关联性给了Core Animation一种方式去主动改变状态值。比如，改变图层的`position`导致Core Animation去移动图层。这些直接操作属性的改变来产生的动画我在网上看到一个词：显式动画。关于可以触发显式动画的属性，可以查看[这里](developer.apple.com/library/etc/redirect/xcode/ios/1151/documentation/Cocoa/Conceptual/CoreAnimation_guide/AnimatableProperties/AnimatableProperties.html)。

### 2.2 图层对象定义自己的几何形状

`图层的一个功能就是管理它内容中的可视化图形。这个可视化几何元素包含内容的边界（content），在屏幕的位置（position）和图层的缩放、变换。和UIView类似，图层也有自己的frame和bounds，但图层也有属性是UIView不具备的，比如锚点（anchor point，后面会介绍）。这些对图层的几何操作是与view有所不同的。`

#### 2.2.1 图层的两套坐标系

* 图层体系提供两套坐标系：基于点的坐标系（point-based coordinate systems）和基于单位的坐标系 （unit coordinate systems）。

* 使用哪个坐标系取决于传递的信息类型。比如，基于点的坐标系被使用在当指定值被直接映射到屏幕坐标系上或是必须被关联到另一个图层，比如是改变了`position`属性。这样说可能有点抽象，可以看看下面的这个图：
![图2.2](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图2.2.png?raw=true)

* 而基于单位的坐标系的使用，可以参考锚点（anchorPoint）的使用。如下图，视图被的边界为[0,1]，“1”就是这个单元。如下图：
![图2.3](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图2.3.png?raw=true)

#### 2.2.2 锚点（anchorPoint）影响着几何变换

* 图层的几何相关操作与图层的锚点息息相关。当我们操作图层的`position`和`transform`属性时锚点的作用显而易见。这样说很抽象，看看下图：
![图2.4](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图2.4.png?raw=true)

* 简要说明下上图。由图可以看出，锚点坐标是相对于图层自己本身的，且是按照单元坐标系进行计算。而`position`是相对于父图层来说的位置，指的的子图层锚点在父图层的坐标位置。以下再看幅图，来看看锚点对`transform`的影响：
![图2.5](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图2.5.png?raw=true)

#### 2.2.3 图层的三维变换

* 每个图层都有两个变换矩阵供程序员操作。CALayer的`transform`属性指定了图层自身的变换和子图层的变换。而`sublayerTransform`属性则是定义了额外的变换只应用于子图层。

* 变换的工作原理是矩阵的乘法。将原坐标通过矩阵变换后得到新的坐标，实现图层上所有元素的改变。由于Core Animation的值支持三维坐标，每个坐标点可以有4个维度值（x,y,z,alpha），所以在矩阵变换的时候需要构造一个4*4的矩阵进行变换。以下是一些常用矩阵，如下图。
![图2.6](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图2.6.png?raw=true)

* 此外，为了使用函数操作变换，Core Animation拓展了KVC支持，运行操作key paths来操作变换。[key paths 链接](developer.apple.com/library/etc/redirect/xcode/ios/1151/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html)

### 2.3 图层树反映不同方面的动画状态

Core Animation中使用到的layer存放在类似树的集合中。(这块提了三个树结构，没感觉有什么必要讲的，感兴趣的可以看文档)

### 2.4 图层(layer)和视图(view)的关系

* 图层不是视图的替代品，我们不能创造可视化接口仅仅基于图层对象。视图的底层构造由图层提供。特别地，图层使视图的绘制和动画内容更加简单、高效，同时又保持着高桢频。但是，有很多工作是图层无法完成的。首先，图层不去处理事件，不去渲染内容，不参与响应者链条（the responder chain）。因此，还是需要视图来处理这些交互。

* 在iOS中，每个视图都有一个图层支持（Mac OS不是这样）。这个图层由系统管理，并会和视图保持同步关系。Apple建议我们直接去操作视图。

## 3. 图层设置（Setting Up Layer Objects）

### 3.1 改变UIView的图层对象

`UIView对象默认内置CALayer对象。`

* 我们能改变UIView内置的图层类（默认CALayer）通过重写`+layerClass`方法。 那么在什么情况下需要重写这个方法呢？
	* 你的视图绘制需要用到Metal或OpenGL ES时，需要使用CAMetalLayer或CAEAGLLayer对象。
	* 如果有一个专门的图层在这个场景可以提供更好的性能。
	* 你想使用一些特别的图层类（关于不同图层类的功能见下图）。

![图3.1](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图3.1.png?raw=true)

### 3.2 为图层提供内容

* 图层是管理内容的数据对象。图层的内容由一个包含了可视化数据的位图组成。我们能为位图提供内容的方式有三（文档有详细做法）：
	* 给图层的`content`属性直接赋值一个image对象（推荐）。
	* 给图层分配一个代理对象，让代理画图层的内容。
	* 定义图层的子类，重写绘制方法。

* 我们唯一需要考虑要自己给图层提供内容的时机是我们自己创建了图层。如果我们使用UIView，就不需要担心这个问题。UIView总是会选择最高效的方式为其图层输送内容。

### 3.3 调整图层样式和外观

#### 3.3.1 图层的背景和边界

* 图层的背景颜色被渲染在其content后面，而图层的边界被渲染在content前面，如下图：
![图3.2](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图3.2.png?raw=true)

		myLayer.backgroundColor = [UIColor greenColor].CGColor; // 背景色
		myLayer.borderColor = [UIColor blackColor].CGColor; // 边框色
		myLayer.borderWidth = 3.0; // 边框宽
		
* 需要注意的是：如果你设置了图层的背景颜色是一个不透明色，同时设置图层的属性`opaque`为YES。这样可以提高性能当图层被合成到屏幕上时和消除图层在backing store时管理alpha通道的操作。同时，如果有设置圆角属性，绝对禁止把图层标记为不透明。

#### 3.3.2 图层的圆角（略）
#### 3.3.3 图层支持嵌入阴影

* CALayer内置属性来设置阴影效果。图层的阴影alpha默认为0。图层阴影默认在图层content之下。基于此，我们操作阴影需要两个步骤，如图：
![图3.3](https://github.com/wwhisdavid/MD_Pictures/blob/master/Core%20Animation/图3.3.png?raw=true)

* 当我们给图层加阴影后，阴影就成了图层内容的一部分，但事实上超出了图层的bounds。此时如果设置了maskToBounds，这个阴影效果将被边界裁剪。如果图层有任何透明内容，这将导致部分阴影被显示出来，而超出边界的阴影则被裁剪。如果想同时有阴影且能裁剪，你需要使用两个图层。[文档](developer.apple.com/library/etc/redirect/xcode/ios/1151/documentation/Cocoa/Conceptual/CoreAnimation_guide/LayerStyleProperties/LayerStyleProperties.html)


## 4. 图层动画（Animating Layer Content）

### 4.1 通过改变属性的动画

* 简单的动画制作分为显式动画和隐式动画。隐式动画使用默认的时间和动画属性来展示动画，而显式动画要求设置属性来使用动画对象。所以隐式动画可以用很少的代码实现简单的动画。

* 简单的动画通过改变图层的属性和让Core Animation使用默认的时间动画这些改变。为了触发隐式动画，我们需要改变图层对象的属性值。

* 隐式动画下的fade实现：

		theLayer.opacity = 0.0;
		
* 显示动画下的fade：

 		CABasicAnimation* fadeAnim = [CABasicAnimation animationWithKeyPath:@"opacity"];
		fadeAnim.fromValue = [NSNumber numberWithFloat:1.0]; // 默认图层当前值
		fadeAnim.toValue = [NSNumber numberWithFloat:0.0];
		fadeAnim.duration = 1.0;
		[theLayer addAnimation:fadeAnim forKey:@"opacity"];
 
* 不像隐式动画更新了图层对象的数据，显式动画不会改变图层数据，显式动画只会产生动画效果。也就是说，当显式动画结束后，Core Animation 将移除图层上的动画对象，并根据原有数据进行重绘。如果想让这个显式动画的结果永久化，必须更新图层数据。

* 显式动画和隐式动画正常情况下在当前run loop的结尾开始执行，并且当前线程必须有run loop。如果改变多个属性或添加多个动画，所以这些属性改变在同一时间（同步）。

### 4.2 使用Keyframe Animation来改变图层属性（核心桢动画）
* 核心桢动画，顾名思义，将动画抽象为桢分解组合展示。
* 直接上代码：

		// 创建一条动画路径
		CGMutablePathRef thePath = CGPathCreateMutable();
		CGPathMoveToPoint(thePath,NULL,74.0,74.0);

		CGPathAddCurveToPoint(thePath,NULL,74.0,500.0,
                                   320.0,500.0,
                                   320.0,74.0);
		CGPathAddCurveToPoint(thePath,NULL,320.0,500.0,
                                   566.0,500.0,
                                   566.0,74.0);
 
		CAKeyframeAnimation * theAnimation;
 
		// 创建动画对象，设置对应key，动画时间，动画路径
		theAnimation=[CAKeyframeAnimation animationWithKeyPath:@"position"];
		theAnimation.path=thePath;
		theAnimation.duration=5.0;
 
		// 给图层加动画
		[theLayer addAnimation:theAnimation forKey:@"position"];
 

#### 4.2.1 指定Keyframe value

* 核心桢的值是核心桢动画的核心。这些值定义了动画流程执行后的行为。

* 当指定一个核心桢值数组，你放到数组中内容依赖于属性需要的数据类型。你可以直接添加一些对象到数组中。然而一些对象必须在添加到数组中之前被转换为id类型，所有标量类型或结构体必须被包装为对象，比如：
	* 对于属性类型为CGRect（例如bounds和frame属性），使用NSValue对象包装每一个矩形。
	* 对于图层的变换属性，使用NSValue包装每一个CATransform3D矩阵。动画这个属性将引起关键帧动画给图层轮流应用每个变换矩阵。
	* 对于borderColor属性，在添加到数组之前，转换CGColorRef数据类型为id类型。
	* 对于属性为CGFlot类型，在添加到数组之前，使用NSNumber包装每个值。
	* 为了动画图层的content属性，指定一个CGImageRef数据类型数组。	 

* 对于CGPoint数据类型的属性，你可以创建一个点（使用NSValue对象包装）数组，或者使用CGPathRef对象指定路径。当你指定一个点数组，关键帧动画对象在每一个连续的点之间绘制一条线，并沿着这些线移动。当你指定一个CGPathRef对象，动画起始于路径的开始点并跟随路径线移动，这包括沿着任何曲面。你可以使用开放的或者封闭的路径。

#### 4.2.2 指定核心桢动画的时间函数

核心帧动画的时间函数和步调比基本动画复杂得多，以下有一些属性可以用来控制它：

* `calculationMode`属性定义算法用来计算动画时间。这个属性的值影响时间关联属性的使用。
	*  线性的和三次曲线动画中，`calculationMode`设置为`kCAAnimationLinear` 或`kCAAnimationCubic`。
	*  节奏动画中，`calculationMode`设置为` kCAAnimationPaced`或` kCAAnimationCubicPaced`，不会依赖于`keyTimes`或`timingFunctions`属性。
	*  离散动画中，`calculationMode`设置为`kCAAnimationDiscrete`,这让动画属性从一个值变道另一个值时不会有插值（拟合）。这个设置使用了`keyTimes`而不管`timingFunctions`。

* keyTimes属性为应用在每一核心帧，指定应用到每一个帧上的计时器。该属性只在calculationMode属性被设置为kCAAnimationLinear，kCAAnimaitonDiscrete，kCAAnimationCubic时被使用。它不使用在节奏动画中。下面解释一发keyTimes：
	* @property(copy) NSArray <NSNumber *> *keyTimes
	* 这个数组放的是递增的NSNumber，且处于[0，1]之间。表示每一帧的动画时间点。比如有5桢动画，则数组有5个数值分别表示5桢动画的动画时间点。 

* timingFunctions属性指定使用在每一个核心帧部分的定时曲线(该属性替换了继承的timingFunction属性)。值为枚举，使用系统定义的几个动画函数。 

* 如果你想自己处理动画的定时，可以使用kCAAnimationLinear或kCAAnimaitonCubic模式与keyTimes和timingFunctions属性。keyTimes定义了应用在每一关键帧的时间点。所有中间值的定时由定时函数控制，定时函数允许你对各个部分应用缓入或缓出曲线定时。如果你不指定任何定时函数，动画将会是线性的。
 
