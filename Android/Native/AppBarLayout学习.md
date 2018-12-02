AppBarLayout是一个垂直的LinearLayout，实现了很多和协调布局一起合作的滚动属性。其子View可以通过setScrollFlags()或在xml布局中通过app:layout_scrollFlags属性设置想要的滚动行为。  

AppBarLayout很多行为依赖于CoordinatorLayout。如果你使用别的ViewGroup装AppBarLyout，很多功能就没有了。  

AppBarLayout不能滚动，但是要有一个可以滚动的兄弟View。兄弟View需要设置AppBarLayout.ScrollingViewBehavior。

## AppBarLayout是什么，效果是怎样的  

使用AndroidStudio建立一个ScrollActivity，模板就是使用的AppBarLayout，不过例子里还使用了CollapsingToolbarLayout，暂时先不考虑这个，因此对例子做了点修改，xml布局如下：  

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ScrollingActivity2">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/app_bar_height"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            app:title="AppBarLayout学习"
            android:id="@+id/toolBar"
            android:layout_width="match_parent"
            android:layout_height="@android:dimen/app_icon_size"></android.support.v7.widget.Toolbar>l

        <ImageView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="fitXY"
            android:src="@drawable/pic_11"/>

    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        tools:showIn="@layout/activity_scrolling2">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="@dimen/text_margin"
            android:text="@string/large_text"/>

    </android.support.v4.widget.NestedScrollView>

</android.support.design.widget.CoordinatorLayout>
```

父布局是CoordinatorLayout，然后主要两个布局：AppBarLayout以及NestedScrollView，NestedScrollView设置了layout_behavior属性，这样AppBarLayout里面的布局就可以跟着一起滚动，不过由于这里还没给AppBarLayout里面的scrollFlags设置参数，目前运行效果如下：  

![](https://ws4.sinaimg.cn/large/006tNbRwly1fxs7ssx9vag30830g9to7.gif)

可以看到下面的布局滚动，上面的AppBarLayout不为所动。下面就开始为View设置不同属性，看看效果。  

## scrollFlags的效果  

scrollFlags一共有五个值，一些值是可以组合作用在View上的。五个值分别是： 

1. scroll：子View随ScrollView一起滚动
2. enterAlways：只要ScrollView向下移动，子View立即响应滚动 
3. enterAlwaysCollapsed：当ScrollView滚动最顶层时，子View响应滚动事件，直至子View完全显示
4. exitUtilCollapsed：只要ScrollView向上滚动，子View立即响应滚动，直到达到最小高度
5. snap：当Scrollview滚动到最顶层时，子View响应滚动事件。松开手指时，依据AppBarLayout移出屏幕区域与生育可视区域对比，自动移向占比大的区域。

### scroll  

修改AppBarLayout的子View的属性如下：  

```
<android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/app_bar_height"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            app:layout_scrollFlags="scroll"
            app:title="AppBarLayout学习"
            android:id="@+id/toolBar"
            android:layout_width="match_parent"
            android:layout_height="@android:dimen/app_icon_size"></android.support.v7.widget.Toolbar>l

        <ImageView
            app:layout_scrollFlags="scroll"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="fitXY"
            android:src="@drawable/pic_11"/>

    </android.support.design.widget.AppBarLayout>
```

运行效果如下： 

![](https://ws3.sinaimg.cn/large/006tNbRwly1fxs85w7775g30830eyttx.gif)

可以看到ToolBar和ImageView就好像是ScrollView里的内容一样，跟着上下滚动；不过需要注意的是，如果单独设置ImageView的为scroll，而不设置ToolBar，是没有效果的，因为ToolBar把ImageView给顶住了，只有下面的ScrollView会滚动。    

scroll属性是其他属性的基础，并且由于AppBarLayout是一个垂直的LinearLayout，因此一旦一个View没有设置该属性，那么该View之后的效果都会生效，可以简单认为是该View把后面View的滚动效果给顶住了。

### enterAlways  

在上面例子的基础上，设置ImageView的scrollFlags="scroll|enterAlways"，如下：  

```
 <ImageView
            app:layout_scrollFlags="scroll|enterAlways"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="fitXY"
            android:src="@drawable/pic_11"/>
