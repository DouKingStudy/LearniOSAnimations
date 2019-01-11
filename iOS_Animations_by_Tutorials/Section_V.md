# 系统学习iOS动画之五：使用UIViewPropertyAnimator



`UIViewPropertyAnimator`是从iOS10开始引入，它能够创建易于交互，可中断和/或可逆的视图动画。

前面部分已经学习了核心动画的大部分功能，视图动画不会有太大的惊喜。

但是这个类让某些类型的视图动画更容易创建，值得学习。



最值得注意的是，当您通过动画师运行动画时，您可以动态调整这些动画 - 您可以暂停，停止，反转和更改已经运行的动画的速度。

如上所述，你可以通过使用图层和视图动画的组合来完成上面提到的所有事情，但是`UIViewPropertyAnimator`可以在同一个类中方便地将许多API包装在一起，这样更容易使用。

此外，这个新类完全取代了`UIView.animate(withDuration...)`API集，也没有使用`CAAnimation`替换您创建的动画，因此您仍然需要经常将这些与`UIViewPropertyAnimator`动画结合使用。

在本书的这一部分中，您将开发一个具有大量不同视图动画的项目，您将使用`UIViewPropertyAnimator`实现这些动画。

**预览：**

[20-UIViewPropertyAnimator入门](#20-UIViewPropertyAnimator入门) —— 了解如何创建基本视图动画和关键帧动画。您将研究使用超出内置缓动曲线的自定义时序。

第21章，使用UIViewPropertyAnimator的中级动画 - 在本章中，您将学习如何使用具有自动布局的动画师。此外，您将学习如何反转动画或制作添加动画，以便在此过程中实现更平滑的变化。

第22章，使用UIViewPropertyAnimator进行交互式动画 - 了解如何根据用户的输入以交互方式驱动动画。为了获得额外的乐趣，您将了解基本和关键帧动画的交互性。

第23章，UIViewPropertyAnimator视图控制器转换 - 使用UIViewPropertyAnimator创建自定义视图控制器转换以驱动转换动画。您将创建静态和交互式过渡。

完成所有这些章节后，您肯定会在家中使用UIViewPropertyAnimator为您的应用程序中的所有类型的动画！

本文的四个章节都是使用同一个项目



## 20-UIViewPropertyAnimator入门


在iOS10之前，创建基于视图的动画的唯一选择是`UIView.animate(withDuration: ...)`一组API，但这组API没有为开发人员提供暂停或停止已经运行的动画的处理方式。此外，对于反转，加速或减慢动画，开发人员只能使用基于图层的`CAAnimation`（核心动画）。

`UIViewPropertyAnimator`就是为了解决上述问题而出现的，它是一个允许保持运行动画的类，允许开发者调整当前运行的动画，并提供有关动画当前状态的详细信息。

当然，简单单一的视图动画直接使用`UIView.animate(withDuration: ...)`就可以了。此外，`UIViewPropertyAnimator`并未实现`UIView.animate(withDuration: ...)`的所有内容。

### 基础动画

本章的[开始项目](README.md#关于代码) **LockSearch** 。 类似于iOS中的锁定屏幕的屏幕。 初始视图控制器有搜索栏，单个窗口小部件和编辑按钮等：

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxukdn545vj31c00u0gy0.jpg)

[开始项目](README.md#关于代码) 已经实现了一些与动画无关的功能。 例如，如果点击“显示更多”，窗口小部件将展开并显示更多项目。 如果点击编辑，会另一个视图控制器，用于编辑窗口小部件列表。

当然，该项目只是模拟iOS中的锁定屏幕，没有实际的功能，用来学习动画。

打开`LockScreenViewController.swift`并向该视图控制器添加一个新的`viewWillAppear(_:)`方法：

```swift
override func viewWillAppear(_ animated: Bool) {
    tableView.transform = CGAffineTransform(scaleX: 0.67, y: 0.67)
    tableView.alpha = 0
}
```

为了创建简单的缩放和淡入淡出视图动画，首先缩小整个表视图并使其透明。

接下来，在视图控制器的视图出现在屏幕上时创建一个动画师。 将以下内容添加到`LockScreenViewController`：

```swift
override func viewDidAppear(_ animated: Bool) {
    let scale = UIViewPropertyAnimator.init(duration: 0.33, curve: .easeIn) {
    }
}
```



在这里，您使用`UIViewPropertyAnimator`的一个[便利构造器](http://andyron.com/2017/swift-14-initialization.html#5-类的继承和构造过程)：`UIViewPropertyAnimator.init(duration:curve:animations:)`。

通过构造器创建动画实例并设置动画的总持续时间和时间曲线。 后一个参数的类型为`UIViewAnimationCurve`，这是一个枚举类型，有四个类型：`easeInOut`、`easeIn`、`easeOut`、`linear`。这与`UIView.animate(withDuration:...)`中的[`option`](Section_I.md#动画缓动)是类似的。



#### 添加动画

在`viewDidAppear(_:)`中添加：

```swift
scale.addAnimations {
    self.tableView.alpha = 1.0
}
```

使用`addAnimations`添加动画代码块，就像`UIView.animate(withDuration...)`的闭包参数`animations`。 使用动画师的不同之处在于可以添加多个动画块。

除了能够有条件地构建复杂的动画外，还可以添加具有不同延迟的动画。 另一个版本的`addAnimations`，有两个参数：
`animation`  动画代码
`delayFactor`  动画开始前的延迟

`delayFactor`与`UIView.animate(withDuration...)`中`delay`不同，它介于0.0到1.0，不是绝对时间是相对时间。



在同一个动画师添加第二个动画，但有一些延迟。继续在上面的代码后添加：

```swift
scale.addAnimations({
    self.tableView.transform = .identity
}, delayFactor: 0.33)
```

实际延迟时间是`delayFactor`乘以动画师的剩余持续时间(remaining duration)。 目前尚未启动动画，因此剩余持续时间等于总持续时间。
所以在上面的情况：

```swift
delayFactor(0.33) * remainingDuration(=duration 0.33) = delay of 0.11 seconds
```



**为什么第二个参数不是一个简单的秒数值？**
想象动画师已经在运行了，你决定在中途添加一些新的动画。 在这种情况下，剩余持续时间不会等于总持续时间，因为自启动动画以来已经过了一段时间。

![image-20181204120317868](https://ws4.sinaimg.cn/large/006tNbRwgy1fxuky4tem3j30e2055q37.jpg)

在这种情况下，`delayFactor`将允许开发者根据剩余可用时间设置延迟动画。 此外，这样设计也确保了不能将延迟设置为长于剩余运行时间。

![image-20181204120335113](https://ws4.sinaimg.cn/large/006tNbRwgy1fxukyfk6x4j30co054glu.jpg)

#### 添加完成闭包

在`viewDidAppear(_:)`中添加：

```swift
scale.addCompletion { (_) in
    print("ready")
}
```

`addCompletion(_:)`就是动画完成闭包，当然，它也可多次调用，来完成多了处理程序。

下面要启动动画，在`viewWillAppear(_:)`的末尾添加：

```swift
scale.startAnimation()
```



### 提取动画

为了代码的清晰，可以把动画代码集中放到一个类中。

创建一个名为`AnimatorFactory.swift`的新文件，并将其默认内容替换为：

```swift
import UIKit

class AnimatorFactory {
  
}
```

然后添加一个类型方法，其中包含刚刚编写的动画代码，但默认情况下不运行动画，而是返回动画师：

```swift
static func scaleUp(view: UIView) -> UIViewPropertyAnimator {
    let scale = UIViewPropertyAnimator(duration: 0.33, curve: .easeIn)

    scale.addAnimations {
        view.alpha = 1.0
    }

    scale.addAnimations({
        view.transform = .identity
    }, delayFactor: 0.33)

    scale.addCompletion { (_) in
                         print("ready")
                        }

    return scale
}
```

该方法将视图作为参数，并在该视图上创建所有动画，最后它返回准备好的动画师。

将`LockScreenViewController`中的`viewDidAppear(_:)`替换为：

```swift
override func viewDidAppear(_ animated: Bool) {
    AnimatorFactory.scaleUp(view: tableView).startAnimation()
}
```

这样看上去代码更加简洁，清晰，把动画代码从视图控制器移出。

这个动画师工厂🏭类`AnimatorFactory`集中处理动画代码，这是设计模式中的工厂模式的一个简单应用。😀



### 运行动画师

<!--

此时你可能会问自己”如果动画对象的唯一目的是立即开始，那么创建动画对象有什么意义呢？“
这是个好问题！

如果您需要运行单个动画块并且不再需要更改，请继续使用`UIView.animate(withDuration: ...)`。您决定使用哪个API的转折点取决于您是想简单地运行动画 - 还是运行动画并最终与之交互。

如果你想使用UIViewPropertyAnimator但你仍然只有一个动画和完成块，并想立即运行它会怎么样？是不是有更简化的方式来创建这样的动画？
为什么，是的，有！你问我很高兴。这就是本章的这一部分被称为运行动画师的原因。在UIViewPropertyAnimator上有一个类方法，它创建一个动画师并立即为你启动它。

-->

当用户使用搜索栏时，将淡入模糊图层（blurView），并在用户完成搜索时将其淡出。

向`LockScreenViewController`类添加一个新方法：

```swift
func toggleBlur(_ blurred: Bool) {
    UIViewPropertyAnimator.runningPropertyAnimator(withDuration: 0.5, delay: 0.1, options: .curveEaseOut, animations: {
        self.blurView.alpha = blurred ? 1 : 0
    }, completion: nil)
}
```

`UIViewPropertyAnimator.runningPropertyAnimator(withDuration:...)`与`UIView.animate(withDuration:...)`有完全相同的参数，使用也相同。

虽然看起来这可能是一种**“即发即忘”**(“fire-and-forget” )的API，但请注意它确实会返回一个动画实例。 因此，您可以添加更多动画，更多完成块，并且通常与当前正在运行的动画进行交互。

现在让我们看看淡入淡出动画的样子。 LockScreenViewController已设置为搜索栏的委托，因此您只需实现所需的方法即可在正确的时间触发动画。

以扩展的方式为`LockScreenViewController`遵守搜索栏的代理协议：

```swift
extension LockScreenViewController: UISearchBarDelegate {
  
  func searchBarTextDidBeginEditing(_ searchBar: UISearchBar) {
    toggleBlur(true)
  }
  
  func searchBarTextDidEndEditing(_ searchBar: UISearchBar) {
    toggleBlur(false)
  }
}
```

 要为用户提供取消搜索的功能，还要添加以下两种方法：

```swift
  func searchBarResultsListButtonClicked(_ searchBar: UISearchBar) {
    searchBar.resignFirstResponder()
  }
  
  func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
    if searchText.isEmpty{
      searchBar.resignFirstResponder()
    }
  }
```

这将允许用户通过点击右侧按钮解除搜索。 

运行，效果：

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxulof5jzqg308q09lwov.gif)

点按搜索栏文本字段，小部件在模糊效果视图下消失；点击搜索栏右侧的按钮时，模糊视图会淡出。



### 基础关键帧动画

`UIViewPropertyAnimator`也可以使用`UIView.addKeyframe`([5-视图的关键帧动画](Section_I.md#5-关键帧动画))。下面创建一个简单的图标抖动动画来展示。



在`AnimatorFactory`中添加类型方法：

```swift
  static func jiggle(view: UIView) -> UIViewPropertyAnimator {
    return UIViewPropertyAnimator.runningPropertyAnimator(withDuration: 0.33, delay: 0
      , animations: {
        UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.25, animations: {
          view.transform = CGAffineTransform(rotationAngle: -.pi/8)
        })
        UIView.addKeyframe(withRelativeStartTime: 0.25, relativeDuration: 0.75, animations: {
          view.transform = CGAffineTransform(rotationAngle: +.pi/8)
        })
        UIView.addKeyframe(withRelativeStartTime: 0.75, relativeDuration: 1.0, animations: {
          view.transform = CGAffineTransform.identity
        })
    }, completion: { (_) in
      
    })
  }
```

第一个关键帧向左旋转，第二个关键帧向右旋转，最后第三个关键帧回到原点 。

要确保图标保持在其初始位置，在完成闭包中添加：

```swift
view.transform = .identity
```

下面就可以在想要运行这个动画的视图上添加动画了。



打开`IconCell.swift`（该文件位于Widget子文件夹中）。这是自定义单元类，对应于窗口小部件视图中的每个图标。
在`IconCell`中添加：

```swift
func iconJiggle() {
    AnimatorFactory.jiggle(view: icon)
}
```

现在**Xcode**抱怨`AnimatorFactory.jiggle`方法返回一个结果没有被使用，这是Xcode善意的提醒😊。

![image-20181204124154609](https://ws1.sinaimg.cn/large/006tNbRwgy1fxum2bblu4j31gy032t98.jpg)

这个问题很容易解决，只需要在`jiggle`方法前添加`@discardableResult`，让Xcode知道这个方法的结果我不要了😏。

> `discardableResult`的[官方解释](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html)：
>
> Apply this attribute to a function or method declaration to suppress the compiler warning when the function or method that returns a value is called without using its result.

```swift
  @discardableResult
  static func jiggle(view: UIView) -> UIViewPropertyAnimator {
```


要最终运行动画，在`WidgetView.swift` 的`collectionView(_:didSelectItemAt:)`中添加：

```swift
if let cell = collectionView.cellForItem(at: indexPath) as? IconCell {
    cell.iconJiggle()
}
```

效果：

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxumdsesopg308q06075c.gif)







### 提取模糊动画

把前面的模糊动画也提取到`AnimatorFactory`中。

```swift
@discardableResult
static func fade(view: UIView, visible: Bool) -> UIViewPropertyAnimator {
    return UIViewPropertyAnimator.runningPropertyAnimator(withDuration: 0.5, delay: 0.1, options: .curveEaseOut, animations: {
        view.alpha = visible ? 1.0 : 0.0
    }, completion: nil)
}
```

替代`LockScreenViewController`中的`toggleBlur（_:）`方法：

```swift
func toggleBlur(_ blurred: Bool) {
    AnimatorFactory.fade(view: blurView, visible: blurred)
}
```



### 防止动画重叠

**如何检查动画师当前是否正在执行其动画？**



如果在同一个图标上快速连续点击，会发现抖动动画没有结束就重新开始了。

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxumve35v2g308k06076o.gif)

解决这个问题，就需要检测视图是否有动画正在运行。

为`IconCell`添加一个属性，并修改`iconJiggle()`：

```swift
  var animator: UIViewPropertyAnimator?

  func iconJiggle() {
    if let animator = animator, animator.isRunning {
      return
    }

    animator = AnimatorFactory.jiggle(view: icon)
  }
```

对比可以发现有所不同：

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxumwxwqd0g308k0600uv.gif)









## 21-深入UIViewPropertyAnimator

上一章节学习了`UIViewPropertyAnimator`的基本使用，这一章节学习更多关于`UIViewPropertyAnimator`的知识。

>  本章的[开始项目](README.md#关于代码) 使用上一章节完成的项目。



### 自定义动画计时

前文已经多次提到：`easeInOut`、`easeIn`、`easeOut`、`linear`（可以理解为物体运动轨迹的曲线类型）。可以参考[视图动画中的动画缓动](Section_I.md#动画缓动) 或者[图层动画中的动画缓动](Section_III.md#动画缓动)，这边就不再介绍了。

#### 内置时间曲线

目前，当您激活搜索栏时，您会在窗口小部件顶部的模糊视图中淡入淡出。 在此示例中，您将删除该淡入淡出动画并为模糊效果本身设置动画。

之前，激活搜索栏时，就会有一个模糊视图中淡入淡出效果。这个部分删除这个效果，修改成对模糊效果本身设置动画。什么意思呢？ 看完下面的操作，应该能明白。

向`LockScreenViewController`类添加一个新方法：

```swift
func blurAnimations(_ blured: Bool) -> () -> Void {
    return {   
    	self.blurView.effect = blured ? UIBlurEffect(style: .dark) : nil
		self.tableView.transform = blured ? CGAffineTransform(scaleX: 0.75, y: 0.75) : .identity
		self.tableView.alpha = blured ? 0.33 : 1.0
    }
}
```

删除`viewDidLoad()`中的两行代码：

```swift
    blurView.effect = UIBlurEffect(style: .dark)
    blurView.alpha = 0
```

替代`toggleBlur(_:)`内容为：

```swift
func toggleBlur(_ blurred: Bool) {
    UIViewPropertyAnimator(duration: 0.55, curve: .easeOut, animations: blurAnimations(blurred)).startAnimation()
}
```

运行，效果：

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxur7gcxngg308k09u11x.gif)



请注意模糊不仅仅是淡入或淡出，实际上它会在效果视图中插入模糊量。



#### 贝塞尔曲线

有时想要对动画的时间非常具体时，使用这些曲线简单地“开始减速”或“慢慢结束”是不够的。

在[10-动画组和时间控制](Section_III.md#10-动画组和时间控制) 中学习了使用`CAMediaTimingFunction`控制图层动画的时间。

之前没有了解背后的原理**贝塞尔曲线**，这边介绍一下它。这边的内容也可应用到图层动画中。

**贝塞尔曲线是什么？**

让我们从简单的事情开始 —— 一条线。它非常简洁，需要在屏幕上画一条线，只需要定义它的两个点的坐标，开始 *(A)* 和结束 *(B)*：

![image-20181204154619268](https://ws3.sinaimg.cn/large/006tNbRwgy1fxure6zmgcj309q036aa0.jpg)



现在让我们来看看曲线。曲线比线条更有趣，因为它们可以在屏幕上绘制任何东西。例如：

![image-20181204154744302](https://ws1.sinaimg.cn/large/006tNbRwgy1fxurfnsi7zj30c706cq37.jpg)

在上面看到的是四条曲线放在一起;它们的两端在小白方块的地方相遇。图中有趣的是小绿圈，它们定义了每条曲线。

所以曲线不是随机的。它们也有一些细节，就像线条一样，可以帮助我们通过坐标定义它们。

您可以通过向线条添加控制点来定义曲线。 让我们在之前的行中添加一个控制点：

![image-20181204154909038](https://ws1.sinaimg.cn/large/006tNbRwgy1fxurh4vhhdj30aw052gly.jpg)

可以想象由连接到线的铅笔绘制的曲线，其起点沿着线AC移动，其终点沿着线CB移动：

![image-20181204154949279](https://ws4.sinaimg.cn/large/006tNbRwgy1fxurhti255j30fu03ljrn.jpg)

网上找了一个动图：

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fyayz6r66pg30a0046tax.gif)

具有一个控制点的Bézier曲线称为 *二次曲线*。有两个控制点的Bézier曲线叫做 *三次曲线*（立方贝塞尔曲线）。
我们使用的内置曲线就是三次曲线。

核心动画使用始终以坐标（0,0）开始的三次曲线，它表示动画持续时间的开始。 当然，这些时间曲线的终点始终是（1,1），表示 动画的持续时间和进度的结束。

让我们来看看 *ease-in* 曲线：

![image-20181204155458179](https://ws4.sinaimg.cn/large/006tNbRwgy1fxurn6i2vqj309107kdg7.jpg)

随着时间的推移（在坐标空间中从左向右水平移动），曲线在垂直轴上的进展非常小，然后大约在动画持续时间的一半时间后，曲线在垂直轴上的进展非常大，最终在(1, 1)处结束。

*ease-out* 和 *ease-in-out*曲线分别是：

![image-20181204155513973](https://ws4.sinaimg.cn/large/006tNbRwgy1fxurngb6dwj30f306tt9l.jpg)



现在已了解Bézier曲线的工作原理，剩下的问题是如何在视觉上设计一些曲线并获得控制点的坐标，方便可以将它们用于iOS动画。

可以使用网站：http://cubic-bezier.com。 这是计算机科学研究员和演讲者Lea Verou的非常方便的网站。 它可以拖动立方Bézier的两个控制点并查看即时动画预览，非常nice😊😊。

![image-20181204155438660](https://ws4.sinaimg.cn/large/006tNbRwgy1fxurmuxor9j31br0i5n0t.jpg)

> 上面贝塞尔的原理说的不够深刻🤦‍♀️，现在只需了解曲线，通过两个控制点可以画曲线。



接下来，向项目中添加自定义计时动画。

把`LockScreenViewController`中的`toggleBlur()`的现有动画替换为：

```swift
func toggleBlur(_ blurred: Bool) {
    UIViewPropertyAnimator(duration: 0.55, controlPoint1: CGPoint(x: 0.57, y: -0.4), controlPoint2: CGPoint(x: 0.96, y: 0.87), animations: blurAnimations(blurred)).startAnimation()
}
```

这边的`controlPoint1` 和`controlPoint2`两个点，就是我们自定义三次曲线的控制点。

可以通过 http://cubic-bezier.com 网站来选着控制点。



#### 弹簧动画

另一个便利构造器`UIViewPropertyAnimator(duration:dampingRatio:animations:)`，用于定义弹簧动画。

这与`UIView.animate(withDuration: delay: usingSpringWithDamping: initialSpringVelocity: options: animations: completion:) `类似，只不过初始速度为0。



<!--

#### Custom timing providers

还有一个构造器`UIViewPropertyAnimator(duration:timingParameters:)`。

这一次，您可以创建一个可以为动画提供任何计时数据的全新对象！ 您可以使用其中一个UIKit对象来定义自定义立方体或基于弹簧的计时，但您也可以自行推出。

您将看到如何创建自定义弹簧动画，然后再转到本章的下一部分，您将在实践中创建一些弹簧动画。
名为timingParameters的第二个参数是UITimingCurveProvider类型 - 由UIKit定义的协议。 UIKit中有两个符合该协议的类：UICubicTimingParameters和UISpringTimingParameters。
我们来看看UISpringTimingParameters。

#### 提供阻尼和速度

即使您使用自定义计时提供程序，您仍然可以选择简单的方法，并提供阻尼比和初始速度，就像使用便捷初始化程序时一样。 代码如下所示：

```swift
let spring = UISpringTimingParameters(dampingRatio:0.5, initialVelocity: CGVector(dx: 1.0, dy: 0.2))

let animator = UIViewPropertyAnimator(duration: 1.0, timingParameters: spring)
```



spring参数表示弹簧的配置，并将其提供给动画对象以用于动画的计时。 如前所述，这仍将计算弹簧“向后”。

注意初始速度是矢量类型。 如果您要为任何视图的位置或大小设置动画，UIKit将在开始时应用二维初始速度。 如果您要为视图的位置设置alpha或单个轴的动画，UIKit将仅考虑初始速度矢量的dx属性。

initialVelocity也是一个可选参数，因此如果您根本不需要设置速度，只需提供阻尼比即可。



#### Custom springs

如果你想对你的弹簧更加具体，你可以在UISpringTimingParameters上使用不同的初始化器，让你指定弹簧的质量，刚度和阻尼，就像你在本书前面对你的层动画所做的那样。
因此，配置自定义弹簧的代码如下：

```swift
let spring = UISpringTimingParameters(mass: 10.0, stiffness: 5.0, damping: 30, initialVelocity: CGVector(dx: 1.0, dy: 0.2))

let animator = UIViewPropertyAnimator(duration: 1.0, timingParameters: spring) 
```



如果您需要快速了解所有这些参数的工作原理，请快速阅读[第11章“图层弹簧”]()。
在下一节中，您将尝试一些自定义时序动画。

-->



### 自动布局动画

前面的文章[系统学习iOS动画之二：自动布局动画](Section_II.md) 学习了自动布局动画。

使用`UIViewPropertyAnimator`的布局约束动画与使用`UIView.animate(withDuration: ...)`创建它们的方式非常相似。 诀窍是更新约束，在动画块中调用`layoutIfNeeded()`。

在`AnimatorFactory`中添加一个新的工厂方法：

```swift
@discardableResult
static func animateConstraint(view: UIView, constraint: NSLayoutConstraint, by: CGFloat) -> UIViewPropertyAnimator {
    let spring = UISpringTimingParameters(dampingRatio: 0.55)
    let animator = UIViewPropertyAnimator(duration: 1.0, timingParameters: spring)

    animator.addAnimations {
        constraint.constant += by
        view.layoutIfNeeded()
    }
    return animator
}
```



在`LockScreenViewController`中`viewWillAppear`里添加：

```swift
dateTopConstraint.constant -= 100
view.layoutIfNeeded()
```

在`viewDidAppear`里添加：

```swift
AnimatorFactory.animateConstraint(view: view, constraint: dateTopConstraint, by: 150).startAnimation()
```

这让时间标签的位置，在应用打开时有一个动画。



接下来，在添加一个约束动画。当点击“Show more”时，窗口小部件会加载内容，并需要更改其高度约束。

重新定义`WidgetCell.swift`中的`toggleShowMore(_:)`方法：

```swift
@IBAction func toggleShowMore(_ sender: UIButton) {
    self.showsMore = !self.showsMore

    let animations = {
        self.widgetHeight.constant = self.showsMore ? 230 : 130
        if let tableView = self.tableView {
            tableView.beginUpdates()
            tableView.endUpdates()
            tableView.layoutIfNeeded()
        }
    }
    let spring = UISpringTimingParameters(mass: 30, stiffness: 10, damping: 300, initialVelocity: CGVector(dx: 5, dy: 0))

    toggleHeightAnimator = UIViewPropertyAnimator(duration: 0.0, timingParameters: spring)
    toggleHeightAnimator?.addAnimations(animations)
    toggleHeightAnimator?.startAnimation()
}
```



在`toggleShowMore(_:)`方法的底部，添加以下代码用来加载窗口小部件中的图标：

```swift
widgetView.expanded = showsMore
widgetView.reload()
```

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxuvd8a3hwg308k08cqhn.gif)



### 视图过渡

在[视图动画的3-过渡动画](Section_I.md#3-过渡动画)，学习了视图过渡。现在用`UIViewPropertyAnimator`做视图过渡。

显示更多按钮的title，"Show More" 和 "Show Less" 两者相互淡入淡出动画。



在`toggleShowMore（_ :)`的`toggleHeightAnimator`定义之前添加这段代码：

```swift
let textTransition = {
    UIView.transition(with: sender, duration: 0.25, options: .transitionCrossDissolve, animations: {
        sender.setTitle(self.showsMore ? "Show Less" : "Show More", for: .normal)
    }, completion: nil)
}
```

在`toggleHeightAnimator`开始之前添加：

```swift
toggleHeightAnimator?.addAnimations(textTransition, delayFactor: 0.5)
```

这将改变按钮标题，具有很好的交叉淡入淡出效果：

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxuvtnlvmbg303m03s772.gif)

效果也可以尝试`.transitionFlipFromTop`等






## 22-用UIViewPropertyAnimator进行交互式动画



前面两个章节介绍了许多`UIViewPropertyAnimator` 的使用，例如基本动画，自定义计时和弹簧动画，以及动画的提取。但是，与以前视图动画 **“即发即忘”**("fire-and-forget")API相比，尚未研究使`UIViewPropertyAnimator`真正有趣的地方。

`UIView.animate(withDuration:...)`提供了动画的设置方法，但是一旦定义动画结束状态，那么动画就会开始执行，而无法控制。 

**但是如果我们想在动画运行时与之交互，怎么办？** 细说，就是动画不是静态的，而是由用户手势或麦克风输入驱动的，就像在前面图层动画 [系统学习iOS动画之三：图层动画](Section_III.md) 所学的一样。

使用`UIViewPropertyAnimator` 创建的动画是完全交互式的：可以启动，暂停，改变速度，甚至可以直接调整进度。

由于`UIViewPropertyAnimator`可以同时驱动预设动画和交互式动画，因而在描述动画师当前的状态时，就有点复杂了😵。下面就看看如何处理动画师的状态。



> 本章的[开始项目](README.md#关于代码) 使用上一章节完成的项目。



### 动画状态机



`UIViewPropertyAnimator`可以检查动画是否已启动(`isRunning`)，是否已暂停或完全停止(`state`)，或动画是否已颠倒(`isReversed`)。

<!--最后，您可以检查动画“完成”的位置，例如从所需的最终状态开始，或者介于两者之间的某个位置。-->

`UIViewPropertyAnimator`有三个描述当前状态的属性：

![image-20181204183027143](https://ws4.sinaimg.cn/large/006tNbRwgy1fxuw4ytvcqj30et02eaa7.jpg)



`isRunning`（只读）：动画当前是否处于运动状态。 默认为`false`，在调用`startAnimation()`时变为`true`，如果暂停或停止动画，或者动画自然完成，它将再次变为`false`。

`isReversed`：默认为`false`，因为我们总是向前开始动画，即动画从开始状态播放到结束状态。 如果更改为`true`，则动画将颠倒，即从介绍状态到开始状态。

`state` （只读）：



`state`默认为`inactive`，这通常意味着刚刚创建了动画师，并且还没有调用任何方法。请注意，这与将`isRunning`设置为`false`不同，`isRunning`实际上只关注正在进行的动画，而当`state`处于`inactive`时，这实际上意味着动画师还没有做任何事情。

`state` 变成 `active`的情况有：

- 调用`startAnimation()`来启动动画

- 在没有开始动画的情况下调用`pauseAnimation()`

- 设置`fractionComplete`属性以将动画“倒回”到某个位置



动画自然完成后，`state`切换回`.inactive`。

如果在动画师上调用`stopAnimation()`，它会将其`state`属性设置为`.stopped`。在这种状态下，你唯一能做的就是完全放弃动画师或者调用`finishAnimation(at:)`来完成动画并让动画师回到`.inactive`。

正如你可能想到的那样，`UIViewPropertyAnimator`只能按特定顺序在状态之间切换。 它不能直接从`inactive`到`stopped`，也不能从`stopped`直接转为`active`。

如果设置了`pausesOnCompletion`，一旦动画师完成了动画的运行而不是自动停止，而是暂停。 这将使我们有机会在暂停状态下继续使用它。

状态流程图：

![image-20181204183357332](https://ws4.sinaimg.cn/large/006tNbRwgy1fxuw8m1813j30fj05iwf8.jpg)

可能有点绕，之后的使用中，如果有疑问，可以再回到这个部分查看。



### 交互式3D touch动画

从这个部分开始，将学习创建类似于3D touch交互的交互式动画：

![image-20190101223452030](/Users/andyron/Library/Application Support/typora-user-images/image-20190101223452030.png)



> 注意：对于本章项目，需要兼容3D touch的iOS设备（没记错的话是6S+）。 
>
> 听闻👂，3D touch这个技术会被在iPhone上取消，好吧，这边是学习类似3D touch 的动画，它的未来如何，就不过问了。

3D touch的动画，可以这样描述：当我们手指按压屏幕上的图标时，动画交互式开始，背景越来越模糊，从图标旁渐渐呈现一个菜单，这个过程会随着手指按压的力度变化而前后变化。

放慢的效果为：

![ScreenRecording_01-04-2019 11-13-10.2019-01-04 11_24_37](/Users/andyron/Downloads/ScreenRecording_01-04-2019 11-13-10.2019-01-04 11_24_37.gif)



`WidgetView.swift`中，`WidgetView`通过扩展遵守`UIPreviewInteractionDelegate`协议。这个协议中就包括了3D touch过程中一些委托方法。

为了让您开始开发动画本身，`UIPreviewInteractionDelegate`方法已经连接到LockScreenViewController上调用相关方法。
WidgetView中的代码如下：

- 3D Touch开始时调用`LockScreenViewController.startPreview(for:)`。

- 当用户按下的过程中，可能更硬（或更柔和）时，反复调用`LockScreenViewController.updatePreview(percent:)`。

- 当peek交互成功完成时，调用`LockScreenViewController.finishPreview()`。

- 最后，如果用户在未完成预览手势的情况下抬起手指，则调用`LockScreenViewController.cancelPreview()`。



在`LockScreenViewController`中添加这三个属性，您需要这些属性来创建窥视交互：

```swift
var startFrame: CGRect?
var previewView: UIView?
var previewAnimator: UIViewPropertyAnimator?
```

`startFrame`  来跟踪动画的开始位置。

 `previewView`  图标的快照视图，动画期间暂时使用它。
`previewAnimator`  将成为驱动预览动画的交互式动画师。



再添加一个属性以保持模糊效果以显示图标框：

```swift
let previewEffectView = IconEffectView(blur: .extraLight)
```

`IconEffectView`是自定义的`UIVisualEffectView`的子类，它包含单个标签的简单模糊视图，使用它来模拟从按下的图标弹出的菜单：

![image-20181219112216359](/Users/andyron/Library/Application Support/typora-user-images/image-20181219112216359.png)

在`LockScreenViewController`遵守`WidgetsOwnerProtocol`协议的扩展中，实现`startPreview(for:)`方法：

```swift
func startPreview(for forView: UIView) {
    previewView?.removeFromSuperview()
    previewView = forView.snapshotView(afterScreenUpdates: false)
    view.insertSubview(previewView!, aboveSubview: blurView)
}
```

`WidgetsOwnerProtocol`协议是一个自定义协议。

只要用户开始按下图标，`WidgetView`就会调用`startPreview(for:)`。 参数`for`是用户开始手势的集合单元格图像。



首先删除任何现有的`previewView`视图，以防万一在屏幕上留下之前的视图。 然后，您可以创建集合视图图标的快照，最后将其添加到模糊效果视图上方的屏幕上。

运行，按压图标。发现图标出现在左上角！😰

![](https://ws3.sinaimg.cn/large/006tNc79gy1fyupp9utvjg309o0ag0tk.gif)



因为尚未设置其位置。 继续添加：

```swift
previewView?.frame = forView.convert(forView.bounds, to: view)
startFrame = previewView?.frame
addEffectView(below: previewView!)
```

现在图标副本位置正确了，完全覆盖在原有图标上。 `startFrame`用来存储起始`frame`，以供之后使用。

 函数`addEffectView(below:)`添加图标快照下方的模糊框。代码为：

```swift
func addEffectView(below forView: UIView) {
    previewEffectView.removeFromSuperview()
    previewEffectView.frame = forView.frame

    forView.superview?.insertSubview(previewEffectView, belowSubview: forView)
}
```



下面创建动画本身，在`AnimatorFactory`中添加类方法：

```swift
static func grow(view: UIVisualEffectView, blurView: UIVisualEffectView) -> UIViewPropertyAnimator {

    view.contentView.alpha = 0
    view.transform = .identity

    let animator = UIViewPropertyAnimator(duration: 0.5, curve: .easeIn)

    return animator
}
```

两个参数，`view`是动画视图，`blurView` 是动画的模糊背景。

在返回动画师之前，为动画师添加动画和完成闭包：

```swift
animator.addAnimations {
    blurView.effect = UIBlurEffect(style: .dark)
    view.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
}

animator.addCompletion { (_) in
    blurView.effect = UIBlurEffect(style: .dark)
}
```

动画代码为`blurView`创建了模糊过渡，为`view`创建一个普通的转换。

之后，在`LockScreenViewController.swift`的`startPreview()`中完成调用：

```swift
previewAnimator = AnimatorFactory.grow(view: previewEffectView, blurView: blurView)
```

现在运行，还没有效果，还需要实现`updatePreview(percent:)`方法：

```swift
func updatePreview(percent: CGFloat) {
    previewAnimator?.fractionComplete = max(0.01, min(0.99, percent))
}
```

当`WidgetView`被按压时，上面个方法会被重复调用。`fractionComplete`在0.01和0.99范围内，因为我不希望在动画才这段结束，我另外指定的方法完成或取消动画。

运行，效果（放慢）：

![ScreenRecording_01-04-2019 18-29-21.2019-01-04 18_40_04](/Users/andyron/Downloads/ScreenRecording_01-04-2019 18-29-21.2019-01-04 18_40_04.gif)



你会（惊喜！）需要更多的动画师。 打开AnimatorFactory.swift并添加一个动画师，它可以解除你的“成长”动画师所做的一切。
您需要此动画师的一种情况是用户取消手势。 当您需要清理UI时，另一个是成功交互的最后阶段。

在`AnimatorFactory`中添加方法：

```swift
static func reset(frame: CGRect, view: UIVisualEffectView, blurView: UIVisualEffectView) -> UIViewPropertyAnimator {

    return UIViewPropertyAnimator(duration: 0.5, dampingRatio: 0.7, animations: {
        view.transform = .identity
        view.frame = frame
        view.contentView.alpha = 0

        blurView.effect = nil
    })
}
```

此方法的三个参数分别是原始动画的起始帧，动画视图和背景模糊视图。 动画块将重置交互开始之前状态中的所有属性。

在`LockScreenViewController.swift`中，实现`WidgetsOwnerProtocol`协议的另一个方法：

```swift
func cancelPreview() {
    if let previewAnimator = previewAnimator {
        previewAnimator.isReversed = true
        previewAnimator.startAnimation()
    }
}
```

`cancelPreview()`是`WidgetView`被按压后，突然抬起手指时调用的方法，取消正在进行的手势。







到目前为止，你还没有开始你的动画师。 您一直在重复设置`fractionComplete`，这会以交互方式驱动动画。
但是，一旦用户取消交互，您就无法继续以交互方式驱动动画，因为您没有更多输入。 相反，通过将`isReversed`设置为`true`并调用`startAnimation()`，可以将动画播放到其初始状态。 现在这是`UIView.animate(withDuration: ...)`无法做到的事情！

再试一次互动。按下动画的一半，然后开始测试`cancelPreview()`。



当您抬起手指时动画会正确播放，但最终黑暗模糊会突然重新出现。

这个问题植根于你的成长动画师的代码。切换回AnimatorFactory.swift并查看grow中的代码（view：UIVisualEffectView，blurView：UIVisualEffectView） - 更具体地说，这部分：

```swift
animator.addCompletion { (_) in
	blurView.effect = UIBlurEffect(style: .dark)
}
```

动画可以向前或向后播放，需要在完成闭包中处理。

`addCompletion()` 的闭包的参数用`_`省略掉了，它其实是一个枚举类型`UIViewAnimatingPosition`，表示动画当前进行的情况。它的值可有三个，可以是`.start`，`.end`或`.current`。

将完成闭包替代为：

```swift
animator.addCompletion { (position) in
	switch position {
	    case .start:
	    blurView.effect = nil
	    case .end:
	    blurView.effect = UIBlurEffect(style: .dark)
	    default:
	    break
	}
}
```

如果动画被返回，则删除模糊效果。 如果成功完成，则明确将效果调整为暗模糊效果。



现在有一个新问题。 *如果取消对某个图标上的按压，则无法再按下它！*
这是因为图标快照仍然位于原始图标上方，挡住按压手势操作。 要解决该问题，值需要在重置动画完成后立即删除快照。

在`LockScreenViewController.swift`的`cancelPreview()`中继续添加：

```swift
previewAnimator.addCompletion { (position) in
	switch position {
	case .start:
	  self.previewView?.removeFromSuperview()
	  self.previewEffectView.removeFromSuperview()
	default:
	  break
	}
}
```

**注意：**，`addCompletion(_:)`可以调用多次，不会被下一个替代。



让我们再添加一个动画师来显示图标菜单。 切换到AnimatorFactory.swift并添加到它：

```swift
static func complete(view: UIVisualEffectView) -> UIViewPropertyAnimator {

	return UIViewPropertyAnimator(duration: 0.3, dampingRatio: 0.7, animations: {
	  view.contentView.alpha = 1
	  view.transform = .identity
	  view.frame = CGRect(x: view.frame.minX - view.frame.minX/2.5,
	                      y: view.frame.maxY - 140,
	                      width: view.frame.width + 120,
	                      height: 60)
	})
}
```

这一次你创建了一个简单的弹簧动画师。 对于动画师，您可以执行以下操作：

- 淡入“自定义操作”菜单项。

- 重置转换。
- 将视图框架直接设置为图标正上方的位置。

菜单的位置根据用户按下的图标而变化。

您将水平位置设置为 `view.frame.minX - view.frame.minX/2.5`，如果图标位于屏幕左侧，则显示右侧菜单，如果图标位于左侧，则显示左侧菜单在屏幕的右侧。请参阅以下差异：

![image-20190102115412511](/Users/andyron/Library/Application Support/typora-user-images/image-20190102115412511.png)

动画师准备好了，所以打开LockScreenViewController.swift并在WidgetsOwnerProtocol扩展中添加最后一个必需的方法：

```swift
func finishPreview() {

    previewAnimator?.stopAnimation(false)

    previewAnimator?.finishAnimation(at: .end)

    previewAnimator = nil
}
```

当您感觉到触觉反馈时，用户按下3D触摸手势时会调用finishPreview（）。

`stopAnimation(_:)`是停止当前在屏幕上运行的动画。参数为`false`，动画师状态为`stopped`；参数为`true`，动画师状态为`inactive`并清除所有动画，而且不调用完成闭包。





一旦你将动画师置于停止状态，你就有了一些选择。你在finishPreview（）中追求的是告诉动画师完成它的最终状态。因此，您调用finishAnimation（at：.end）;这将使用计划动画的目标值更新所有视图并调用您的完成。

此手势不再需要previewAnimator，因此您可以将其删除。

您可以使用以下方法之一调用finishAnimation（at :)：

`start`：将动画重置为初始状态。
`current`：从动画的当前进度更新视图的属性并完成。



调用`finishAnimation(at:)`后，您的动画师处于`inactive`。

回到Widgets项目。由于你摆脱了预览动画师，你可以运行完整的动画师来显示菜单。将以下内容附加到finishPreview（）的末尾：

```swift
AnimatorFactory.complete(view: previewEffectView).startAnimation()
```



运行，按压图标：

![image-20190102115643202](/Users/andyron/Library/Application Support/typora-user-images/image-20190102115643202.png)



### 关闭模糊视图

目前，菜单弹出，模糊视图显示后，还没有回到原来视图的操作，下面添加这个操作。

在`finishPreview()`中添加以下代码,以准备交互式模糊：

```swift
blurView.effect = UIBlurEffect(style: .dark)
blurView.isUserInteractionEnabled = true
blurView.addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(dismissMenu)))
```

先确保将模糊效果设置为`.dark`，然后模糊视图本身上启用用户交互，并未模糊视图添加点击手势操作，允许用户点击图标周围的任何位置用来关闭菜单。

`dismissMenu()`代码为：

```swift
@objc func dismissMenu() {
    let reset = AnimatorFactory.reset(frame: startFrame!, view: previewEffectView, blurView: blurView)
    reset.addCompletion { (_) in
                         self.previewEffectView.removeFromSuperview()
                         self.previewView?.removeFromSuperview()
                         self.blurView.isUserInteractionEnabled = false
                        }
    reset.startAnimation()
}
```



### 交互式关键帧动画

在[20-UIViewPropertyAnimator入门](#基础关键帧动画)学习了 用`UIViewPropertyAnimator`制作关键帧动画，现在再给关键帧动画添加交互式操作。

为了尝试一下，你将为成长动画添加一个额外的元素 - 在用户按下图标时以交互方式擦洗的元素。

删除`AnimatorFactory`的`grow()`方法中的代码：

```swift
animator.addAnimations {
  blurView.effect = UIBlurEffect(style: .dark)
  view.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
}
```

替换为：

```swift
animator.addAnimations {
    UIView.animateKeyframes(withDuration: 0.5, delay: 0.0, animations: {

        UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 1.0, animations: {
            blurView.effect = UIBlurEffect(style: .dark)
            view.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
        })

        UIView.addKeyframe(withRelativeStartTime: 0.5, relativeDuration: 0.5, animations: {
            view.transform = view.transform.rotated(by: -.pi/8)
        })

    })
}
```

第一个关键帧运行您之前的相同动画。
第二个关键帧是简单旋转，效果：

![ScreenRecording_01-04-2019 23-29-27.2019-01-04 23_32_31](/Users/andyron/Downloads/ScreenRecording_01-04-2019 23-29-27.2019-01-04 23_32_31.gif)





## 23-UIViewPropertyAnimator视图控制器转场



在[系统学习iOS动画之四：视图控制器的转场动画](#Section_IV.md)中，学习了如何创建自定义视图控制器转场。这个部分学习使用`UIViewPropertyAnimator`来自定义视图控制器转场。 你看到了它们的灵活性和强大性，所以你可能很想知道如何使用来创建它们。

本章的[开始项目](README.md#关于代码) 使用上一章节完成的项目。



### 静态视图控制器转场

现在，点击**”Edit“**按钮时，体验非常糟糕😰。 该按钮在当前按钮上显示一个新的视图控制器，只要您点击该第二个屏幕中的任何可用选项，它就会消失。
让我们调高一点！

创建一个新文件`PresentTransition.swift`。 将其默认内容替换为：

```swift
import UIKit

class PresentTransition: NSObject, UIViewControllerAnimatedTransitioning {

    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.75
    }

    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {

    }
}
```



`UIViewControllerAnimatedTransitioning`协议已经在[系统学习iOS动画之四：视图控制器的转场动画](#Section_IV.md)中学过。

将创建一个过渡动画，动画模糊图层并在其上移动新的视图控制器。

在`PresentTransition`中添加一个新方法，为转场创建动画制作工具：

```swift
func transitionAnimator(using transitionContext: UIViewControllerContextTransitioning) -> UIViewImplicitlyAnimating {
    let duration = transitionDuration(using: transitionContext)
    
    let container = transitionContext.containerView
    let to = transitionContext.view(forKey: .to)!
    
    container.addSubview(to)
}
```

在上面的代码中，为视图控制器转场做了必要的准备工作。 首先获取动画持续时间，然后获取目标视图控制器的视图，最后将此视图添加到过渡容器中。

接下来，可以设置动画并运行它。 将下面代码添加到上面的方法`transitionAnimator(using:)`中：

```swift
to.transform = CGAffineTransform(scaleX: 1.33, y: 1.33).concatenating(CGAffineTransform(translationX: 0.0, y: 200))
to.alpha = 0
```

这会向上伸展，然后向下移动目标视图控制器的视图，最后将其淡出。 现在它已经准备好在屏幕上动画了。

在`to.alpha = 0`之后添加动画师来运行转换：

```swift
let animator = UIViewPropertyAnimator(duration: duration, curve: .easeOut)

animator.addAnimations({
    to.transform = CGAffineTransform(translationX: 0.0, y: 100)
}, delayFactor: 0.15)

animator.addAnimations({
    to.alpha = 1.0
}, delayFactor: 0.5)
```

动画师中有两个动画：将目标视图控制器的视图移动到最终位置和淡入。

最后添加完成闭包：

```swift
animator.addCompletion { (_) in                    			  
  transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
}

return animator

```

在`animateTransition(using:)`中调用上面的方法`transitionAnimator(using:)`：

```swift
transitionAnimator(using: transitionContext).startAnimation()
```



那应该暂时做到。 让我们将视图控制器连接到过渡动画师并尝试动画。

在`LockScreenViewController`中定义常量属性：

```swift
let presentTransition = PresentTransition()
```

当UIKit要求您提供演示动画和交互控制器时，您将向UIKit提供此对象。 

让`LockScreenViewController`遵守`UIViewControllerTransitioningDelegate`协议：

```swift
// MARK: - UIViewControllerTransitioningDelegate
extension LockScreenViewController: UIViewControllerTransitioningDelegate {
  
  func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
    return presentTransition
  }
}
```

`UIViewControllerTransitioningDelegate`协议在 [系统学习iOS动画之四：视图控制器的转场动画](#Section_IV.md) 中学习过，可查看。

`animationController(forPresented:presents:source:)`方法是告诉UIKit，我想自定义视图控制器转场。

在`LockScreenViewController`中，找到点击**Edit**按钮的Action`presentSettings(_:)`，添加代码：

```swift
settingsController = storyboard?.instantiateViewController(withIdentifier: "SettingsViewController") as! SettingsViewController
settingsController.transitioningDelegate = self
present(settingsController, animated: true, completion: nil)
```

运行，点击**Edit**按钮，设置控制器似乎有点偏：

![IMG_1035E53ABC83-1](https://ws3.sinaimg.cn/large/006tNbRwgy1fyc6z0vsfjj309q07cwfz.jpg)

你会想要处理一些粗糙的边缘，但你的工作几乎已经完成。
要纠正的第一件事是目标视图控制器不需要纯色背景。 打开`Main.storyboard`（它在Assets项目文件夹中）并选择设置视图控制器视图。

将视图的背景更改为**Clear Color**



再试一次这种转变。 这次您应该看到设置视图控制器的内容直接显示在锁定屏幕上：

![IMG_AB757FD312C4-1](https://ws4.sinaimg.cn/large/006tNbRwgy1fyc7232ygoj309q09z78p.jpg)

看起来这种过渡可以用更多的动画来做。 例如，淡化小部件顶部的模糊，以便用户可以更好地看到顶部的模态视图控制器，这不是很好吗？
既然你已经是专业人士，那就让我们做一些新的事 - “动画注入”！ （无需查看该术语 - 我刚刚为本章提出了这个问题）。
您将向动画师添加一个新属性，允许您将任何自定义动画注入过渡。 这将允许您使用相同的过渡类来生成略有不同的动画。

在`PresentTransition`中添加两个新属性：

```swift
var auxAnimations: (() -> Void)?
var auxAnimationsCancel: (() -> Void)?
```

在`transitionAnimator(using:)`方法中动画师返回之前添加：

```swift
if let auxAnimations = auxAnimations {
    animator.addAnimations(auxAnimations)
}
```

如果您已向对象添加任意任意动画块，则会将其添加到动画制作者动画的其余部分。 取消块允许您在取消转换时执行可能需要的任何展开。

这允许您根据具体情况在转换中添加自定义动画。 例如，让我们为当前转换添加模糊动画。
打开`LockScreenViewController`并在presentSettings（）的顶部插入以下内容：

```swift
presentTransition.auxAnimations = blurAnimations(true)
```

这会将你在很多章节前创建的模糊动画添加到视图控制器转换中！
再试一次过渡，看看这一行如何改变它：

![image-20190111123452428](/Users/andyron/Library/Application Support/typora-user-images/image-20190111123452428.png)

重复使用动画简直太神奇了吗？



现在，当用户解除显示的控制器时，您还需要隐藏模糊。`SettingsViewController`已经有一个`didDismiss`属性，所以你只需要将该属性设置为一个动画模糊的块。
在`settingsController`出现之前的倒数第二行的`presentSettings(_:)`中，插入：

```swift
    settingsController.didDismiss = { [unowned self] in
      self.toggleBlur(false)
    }
```

现在点击设置屏幕中的一个选项将忽略它。 然后模糊将消失，用户将成功恢复到第一个视图控制器：

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fyc7kqfivvg309q0ha45w.gif)

本章的这一部分到此结束。 您的视图控制器转换准备就绪！



### 交互视图控制器转场

作为本书UIViewPropertyAnimator部分的最后一个主题，您将创建一个交互式视图控制器转换。 您的用户将通过下拉窗口小部件表来推动转换。

首先，让我们使用强大的`UIPercentDrivenInteractionTransition`类来启用视图控制器转换的交互性。
打开`PresentTransition.swift`把下面：

```swift
class PresentTransition: NSObject, UIViewControllerAnimatedTransitioning
```

替换为：

```swift
class PresentTransition: UIPercentDrivenInteractiveTransition, UIViewControllerAnimatedTransitioning {
```

`UIPercentDrivenInteractiveTransition`是一个定义基于“百分比”的转换方法的类，例如：

`update(_:)` 以回退过渡。
`cancel() ` 取消视图控制器转换。
`finish()`  播放转换直到完成。



您可能还记得[19-交互式导航控制器转场](Section_IV.md#19-交互式导航控制器转场)中的这些内容，但您没有看到一些专门用于使用UIViewPropertyAnimator的高级API。

为使动画过渡更容易实现的一些新功能包括：

`timingCurve`：如果您的用户以交互方式驱动转换，并且在需要播放转换时直到结束，您可以通过设置此属性为动画提供自定义时序曲线。 这可以是立方体，弹簧或其他自定义时序提供者。

`wantsInteractiveStart`：默认情况下这是真的，因为您可能主要将此类用于交互式转换。 但是，如果将该属性设置为false，则转换将以非交互方式启动，您可以暂停它并稍后转到交互模式。

`pause()` ：调用此方法暂停非交互式转换并切换到交互模式。

向`PresentTransition`添加一个新方法：

```swift
  func interruptibleAnimator(using transitionContext: UIViewControllerContextTransitioning) -> UIViewImplicitlyAnimating {
    return transitionAnimator(using: transitionContext)
  }
```



这是`UIViewControllerAnimatedTransitioning`协议上的一个方法。 它允许您向UIKit提供可中断的动画师，它将用于您的过渡动画。

您的过渡动画师类现在有两种不同的行为：

1. 如果以非交互方式使用它（当用户按下编辑按钮时），UIKit将调用`animateTransition(using:)`来设置转换动画。
2. 如果以交互方式使用它，UIKit将调用`interruptibleAnimator(using:)`，获取动画师，并使用它来推动这种转场。

![image-20190111124837379](/Users/andyron/Library/Application Support/typora-user-images/image-20190111124837379.png)



切换到LockScreenViewController.swift并在UIViewControllerTransitioningDelegate扩展中添加这个新方法：

```swift
  func interactionControllerForPresentation(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
    return presentTransition
  }
```

这将让UIKit知道您在视图控制器演示期间计划一些有趣的交互性。

接下来，仍然在您打开的文件中，添加两个新属性; 您将需要它们来跟踪用户的手势：

```swift
  var isDragging = false
  var isPresentingSettings = false
```



当用户拉下表时，你会将`isDragging`标志设置为`true`，一旦用户拉得足够远，你将依次将`isPresentingSettings`设置为`true`。

要跟踪用户拉出表视图的距离，您需要实现一些滚动视图委托方法。 添加新扩展并插入第一个方法：

```swift
func scrollViewWillBeginDragging(_ scrollView: UIScrollView) {
    isDragging = true
}
```



这可能看起来有点多余，因为UITableView已经有一个属性来跟踪它当前是否被拖动，但这次你要自己做一些自定义跟踪。
接下来添加委托方法以跟踪用户的进度：

```swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    guard isDragging else { return }

    if !isPresentingSettings && scrollView.contentOffset.y < -30 {
        isPresentingSettings = true
        presentTransition.wantsInteractiveStart = true
        presentSettings()
        return
    }
}
```



首先，检查是否启用了`isDragging`标志; 你不想跟踪表视图的偏移量。 然后检查用户是否已经拉得足够远以开始转换。
如果两个条件都为真，则准备转换设置。 将isPresentingSettings设置为true，将过渡动画设置为交互模式，最后调用presentSettings（）。
presentSettings（）负责在交互模式下启动视图控制器转换，因为您事先将wantsInteractiveStart设置为true。



接下来，您需要添加代码以交互方式更新它。 将以下内容追加到scrollViewDidScroll（_ :)的末尾：

```swift
if isPresentingSettings {
    let progess = max(0.0, min(1.0, ((-scrollView.contentOffset.y) - 30) / 90.0))
    presentTransition.update(progess)
}
```



您可以根据用户拉出表视图的距离计算0.0到1.0范围内的进度，并在过渡动画师上调用update（_ :)以将动画定位到当前进度。
立即尝试过渡，当您向下拖动时，您将看到表格视图逐渐模糊。

![2019-01-11 13-16-13.2019-01-11 13_22_39](/Users/andyron/myfield/temporary/backup image/iOS动画/2019-01-11 13-16-13.2019-01-11 13_22_39.gif)

您还需要注意完成和取消过渡。 添加到与以前相同的扩展名：

```swift
func scrollViewWillEndDragging(_ scrollView: UIScrollView, withVelocity velocity: CGPoint, targetContentOffset: UnsafeMutablePointer<CGPoint>) {
    let progress = max(0.0, min(1.0, ((-scrollView.contentOffset.y) - 30) / 90.0))

    if progress > 0.5 {
        presentTransition.finish()
    } else {
        presentTransition.cancel()
    }

    isPresentingSettings = false
    isDragging = false
}
```



这段代码应该看起来很相似;它与您在第19章“交互式UINavigationController过渡”中使用的方法相同。如果用户已经超过距离的一半（您决定“足够远”），则认为转换成功并将动画播放到最后。如果用户未拖动超过一半的距离，则取消转换。无论哪种方式，您重置两个标志的值，并且转换的交互部分结束。
然而，这种转变尚未完成 - 在生产准备就绪之前，还有很多事情要做好准备。



切换到PresentTransition.swift并找到transitionAnimator（使用:)。在完成块中，忽略该参数并始终使用相同的值调用completeTransition（_ :)。
您可以通过检查动画师完成的位置并提供相关值来帮助UIKit。将现有的addCompletion（...）替换为：

```swift
    animator.addCompletion { (position) in
      switch position {
      case .end:
        transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
      default:
        transitionContext.completeTransition(false)
      }
    }
```




实际上，仅当动画师在其.end位置完成时，视图控制器转换才成功。任何其他情况都意味着转换已被取消，因此您可以直接调用completeTransition（false）。

尝试上下摆动桌子，然后转换开始，但随后取消。 迟早你会看到像这样可怕的东西：

![image-20190111133132818](/Users/andyron/Library/Application Support/typora-user-images/image-20190111133132818.png)

>  注意：此错误似乎已在iOS11中修复，但您仍需要对仍在使用iOS10的所有用户使用下面介绍的修复程序。



这是与视觉效果视图相关的问题。在动画块中设置效果时，如果动画被反转或取消，它似乎无法正确删除，因此您最终会弄得一团糟。还记得你添加到PresentTransition的auxAnimationsCancel属性吗？是时候使用它了。
找到对animator.addCompletion的调用，并在默认值后添加以下行：case：

```swift
self.auxAnimationsCancel?()
```



如果动画师没有完成，你将运行存储在该块中的任何内容。切换回LockScreenViewController.swift并找到presentSettings。在设置auxAnimations属性后，添加以下行：

```swift
presentTransition.auxAnimationsCancel = blurAnimations(false)
```



再次构建和运行，你的像素化问题应该已经消失。但是还有另一个问题。想想你的非交互式过渡。点击编辑。出了点问题！
只要用户点击按钮，您就需要更改代码以将视图控制器转换显式设置为非交互式。

切换回LockScreenViewController.swift并找到小部件表数据源方法tableView（_：cellForRowAt :)。您将看到代码为“编辑”按钮指定了一个闭包，如下所示：

```swift

```



就在self.presentSettings（）行之前，插入：

```swift
self.presentTransition.wantsInteractiveStart = false
```



这可确保您以非交互方式呈现设置视图控制器。 再试一次应用程序，试试过渡。

![2019-01-11 13-40-54.2019-01-11 13_41_51](/Users/andyron/myfield/temporary/backup image/iOS动画/2019-01-11 13-40-54.2019-01-11 13_41_51.gif)





### 可中断的转场动画

接下来，您将考虑在转场期间在非交互模式和交互模式之间切换。

UIViewPropertyAnimator与视图控制器转换的集成旨在解决用户开始转换到另一个控制器的情况，但在中途改变他们的想法。

在本章的这一部分中，您将添加代码以在点击编辑后开始显示设置控制器，但如果用户在动画期间再次点击屏幕，则暂停转换。

切换到PresentTranstion.swift。您需要稍微改变动画师，不仅要分别处理交互式和非交互式模式，还要同时处理相同的过渡。
再添加两个属性：

```swift
  var context: UIViewControllerContextTransitioning?
  var animator: UIViewPropertyAnimator?
```

您将使用这些来跟踪动画的当前背景以及动画师处理其动画。
向下滚动到transitionAnimator（使用:)，并在`return animator`之前的底部，插入以下内容：

```swift
    self.animator = animator
    self.context = transitionContext
```

每次为转换创建新的动画师时，您也会存储对它的引用。

转换完成后释放这些资源也很重要。 在之前插入的两行之后添加一个新的完成块：

```swift
animator.addCompletion { [unowned self] _  in
	self.animator = nil
	self.context = nil
}
```



现在，您可以向类中添加一个方法来中断转换：

```swift
func interruptTransition() {
    guard let context = context else { return }
    context.pauseInteractiveTransition()
    pause()
}
```



您可以调用pauseInteractiveTransitioning（）来暂停动画师，并在转换动画师上暂停（）以使其处于交互模式。
要在非交互式转换期间允许触摸，您必须明确将动画设置为能够处理用户活动。 滚动回transitionAnimator（使用:)并将此行插入底部：

```swift
animator.isUserInteractionEnabled = true
```

您确保过渡动画是交互式的，这样用户可以在暂停后继续与屏幕进行交互。



您将允许用户向上或向下滚动以分别完成或取消转换。 为此，切换回LockScreenViewController.swift并添加一个新属性：

```swift
var touchesStartPointY: CGFloat? 
```

如果用户在转换期间触摸屏幕，您可以将其暂停并存储第一次触摸的位置：

```swift
  override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    guard presentTransition.wantsInteractiveStart == false, presentTransition.animator != nil else {
      return
    }
    
    touchesStartPointY = touches.first!.location(in: view).y
    presentTransition.interruptTransition()
  }
```



您检查触摸是否在非交互式转换期间发生，并且转换的动画师当前正在运行。 在这种情况下，您存储当前触摸位置，然后调用自定义方法interruptTransition（）。 这将暂停转换并使其保持交互模式。
运行应用程序，点击编辑，然后再次快速再次点击。 转换将在屏幕上冻结，如下所示：



现在，您需要跟踪用户触摸并查看用户是向上还是向下平移。 添加以下内容：

```swift
 override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
  guard let startY = touchesStartPointY else { return }
  
  let currentPoint = touches.first!.location(in: view).y
  if currentPoint < startY - 40 {
    touchesStartPointY = nil
    presentTransition.animator?.addCompletion({ (_) in
      self.blurView.effect = nil
    })
    presentTransition.cancel()
    
  } else if currentPoint > startY + 40 {
    touchesStartPointY = nil
    presentTransition.finish()
  }
}
```



通过这个相当大的代码块，您的非交互式转换交互式转换已经完成！
你观察了两种不同的情况。 首先，如果用户将其触摸向下移动超过40点，则取消转换并重置模糊效果。 如果用户向上移动触摸超过40点，则表示您已成功完成转换。
最后一次尝试应用程序。 点击编辑，再次点击以暂停转换，并根据您平移的方向取消或完成转换。
这就是本书的这一部分！

你已经学到了很多关于UIViewPropertyAnimator以及如何充分利用它的知识。 你已经完成了相当冗长的四章，但是你取得了很多成就，而且项目看起来很棒：

