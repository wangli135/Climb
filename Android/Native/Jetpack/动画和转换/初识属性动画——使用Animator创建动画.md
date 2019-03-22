属性动画不仅仅能作用于View，而能作用于任何对象。  
## 与ViewAnimation的区别  
ViewAnimation只支持几种动画：scale、transition、rotate、alpha四种类型。并且缺陷是只是改变了显示位置，实际位置并没有改变。  
一个demo解释一切，如下图：  
![与ViewAnimation的区别](https://ws1.sinaimg.cn/large/006tNc79ly1fz7f64q5fwg30bv0jxwho.gif)  
Hello按钮使用ViewAnimation进行移动，点击事件在移动后，但是响应还在最初的位置；而属性动画移动的World按钮则不同，响应是跟着按钮走的。  
## 属性动画原理  
### 属性动画的一些参数  
创建一个属性动画，一般需要设置几个参数，如下：  
- duration：动画的持续时长，默认300ms
- Time interpolation：时间插值器，是一个函数，property=f(time)，随着时间计算属性的函数  
- 重复次数和行为：可以指定动画是否重复，以及重复次数；也可以指定动画是否reverse  
- AnimatorSet：可以组合多个动画，同时作用or分批作用  
- 帧刷新延迟：可以指定多久刷新一帧动画，默认值是每10s刷新一帧，但实际上的值还是要依赖于系统的实际运行情况。  

### 原理说明  
以官方图为例：  
![线性Demo](https://ws1.sinaimg.cn/large/006tNc79ly1fz7hxlw65bj30fs04mq35.jpg)  
一个动画，40ms，从左向右移动40pixel，每隔10ms，新的帧被画出来了，动画停止时，View停在了最终位置。  
上面的例子是一个线性的效果，时间插值器的函数是：  
```
pixel=1*time
```
下面是一个非线性的例子，如图：  
![非线性Demo](https://ws3.sinaimg.cn/large/006tNc79ly1fz7i1ix2rej30fs04nq39.jpg)  
这个只需要改变插值器即可，可以看到先加速到一半，再开始减速。  
下面开始正式说明属性动画的原理，首先看下图：  
![原理图](https://ws1.sinaimg.cn/large/006tNc79ly1fz7ii7gs4fj30k306o74k.jpg)  
可以看到核心是[ValueAnimator](https://developer.android.com/reference/android/animation/ValueAnimator.html?hl=zh-cn)这个类会追踪动画的时长，当前属性值。  
ValueAnimator封装了TimeInterpolation和TypeEvaluator，TimeInterpolation用来计算每一帧对应时间的属性，TypeEvaluator用来从动画中获取属性值。  
创建一个动画并开启后，属性动画主要有三步操作：  
1. 根据时间流失，得到一个已过时间因子，这个值的范围是[0,1]，以上面的例子为例，总时长40ms，而每一帧10ms，第一帧的已过时间因子就是0.25  
2. 得到已过时间因子后，ValueAnimator会调用TimeInterpolation计算得到属性因子，以上面的例子为例，已过时间因子是0.25，线性插值器的计算结果也是0.25 
3. 得到了插值器的结果后，ValueAnimator会调用TypeEvaluator计算具体的属性值，然后改变View的该属性值。  


每一帧，经过这么计算，就是属性动画的原理。  

## 关于API  
主要是[ValueAnimator](https://developer.android.com/reference/android/animation/ValueAnimator?hl=zh-cn),[ObjectAnimator](https://developer.android.com/reference/android/animation/ObjectAnimator?hl=zh-cn)，[AnimatorSet](https://developer.android.com/reference/android/animation/AnimatorSet?hl=zh-cn)，类结构图如下所示：  
![Animator类结构图](https://ws2.sinaimg.cn/large/006tNc79ly1fz7k3vx2j0j30om05ogmi.jpg)  
## 使用  
Animator和Animation一样，既可以代码实现，也可以在xml中定义，下面分别说明两种方式分别是如何操作的。  
### 使用ValueAnimator  
ValueAnimator有一些静态方法，ofInt、ofFloat、ofObject。  
先看效果，  
![ValueAnimator效果](https://ws3.sinaimg.cn/large/006tNc79ly1fz881zz2gog30bv0jxgmw.gif)  
1. xml定义动画 
```
<?xml version="1.0" encoding="utf-8"?>
<animator xmlns:android="http://schemas.android.com/apk/res/android"
          android:duration="3000"
          android:interpolator="@android:anim/accelerate_interpolator"
          android:valueFrom="0"
          android:valueTo="1000"
          android:valueType="floatType"
    />
```
配合代码：  
```
val animatorFromXml = AnimatorInflater.loadAnimator(this, R.animator.value_move_view)
        btnMoveFromXml.setOnClickListener {
            if (animatorFromXml is ValueAnimator) {
                animatorFromXml.apply {
                    addUpdateListener {
                        tvShow.translationX = it.animatedValue as Float
                    }
                    start()
                }
            }
        }
```
2. 代码定义动画  
```
val animator = ValueAnimator.ofFloat(0f, 1000f)
                .apply {
                    duration = 3000
                    interpolator = AccelerateInterpolator()
                    addUpdateListener {
                        tvShow.translationX = it.animatedValue as Float
                    }
                }
        btnMove.setOnClickListener {
            animator.start()
        }
```
上面两种实现是一样的效果，耗时3s，transitionX从0变为1000，ValueAnimator需要添加UpdateListener得到实际的属性值，然后赋值给对应View。  
### 使用ObjectAnimator  
效果与上面相同，如下：  
![ObjectAnimator效果](https://ws2.sinaimg.cn/large/006tNc79ly1fz888hvdcmg30bv0jxdi5.gif)  
1. xml定义动画 
```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
                android:duration="3000"
                android:interpolator="@android:anim/accelerate_interpolator"
                android:propertyName="transitionX"
                android:valueFrom="0"
                android:valueTo="1000"
                android:valueType="floatType"
    />
```
配合代码：  
```
val animatorFromXml = AnimatorInflater.loadAnimator(this, R.animator.object_move_view)
        btnMoveFromXml.setOnClickListener {
            if(animatorFromXml is ValueAnimator){
                animatorFromXml.apply {
                    addUpdateListener {
                        tvShow.translationX=animatedValue as Float
                    }
                    start()
                }
            }
        }
```
2. 代码定义动画
```
val animator = ObjectAnimator.ofFloat(tvShow,View.TRANSLATION_X,0f,1000f).apply {
            duration=3000
            interpolator=AccelerateInterpolator()
        }
        btnMove.setOnClickListener {
            animator.start()
        }
```
与ValueAnimator相比，ObjectAnimator传入了要运动的View，所以也就不需要在UpdateListener中控制View。  
### AnimatorSet  
如果需要同时开启多个动画，那么可以使用AnimatorSet，串联组织多个动画。  
先看效果：  
![AnimatorSet效果](https://ws2.sinaimg.cn/large/006tNc79ly1fz8a8vm7vlg30bv0jxmy3.gif)  
具体效果是首先透明度变化，然后transitionX和transitionY一起变化，最后透明度再变化一波。代码如下：  
```
val alphaStartAnim = ObjectAnimator.ofFloat(tvShow, View.ALPHA, 1f, 0f, 1f)
        val xAnim = ObjectAnimator.ofFloat(tvShow, View.TRANSLATION_X, 0f, 1000f)
        val yAnim = ObjectAnimator.ofFloat(tvShow, View.TRANSLATION_Y, 0f, 500f)
        val alphaEndAnim = ObjectAnimator.ofFloat(tvShow, View.ALPHA, 1f, 0f, 1f)
        btnMove.setOnClickListener {
            AnimatorSet().apply {
                play(xAnim).after(alphaStartAnim)
                play(xAnim).with(yAnim)
                play(xAnim).before(alphaEndAnim)
                start()
            }
        }
```
## 总结  
至此，我们可以使用ValueAnimator、ObjectAnimator或AnimatorSet进行创建动画，然后作用于View或其他对象。    

关于代码，参考[Github](https://github.com/wangli135/ClimbDemo/tree/master/jetpackdemo/src/main/java/com/xingfeng/jetpackdemo/animation/propertyanimator)


## 参考 
- https://developer.android.com/guide/topics/graphics/prop-animation?hl=zh-cn  
- https://blog.csdn.net/guolin_blog/article/details/43536355
- https://blog.csdn.net/guolin_blog/article/details/43816093  
