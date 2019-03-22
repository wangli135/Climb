属性动画中有一个重要的概念就是插值器——Interpolator，根据流失的时间因子计算得到属性因子。Android中默认的插值器是AccelerateDecelerateInterpolator，内置了很多插值器，本文将以一个例子介绍各种插值器的效果，以及如何自定义Interpolator。  
话不多说，先看demo，如下图：  
![Interpolator Demo](https://ws2.sinaimg.cn/large/006tNc79ly1fz8hw9mc51g30bv0jxtx1.gif)  
## 官方Interpolator介绍  
除了最后一个是自定义Interpolator外，其他都是系统自带的。下面主要介绍下效果就好了：  
- AccelerateDecelerateInterpolator：先加速、再减速，默认的插值器 
- LinearInterpolator：线性插值器  
- AccelerateInterpolator：加速  
- DecelerateInterpolator：减速  
- AnticipateInterpolator：开始时先反向
- BounceInterpolator：达到最终位置会先反弹，类似弹弹球着地的效果  
- OvershootInterpolator:达到最终位置会超出一些  
- AnticipateOvershootInterpolator：AnticipateInterpolator与OvershootInterpolator的结合  
- CycleInterpolator：正弦效果，可以指定回荡的次数  
- PathInterpolator：根据指定的path进行运动，可以实现贝塞尔曲线    

Interpolator既可以在代码中指定给动画，同样也可以在xml文件中使用，这块可以到参考文章中查看。  
## 自定义Interpolator  
先介绍一个网站，里面有各种Interpolator的效果以及数学公式定义，网址是 http://inloop.github.io/interpolator/  。  
### 先看官方Interpolator找找灵感  
Interpolator的核心是下面这个方法：  
```
float getInterpolation(float input);
```
其中input就是流失的时间因子，范围是[0,1]，输出是属性因子。    
LinearInterpolator的函数是个一次函数，样式如下：  
![](https://ws4.sinaimg.cn/large/006tNc79ly1fz8iaoszhmj30pw0p2dgp.jpg)  
相对应的，LinearInterpolator的方法实现如下：
```
 public float getInterpolation(float input) {
        return input;
    }
```
而AccelerateInterpolator的函数是个二次函数，样式如下：  
![](https://ws1.sinaimg.cn/large/006tNc79ly1fz8id2flxaj30pu0pw0uo.jpg)   
相对应地，实现如下：  
```
public float getInterpolation(float input) {
        if (mFactor == 1.0f) {
            return input * input;
        } else {
            return (float)Math.pow(input, mDoubleFactor);
        }
    }
```
看完了官方的两个例子，再来看看如何自定义Interpolator。  
### 自定义Interpolator——SpringInterpolator  
可以看到，我们自定义的Interpolator在达到终点后，有多次震荡的效果，是不是很像弹簧？这个可以通过自定义Interpolator实现，也可以通过DynamicAnimation实现，具有可以参考[让View具有弹性效果的动画——SpringAnimation](让View具有弹性效果的动画——SpringAnimation.md)。  
SpringInterpolator的函数很复杂，样式如下：  
![](https://ws3.sinaimg.cn/large/006tNc79ly1fz8ihmqt9pj30q20pqq5a.jpg)

然后就是根据函数实现方法了，具体如下：  
```
class SpringInterpolator(val factor: Float) : Interpolator{
    
    override fun getInterpolation(input: Float): Float {
        return (Math.pow(2.0, -10.0 * input) * Math.sin((input - factor / 4) * (2 * Math.PI) / factor) + 1).toFloat()
    }
}
```
这么复杂的函数，如果你问我是怎么会的，那我只能告诉你，去上面那个网址看吧，你会发现自定义Interpolator原来这么容易。    


**其实自定义Interpolator的核心就是这个上面的函数样式，以及将其再转换成代码就ok了。**
## 总结  
关于代码，参考[Github](https://github.com/wangli135/ClimbDemo/tree/master/jetpackdemo/src/main/java/com/xingfeng/jetpackdemo/animation/propertyanimator)

## 参考  
- https://developer.android.com/guide/topics/graphics/prop-animation?hl=zh-cn#interpolators  
- http://wiki.jikexueyuan.com/project/android-animation/2.html  
- https://medium.com/mindorks/understanding-interpolators-in-android-ce4e8d1d71cd  
- https://robots.thoughtbot.com/android-interpolators-a-visual-guide  
