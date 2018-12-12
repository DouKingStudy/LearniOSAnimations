# 系统学习iOS动画之六：3D动画



到目前为止，之前的文章只使用了二维动画——这是在平面设备屏幕上动画元素的最自然方式。 毕竟，从iOS 7扁平化后的世界中的按钮，文本字段，开关和图像没有了第三维; 这些元素存在于由X和Y轴定义的平面中：

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fx962j0d7jj309a03g0sj.jpg)

**核心动画**可以帮助我们摆脱这个二维世界; 虽然它不是真正的3D框架，但**核心动画**有很多好的方法可以帮助我们在3D空间中描绘二维对象。

换句话说，图层和动画仍然以二维方式进行描绘，但可以在3D空间中旋转和定位每个元素的2D平面，如下所示：

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fx9630g50tj30ab04jdfv.jpg)

上面显示的是在3D空间中旋转的两个2D图像。 透视变形使我们可以从渲染器的角度了解它们的位置。

本文将学习如何在3D空间中定位和旋转图层。`CATransform3D`类似于`CGAffineTransform`，但除了在x和y方向上缩放，倾斜和平移之外，它还带来了第三维：z。 z轴直接从设备屏幕朝向您的眼睛。

请考虑以下几个示例，以更好地了解透视的工作原理。

将相机设置得非常靠近屏幕会相应地扭曲图层的视角：