```

此时滚动效果如下所示：  

![](https://ws3.sinaimg.cn/large/006tNbRwly1fxs8bibj50g30830eq4o7.gif)

此时可以看到，向上滚动，没有区别；向下滚动时，由于ImageView设置了enterAlways，因此首先滚动，直至出现了，然后ScrollView滚动，最后才是ToolBar显示。

**可以理解为设置了enterAlways属性的View在向下滚动时的优先级高于ScrollView本身，可以实现分段滚动的效果。**  

### enterAlwaysCollapsed  

enterAlwaysCollapsed是进一步修饰enterAlways属性的，上面的例子看到设置了enterAlways后，向下滚动时，ImageView首先滚动，然后才是ScrollView滚动，而设置了enterAlwaysCollapsed之后，再配合minHeight属性，可以有不同的效果，先看xml设置：  

```
 <ImageView
            android:minHeight="60dp"
         app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="fitXY"
            android:src="@drawable/pic_11"/>
```

效果如下图：  

![](https://ws1.sinaimg.cn/large/006tNbRwly1fxs8tmf3bxg30830eq7vy.gif)

可以看到设置了minHeight以及enterAlwaysCollapsed后，ImageView先滚动到最小高度，然后ScrollView滚动，最后ImageView和ToolBar一起滚动。  

### exitUtilCollapsed  

当向上滑动时，称为exit；向下滑动时，称为enter，这样理解起enterAlways和enterAlwaysCollpased就很好理解了，理解exitUtilCollapsed也好理解了。  

exitUtilCollapsed用于设置向上滚动时的最小高度，吸顶的功能。举个例子：  

```
 <ImageView
            android:minHeight="60dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="fitXY"
            android:src="@drawable/pic_11"/>
```

效果如下：  

![](https://ws2.sinaimg.cn/large/006tNbRwly1fxs95392yrg30830eqwu1.gif)

可以看到，一开始跟着ScrollView一起向上滚动，当到达最小高度后，就不再滚动，吸顶了。  向下滚动时，当ScrollView滚动顶部了，才继续滚动了。  

### snap  

snap是一个根据View在屏幕上显示范围进行调整的一个属性，看下效果其实就明白是怎么回事了。  

```
 <ImageView
            android:minHeight="60dp"
            app:layout_scrollFlags="scroll|snap"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="fitXY"
            android:src="@drawable/pic_11"/>
```

效果如下图：  

![](https://ws3.sinaimg.cn/large/006tNbRwly1fxs9dcip1vg30830eqb1i.gif)

snap属性会根据View的显示范围，判定是显示还是隐藏。  

## 监听AppBarLayout滚动  

可以通过设置监听器来监听AppBarLayout的移动，比如说随着滚动，更改AppBarLayout的透明度，代码如下：  

```
app_bar.addOnOffsetChangedListener(AppBarLayout.OnOffsetChangedListener { appBarLayout, offset ->
            val toolBarHeight = toolBar.height
            if (Math.abs(offset) <= toolBarHeight) {
                appBarLayout.alpha = 1.0f - Math.abs(offset) * 1.0f / toolBarHeight
            }
        })
```

运行效果如下：  

![](https://ws4.sinaimg.cn/large/006tNbRwly1fxsfxvn46og30830eq7wh.gif)

当向上滑时，offset的变化是0-->负数；向下滑时，负数--->0。  

## 总结

AppBarLayout是一个垂直的LinearLayout，内部可以布局多个View，在CoordinatorLayout内部与ScrollView共同作用，一共有五种scrollFlags设置，scroll是其他属性的基础。   

后面会继续学习与CollapsingToolbarLayout一起的使用。

[代码地址](https://github.com/wangli135/ClimbDemo/tree/master/behaviordemo)  

## 参考  

- https://developer.android.com/reference/android/support/design/widget/AppBarLayout  
- https://www.jianshu.com/p/5e48fb725e3f  
- https://www.jianshu.com/p/d4fd636d7c44