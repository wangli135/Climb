Android的Transition框架提供了在两个布局，即Scene之间的自动过渡，内置了AutoTransition、Fade、ChangeBounds动画，除此之外，也提供了给我们自定义Transition的途径。先看个效果：  
![自定义Transition效果](https://ws4.sinaimg.cn/large/006tNc79ly1fze79ro0p5g30bv0jxq5n.gif)  
可以看到，在两个场景中，背景色在渐渐变化。 ## 自定义Transition  
将以上述例子，说明自定义Transition的步骤。  
### 继承Transition  
继承Transition，代码框架如下：  
```
class CustomTransition : Transition() {

    override fun captureStartValues(transitionValues: TransitionValues) {}

    override fun captureEndValues(transitionValues: TransitionValues) {}

    override fun createAnimator(
        sceneRoot: ViewGroup,
        startValues: TransitionValues?,
        endValues: TransitionValues?
    ): Animator? {}
}
```
其中前两个方法是Transition中的抽象方法，为记录开始布局和结束布局的属性值所用；第三个方法为创建过渡动画，可以看到是一个属性动画Animator。  
### 获取View属性值  
通过对已有Transition代码的学习以及官方指导，可以这么写：  
```
override fun captureStartValues(transitionValues: TransitionValues?) {
        catureValues(transitionValues)
    }

    private fun catureValues(transitionValues: TransitionValues?) {
        transitionValues?.values!![BACKGROUND_KEY] = transitionValues?.view?.background
    }

    override fun captureEndValues(transitionValues: TransitionValues?) {
        catureValues(transitionValues)
    }
```
最终都调用captureValues()方法，在该方法中保存属性值，属性值保存在transitionValues的values中，该values是一个map，这里的BACKGROUND_KEY的定义如下：  
```
    private val BACKGROUND_KEY = "com.xingfeng.jetpackdemo.animation.layoutanimate:CustomTransition:background"
```
官方建议我们以"package_name:class_name:property_name"命名一个Key，这样可以很好的避免重复。  
### 创建过渡动画  
接下来就是在createAnimator()方法中实现动画了，本文的代码如下：  
```
override fun createAnimator(sceneRoot: ViewGroup?, startValues: TransitionValues?, endValues: TransitionValues?): Animator? {
        if (startValues == null || endValues == null) {
            return null
        }

        val startDrawable = startValues?.values!![BACKGROUND_KEY] as Drawable
        val endDrawable = endValues?.values!![BACKGROUND_KEY] as Drawable
        if (startDrawable is ColorDrawable && endDrawable is ColorDrawable) {
            val startColor = startDrawable.color
            val endColor = endDrawable.color
            if (startColor != endColor) {
                return ObjectAnimator.ofObject(ArgbEvaluator(), startColor, endColor).apply {
                    addUpdateListener {
                        endValues?.view!!.setBackgroundColor(it?.animatedValue as Int)
                    }
                    duration = 3000
                }
            }
        }
        return null
    }

```
createAnimator()方法的参数有三个，其中后两个保存了起始和结束的属性值，通过key可以取出值，这里如果两个view的背景是ColorDrawable，那么将执行一个动画，否则为null。
## 总结  
可以看到自定义Transition的流程是比较简单的，主要可以说是两步：  
1. 在captureValues()方法中保存刚兴趣的属性值，该方法是captureStartValues()和captureEndValues()的委托；  
2. 在createAnimator()中取出感兴趣的属性，创建属性动画即可。  


关于代码，请参考[Github地址](https://github.com/wangli135/ClimbDemo/tree/master/jetpackdemo/src/main/java/com/xingfeng/jetpackdemo/animation/layoutanimate)

## 参考  
- https://developer.android.google.cn/training/transitions/custom-transitions
- https://medium.com/@belokon.roman/custom-transitions-in-android-f8949870bd63 
- https://github.com/lgvalle/Material-Animations

