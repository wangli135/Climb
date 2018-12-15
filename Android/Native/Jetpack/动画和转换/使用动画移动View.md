经常会遇到屏幕上的View位置更新的情况，这时如果中间添加一些动画，会显得很自然并且很酷炫，而不是硬巴巴的。  

Android中，修改View的位置方式很多，修改x、y、params以及动画之类的。本文将介绍使用ObjectAnimator进行View的移动。  

## 使用ObjectAnimator改变View位置  

ObjectAnimator类提供了很简单的方式操作View的属性，提供了静态方法根据你想要操作的属性生成ObjectAnimator实例。因为想移动View的位置，这里考虑View的属性是translationX和translationY。  

下面的例子将TextView在2s里从左向右移动了400像素，代码如下：

```Kotlin
moveBtn.setOnClickListener {
            ObjectAnimator.ofFloat(tvShow, "translationX", 400f)
                    .apply {
                        duration = 2000
                        start()
                    }
        }
```

效果如下：  

![移动View效果](https://ws2.sinaimg.cn/large/006tNbRwly1fy7feclmurg30c40k8mxi.gif)

关于ObjectAnimator的介绍，后续将有文章详细介绍，敬请期待。在这之前，可以看[官方文档](https://developer.android.com/guide/topics/graphics/prop-animation#object-animator)  

## 添加弯曲动作  

ObjectAnimator的使用很简单，但是只提供了直线移动。使用弯曲动画可以让效果更生动，更符合Google的Material Design。  

### 使用PathInterpolator  

PathInterpolator是Android 5.0新加入的类，基于贝塞尔曲线或Path对象。这个类指定弯曲动画在一个1*1的正方形中，并且锚点在（0，0）和（1，1）,其他控制点则通过构造方法传入。  创建PathInterpolator的一种方式是创建Path对象，然后将它应用于PathInterpolator上。  

比如：  

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    val path = Path().apply {
        arcTo(0f, 0f, 1000f, 1000f, 270f, -180f, true) 
    } 
    val pathInterpolator = PathInterpolator(path)
} 
```

当然，也可以通过在xml中定义PathInterpolator，比如：  

```
<pathInterpolator xmlns:android="http://schemas.android.com/apk/res/android"
    android:controlX1="0.4"
    android:controlY1="0"
    android:controlX2="1"
    android:controlY2="1"/>
```

应用给View，必须这么使用：  

```
val path = Path().apply {
                lineTo(400f, 0f)
                moveTo(400f, 0f)
                arcTo(0f, 0f, 1000f, 1000f, 270f, -180f, true)
                lineTo(400f, 400f)
            }
            ObjectAnimator.ofFloat(tvShow, View.X, View.Y, path).apply {
                duration = 2000
                start()
            }
```

效果如下：  

![完全动画](https://ws4.sinaimg.cn/large/006tNbRwly1fy7gnjziyjg30c40k8wen.gif)

可以看到，将根据Path定义的顺序执行。

关于官方代码中下段代码：  

```
val animation = ObjectAnimator.ofFloat(view, "translationX", 100f).apply {
    interpolator = pathInterpolator 
    start() 
} 
```

执行后，App将会崩溃，崩溃原因是Path must be start at（0，0）and end at（1，1）。

但是这样的代码是可行的。  

```
val path = Path().apply {
                cubicTo(0.2f,0f,0.1f,1f,0.5f,1f)
                lineTo(1f,1f)
            }
            ObjectAnimator.ofFloat(tvShow, "translationX", 400f).apply {
                duration = 2000
                interpolator = PathInterpolator(path)
                start()
            }
```

或者这样的Path，  

```
val path = Path().apply {
                lineTo(0.2f,0.2f)
                moveTo(0.2f,0.2f)
                lineTo(1f,1f)
            }
```

当动画中使用了最后值时，这时Path必须得以（0，0）开始，（1，1）结束；而不规定结束值，那么就按照Path运作的最后值停留。  

系统默认提供了三种PathInterpolator，分别是:  

1. `@interpolator/fast_out_linear_in.xml`  
2. `@interpolator/fast_out_slow_in.xml`
3. `@interpolator/linear_out_slow_in.xml`

参考：  

- https://developer.android.com/training/animation/reposition-view
- https://www.jianshu.com/p/e25f7aca3e73