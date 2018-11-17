[TabLayout](https://developer.android.com/reference/android/support/design/widget/TabLayout)是开发中经常使用到的控件，经常与ViewPager一起配合使用，一组tab，可以点击、可以滚动。这不，我们的app中也是用到了这个控件，之前对这个控件只停留在最基本的用法，因此开发时也去查了些资料，趁着周末，就系统地再学习一下。  

## 基本操作  

 使用之前，首先需要在gradle文件中加入design库，

```gradle
    implementation 'com.android.support:design:28.0.0'
```

首先看一下最默认的行为与效果。  代码如下：  

``` 布局文件
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.design.widget.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="56dp">

    </android.support.design.widget.TabLayout>

</android.support.constraint.ConstraintLayout>
```

```Java
tabLayout.addTab(tabLayout.newTab().setText("Tab 1").setIcon(R.mipmap.ic_launcher));
        tabLayout.addTab(tabLayout.newTab().setText("Tab 2"));
        tabLayout.addTab(tabLayout.newTab().setText("Tab 3").setIcon(R.mipmap.ic_launcher));
        tabLayout.setOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
            @Override
            public void onTabSelected(TabLayout.Tab tab) {
                Toast.makeText(MainActivity.this,tab.getText(),Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onTabUnselected(TabLayout.Tab tab) {

            }

            @Override
            public void onTabReselected(TabLayout.Tab tab) {
                Toast.makeText(MainActivity.this,tab.getText(),Toast.LENGTH_SHORT).show();
            }
        });
```

setOnTabSelectedListener是一个过时方法，官方建议使用addOnTabSelectedListener代替，其中监听器接口是一样的，分别表示tab选中、未选中，再次选中状态。其中再次选中状态可以用于实现在选中tab的前提下，再点击时，滚动到最顶部的效果，比如很多资讯类app就是这么实现的。

效果图如下：  

![](https://ws1.sinaimg.cn/large/006tNbRwly1fxav9pqnogj30u01hcgmj.jpg)

默认效果可以看到指示器红色，三个tab平分布局，有icon的显示在文字上方。  如果TabLayout的宽度wrap_content，那么三个tab将会挤到左边，每个tab的效果是wrap_content。

以上tab是通过代码添加的，也可以在xml中进行添加，效果等效于  

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.design.widget.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="56dp">

        <android.support.design.widget.TabItem
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:icon="@mipmap/ic_launcher"
            android:text="Tab 1" />

        <android.support.design.widget.TabItem
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="Tab 2" />

        <android.support.design.widget.TabItem
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:icon="@mipmap/ic_launcher"
            android:text="Tab 3" />

    </android.support.design.widget.TabLayout>

</android.support.constraint.ConstraintLayout>
```

### xml属性说明  

对于TabLayout的样式修改，一些可以通过修改属性就行修改。

#### 修改指示器  

可以修改指示器的颜色和高度，比如：  

```
<android.support.design.widget.TabLayout
        app:tabIndicatorColor="@android:color/holo_green_dark"
        app:tabIndicatorHeight="10dp"
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="56dp"/>
```

效果如下：  

![](https://ws3.sinaimg.cn/large/006tNbRwly1fxavpl19rxj30u01hc758.jpg)  
#### 修改文字样式  
1. 选中和未选中颜色设置   
```
app:tabSelectedTextColor="#FF0000"
        app:tabTextColor="#0000FF"
```
效果如下图：  
![](https://ws1.sinaimg.cn/large/006tNbRwly1fxayfqmtfbj30u01hcwff.jpg)
2. 修改文字大小  
使用textAppearence属性，在style文件中设置大小。  
```
        app:tabTextAppearance="@style/my_tab_text_style"
```

my_tab_text_style内容如下：  
```
<style name="my_tab_text_style">
        <item name="android:textSize">20sp</item>
        <item name="android:textStyle">bold</item>
    </style>
```
效果如下： 
![](https://ws3.sinaimg.cn/large/006tNbRwly1fxayk5w8t6j30u01hct9o.jpg)  
#### 修改tab样式  
1. 使用padding参数，可以使用tabPadding进行设置，比如：  

   ```
   <android.support.design.widget.TabLayout
           app:tabPaddingTop="20dp"
           android:paddingBottom="20dp"
           android:id="@+id/tabLayout"
           android:layout_width="match_parent"
           android:layout_height="wrap_content">
   ```

   可以看到预览图的效果如下：  

   ![](https://ws1.sinaimg.cn/large/006tNbRwly1fxaypwmw8ij30l20bmt9j.jpg)

2. tabMode。tabMode支持两种值，MODE_FIXED和MODE_SCROLLABLE；当tab比较多，一屏容纳不下时，会使用MODE_SCROLLABLE，这时可以隐藏部分MODE；而FIXED的就会始终显示。 

   当在xml布局中添加了很多TabItem后，预览效果如下图：  

   ![](https://ws3.sinaimg.cn/large/006tNbRwly1fxaytvijdkj30n20asmy1.jpg)

   这时使用的就是FIXED模式，可以看到TabLayout默认就是FIXED模式；当改成MODE_SCROLLABLE后，

   ```
           app:tabMode="scrollable"
   ```

   预览样式如下图：  

   ![](https://ws4.sinaimg.cn/large/006tNbRwly1fxayw6fdlcj30pw0aidh0.jpg)

3. Tab位置。当只有三个tab时，默认分散了，如果想三个tab聚合起来，可以通过设置tabGravity属性进行设置，比如：  

   ```
           app:tabGravity="center"
   ```

   设置后的效果如下：

   ![](https://ws1.sinaimg.cn/large/006tNbRwly1fxayzolekfj30p009iq3k.jpg)

   设置前的效果就是前面三个tab平铺的效果。改属性的默认值是fill。  

   另外，可以通过设置tabContentStart设置偏移量，类似margin。  

   比如设置：  

   ```
           app:tabContentStart="100dp"
   ```

   后的效果如下图：  

   ![](https://ws3.sinaimg.cn/large/006tNbRwly1fxaza28f04j30ig06s0sy.jpg)

4. 设置tab背景。tabBakcground属性，比如设置的样式如下：  

   ```
   <shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle"
       >
   
       <size
           android:width="50dp"
           android:height="30dp"
           ></size>
   
       <gradient
           android:startColor="#FF0000"
           android:endColor="#0000FF"
           ></gradient>
   
   </shape>
   ```

   效果图：  

   ![](https://ws2.sinaimg.cn/large/006tNbRwly1fxaz496conj30i6076glt.jpg)

5. tab宽度，可以通过设置tabMaxWidth和tabMinWidth进行限制。    
### TabItem样式自定义  
以上的xml样式，都可以通过相应的set方法进行设置，但是如果想改变默认的tab样式，那么就需要代码的操作了。默认的tab样式，icon在上，text在下；下面改个icon在左，text在右的样式。    

首先定义一个布局：  

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_height="48dp"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <ImageView
        android:padding="12dp"
        android:id="@+id/iv_icon"
        tools:src="@mipmap/ic_launcher"
        android:layout_width="48dp"
        android:layout_height="48dp" />

    <TextView
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        android:textSize="18sp"
        android:textColor="#333333"
        android:id="@+id/tv_text"
        android:layout_width="0dp"
        android:layout_height="21dp"
        app:layout_constraintLeft_toRightOf="@id/iv_icon"
        app:layout_constraintRight_toRightOf="parent"
        tools:text="hha" />


</android.support.constraint.ConstraintLayout>
```

然后在代码中更改TabLayout现有Tab样式，代码如下：  

```
tabLayout=findViewById(R.id.tabLayout);

        for (int i = 0; i < tabLayout.getTabCount(); i++) {
            TabLayout.Tab tab = tabLayout.getTabAt(i);
            View view=LayoutInflater.from(this).inflate(R.layout.my_tab_layout,null);
            tab.setCustomView(view);
            ((ImageView)view.findViewById(R.id.iv_icon)).setImageDrawable(tab.getIcon());
            ((TextView)view.findViewById(R.id.tv_text)).setText(tab.getText());
        }
```

运行结果如下图：  

![](https://ws1.sinaimg.cn/large/006tNbRwly1fxb0dr04vzj30u01hcmy2.jpg)  
可以看到tab样式已经变成了icon在左，文字在右的样式。  

## 集成ViewPager 
布局如下：  
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.design.widget.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="56dp">
    </android.support.design.widget.TabLayout>

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        app:layout_constraintTop_toBottomOf="@id/tabLayout"
        app:layout_constraintBottom_toBottomOf="parent"
        android:layout_width="match_parent"
        android:layout_height="0dp"></android.support.v4.view.ViewPager>

</android.support.constraint.ConstraintLayout>
```
代码如下：  
```
public class MainActivity extends AppCompatActivity {

    private TabLayout tabLayout;
    private ViewPager viewPager;

    private int[] imgs=new int[]{
            R.drawable.pic_11,
            R.drawable.pic_12,
            R.drawable.pic_11
    };

    private String[] tabs=new String[]{
      "Tab 1",
      "Tab 2",
      "Tab 3",
    };


    private List<ImgFragment> imgFragments=new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tabLayout=findViewById(R.id.tabLayout);
        viewPager=findViewById(R.id.viewpager);

        for (int i = 0; i < imgs.length; i++) {
            imgFragments.add(ImgFragment.newInstance(imgs[i]));
        }

        viewPager.setAdapter(new FragmentPagerAdapter(getSupportFragmentManager()) {
            @Override
            public Fragment getItem(int i) {
                return imgFragments.get(i);
            }

            @Override
            public int getCount() {
                return imgFragments.size();
            }

            @Nullable
            @Override
            public CharSequence getPageTitle(int position) {
                return tabs[position];
            }
        });

        tabLayout.setupWithViewPager(viewPager);

    }
}

```
运行效果如下：  
![](https://ws2.sinaimg.cn/large/006tNbRwly1fxb117x4xgg30dq0kckjl.gif) 
这里需要注意的是：当调用了setupWithViewPager之后，tab值默认将会从getPageTitle中获取；如果这个时候没有重写PageAdapter的getPageTitle，那么效果将会如下图：  
![](https://ws3.sinaimg.cn/large/006tNbRwly1fxb14rz7tdj30qy182433.jpg)
这个时候可以发现TabLayout上面都是空的，但其实是有tab的，只不过tab内容为空而已。这个时候可以通过代码重新设置，比如：  
```
tabLayout.setupWithViewPager(viewPager);
        for (int i = 0; i < tabLayout.getTabCount(); i++) {
            TabLayout.Tab tab=tabLayout.getTabAt(i);
            tab.setText(tabs[i]);
            tab.setIcon(imgs[i]);
        }
```
效果如下图：  
![](https://ws3.sinaimg.cn/large/006tNbRwly1fxb17egnnqj30qy182tdi.jpg)
可以看到，不止设置了text，还设置了title，使用PageAdapter只能设置text。  

## 总结  

至此，TabLayout的基本用法也就是这样了；除了这个，还有与Toolbar以及协调布局共同使用的情况，这个以后有机会会继续深入的学习下。  

[DEMO地址](https://github.com/wangli135/ClimbDemo/tree/master/tablayoutdemo)  



## 参考内容  

- https://developer.android.com/reference/android/support/design/widget/TabLayout  
- [TabLayout之自定义样式](https://www.jianshu.com/p/ed129686f2cc)
- [MaterialDesign之对TabLayout的探索](https://www.jianshu.com/p/bbefb97cccdd)
- [Design库-TabLayout属性详解](https://www.jianshu.com/p/d4c0bc0cab38)


