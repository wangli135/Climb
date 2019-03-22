Google除了提供了属性动画之外，还提供了一种基于物理的动画，叫做DynamicAnimation,与物理世界更贴近，关于这块可以参考https://www.jianshu.com/p/46b1cdc253e9。  

目前主要有两种DynamicAnimation，分别是：  

- [Spring Animation](https://developer.android.com/guide/topics/graphics/spring-animation?hl=zh-cn) 类比弹力  

- [Fling Animation](https://developer.android.com/guide/topics/graphics/fling-animation?hl=zh-cn) 类比速度、动量  

本文主要介绍Fling Animation。话不多说，先看下官方demo示例：  

![Fling Animation官方demo](https://developer.android.com/images/guide/topics/graphics/fling-animation.gif?hl=zh-cn)

在松手后，会继续有动画的效果，逐渐减慢直至停止，是不是和现实生活中很类似？因为有摩擦力，所以会不断减少，这时高中老师教给我们的牛顿力学可以发挥用场了。  

再来看下本文最终的demo示例：  

![本文demo示例](https://ws3.sinaimg.cn/large/006tNc79ly1fz6bxxvqy5g30bw0jbtjs.gif)  
拖动ImageView，松手的一瞬间，如果垂直方向的加速度大于水平方向的，那么垂直方向进行动画；反之水平方向运动，运动范围限制在屏幕中。  
## FlingAnimation的使用
FlingAnimation的使用主要分为两步骤：
1. 添加支持库  

  ```
  dependencies {
        implementation 'com.android.support:support-dynamic-animation:28.0.0'
    }
  ```

2. 创建一个FlingAnimation  
  ```kotlin
  val fling = FlingAnimation(view, DynamicAnimation.SCROLL_X)
  ```

  FlingAnimation的创建需要指定View以及动画的属性，接下来就是设置一些属性，  

  - setStartVelocity(float)：设置起始加速度，单位是改变的属性每秒，默认是0。如果需要使用dp转pixel，可以使用下段代码：    
  ```
  float pixelPerSecond = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dpPerSecond,
         getResources().getDisplayMetrics());
  ```
  - setMinValue(float)：设置动画的最小值。这个值是创建FlingAnimation中的属性值的最小值，也就是说属性值不过小于该值。  
  - setMaxValue(float)：与上面类似，只不过是最大值，min<=属性值<=max。  
  - setFriction(float)：设置摩擦力，学过力学的都知道，没有摩擦力，那么将一直运动下去；而摩擦力越大，那么将会越快停止，默认值是1。  
  - setMinimumVisibleChange(float)：当创建一个单位不是pixel的自定义属性时，需要设置该值；DynamicAnimation.ViewProperty里面的属性是不需要设置该值的。  

## Demo示例代码

学完了理论知识，就看一下代码了，布局很简单，就一个ImageView，将touch事件交给了GestureDetector，然后在onFling()方法中实现FlingAnimation动画；有一点需要注意的是，FlingAnimation改变的是transitionX和transitionY属性，为了限制在屏幕内动画，因此计算了x和y方向的最大值，具体代码如下：  

```Kotlin
class FlingAnimationActivity : AppCompatActivity() {

    lateinit var gestureDetector: GestureDetector
    var maxTransitionX: Int? = null
    var maxTransitionY: Int? = null
    
    private val gestureListener = object : GestureDetector.SimpleOnGestureListener() {
        override fun onDown(e: MotionEvent?): Boolean {
            return true
        }

        override fun onFling(e1: MotionEvent?, e2: MotionEvent?, velocityX: Float, velocityY: Float): Boolean {
            if (Math.abs(velocityX) > Math.abs(velocityY)) {
                FlingAnimation(ivImg, DynamicAnimation.TRANSLATION_X).apply {
                    setStartVelocity(velocityX)
                    setMinValue(0f)
                    setMaxValue(maxTransitionX!!.toFloat())
                    friction = 1.1f
                    start()
                }
            } else {
                FlingAnimation(ivImg, DynamicAnimation.TRANSLATION_Y).apply {
                    setStartVelocity(velocityY)
                    setMinValue(0f)
                    setMaxValue(maxTransitionY!!.toFloat())
                    friction = 1.1f
                    start()
                }
            }
            return true
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_fling)

        mainLayout.viewTreeObserver.addOnGlobalLayoutListener {
            maxTransitionX = mainLayout.width - ivImg.width
            maxTransitionY = mainLayout.height - ivImg.height
        }

        gestureDetector = GestureDetector(this, gestureListener)

        ivImg.setOnTouchListener { v, event ->
            gestureDetector!!.onTouchEvent(event)
        }
    }
}
```

## 参考文章  

- https://developer.android.com/guide/topics/graphics/fling-animation?hl=zh-cn
- https://developer.android.com/reference/android/support/animation/FlingAnimation
- https://proandroiddev.com/introduction-to-physics-based-animations-in-android-1be27e468835
- https://medium.com/@temidjoy/android-animations-and-transistions-fling-animation-f5bf42bfef55
- https://www.jianshu.com/p/46b1cdc253e9  

  