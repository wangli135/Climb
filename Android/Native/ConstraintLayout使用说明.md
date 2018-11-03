刚学Android那会，Android是有五大布局的：FrameLayout、LinearLayout、RelativeLayout、AbsouteLayout、TableLayout。现在为止，Android的布局家族得到了扩充，现在在公司做项目使用的最多的是ConstraintLayout，之前接触的比较少，因此趁着周末时间，好好学习了一下，功能非常nb，不愧是Android Studio默认创建项目的首选布局。  

[ConstraintLayou](https://developer.android.google.cn/reference/android/support/constraint/ConstraintLayout)与RelativeLayout类似，但功能更强大，且性能更高。  

## 官方文档  

目前ConstraintLayout支持以下几种约束：  

- 相对布局

- Margins

- 中心布局  

- 圆形布局  

- 可见性行为

- 尺寸约束

- 链

- 虚拟辅助工具

- 优化器  

### 相对布局  

相对位置是一个最基本的功能，一个控件相对另一个控件是如何布局的。你可以控制在水平和垂直两个方向上控制控件。  

- 水平方向：left、right、start、end边
- 垂直方向：top、bottom边和文字baseline   
  

layout_constraintXXX_toYYYOf属性，其中XXX和YYY可以为start、left、end、right、top和bottom，意思是让控件的哪一边和另一个控件的哪一边挨着。  

关于相对位置，官方给出的图如下：  
![相对位置示意图](https://upload-images.jianshu.io/upload_images/4179925-064b2cef33079d6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840/format/webp)

用法：  
```
 <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button2"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toRightOf="@+id/button1"
        app:layout_constraintTop_toTopOf="parent" />
```
layout_constraintXXX_toYYYOf属性后面即可跟控件id，也可以跟parent，parent代表着父布局，即ConstrainLayout。  
### Margins
![相对位置带Margins](https://developer.android.google.cn/reference/android/support/constraint/resources/images/relative-positioning-margin.png)
和其他布局一样，都是可以指定Margins的，比如在指定了边重合后，可以再加上Margins实现上图的效果。  
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="A"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />


    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="30dp"
        android:text="B"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toRightOf="@id/button1"
        app:layout_constraintTop_toTopOf="parent" />
    
</android.support.constraint.ConstraintLayout>
```
当约束的目标可见性是GONE，可以设置一个另外的margin，通过以下属性进行设置：  
- layout_goneMarginStart
- layout_goneMarginEnd
- layout_goneMarginLeft
- layout_goneMarginTop
- layout_goneMarginRight
- layout_goneMarginBottom

###  中心布局和bias  

举个例子：  

```
<android.support.constraint.ConstraintLayout ...>
             <Button android:id="@+id/button" ...
                 app:layout_constraintLeft_toLeftOf="parent"
                 app:layout_constraintRight_toRightOf="parent/>
         </>
         
```

如果需要满足这个约束，那么button尺寸得和ConstraintLayout一样宽。但最终的效果如下：  

![](https://developer.android.google.cn/reference/android/support/constraint/resources/images/centering-positioning.png)

这样就好像拉伸了控件一样，实现了水平居中的效果，同样可用于垂直居中。

#### Bias  

默认地，两边拉伸一样大，但如果想两边拉伸的不一样，可以使用如下两个属性：  

- layout_constraintHorizontal_bias
- layout_constraintVertical_bias
  

![](https://developer.android.google.cn/reference/android/support/constraint/resources/images/centering-positioning-bias.png)

代码如下：  

```
<android.support.constraint.ConstraintLayout ...>
             <Button android:id="@+id/button" ...
                 app:layout_constraintHorizontal_bias="0.3"
                 app:layout_constraintLeft_toLeftOf="parent"
                 app:layout_constraintRight_toRightOf="parent/>
         </>
```

这样两边比就是0.3：0.5，类似LinearLayout的weight效果。  

### 圆形布局(1.1加入)

可以指定某一控件以相对另一控件的某一角度和距离进行布局。  

- `layout_constraintCircle` : references another widget id
- `layout_constraintCircleRadius` : the distance to the other widget center
- `layout_constraintCircleAngle` : which angle the widget should be at (in degrees, from 0 to 360)

![](https://developer.android.google.cn/reference/android/support/constraint/resources/images/circle1.png)

### 可见性行为  

ConstraintLayout在处理控件可见性为GONE时有特殊的规则。  

当控件可见性为GONE时，    

- 布局阶段会认为是一个点
- 它的约束关系仍在，但是margin会是0。  

效果如下图：  
![](https://developer.android.google.cn/reference/android/support/constraint/resources/images/visibility-behavior.png)

### 尺寸约束  

#### ConstraintLayout最小尺寸

可以为ConstraintLayout设置最小尺寸和最大尺寸。当ConstraintLayout的尺寸设置WARP_CONTENT时，这些最小尺寸和最大尺寸会被使用。

#### 控件尺寸约束  

控件的尺寸是通过layout_width和layout_height进行约束的，可以使用三种值：  

- 某一指定值
- WARP_CONTENT
- 0dp，等价于"MATCH_CONSTRAINT"  

#### Ratio

可以定义控件的尺寸和另一个控件的尺寸比例。前提条件是至少有一个尺寸是0dp，然后指定layout_constraintDimensionRatio，如下：  

```
 <Button android:layout_width="wrap_content"
                   android:layout_height="0dp"
                   app:layout_constraintDimensionRatio="1:1" />
```

这种情况下，宽高将是1：1。

ratio可以有两种表现形式：  

- 一个浮点数：表示宽/高
- 一个比例形式，表示宽：高  

如果两个尺寸都为0dp，系统根据指定的ratio设置满足的最大尺寸。
```
 <Button android:layout_width="0dp"
                   android:layout_height="0dp"
                   app:layout_constraintDimensionRatio="H,16:9"
                   app:layout_constraintBottom_toBottomOf="parent"
                   app:layout_constraintTop_toTopOf="parent"/>
```

H字母在前表示，前面是高度：宽度；

### 链  

### 虚拟辅助工具  

- Guideline
- Barrier
- Group

### 优化 （1.1新特性） 

可以在ConstraintLayout中添加app:layout_optimizationLevel进行优化，目前支持的参数有：  

- **none** : no optimizations are applied
- **standard** : Default. Optimize direct and barrier constraints only
- **direct** : optimize direct constraints
- **barrier** : optimize barrier constraints
- **chain** : optimize chain constraints (experimental)
- **dimensions** : optimize dimensions measures (experimental), reducing the number of measures of match constraints elements  
  
## 高级用法——辅助工具

### Guideline

可以将Guideline看做是辅助线，不会在布局中显示的。Guideline中有两种，垂直的和水平的。

- 水平的Guideline，高度为0dp，宽度为ConstraintLayout的宽度

- 垂直的Guideline，宽度为0dp，高度为ConstraintLayout的高度

有三种方法去定位一个辅助线的位置：  

- layout_constraintGuide_begin：对于水平的，相当于topMargin；对于垂直的，相当于leftMarigin

- layout_constraintGuide_end：对于水平的，相当于bottomMargin；对于垂直的，相当于rightMarigin

- layout_constraintGuide_percent：设定相对布局的百分比，如果需要居中，就设置0.5  

```
 <android.support.constraint.Guideline
        android:id="@+id/guideline"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:orientation="vertical"
        app:layout_constraintGuide_percent="0.5" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="hello"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="@+id/guideline"
        app:layout_constraintTop_toTopOf="parent" />
```

居左1/4，guideline居中，textview又居左和guideline的中，就是1/4的地方。  

### Barrier  

[Barrier](https://developer.android.google.cn/reference/android/support/constraint/Barrier)是栅栏的意思，Java并发库里有个栅栏，这里意思差不多，都是给个底线，超越不得的。  

Barrier以一组控件作为输入，基于这些控件创造出一个虚拟的辅助线。  

比如下图，将会创造一个左边的栅栏沿着控件的左边。  

![](https://developer.android.google.cn/reference/android/support/constraint/resources/images/barrier-buttons.png)

```
<android.support.constraint.Barrier
              android:id="@+id/barrier"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              app:barrierDirection="start"
              app:constraint_referenced_ids="button1,button2" />
```

当创建了Barrier之后，其他控件可以和Barrier建立约束。  

### Group

[Group](https://developer.android.google.cn/reference/android/support/constraint/Group)可以将一组控件组合起来，一起控制可见性。不需要分开设置。这个真的是太有用了，目前我们的项目中经常需要某一中情况下隐藏几个textview，那一种情况下又是开那几个TextView，可以改变用Group。  

用法如下：  

```
 <android.support.constraint.Group
              android:id="@+id/group"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:visibility="visible"
              app:constraint_referenced_ids="button4,button9" />
```

## 总结  

ConstraintLayout是一个性能很高的布局，可以应对复杂的布局，重要的还是要多实践实践。