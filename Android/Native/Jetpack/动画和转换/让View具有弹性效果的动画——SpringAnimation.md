SpringAnimation和FlingAnimation一样，是DynamicAnimation的两种类型。Spring模拟的是物理世界的弹力，弹弹弹，弹走鱼尾纹，，，  
先看下效果：  
![demo示例](https://ws4.sinaimg.cn/large/006tNc79ly1fz7berefrgg30bv0jxe81.gif)  
在某些参数下，可以看到图片有来回震荡的效果。  
## SpringAnimation的基本使用 
1. 添加支持库  
```
 dependencies {
      implementation 'com.android.support:support-dynamic-animation:28.0.0'
  }
```
2. 代码实现  
```
SpringAnimation(img, DynamicAnimation.TRANSLATION_Y, 0f) 
```
### SpringForce
SpringForce定义着动画的各种属性值，其中有两个重要的属性：DampingRatio和Stiffness。
DampingRatio可以理解成反弹次数，值越大，反弹次数越少；值为1，则不反弹。  
Stiffness可以理解成要恢复成未拉伸状态所需的时间，值越大，恢复到之前的状态的时间就越短。 
> DampingRatio为0将无限震荡，就是无阻尼状态。这个时候是不能通过skipToEnd()取消动画的。

Demo中的例子就是调节这两个属性，然后就会有不同的效果。  
代码如下：  
```
class SpringAnimationActivity : AppCompatActivity() {

    lateinit var xSpringAnimation: SpringAnimation
    lateinit var ySpringAnimation: SpringAnimation

    var xDiffLeft: Float? = null
    var yDiffTop: Float? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_spring_animation)

        val springForce = SpringForce(0f).apply {
            setDampingRatio(SpringForce.DAMPING_RATIO_HIGH_BOUNCY)
            setStiffness(SpringForce.STIFFNESS_HIGH)
        }

        dampingRatioGroup.setOnCheckedChangeListener { group, checkedId ->
            when (checkedId) {
                R.id.dampingRatioHigh -> springForce.dampingRatio = SpringForce.DAMPING_RATIO_HIGH_BOUNCY
                R.id.dampingRatioMedium -> springForce.dampingRatio = SpringForce.DAMPING_RATIO_MEDIUM_BOUNCY
                R.id.dampingRatioLow -> springForce.dampingRatio = SpringForce.DAMPING_RATIO_LOW_BOUNCY
                R.id.dampingRatioNo -> springForce.dampingRatio = SpringForce.DAMPING_RATIO_NO_BOUNCY
            }
        }

        stiffnessGroup.setOnCheckedChangeListener { group, checkedId ->
            when (checkedId) {
                R.id.stiffnessHigh -> springForce.stiffness = SpringForce.STIFFNESS_HIGH
                R.id.stiffnesMedium -> springForce.stiffness = SpringForce.STIFFNESS_MEDIUM
                R.id.stiffnesLow -> springForce.stiffness = SpringForce.STIFFNESS_LOW
                R.id.stiffnesVeryLow -> springForce.stiffness = SpringForce.STIFFNESS_VERY_LOW
            }
        }

        xSpringAnimation = SpringAnimation(ivImg, DynamicAnimation.TRANSLATION_X).setSpring(springForce)
        ySpringAnimation = SpringAnimation(ivImg, DynamicAnimation.TRANSLATION_Y).setSpring(springForce)

        ivImg.setOnTouchListener { v, event ->
            val actionCode = event.action
            if (actionCode == MotionEvent.ACTION_DOWN) {
                xDiffLeft = event.rawX - ivImg.x
                yDiffTop = event.rawY - ivImg.y
                xSpringAnimation.cancel()
                ySpringAnimation.cancel()
            } else if (actionCode == MotionEvent.ACTION_MOVE) {
                ivImg.x = event.rawX - xDiffLeft!!
                ivImg.y = event.rawY - yDiffTop!!
            } else if (actionCode == MotionEvent.ACTION_UP) {
                xSpringAnimation.start()
                ySpringAnimation.start()
            }
            true
        }
    }
}
```
### 自定义属性 
可以动画的不止DynamicAnimation中定义的一些属性，下面的例子，定义了scale的属性，起始就是将scaleX和scaleY进行了整合，代码如下：
```
SpringAnimation(it, object : FloatPropertyCompat<View>("scale") {
                    override fun setValue(view: View?, scale: Float) {
                        view!!.scaleX = scale
                        view!!.scaleY = scale
                    }

                    override fun getValue(view: View?): Float {
                        return view!!.scaleX
                    }
                }).setSpring(SpringForce(2f)
                        .setDampingRatio(SpringForce.DAMPING_RATIO_HIGH_BOUNCY)
                        .setStiffness(SpringForce.STIFFNESS_LOW))
                        .setMinimumVisibleChange(0.05f)
                        .start()
```
需要调用setMinimumVisibleChange()方法，设置最小可见变化，不然没有震荡的效果。  
关于setMinimumVisibleChange()的原则，参考https://developer.android.com/guide/topics/graphics/fling-animation?hl=zh-cn#setting-minimum-visible-change  
效果如下：  
![自定义属性](https://ws4.sinaimg.cn/large/006tNc79ly1fz7bqxzfzvg30bv0jxts8.gif) 
具体代码如下：  
```
var changed = false
        val sForce = SpringForce()
                .setDampingRatio(SpringForce.DAMPING_RATIO_HIGH_BOUNCY)
                .setStiffness(SpringForce.STIFFNESS_LOW)
        val propertyCompat = object : FloatPropertyCompat<View>("scale") {
            override fun setValue(view: View?, scale: Float) {
                view!!.scaleX = scale
                view!!.scaleY = scale
            }

            override fun getValue(view: View?): Float {
                return view!!.scaleX
            }
        }
        ivImg2.setOnClickListener {
            if (!changed) {
                SpringAnimation(it, propertyCompat).setSpring(sForce.setFinalPosition(2f))
                        .setMinimumVisibleChange(0.00390625f)
                        .start()
            } else {
                SpringAnimation(it, propertyCompat).setSpring(sForce.setFinalPosition(1f))
                        .setMinimumVisibleChange(0.00390625f)
                        .start()
            }
            changed = !changed
        }
```
## Chained Spring效果  
曾经做过一个需求，点击发布按钮，弹出几个发布选项，有一个弹簧效果，下面仿写了这个效果，如下图：  
![Chained Spring效果](https://ws1.sinaimg.cn/large/006tNc79ly1fz7bsrto3pg30bv0jxtad.gif)
可以看到联动的效果，最左边的带动中间，中间再带动最右边的。  
实现主要是通过addUpdateListener()以及startToFinalPosition()实现的。  
代码如下：
```
class ChainedSpringActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_chained_spring)

        val springForce = SpringForce(-600f).apply {
            dampingRatio = SpringForce.DAMPING_RATIO_HIGH_BOUNCY
            stiffness = SpringForce.STIFFNESS_LOW
        }
        val tvTextSpringAnimation = SpringAnimation(tvPublishText,
                DynamicAnimation.TRANSLATION_Y)
        val tvVideoSpringAnimation = SpringAnimation(tvPublishVideo,
                DynamicAnimation.TRANSLATION_Y).apply {
            addUpdateListener { animation, value, velocity ->
                tvTextSpringAnimation.animateToFinalPosition(value)
            }
        }
        val tvPicSpringAnimation = SpringAnimation(tvPublishPic,
                DynamicAnimation.TRANSLATION_Y).apply {
            spring = springForce
            addUpdateListener { dynamicAnimation, value, velocity ->
                tvVideoSpringAnimation.animateToFinalPosition(value)
            }
        }
        fab.setOnClickListener {
            tvPicSpringAnimation.start()
        }
        
    }
}
```
## 取消动画  
- cancel()：立即停止动画
- skipToEnd()：恢复到最终位置并停止动画。**需要注意的是，在无阻尼的情况下，不能调用该方法，为了安全，可以先调用canSkipToEnd()进行判断，有阻尼的情况下返回true，否则返回false**    

> 一般来说，skipToEnd()会有跳跃的效果。

## 总结  
SpringAnimation主要是通过设置SpringForce进行动画的控制，SpringForce的DampingRatio和Stiffness分别表示阻尼系数和生硬度，DampingRatio越大，反弹次数越少，Sniffness越大，达到最终位置的时间就越短。


## 参考文章  
- https://developer.android.com/guide/topics/graphics/spring-animation?hl=zh-cn
- https://www.jianshu.com/p/46b1cdc253e9 
- https://proandroiddev.com/introduction-to-physics-based-animations-in-android-1be27e468835
- https://medium.com/@temidjoy/android-animations-and-transistions-fling-animation-f5bf42bfef55  

  