![image-20181204230708868](https://ws1.sinaimg.cn/large/006tNbRwgy1fxv44v769fj30b60a4jsk.jpg)

如果将相机离物体比较远时的视角：

![image-20181204230723724](https://ws2.sinaimg.cn/large/006tNbRwgy1fxv45b8kb6j309s0anwfo.jpg)

最后，如果你在相机和屏幕之间设置了很大的距离：

![image-20181204230830029](https://ws2.sinaimg.cn/large/006tNbRwgy1fxv469w6efj309508kq3v.jpg)



[24-简单的3D动画](#24-简单3D动画) —— 尝试新发现的有关相机距离和视角的知识。设置图层的透视图，处理图层的变换以旋转，平移和缩放三维图层。

[25-中级3D动画](#25-中级3D动画) —— 在前一章的基础上，既然你知道了m34和相机距离的秘密，你就可以创建具有多个视图的各种3D动画。



## 24-简单3D动画



本章将尝新发现的有关相机距离和视角的知识。

开始项目 [Office Buddy](README.md#关于代码)是一个办公室帮助应用程序，供员工访问有关日常公司生活的分类信息。这个应用很简单就是点击左上角的按钮或者左右滑到，然后左边侧栏出现。下面👇将向这个开始项目中添加一些3D元素。

开始项目预览：

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxv4fvitbsg308q0fntd3.gif)



### 创造3Dtransformations



打开`ContainerViewController.swift`，`ContainerViewController`在屏幕上显示菜单视图控制器和内容视图控制器。 它还处理平移手势，以便用户可以打开和关闭菜单。

您的第一个任务是构建一个类方法，该方法为侧面菜单的给定百分比“开放性”创建相应的3D变换。
将以下方法声明添加到`ContainerViewController`：

```swift
func menuTransform(percent: CGFloat) -> CATransform3D {

}
```

上述方法接受菜单当前进度的单个参数，该参数由`handleGesture(_ :)`中的代码计算，并返回`CATransform3D`的实例。 您将直接将此方法的结果分配给菜单图层的transform属性。

将以下代码添加到上面方法中：

```swift
var identity = CATransform3DIdentity
identity.m34 = -1.0/1000
```

这段代码可能看起来有点令人惊讶; 到目前为止，您只使用函数来创建或修改变换。 但是，这一次，您正在修改其中一个类的属性。

>  注意：`CATransform3D`和`CGAffineTransform`分表表示`4*4`和`3*3`的数学[矩阵](https://zh.wikipedia.org/wiki/矩阵)，在Swift和OC中都是用结构体表示的。
>
>  属性`m34`指矩阵的第3行第4列，这个属性比较常用，表示透视效果，`m34 = -1 / D`，D可以理解为**相机距离**，D越小，透视效果越明显，必须在有旋转效果的前提下，才会看到透视效果。



### 相机距离 

对于普通应用程序中的UI元素，相机距离大概可以表示：
*0.1 ... 500*：非常接近，透视失真。
*750 ... 2,000*：视角不错，内容清晰可见。
*2000+*：几乎没有透视失真。

对于Office Buddy应用程序，1000点的距离将为菜单提供一个非常微妙的视角。 

将以下代码添加到`menuTransform(percent:)`的底部：

```swift
let remainingPercent = 1.0 - percent
let angle = remainingPercent * .pi * -0.5
```

将以下代码添加到`menuTransform(percent:)`的底部：

```swift
let rotationTransform = CATransform3DRotate(identity, angle, 0.0, 1.0, 0.0)
let translationTransform = CATransform3DMakeTranslation(menuWidth * percent, 0, 0)
return CATransform3DConcat(rotationTransform, translationTransform)
```

在这里，使用`rotationTransform`将图层绕y轴旋转。 菜单从左侧移动，因此还需要创建平移变换以沿x轴移动它，最终将菜单宽度设置为100％。 最后，连接两个转换并返回结果。



从`setMenu(toPercent:)`中删除下面：

```swift
menuViewController.view.frame.origin.x = menuWidth * CGFloat(percent) - menuWidth
```

替代为： 

```swift
menuViewController.view.layer.transform = menuTransform(percent: percent)
```

菜单栏的位置通过转换来控制了。



运行项目， 向右平移查看菜单如何围绕其y轴旋转：

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxvodwgodgg306y08edhk.gif)

菜单以3D形式旋转，但它围绕其水平中心旋转，菜单与内容视图控制器中间有间隙。



### 移动图层的锚点

默认情况下，图层的锚点的x坐标为0.5，表示它位于中心。 将锚点的x设置为1.0，就不会出现上面的那种间隙，如下所示：
![image-20181205104831488](https://ws1.sinaimg.cn/large/006tNbRwgy1fxvoenmznij30a207xq3b.jpg)



所有变换都是围绕图层的锚点计算的。

在`viewDidLoad()`中找到以下行：

```swift
menuViewController.view.frame = CGRect(x: -menuWidth, y: 0, width: menuWidth, height: view.frame.height)
```

现在在该行上方插入以下代码（在设置视图帧之前插入行非常重要，否则设置锚点将偏移视图）：

```swift
menuViewController.view.layer.anchorPoint.x = 1.0
```

这会使菜单围绕其右边缘旋转。

运行效果：

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxvokxjjzcg306y08e764.gif)

这看起来好多了！


### 通过阴影创建远景

阴影为3D动画带来了很多真实感。这里不需要使用任何先进的着色技术，只要旋转时更改`alpha`。

将以下代码添加到`setMenu(toPercent:)`：

```swift
menuViewController.view.alpha = CGFloat(max(0.2, percent))
```

0.2让菜单最小还可见，百分比让菜单越小透明度越低。

由于此应用程序的背景为黑色，因此降低菜单视图的alpha值会使菜单中显示黑色并模拟阴影效果。

运行效果：

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxvoq48apag306y08edhu.gif)

这是一个让3D效果更加真实的小细节。



如果仔细观察，会发现第一次点击按钮时，菜单不是以3D效果展示，以后才是。这是因为第一次切换菜单之前，设置3D动画参数和图层转换。在`viewDidLoad()`中添加：

```swift
setMenu(toPercent: 0.0)
```



### 光栅化的效率

让动画更加“完美”。如果在来回平移时盯着菜单足够长，会注意到菜单项的边框看起来像素化，如下所示：

![image-20181205110350367](https://ws3.sinaimg.cn/large/006tNbRwgy1fxvoukq8jyj30b706tmxa.jpg)

核心动画不断重绘菜单视图控制器的所有内容，并在所有元素移动时重新计算所有元素的透视失真，这个过程中会出现*锯齿状边缘*。

最好让Core Animation知道我们不会在动画期间更改菜单内容，以便它可以渲染菜单一次并简单地旋转渲染和缓存的图像。
这听起来很复杂，但很容易实现。

找到`handleGesture()`中的`.began`代码块，此代码在用户平移操作时执行。

将以下代码添加到`.began`代码块的末尾：

```swift
menuViewController.view.layer.shouldRasterize = true
menuViewController.view.layer.rasterizationScale = UIScreen.main.scale
```

`shouldRasterize`让核心动画将图层内容缓存为图像。 然后设置`rasterizationScale`以匹配当前的屏幕比例。

运行，效果：

![image-20181205110814153](https://ws2.sinaimg.cn/large/006tNbRwgy1fxvoz544nmj308203uwed.jpg)

为避免在使用应用程序时进行任何不必要的缓存，应该在动画完成后立即关闭光栅化。
在`.failed`代码块找到动画完成闭包并添加以下代码：

```swift
self.menuViewController.view.layer.shouldRasterize = false
```

现在，只在动画期间激活光栅化。提高了效率！😊



### 菜单按钮的3D旋转动画

菜单展示时，菜单按钮也进行自身的旋转。具体来说，您将围绕x轴和y轴创建旋转，以使菜单按钮在其对角线上翻转。

在`ContainerViewController`的`setMenu(toPercent:)`中添加：

```swift
let centerVC = centerViewController.viewControllers.first as? CenterViewController
if let menuButton = centerVC?.menuButton {
    menuButton.imageView.layer.transform = buttonTransform(percent: percent)
}
```

`buttonTransform`函数为：

```swift
func buttonTransform(percent: CGFloat) -> CATransform3D {
    var identity = CATransform3DIdentity
    identity.m34 = -1.0/1000

    let angle = percent * .pi
    let rotationTransform = CATransform3DRotate(identity, angle, 1.0, 1.0, 0.0)

    return rotationTransform
}
```


效果如下：

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxvpgah492g306y067799.gif)





## 25-中级3D动画

在上一章[24-简单3D动画](#24-简单3D动画)中，学习了将透视应用到单个视图制作出简单的3D效果的动画； 事实上，一旦我们知道m34和相机距离的秘密，就可以创建各种3D动画。

本章以前面的内容为基础，学习如何使用多个视图创建有意思的3D动画。

本章的[开始项目](README.md#关于代码) ***ImageGallery***是一个简单的飓风图库。 



### 探索开始项目
本章的开始项目是：

![image-20181212190316969](https://ws2.sinaimg.cn/large/006tNbRwgy1fy461l43y4j307m083glg.jpg)

只是一个空白屏幕，顶部有两个按钮。

打开`ViewController.swift`，会看到一个名为`images`的数组，此数组就是一些图片信息。

 `ImagViewCard`类继承自`UIImageView`并且有一个字符串属性`title`来保存飓风标题，有一个名为`didSelect`的属性，以便您可以轻松地在图像上设置点击处理程序。



第一个任务是将所有图像添加到视图控制器的视图中。 将以下代码添加到`viewDidAppeae(_:)`的末尾：

```swift
for image in images {
    image.layer.anchorPoint.y = 0.0
    image.frame = view.bounds

    view.addSubview(image)
}
```

在上面的代码中，循环遍历所有图像，在y轴上将每个图像的锚点设置为0.0，并调整每个图像的大小，使其占据整个屏幕。 设置锚点可让图像围绕其上边缘而不是中心的默认值旋转，如下图所示：

![image-20181205113638430](https://ws1.sinaimg.cn/large/006tNbRwgy1fxvpsq3ekwj30d7068417.jpg)

运行只会看到最后一张图片`Hurricane Irene`，因为图片位置相同，叠加在一起来

显示飓风图像的名字，在`viewDidAppear(_:)`的末尾添加以下行：

```swift
navigationItem.title = images.last?.title
```



注意，目前没有在图像上设置任何透视转换;之后将直接在视图控制器的视图上设置透视图。

在上一章中，在单个视图上调整了`transfor`m属性，然后在3D空间中旋转它。但是，由于您当前的项目有更多的个人视图，需要在3D中操作，您可以设置其父视图的透视图，从而节省大量工作。

将以下代码添加到`viewDidAppear(_:)`：

```swift
var perspective = CATransform3DIdentity
perspective.m34 = -1.0/250.0
view.layer.sublayerTransform = perspective
```

在这里，您可以使用图层属性`sublayerTransform`来设置视图控制器图层的所有子图层的透视图。 然后将子层转换与每个单独层的自身变换组合。

这使您可以专注于管理子视图的旋转或平移，而无需担心透视。 您将在下一节中更详细地了解它的工作原理。

### 改变图库

`toggleGallery(_:)`连接着右上方的“浏览”按钮，在此处将3D变换应用于四个图像。

将以下变量添加到`toggleGallery(_:)`：

```swift
var imageYOffset: CGFloat = 50.0

for subview in view.subviews {
    guard let image = subview as? ImageViewCard else {
        continue
    }
}
```

由于您不只是将所有图像旋转到原位而只是移动它们以产生”扇形“动画，因此您可以使用`imageYOffset`来设置每个图像的偏移。
接下来，您需要遍历所有图像并运行其各自的动画。

在这里，您循环浏览视图控制器视图的所有子视图，并仅对作为`ImageViewCard`实例的子视图执行操作。
在上面添加的`guard`块之后添加以下代码，以替换此处的更多代码注释：

```swift
var imageTransform = CATransform3DIdentity
// 1
imageTransform = CATransform3DTranslate(imageTransform, 0.0, imageYOffset, 0.0)
// 2
imageTransform = CATransform3DScale(imageTransform, 0.95, 0.6, 1.0)
// 3
imageTransform = CATransform3DRotate(imageTransform, .pi/8, -1.0, 0.0, 0.0)
```

首先将标识转换分配给imageTransform，然后对其添加一系列调整。 这是每个单独的调整对图像的作用：

`// 1` 使用`CATransform3DTranslate`在y轴上移动图像; 这会使图像偏离其默认的**0.0 y坐标**，如下所示：

![image-20181212182739840](https://ws2.sinaimg.cn/large/006tNbRwgy1fy450l67o1j30cj06a772.jpg)

之后，将要分别计算每个图像的`imageYOffset`，否则图片还是叠加在一起。

`// 2` 通过使用`CATransform3DScale`调整转换的比例分量来缩放图像。 可以在x轴上稍微缩小图像，但是在y轴上将其缩小到60％以丰富旋转3D效果：

![image-20181212182903109](https://ws3.sinaimg.cn/large/006tNbRwgy1fy451zjly0j30fy08p77t.jpg)

`// 3` 最后，使用`CATransform3DRotate`将图像旋转22.5度，使其具有一些透视变形，如下所示：

![image-20181212182944370](https://ws1.sinaimg.cn/large/006tNbRwgy1fy452p4h3vj30fg082dhq.jpg)

请记住，之前已经设置了锚点，因此图像围绕其顶部边缘旋转。

现在你看到通过view.layer.sublayerTransform设置上面的m34值的值; 您的旋转变换只需重新使用子层变换中的m34值，而无需在此处应用它。 那很方便！

现在剩下的就是将转换应用于每个图像。 添加以下行（仍在for代码块中）：

```swift
image.layer.transform = imageTransform
```


将以下行添加到for块的末尾，修改每个图像的位置：

```swift
imageYOffset += view.frame.height / CGFloat(images.count)
```

这会调整每个图像的y偏移量，具体取决于它在堆栈中的位置。 将屏幕高度除以图像数量，以便它们在屏幕上均匀分布。
运行后效果：

![image-20181205115546758](https://ws2.sinaimg.cn/large/006tNbRwgy1fxvqcmw3j4j308w0fu0ym.jpg)

下面让它动起来！

### 动画图库

在上面的`image.layer.transform = imageTransform`的前面添加：

```swift
let animation = CABasicAnimation(keyPath: "transform")
animation.fromValue = NSValue(caTransform3D: image.layer.transform)
animation.toValue = NSValue(caTransform3D: imageTransform)
animation.duration = 0.33
image.layer.add(animation, forKey: nil)
```

这段代码非常熟悉：在transform属性上创建一个图层动画，并将其从当前值设置为之前设计的`imageTransform`。
运行后， 点击“浏览”按钮，效果：

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxvqjb5gwig308q0fo49f.gif)

你现在已经完成了画廊; 当您在用户点击“浏览”按钮时添加关闭风扇的功能时，您将在“挑战”部分重新访问它。

### 更多一点交互

为图像库添加一点交互性：点击图像，变成全屏，并且位置移到最前面，以便用户可以更好地查看它。

`ImageViewCard`已经具有名为`didSelect`的闭包表达式属性，当用户点击图像，就将点击的图像视图作为输入参数给这个闭包。

首先将以下代码添加`viewDidAppear()`的for循环体内：

```swift
image.didSelect = selectImage
```

在`ViewController`中添加方法：

```swift
func selectImage(selectedImage: ImageViewCard) {

    for subview in view.subviews {
        guard let image = subview as? ImageViewCard else {
            continue
        }
        if image === selectedImage {

        } else {

        }
    }
}
```



现在您还需要两个动画：一个用于为所选图像设置动画，另一个用于为图库中的所有其他图像设置动画。
你将反过来解决这个问题并首先淡出未选择的图像。

上面的方法还缺少两个动画，当`image === selectedImage`，就是所选图像的动画；或者，未选择的所有其他图像的动画，前者代码为：

```swift
UIView.animate(withDuration: 0.33, delay: 0.0, options: .curveEaseIn, animations: {
    image.alpha = 0.0
}, completion: { (_) in
    image.alpha = 1.0
    image.layer.transform = CATransform3DIdentity
})
```

后者代码为：

```swift
UIView.animate(withDuration: 0.33, delay: 0.0, options: .curveEaseIn, animations: {
    image.layer.transform = CATransform3DIdentity
}, completion: {_ in
    self.view.bringSubview(toFront: image)
})
```

在这里，没有对动画进行3D变换，然后确保图像位于视图堆栈的顶部，以便它可见。



最后，将以下代码添加到`selectImage(selectedImage:)`的末尾，更新标题：

```swift
self.navigationItem.title = selectedImage.title
```



### 切换图库

这小结工作是将使“浏览”按钮可以关闭图库视图。

向`ViewController`添加一个`isGalleryOpen`的新属性，并将其初始值设置为`false`。 

需要在代码中的两个位置更新此属性的值：

- 在`toggleGallery(_:)`结束时将其设置为`true`
- 在`selectImage(selectedImage:)`结束时将其设置为`false`



在`toggleGallery()`的顶部，添加一个检查以查看图库是否已打开。 如果打开，则遍历所有图像并将其转换设置为原始值。 不要忘记重置`isGalleryOpen`并返回，因此其余的方法代码也不会执行。

```swift
if isGalleryOpen {
    for subview in view.subviews {
        guard let image = subview as? ImageViewCard else {
            continue
        }

        let animation = CABasicAnimation(keyPath: "transform")
        animation.fromValue = NSValue(caTransform3D: image.layer.transform)
        animation.toValue = NSValue(caTransform3D: CATransform3DIdentity)
        animation.duration = 0.33

        image.layer.add(animation, forKey: nil)
        image.layer.transform = CATransform3DIdentity

    }

    isGalleryOpen = false
    return
}
```



本章的最后效果：

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxvr9roxswg308q0fo7wh.gif)