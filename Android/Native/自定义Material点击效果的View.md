最近在做项目的时候，遇到一个需求，需要自定义一个View；写到布局文件里面，希望也有Material的波纹点击效果，需要怎么弄呢？  

## ?attr/selectableItemBackground
将该View的background属性设为标题的样式即可，这样在5.0以上就有了波纹效果，在5.0以上就是selector的效果。   

这样设置了后，就有了系统默认的效果。  

### 实验  

其实，现在写个Button，默认也都是有波纹点击效果的，下面分别对三个button，第一个没设置background，第二个和第三个的属性如下：  

```xml
//Button 1
android:background="?attr/selectableItemBackground"
//Button 2
android:background="?attr/selectableItemBackgroundBorderless"
```

效果如下：  

![](https://s1.ax1x.com/2018/09/23/iuBF4x.gif)

可以清楚看到selectableItemBackground和selectableItemBackgroundBorderless的区别，并且设置了之后，button的背景将与其父View的背景色一样。  

### 原理  

设置了attr/selectableItemBackground，其实是给View设置了一个[RippleDrawable](https://developer.android.google.cn/reference/android/graphics/drawable/RippleDrawable)的背景。  

### 自定义RippleDrawable  

和其他很多Drawable一样，RippleDrawable也是可以通过写xml的形式来定义的，下面创建一个drawable xml文件，其定义如下：  

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#ff00ff00" android:radius="10dp">
   <item android:drawable="@color/whiteColor"></item>
</ripple>
```

其中ripple中的color表示波纹的颜色，radius表示波纹的半径，item是默认的背景色，效果如下图：  

![](https://s1.ax1x.com/2018/09/23/iuBhI1.gif)

可以看到Button3的背景不再和父View一样是红色，而变成了白色。  

当不设置item时，比如：  

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#ff00ff00">
</ripple>
```

效果将会是一个没有边界的点击效果，和bordless一样。

#### 5.0版本以下的兼容性  

使用自定义ripple后，在5.0版本以上用不了，替代方法是将上面的布局放到drawable-v21目录下，在drawable目录下创建一个同名文件，使用selector作为背景色。我这边是采用的这种方案。  





