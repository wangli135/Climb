# 介绍  

[JiaoZiVideoPlayer](https://github.com/lipangit/JiaoZiVideoPlayer)是一款Android视频播放器，先如今抖音之类的小视频风靡，还不赶紧学习下如何开发小视频应用？  

# 使用  

## 入门使用  

1. 申明网络权限
2. 引入库，包括JiaoZiVideoPlayer和Glide  
```
 implementation 'com.github.bumptech.glide:glide:4.8.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'
    implementation 'cn.jzvd:jiaozivideoplayer:6.3.1'
```
3. 在布局中加入播放器布局  
```
<cn.jzvd.JzvdStd
        android:id="@+id/videoplayer"
        android:layout_width="match_parent"
        android:layout_height="200dp" />
```
4. 在Activity中初始化及加载视频  
```java
JzvdStd myJzvdStd;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        myJzvdStd = findViewById(R.id.videoplayer);
        myJzvdStd.setUp("http://jzvd.nathen.cn/342a5f7ef6124a4a8faf00e738b8bee4/cf6d9db0bd4d41f59d09ea0a81e918fd-5287d2089db37e62345123a1be272f8b.mp4"
                , "饺子快长大", JzvdStd.SCREEN_WINDOW_NORMAL);
        Glide.with(this).load("http://jzvd-pic.nathen.cn/jzvd-pic/1bb2ebbe-140d-4e2e-abd2-9e7e564f71ac.png").into(myJzvdStd.thumbImageView);
    }

    @Override
    public void onBackPressed() {
        if (Jzvd.backPress()) {
            return;
        }
        super.onBackPressed();
    }

    @Override
    protected void onPause() {
        super.onPause();
        Jzvd.releaseAllVideos();
    }
```
5. 效果  
![jiaozi简单demo](https://ws3.sinaimg.cn/large/006tNbRwgy1fvkp4m23l2g30au0i2b2e.gif)

## 简单功能  

1. 自动播放  

```java
myJzvdStd.startButton.performClick();
myJzvdStd.startVideo();
```
可以在onResume()中加入上段代码，视频将会自动下载播放  

2. 全屏播放  

```java
           JzvdStd.startFullscreen(this, JzvdStd.class, "http://2449.vod.myqcloud.com/2449_22ca37a6ea9011e5acaaf51d105342e3.f20.mp4", "嫂子辛苦了");
```
3. 跳转到某一时间进行播放  
   - 在播放器播放之前，setUp之前进行设置  
```
myJzvdStd.seekToInAdvance = 20000;
```
   - 在播放中强制跳转
```
JZMediaManager.seekTo(30000);
```
参数表示时间，单位是毫秒。  
4. 小窗播放  
```
myJzvdStd.startWindowTiny();
```
效果图如下：  
![小窗播放](https://ws3.sinaimg.cn/large/006tNbRwgy1fvkpmb8lqgg30au0i2x6p.gif)

## 扩展播放器  
如果需要实现自己的功能，可以继承JzvdStd进行扩展功能，下面举几个例子。
### 小屏播放无声音，全屏有声音
```
public class JzvdStdVolumeAfterFullscreen extends JzvdStd {

    public JzvdStdVolumeAfterFullscreen(Context context) {
        super(context);
    }

    public JzvdStdVolumeAfterFullscreen(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public void onPrepared() {
        super.onPrepared();
        if(currentScreen==SCREEN_WINDOW_FULLSCREEN){
            JZMediaManager.instance().jzMediaInterface.setVolume(1.0f,1.0f);
        }else{
            JZMediaManager.instance().jzMediaInterface.setVolume(0.0f,0.0f);
        }
    }

    //进入全屏，关闭静音
    @Override
    public void startWindowFullscreen() {
        super.startWindowFullscreen();
        JZMediaManager.instance().jzMediaInterface.setVolume(1.0f,1.0f);
    }

    //退出全屏，开启静音
    @Override
    public void playOnThisJzvd() {
        super.playOnThisJzvd();
        JZMediaManager.instance().jzMediaInterface.setVolume(0.0f,0.0f);
    }
}
```
接下来，在xml中将JzvdStd更改为该布局即可实现全屏有声音，小屏无声音。  
### 小屏状态不显示标题，全屏状态显示标题  
```
public class JzvdStdShowTitleAfterFullscreen extends JzvdStd {
    public JzvdStdShowTitleAfterFullscreen(Context context) {
        super(context);
    }

    public JzvdStdShowTitleAfterFullscreen(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public void setUp(JZDataSource jzDataSource, int screen) {
        super.setUp(jzDataSource, screen);
        if (currentScreen == SCREEN_WINDOW_FULLSCREEN) {
            titleTextView.setVisibility(View.VISIBLE);
        } else {
            titleTextView.setVisibility(View.INVISIBLE);
        }
    }
}
```
# 参考  

   -  [JiaoZiVideoPlayer使用说明（持续更新中...）](https://www.jianshu.com/p/4c187a09b838)
   -  [饺子视频播放器的小白使用说明](https://shimo.im/docs/xj5F85W1gqEEBXRJ)

