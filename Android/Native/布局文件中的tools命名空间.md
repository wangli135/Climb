参加工作，发现自己对很多Andriod的细节了解的还是太少了，现在想想当时在学校，有点重java se而	轻android了，都去看java集合库、并发库的源码了，而对Android的很多细小的知识点忽略了。but，我是一个Android开发者啊，所以现在得慢慢的把这些再补起来了。  

Android的布局文件写在xml中，一共有三种命名空间，分别是：android、app以及tools。

```XML
xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
```

之前对android和app和熟悉，但是对tools的认知为0。工作中初步认识是，用tools可以设置看看效果，到了真实环境，那些设置的就不见了。因此周末时间就赶紧来补补课了。  

## tools命名空间

tools命名空间下的属性允许设计时特性（只在设计时可见）和编译时行为。编译工具会将tools的属性移除，从而不影响apk的大小。举个例子，有个Textview，我们要看看显示效果，如果用android：text设置，有时忘记删除；但是用tools：text就不用管了，既可以设计时看到效果，又不需要编译前删除。  

### 设计时观察属性  

下面的属性设置只在设计的预览图中可见。

- tools：取代android
在View下，能用android可以换为tools，比如tools:text，tools:src,tools:visibility...

- tools：itemCount  
```
<android.support.v7.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:itemCount="3"/>
```
指定RecyclerView的可见数目

- tools:listitem / tools:listheader / tools:listfooter  
可用于AdapterView  
```
<ListView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@android:id/list"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:listitem="@layout/sample_list_item"
    tools:listheader="@layout/sample_list_header"
    tools:listfooter="@layout/sample_list_footer" />

```
- tools:minValue / tools:maxValue  
可用于NumberPicker  
```
<NumberPicker xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/numberPicker"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    tools:minValue="0"
    tools:maxValue="10" />
```
### 资源压缩属性  
开启资源压缩，是需要在build.gradle文件中设置的，如下：  
```
android {
    ...
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
}
```
- tools:keep  
开启资源压缩后，这个用于保留没用被用到的资源。  
- tools:discard  
手动表明资源废弃

## 参考  
- [Tools attributes reference](https://developer.android.com/studio/write/tool-attributes)
- [[译]精通 Android 中的 tools 命名空间](https://www.jianshu.com/p/a39dddb46bd8)