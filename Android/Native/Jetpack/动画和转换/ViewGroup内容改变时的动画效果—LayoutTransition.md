向一个ViewGroup中添加View或移除View时，针对当前所有的View，是可以有一个动画效果的，这个动画效果主要靠[LayoutTransition](https://developer.android.com/reference/android/animation/LayoutTransition?hl=zh-cn)实现。

话不多说，先上图看下效果（仿照Android API Demos里面的例子）。

![Layout引起的动画](https://ws1.sinaimg.cn/large/006tNc79ly1fz5y8ujp7cg30bw0jbwum.gif)

可以看到，当添加或删除View时，下面View中的Button都是有动画效果的，这种实现就是通过LayoutTransition实现的。  

## LayoutTransition是什么  

### 开启LayoutTransition

LayoutTransition可以使**ViewGroup**在**布局发生变化**时产生动画。默认情况下，ViewGroup是没有开启这个动画的，开启方式主要有两种：  

1. xml属性设置，设置ViewGroup的android:animateLayoutChanges="true"

2. 代码设置，创建LayoutTransition实例，并调用setLayoutTransition()方法进行设置  

### 动画类型

LayoutTransition的核心概念是有两种类型的变化会引起四种动画，两种类型的变化分别是add和remove以及对应的VISIBLE以及GONE。以add为例，当add进一个View时，该View有appearing动画，而其他View因该View会发生change-appearing的动画；同理，remove时，被remove掉的View有disappearing动画，而其他View因该View会发生disappearing的动画。  

四种动画对应着demo中的四个选项：in、out、inchange和outchange。  

### 动画时序问题

当add一个view时，其他View首先执行change-appearing动画，因为需要为view腾出空间，然后view才执行appearing动画；同理，当remove一个view时，被remove的View首先执行disappearing动画为其他View腾出位置，然后其他View再执行change-disappearing动画，这都是很好理解的。  

如果在一个disappearing动画完成之前开启一个appearing动画，那么disappearing动画会立即停止，并且已发生的效果会取消，反之效果类似。  

### 原理  

LayoutTransition中指定的动画时长、效果都是临时的。实际的值是在每次动画时设置的。举个例子，CHANGE_APPEARING动画会作用left、top、right、bottom、scrollX和scrollY属性，当动画开始时，这些属性值会根据pre-和post-layout的值进行更新。

## Demo实现代码  

LayoutTransition中有四种动画，可以针对每一种动画进行开启、关闭的设置，或者修改该种动画，具体方法是：  

```
	enableTransitionType(int transitionType)
		disableTransitionType(int transitionType)
			setAnimator(int transitionType, Animator animator)
```

Demo中的代码主要包括两部分，一部分是使用默认的LayoutTransition，对其中四种动画进行单独设置；第二部分是使用了自定义的LayoutTransition，主要是改变了APPEARING动画，有一个旋转的效果。底部的布局是一个FlowLayout，可以实现换行布局的效果。  

xml布局比较简单，就不贴代码了，kotlin代码如下：

```
class LayoutAnimateActivity : AppCompatActivity() {

    var index = 0
    val layoutTransition = LayoutTransition()//初始化LayoutTransition实例

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_auto_layout_animate)

        //开启ViewGroup的Layout Animate
        viewGroup.layoutTransition = layoutTransition

        //控制APPEARING动画
        inCb.setOnCheckedChangeListener { buttonView, isChecked ->
            if (isChecked)
                layoutTransition.enableTransitionType(LayoutTransition.APPEARING)
            else
                layoutTransition.disableTransitionType(LayoutTransition.APPEARING)
        }
        //控制DISAPPEARING动画
        outCb.setOnCheckedChangeListener { buttonView, isChecked ->
            if (isChecked)
                layoutTransition.enableTransitionType(LayoutTransition.DISAPPEARING)
            else
                layoutTransition.disableTransitionType(LayoutTransition.DISAPPEARING)
        }
        //控制CHANGE-APPEARING动画
        inChangeCb.setOnCheckedChangeListener { buttonView, isChecked ->
            if (isChecked)
                layoutTransition.enableTransitionType(LayoutTransition.CHANGE_APPEARING)
            else
                layoutTransition.disableTransitionType(LayoutTransition.CHANGE_APPEARING)
        }
        //控制CHANGE-DISAPPEARING动画
        outChangeCb.setOnCheckedChangeListener { buttonView, isChecked ->
            if (isChecked)
                layoutTransition.enableTransitionType(LayoutTransition.CHANGE_DISAPPEARING)
            else
                layoutTransition.disableTransitionType(LayoutTransition.CHANGE_DISAPPEARING)
        }
        //设置自定义动画，改变APPEARING动画
        customAnimateCb.setOnCheckedChangeListener { buttonView, isChecked ->
            if (isChecked) {
                viewGroup.layoutTransition = LayoutTransition().also {
                    it.setAnimator(LayoutTransition.APPEARING, ObjectAnimator.ofFloat(null, View.ROTATION_X, 0f, 360f))
                }
            } else {
                viewGroup.layoutTransition = layoutTransition
            }
        }
        //Add View
        btnAdd.setOnClickListener {
            val view = Button(this).apply {
                text = "Button ${index++}"
            }
            viewGroup.addView(view, 0, ViewGroup.LayoutParams(
                    ViewGroup.LayoutParams.WRAP_CONTENT,
                    ViewGroup.LayoutParams.WRAP_CONTENT
            ))
        }
        //Remove View
        btnRemove.setOnClickListener {
            var childCount = viewGroup.childCount
            if (childCount > 0) {
                viewGroup.removeViewAt(0)
            }
        }
    }
}
```



### 自定义LayoutTransition动画  

改变APPEARING的动画是设置了一个Animate，上面使用了一个ObjectAnimate，定义如下：  

```
LayoutTransition().also {
                    it.setAnimator(LayoutTransition.APPEARING, ObjectAnimator.ofFloat(null, View.ROTATION_X, 0f, 360f))
                }
```

## 总结

LayoutTransition是ViewGroup发生Layout改变时的动画，既可以通过xml进行设置开启，也可以代码设置开启，setTransition=null就是关闭动画。LayoutTransition整个包含四种动画，可以单独进行开启、关闭和设置，可以根据需要进行定制化。  关于代码，请查看[Github](https://github.com/wangli135/ClimbDemo/tree/master/jetpackdemo/src/main/java/com/xingfeng/jetpackdemo/animation/layoutanimate)

## 参考  
- https://developer.android.com/training/animation/layout?hl=zh-cn  
- https://blog.csdn.net/harvic880925/article/details/50985596

> 关注我的技术公众号，不定期会有技术文章推送，不敢说优质，但至少是我自己的学习心得。微信扫一扫下方二维码即可关注：  
![二维码](https://ws1.sinaimg.cn/large/006tNc79ly1fz0b9yxfsbj309k09kt8n.jpg)
