系统学习iOS动画之零：说明和目录
---------

之前也做过一些iOS动画，但一直没有系统学习过，这次我用[RW网站](https://www.raywenderlich.com)的书 [《iOS Animations by Tutorials》](https://store.raywenderlich.com/products/ios-animations-by-tutorials) 来系统地学习iOS动画。这本书示例项目不是太复杂但很完善。

这个系列的几篇文章是我学习  的几篇笔记。

动画制作很有趣，可以为用户界面注入活力。 打开菜单或向右滑动时，谁不喜欢有一点视觉刺激的应用程序呢😏？

如果使用得当，动画可以向用户传达信息，并将用户注意力吸引到界面的重要部分。

**Xcode 10.1, Swift 4.2, macOS Mojave 10.14.1**

本书的每一章都包含一个入门项目，并详细介绍了少量动画技术; 这使您可以按任何顺序完成本书中的章节。
但是，对于初学者，我们建议您按顺序完成各章，因为这些概念是相互依赖的。 另外，如果您按照教程并执行动手操作，请记住，您将充分利用本书。
对于高级开发人员，由于您可能还不熟悉这些熟悉的动画API的Swift语法，因此在前面的章节中仍然有价值，因为它涵盖了基础知识。 但是，如果您对此感到满意，请随时跳到您最感兴趣的主题。



## 关于代码

原书提供的代码，每章都会有*开始项目*和*最终完成项目*代码（这应该是RW网站的惯例了😀），有的章节还有有*挑战项目*。建议按顺序阅读，因为前后章节知识点有一定关联。

**开始项目**都是相对简单项目或者是前一个章节的项目，可以直接使用原书提供的，也可以自己从头创建一下（我自己就是这么干的🤓🤓）。

我完成每一章节代码放在GitHub上 [andyRon/LearniOSAnimations](https://github.com/andyRon/LearniOSAnimations)，代码中加一些中文注释便于理解。

> 悄悄地说，如果暂时手头没有多余💰购买正版，可以私信我获取电子书+代码。

### 项目预览和对应章节


- **BahamaAirLoginScreen**  1 2 3    8 9 10 11 12 

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fx69ltw09dg308s0avwtn.gif)

- **Flight Info**           4 5

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxcajmwugeg308m0fndxy.gif)

- **Packing List**          6 7

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fw8qbtmmeag308s0fnafk.gif)

- **MultiplayerSearch**   13

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxmnjaf154g308m0fn1gb.gif)

- **SlideToReveal**       14

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fy6f1w4io8g308o0fpjw8.gif)

- **PullToRefresh**       15

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fx7j42np9ig308q08r0w0.gif)

- **Lris**                16

- **BeginnerCook**        17



- **LogoReveal**          18 19



- **LockSearch** 		20 21 22 23 

- **OfficeBuddy** 	24

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxvpgah492g306y067799.gif)

- **ImageGallery** 	25

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxvr9roxswg308q0fo7wh.gif)

- **Snow Scene**26

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxvzhofmleg30ku112b2a.gif)

- **SouthPoleFun**  	27

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxw1bdetvhg30fm08sn0q.gif)






## 目录

 [《iOS Animations by Tutorials》](https://store.raywenderlich.com/products/ios-animations-by-tutorials) 全书分为7个部分，27小章节，内容非常丰富，我对应7个部分分别总结为7篇文章，有几篇文章可能比较长，特别是动图比较多，用手机看的小伙伴请慎重，对自己温柔一点🥴。

之所以把目录结构列出了，是我认为系统的学习很重要，每个知识点对应在系统点上，这样那边出问题也好及时发现，要不可能越学越乱，这方面我自己深有体会的😕🤔。

[系统学习iOS动画之一：视图动画](Section_I.md)
[系统学习iOS动画之二：自动布局](Section_II.md)
[系统学习iOS动画之三：图层动画](Section_III.md)
[系统学习iOS动画之四：视图控制器的转场](Section_IV.md)
[系统学习iOS动画之五：使用UIViewPropertyAnimator](Section_V.md)
[系统学习iOS动画之六：3D动画](Section_VI.md)
[系统学习iOS动画之七：其它类型的动画](Section_VII.md)



## 关于英文

有的英文不怎么好翻译，为了不让我的翻译产生歧义，我尽量在使用专业词汇时，也把英文给出。

英语水平有限，勉强能看懂和理解书籍，但有的部分翻译成中文，就有点尴尬了，所以遇到一些不好翻译的词时，我就把英文原文也列出来。

另外加一个学到英语词汇😊
wrangle  	争吵，口角
thrill		(使)兴奋
convey	  表达，运输
subtle
elaborate
stiff
mimic
keen
acceleration
deceleration
oscillation
snappy



GIF很多都使用**Slow Animations(+T)**，所以看上去比较慢。

添加持续时间



## 动画相关的类总结

UIKit

​	.animate(withDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion:)

​	.transition(...)

​	

QuartzCore



CAAnimation

​	CAPropertyAnimation

​	CABasicAnimation 

​		CASpringAnimation 	

​	CAKeyframeAnimation

​	CATransition

​	CAAnimationGroup



CATransform3D

CGAffineTransform

CATransform3DRotate

CALayer  CAEmitterLayer















