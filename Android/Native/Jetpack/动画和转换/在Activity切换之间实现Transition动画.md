在Android 5.0执行，如果需要在Activity切换之间实现动画，需要实现        overridePendingTransition()方法，并实现入场动画和退场动画。而在5.0之后，出现了一些Material Design的转场动画，先看下demo样子。  
![转场动画的动画demo1](https://ws2.sinaimg.cn/large/006tNc79ly1fzfeidyvqeg30aq0ipe81.gif)  
上面的例子中，Slide效果还是比较明显的，Explode和Fade不是很容易看清，后面两个是Share Element的动画，最后两个是ActivityOptionsCompat的另外两种效果。  
## 实现  
### 使用Transition开启Activity
以ExplodeActivity为例，完整代码如下：  
```
class ExplodeActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        with(window) {
            enterTransition = Explode()
            exitTransition = Explode()
        }
        setContentView(R.layout.activity_explode)
    }

}
```
Android Transition框架提供了三种自带动画，分别是Explode、Fade和Slide，更好的动画效果见下图：  
![](https://ws1.sinaimg.cn/large/006tNc79ly1fzfev5xs4zg30oq0f4hdt.gif)  
> 图片来源自https://github.com/lgvalle/Material-Animations  

### Share Elements的跳转  
实现这种转场动画，Activity的theme需要设置：  
```
<style name="BaseAppTheme" parent="android:Theme.Material">
  <!-- enable window content transitions -->
  <item name="android:windowActivityTransitions">true</item>
</style>
```
上面等同于在Activity的onCreate()方法的setContentView()之前调用如下代码：  
```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        with(window) {
            requestFeature(Window.FEATURE_CONTENT_TRANSITIONS)
        }
        setContentView(R.layout.activity_share_view)

        ivImage.transitionName = "image"
    }
```
在theme文件中可以指定进入、退出动画，同理，在代码中也是可以指定的。
有时候，两个页面的不同View之间可以设置转场动画，比如demo中大幂幂的图片，在第二个Activity也有使用，这个很简单，只需要给View关联上transitionName字段，并在启动Activity时将View与transitionName进行关联，即可。比demo中的例子为例，  
ActivityTransitionActivity中的启动代码如下
```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                ActivityCompat.startActivity(this,
                        Intent(this, ShareViewActivity::class.java),
                        ActivityOptionsCompat.makeSceneTransitionAnimation(this, ivShareIv, "image").toBundle())
            } else {
                startActivity(Intent(this, ShareViewActivity::class.java))
            }
```
需要注意的是，这里需要使用ActivityOptionsCompat.makeSceneTransitionAnimation()方法，"image"就是transitionName，ShareViewActivity中的代码如下：  
```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        with(window) {
            requestFeature(Window.FEATURE_CONTENT_TRANSITIONS)
        }
        setContentView(R.layout.activity_share_view)

//关键
        ivImage.transitionName = "image"
    }

```
可以看到在onCreate()方法里设置了ivImage的transitionName="image"，除了代码设置，也可以在xml中设置该属性的。  

#### 多个View的共享实现  
上面只有一个View，如果有多个View，如何实现呢？也很easy，启动代码如下：  
```
tvShare.setOnClickListener {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                val options = ActivityOptionsCompat.makeSceneTransitionAnimation(
                        this,
                        UtilPair<View, String>(ivShareIv, "image"),
                        UtilPair<View, String>(tvShare, "text")
                )
                ActivityCompat.startActivity(this,
                        Intent(this, ShareViewActivity::class.java).apply {
                            putExtra("showTwoView", true)
                        },
                        options.toBundle()
                )
            } else {
                startActivity(Intent(this, ShareViewActivity::class.java))
            }
        }

```
需要使用Pair将每组View与transitionName关联，最后调用ActivityCompat开启新的页面。  
### ActivityOptionsCompat  
[ActivityOptionsCompat](https://developer.android.google.cn/reference/android/support/v4/app/ActivityOptionsCompat)除了共享元素的动画外，还有另外两个方法：  
- makeScaleUpAnimation()
- makeThumbnailScaleUpAnimation()  


最后两个就是使用该方法实现的。  

### 关闭Activity  
如果需要反向转场动画，那么需要使用Activity.finishAfterTransition()代替Activity.finish()。  

## 总结  
重点推荐[Material-Animations](https://github.com/lgvalle/Material-Animations)这个开源项目，demo很酷炫，原理也讲得很好。  
关于代码，请参考[Github](https://github.com/wangli135/ClimbDemo/tree/master/jetpackdemo/src/main/java/com/xingfeng/jetpackdemo/animation/activitytransition)

## 参考  
- https://developer.android.com/training/transitions/start-activity?hl=zh-cn  
- https://android.jlelse.eu/android-material-design-activity-transition-55de706ab967  
- https://www.jianshu.com/p/a43daa1e3d6e  
- https://github.com/lgvalle/Material-Animations  

> 关注我的技术公众号，不定期会有技术文章推送，不敢说优质，但至少是我自己的学习心得。微信扫一扫下方二维码即可关注：  
> ![二维码](https://ws1.sinaimg.cn/large/006tNc79ly1fz0b9yxfsbj309k09kt8n.jpg)