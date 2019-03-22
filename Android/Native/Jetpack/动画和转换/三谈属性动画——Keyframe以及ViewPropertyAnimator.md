经过[初识属性动画——使用Animator创建动画](初识属性动画——使用Animator创建动画.md)和[再谈属性动画——介绍以及自定义Interpolator插值器](再谈属性动画——介绍以及自定义Interpolator插值器.md)，对属性动画已经介绍的差不多了，还剩下最后两个概念，Keyframe和ViewPropertyAnimator。  
## Keyframe  
动画归根结底是一些帧的组合，一旦设定了一个动画后，中间的每帧，Android系统会帮我们计算好，而[Keyframe](https://developer.android.google.cn/reference/android/animation/Keyframe)允许我们定义动画中的一些关键帧，该对象主要有fraction和value组成，其中fraction代表着动画的进度、value代表着动画的值，可以设置单独的Interpolator，这个Interpolator作用于前一帧与当前帧。  
举个例子：  
```
val kf1 = Keyframe.ofFloat(0.2f, 100f).apply {
            interpolator = AnticipateInterpolator()
        }
        val kf2 = Keyframe.ofFloat(0.4f, 200f).apply {
            interpolator = LinearInterpolator()
        }
        val kf3 = Keyframe.ofFloat(0.6f, 300f)
        val kf4 = Keyframe.ofFloat(0.8f, 400f).apply {
            interpolator = BounceInterpolator()
        }
        val kf5 = Keyframe.ofFloat(1.0f, 500f).apply {
            interpolator = SpringInterpolator(0.2f)
        }

        btnMove.setOnClickListener {
            ObjectAnimator.ofPropertyValuesHolder(tvShow,
                    PropertyValuesHolder.ofKeyframe(View.TRANSLATION_Y, kf1, kf2, kf3, kf4, kf5)).apply {
                duration = 3000
                start()
            }
        }
```
上面代码定义了5个Keyframe，分别设置了不同的Interpolator，然后再用PropertyValuesHolder包装一下，最终效果如下：  
![Keyframe效果](https://ws3.sinaimg.cn/large/006tNc79ly1fze98kil37g30bv0jx74y.gif)  
## ViewPropertyAnimator  
如果想在一个View上使用属性动画，可以这么操作：  
```
val animX = ObjectAnimator.ofFloat(myView, "x", 50f)
val animY = ObjectAnimator.ofFloat(myView, "y", 100f)
AnimatorSet().apply {
    playTogether(animX, animY)
    start()
}
```
当然，也可以这么操作：  
```
val pvhX = PropertyValuesHolder.ofFloat("x", 50f)
val pvhY = PropertyValuesHolder.ofFloat("y", 100f)
ObjectAnimator.ofPropertyValuesHolder(myView, pvhX, pvhY).start()
```
可以发现，都是挺麻烦的。View作为最常被动画的对象，Android提供了一种封装，这就是[ViewPropertyAnimator](https://developer.android.google.cn/reference/android/view/ViewPropertyAnimator)，使用方式也是很简单，比如上面的代码等效于：  
```
myView.animate().x(50f).y(100f)
```
View.animate()方法会返回一个ViewPropertyAnimator，该对象具备View的常用属性的变换，比如：  
- transitionX、transitionY、transitionZ
- rotation、rotationX和rotationY  
- scaleX、scaleY  
- x、y、z
- alpha  

## 总结  
至此，学习完了属性动画的知识点，属性动画在Android Transition框架中很重要，是构成各种转场动画的关键，会实现各种酷炫的动画是很厉害的，但其实都离不开这些基础，剩下的更多是数学。  

关于代码，参考[Github](https://github.com/wangli135/ClimbDemo/tree/master/jetpackdemo/src/main/java/com/xingfeng/jetpackdemo/animation/propertyanimator)  

## 参考  
- https://developer.android.google.cn/guide/topics/graphics/prop-animation#keyframes  
- https://developer.android.google.cn/reference/android/animation/Keyframe  
- https://developer.android.google.cn/reference/android/view/ViewPropertyAnimator  

