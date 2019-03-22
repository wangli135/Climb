同一个Activity之间，布局切换是可以有动画效果的，下面是仿照API Demo中的一个例子，如下图：  
![](https://ws1.sinaimg.cn/large/006tNc79ly1fze1sz80i4g30bv0jxmzz.gif)  
在同一个Activity中，通过选中不同的Scene，切换不同的布局。  
## 实现  
在两个Layout之间进行动画的基本步骤如下：  
1. 为起始和结束Layout创建[Scene](https://developer.android.google.cn/reference/android/transition/Scene)对象，一般来说，当前布局就是起始布局；  
2. 创建一个[Transition](https://developer.android.google.cn/reference/android/transition/Transition)对象，定义你想要的动画;
3. 调用TransitionManager.go()方法，系统将会自动进行布局动画。  

步骤是不是很简单？整个流程图如下：  
![流程图](https://ws2.sinaimg.cn/large/006tNc79ly1fze1ybho5pj30e206iglz.jpg)  

### 创建Scene  
Scene可以理解为对布局的一个快照，包含了View的层次以及各种属性相关的信息。Transition框架可以自动在起始和结束Scene之间进行动画。  
#### 从布局文件中创建Scene    
调用Scene.getSceneForLayout()方法创建一个Scene，其中第一个参数是一个ViewGroup，第二个参数是布局id。  
```
Scene.getSceneForLayout(scene_root, R.layout.layout_scene_1, this)
```
#### 从代码中创建Scene  

xml布局中定义的View层次也是可以通过代码定义的，只不过比较麻烦，这里就不介绍了。  
### 应用Transition  
可以使用android已经提供的一些Transition，比如AutoTransition、Fade，或者定义自己的Transition。然后调用TransitionManager.go()方法即可。  
#### 创建Transition  

| 类             | 标签              | 属性                                                 | 效果                       |
| -------------- | ----------------- | ---------------------------------------------------- | -------------------------- |
| AutoTransition | <autoTransition/> |                                                      | 淡出、移动和改变尺寸、淡入 |
| Fade           | <fade/>           | android:fadingMode="[fade_in ,fade_out,fade_in_out]" | 控制淡出淡入               |
| ChangeBounds   | <changeBounds/>   |                                                      | 移动和改变尺寸             |

以上就是内置的类型以及在xml中对应的标签。Transition和属性动画、View Animation一样，都是可以在xml中定义的，举个例子，  
```
<fade xmlns:android="http://schemas.android.com/apk/res/android" />
```
然后就可以通过代码获取到Transition实例，代码是  
```
var mFadeTransition: Transition =
    TransitionInflater.from(this)
                      .inflateTransition(R.transition.fade_transition)
```
而如果使用代码定义Transition，可以这么做：  
```
var mFadeTransition: Transition = Fade()  
```
#### 应用Transition  
调用如下代码即可：  
```
TransitionManager.go(mEndingScene, mFadeTransition)
```
是不是很easy？这个demo的代码如下所示：  
```
secneRg.setOnCheckedChangeListener { group, checkedId ->
            var endScene: Scene? = null
            when (checkedId) {
                R.id.scene1Rb -> {
                    endScene = Scene.getSceneForLayout(scene_root, R.layout.layout_scene_1, this)
                }
                R.id.scene2Rb -> {
                    endScene = Scene.getSceneForLayout(scene_root, R.layout.layout_scene_2, this)
                }
                R.id.scene3Rb -> {
                    endScene = Scene.getSceneForLayout(scene_root, R.layout.layout_scene_3, this)
                }
                R.id.scene4Rb -> {
                    endScene = Scene.getSceneForLayout(scene_root, R.layout.layout_scene_4, this)
                }
            }
            TransitionManager.go(endScene, TransitionSet().apply {
                addTransition(Fade())
                addTransition(ChangeBounds())
            })
        }
```
#### 选择指定目标View  
默认情况下，整个View层次都是作为动画的对象，如果不想某些View有动画效果，可以在设置动画之前调用removeTarget()来进行清除。  
### Transition框架的限制  
Transition框架有一些使用限制，  
1. 应用于SurfaceView的动画不会起效，因为其更新在非UI线程；
2. 继承AdapterView的，比如ListView，不能应用Transition  
3. 如果你想在TextView中改变大小，那么在对象完成动画之前，文字会显示异常，为了避免这种情况，不要动画可能包含文字的View。  

## 总结  
关于代码，请参考[Github地址](https://github.com/wangli135/ClimbDemo/tree/master/jetpackdemo/src/main/java/com/xingfeng/jetpackdemo/animation/layoutanimate)

## 参考  
- https://github.com/lgvalle/Material-Animations
- https://developer.android.google.cn/training/transitions/  
