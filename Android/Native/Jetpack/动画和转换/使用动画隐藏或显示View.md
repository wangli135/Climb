一般来说，当内容更换时，有动画的话会更好过渡，用户也会体验较好。有三种比较常见的动画用于隐藏或显示内容：Circle Reveal动画、淡入淡出效果、卡片翻转效果。  

下面将分别介绍这三种常见的动画效果：  

## 淡入淡出动画  

淡入淡出动画一般是一个View在渐渐消失，另一个View同时在渐渐出现。 

先看效果，如下图：  
![淡入淡出效果动画](https://ws2.sinaimg.cn/large/006tNbRwly1fy0fpxr8zrg309n0h2k8c.gif)

可以看到效果是一个文本渐渐出现，loading渐渐消失。  

### 创建xml布局  

```
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".animation.CrossFadeActivity">

    <ScrollView
        android:id="@+id/content"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            style="?android:textAppearanceMedium"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:lineSpacingMultiplier="1.2"
            android:padding="16dp"
            android:text="@string/large_text"/>

    </ScrollView>

    <ProgressBar
        android:id="@+id/loading_spinner"
        style="?android:progressBarStyleLarge"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"/>
</FrameLayout>
```

### 代码实现动画  

```
class CrossFadeActivity : AppCompatActivity() {

    private var mShortAnimationDuration: Int = 5000

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_cross_fade)
        crossFade()
    }

    private fun crossFade() {
        content.apply {
            visibility = View.VISIBLE
            alpha = 0f

            animate().alpha(1.0f)
                    .setDuration(mShortAnimationDuration.toLong())
                    .withEndAction {
                        content.visibility = View.VISIBLE
                    }.start()

        }

        loading_spinner.animate()
                .alpha(0.0f)
                .setDuration(mShortAnimationDuration.toLong())
                .withEndAction {
                    loading_spinner.visibility = View.GONE
                }.start()

    }

}
```

这里使用的属性动画，初始时内容alpha=0，渐渐变为1，而loading的alpha则由1变为0。喂了看出效果因此，设置了5s。  

## 卡片翻转动画  

先看效果：  

![卡片翻转动画效果](https://ws3.sinaimg.cn/large/006tNbRwly1fy0gbaumkog309n0h2e81.gif)

这里采用自定义Fragment的专场动画，其中有两个Fragment，布局都很简单，就不展示了。  

### 动画代码  

```card_flip_left_in
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Before rotating, immediately set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:duration="0" />

    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="180"
        android:valueTo="0"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 1. -->
    <objectAnimator
        android:valueFrom="0.0"
        android:valueTo="1.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="@integer/card_flip_time_half" />
</set>
```

其他类似，分别表示左进、左出、右进、右出的动画。  

### Kotlin代码  

```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_card_flip)
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                    .add(R.id.container, CardFrontFragment())
                    .commit()
        }
    }

    override fun onBackPressed() {
        flipCard()
    }

    private var mShowingBack: Boolean = false

    private fun flipCard() {
        if (mShowingBack) {
            supportFragmentManager.popBackStack()
            mShowingBack = false
            return
        }

        mShowingBack = true

        supportFragmentManager.beginTransaction()
                .setCustomAnimations(
                        R.animator.card_flip_right_in,
                        R.animator.card_flip_right_out,
                        R.animator.card_flip_left_in,
                        R.animator.card_flip_left_out
                )
                .replace(R.id.container, CardBackFragment())
                .addToBackStack(null)
                .commit()
    }
```

重写了返回键，就是为了看效果。  

去掉自定义的动画，转场如下图所示：  

![Fragment默认的转场动画](https://ws2.sinaimg.cn/large/006tNbRwly1fy0gkezt10g309n0h2q4o.gif)

是不是很突兀？看来有个动画还是真的不一样的。  

## Circle Reveal动画  

话不多说，先看效果：  

![Circle Reveal动画效果](https://ws1.sinaimg.cn/large/006tNbRwly1fy0h4f5wbmg309n0h2q40.gif)

ViewAnimationUtils.createCircularReveal()方法使我们可以有上述的效果。但是该类在API 21之上才有效果。  

### 显示代码  

显示代码如下：  

```
private fun showView() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            val cx = text.width / 2
            val cy = text.height / 2

            val finalRadius = Math.hypot(cx.toDouble(), cy.toDouble()).toFloat()
            val anim = ViewAnimationUtils.createCircularReveal(text, cx, cy, 0f, finalRadius)
            text.visibility = View.VISIBLE
            anim.start()
        } else {
            text.visibility = View.VISIBLE
        }
    }
```

### 隐藏代码  

```
private fun hideView() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            val cx = text.width / 2
            val cy = text.height / 2

            val initRadius = Math.hypot(cx.toDouble(), cy.toDouble()).toFloat()
            val anim = ViewAnimationUtils.createCircularReveal(text, cx, cy, initRadius, 0f)
            anim.addListener(object : AnimatorListenerAdapter() {
                override fun onAnimationEnd(animation: Animator?) {
                    super.onAnimationEnd(animation)
                    text.visibility = View.INVISIBLE
                }
            })
            anim.start()
        } else {
            text.visibility = View.INVISIBLE
        }
    }
```

## 总结  

动画确实很炫酷，不过要慎用，这周开发就遇到了坑，有机会会以文章的形式记录下来。  

关于代码，请见 [Github](https://github.com/wangli135/ClimbDemo/tree/master/jetpackdemo)



参考：  

- https://developer.android.com/training/animation/reveal-or-hide-view?hl=zh-cn

