有时候，一个列表中的Item会有EditText的出现，而由于View复用机制，如果不好好处理EditText，将会出现一些问题。之前做项目中也遇到了这个问题，通过摸索以及思考，最终得到了解决方案。    

其实有些问题的出现，还是由于没有理解RecyclerView的复用机制和EditText，主要原因还是菜，哈哈。  

**菜是原罪**

## EditText在RecyclerView中的问题  

例子是这样的，每个Item包含一个title、一张图片以及一个评分，这个评分就是通过输入框来输入的。  

### 问题1——复用机制、未绑定数据导致的

先看下第一段Adapter里面的逻辑：  

```PicViewHolder
class PicViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

    var ivPic: ImageView = itemView.findViewById(R.id.ivPic)
    var etScore: EditText = itemView.findViewById(R.id.etScore)
    var tvTitle: TextView = itemView.findViewById(R.id.tvTitle)

    fun updateView(picItem: PicItem) {
        ivPic.setImageResource(picItem.picResId)
        tvTitle.text = picItem.title
        if (picItem.score == null) etScore.hint = "请输入分数" else etScore.setText(picItem.score)
    }
}
```
上面是ViewHolder的代码，Adapter的onBindViewHolder就是调用了updateView()方法，这里面暂时还没把输入框的内容和PicItem绑定。先看下效果如下：  
![EditText未绑定数据的情形](https://ws3.sinaimg.cn/large/006tNbRwly1fy8keuki9xg307z0dbx6q.gif)  

这里，每张图片输入图片title对应的分数，可以看到，由于未绑定数据和RecyclerView的复用机制的存在，在一些图片中还没输入分数，就已经出现分数了。那下面先来进行数据的绑定。  

### 问题2——错误的绑定机制  

要想在EditText输入后绑定数据，怎么搞？TextWatcher了解一下，操作方式如下：  

```
class PicViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

    var ivPic: ImageView = itemView.findViewById(R.id.ivPic)
    var etScore: EditText = itemView.findViewById(R.id.etScore)
    var tvTitle: TextView = itemView.findViewById(R.id.tvTitle)
    var myTextWatcher: MyTextWatcher=MyTextWatcher()

    fun updateView(picItem: PicItem) {
        ivPic.setImageResource(picItem.picResId)
        tvTitle.text = picItem.title
        if (picItem.score == null) etScore.hint = "请输入分数" else etScore.setText(picItem.score)
        myTextWatcher.picItem=picItem
        etScore.addTextChangedListener(myTextWatcher)
    }
}

class MyTextWatcher: TextWatcher {

    lateinit var picItem:PicItem

    override fun afterTextChanged(s: Editable?) {
        picItem?.apply {
            score=s?.toString()
        }
    }

    override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {
    }

    override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
    }
}
```

效果如下：  

![TextWatcher的不正确使用](https://ws2.sinaimg.cn/large/006tNbRwly1fy8kvdf76vg307z0dbu0z.gif)  

可以发现，还是乱的。想了想，这是咋回事呢？原来是因为这里是addTextWatcher，而不是setTextWatcher，也就是在复用的时候，同一个EditText添加了多个TextWatcher，怪不得分数9还能出现在上面了。  

想清了原因，那就好办了。  首先我是试了一个，removeTextWatcher的方法，那就是在Adapter的detachViewHolderFromWindow方法中移除TextWatcher，如下：  

```
class PicAdapter(val picItems: List<PicItem>) : RecyclerView.Adapter<PicViewHolder>() {
    override fun onViewDetachedFromWindow(holder: PicViewHolder) {
        super.onViewDetachedFromWindow(holder)
        holder.etScore.removeTextChangedListener(holder.myTextWatcher)
    }
}
```

实践发现，依赖onViewDetachedFromWindow是不靠谱的。  

## 解决方案  

经过思考，由于RecyclerView的复用机制，导致了以下关系的存在：  

一个ViewHolder——>一个EditText——>多个TextWatcher——>多个PicItem  

这里我们可以将多个TextWatcher始终绑定一个，那就需要在ViewHolder的初始化里面操作，而不是在updateView，因为会多次bind，这就得到了以下关系：  

一个ViewHolder——>一个EditText——>一个TextWatcher——>多个PicItem    

那么也就是说TextWatcher负责多个PicItem的更新，怎么做呢？  

很简单，在updateView()，也就是bind过程中每次去更新PicItem就可以了。最终代码如下：  

```
class PicViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

    var ivPic: ImageView = itemView.findViewById(R.id.ivPic)
    var etScore: EditText = itemView.findViewById(R.id.etScore)
    var tvTitle: TextView = itemView.findViewById(R.id.tvTitle)
    var myTextWatcher: MyTextWatcher = MyTextWatcher()

    init {
        etScore.addTextChangedListener(myTextWatcher)
    }

    fun updateView(picItem: PicItem) {
        myTextWatcher.picItem = picItem
        ivPic.setImageResource(picItem.picResId)
        tvTitle.text = picItem.title
        if (picItem.score == null) {
            etScore.hint = "请输入分数"
            etScore.setText("")
        } else {
            etScore.setText(picItem.score)
        }
    }
}
```

最终效果如下：  

![正确效果](https://ws4.sinaimg.cn/large/006tNbRwly1fy8lij87hxg307z0dbqva.gif)

模拟器滑动起来感觉有点便秘，将就看着吧。。。。  

另外如果对这个美女感兴趣，她叫刘芸，不要谢我，去百度搜图片去吧。。。😆😀😀  

## 总结  

其实后来想想，如果能明白RecyclerView复用机制，EditText的TextWatcher机制，其实很容易解决这种问题，那么绕路了的原因就是因为**菜**。哎，不多说了，学习去了。。  



代码地址：https://github.com/wangli135/ClimbDemo/tree/master/app

