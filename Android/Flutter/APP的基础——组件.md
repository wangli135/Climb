一个APP除了逻辑外，最主要的就是UI，Flutter提供了很多已有的View（在Flutter中叫Widgets）。本文将主要介绍这些Widgets，这是创建APP的基础。  

Widgets分为几类，下面分别介绍。  



# Basics  

基础组件是创建APP前必须要了解的，**非常重要**。

## Row  

 横向布局一系列子Widgets。（和LinearLayout的水平方向大致）。Row组件不支持滚动，因此如果子Widgets超过了Row的空间，那么将会报错。如果想使用一个可以滚动的组件，可以使用ListView。

如果需要子Widgets进行填充，那么可以使用Expand组件进行包装。  

```dart
void main() {
  runApp(new MaterialApp(
    home: new RowDemo(),
  ));
}

class RowDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Row(
      children: <Widget>[
       const Text(
          'Flutter\'s hot reload helps you quickly and easily experiment, build UIs, add features, and fix bug faster. Experience sub-second reload times, without losing state, on emulators, simulators, and hardware for iOS and Android.',
          textAlign: TextAlign.start,
        ),
        const Text(
          'Hello World',
          textAlign: TextAlign.start,
        ),
      ],
    );
  }
}
```

在Row中摆放两个Text，其中第一个Text内容很多，超过了屏幕显示范围，导致第二个Text没有显示，下面是结果：  

![RowDemo_超过Row范围](http://ww1.sinaimg.cn/large/ab7f4b86gy1fva0xfnin8j20u01hcq40.jpg)

可以看到Flutter将会提示右边超过了多少像素，那么如果需要将两个Text同时显示，可以使用Expand对第一个Text进行包装，如下：  
```dart
class RowDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Row(
      children: <Widget>[
        const Expanded(
          child: const Text(
            'Flutter\'s hot reload helps you quickly and easily experiment, build UIs, add features, and fix bug faster. Experience sub-second reload times, without losing state, on emulators, simulators, and hardware for iOS and Android.',
            textAlign: TextAlign.start,
          ),
        ),

        const Text(
          'Hello World',
          textAlign: TextAlign.start,
        ),
      ],
    );
  }
}
```

这个时候的显示结果如下：  

![RowDemo_Expand](http://ww1.sinaimg.cn/large/ab7f4b86gy1fva11so5z6j20u01hcq3n.jpg)

可以看到，第一个Text在垂直方向进行了扩展，给第二个Text腾了个位子。  

如果需要像LinearLayout一样设置Weight，可以将二个Text也用Expand进行包装，结果如下图：  

![RowDemo_Weight属性](http://ww1.sinaimg.cn/large/ab7f4b86gy1fva1574cr2j20u01hcgn3.jpg)

### 布局算法  

Row的布局分为六步：  

1. 使用不受限制的水平约束和受限制的垂直约束布局非Expanded组件

2. 在剩下的水平空间中按照flex参数将设下的组件进行分割

3. 使用不受限制的水平和垂直约束，布局剩下的组件

4. 在满足传入高度约束的前提下，Row的height是最大子组件的height

5. Row的高度决定于mainAxisSize属性。

6. 按照mainAxisAlignment和crossAxisAlignment属性进行摆放组件位置。

## Column  

Column与Row相对，垂直方向的LinearLayout，因此不过多介绍。

## Text

Text组件和TextView相似，使用单一style显示文本。  

## Image

Image组件和ImageView相似，显示一张图片。Image提供了不少构造方法，用于从不同源加载图片。  

1. new Image，从ImageProvider中获取图片

2. new Image.asset，从AssetBundle中获取图片

3. new Image.network，从网络获取图片

4. new Image.file，从File获取图片

5. new Image.memory，从Unit8List中获取图片

## Icon

Icon代表着不能交互的图标，如果需要交互，那么需要使用IconButton。Icons工具类提供了很多常用的Icon，不得不说，Flutter这一点太nice了。  

## Container

Container是一个可以放置一个子Widget的容器，可以设置padding、margin这些参数。

在绘制过程中，Container首先应用传入的transform，然后用decoration绘制padded区域，然后绘制子组件，最后绘制foregroundDecoration。  

## AppBar

![](https://flutter.github.io/assets-for-api-docs/assets/material/app_bar.png)

AppBar中的参数说明如上图。

## Scaffold

这个组件实现了material design基本的布局。提供了显示drawers、snack bars和bottom sheets。

参数有如下：  

```dart
this.appBar,
    this.body,
    this.floatingActionButton,
    this.floatingActionButtonLocation,
    this.floatingActionButtonAnimator,
    this.persistentFooterButtons,
    this.drawer,
    this.endDrawer,
    this.bottomNavigationBar,
    this.backgroundColor,
    this.resizeToAvoidBottomPadding: true,
    this.primary: true,
```

# Text

显示和格式化文本。  

## Text  

单一style的文本。

## RichText

一段富文本。  

使用多种style显示文本。

```
new RichText(
  text: new TextSpan(
    text: 'Hello ',
    style: DefaultTextStyle.of(context).style,
    children: <TextSpan>[
      new TextSpan(text: 'bold', style: new TextStyle(fontWeight: FontWeight.bold)),
      new TextSpan(text: ' world!'),
    ],
  ),
)
```

## DefaultTextStyle

# Input  

除了Material和Cupertino中的输入组件。  

## Form  

将多个组件组合蕲艾的容器，比如说TextField组件。  

每一个单独的form字段必须使用FormField组件进行包装，调用FormState来保存、充值或更新状态。  

可以使用Form.of获取一个FormState。  

## FormField  

一个单独的form字段。  

这个组件管理当前form的状态。当在Form中使用时，你可以使用FormState上的方法来查询和操作form数据。

# Material组件  

## App结构和导航  

- Scaffod

- Appbar

- BottomNavigatonBar

- TabBar

- TabBarView

- MaterialApp

- WidgetsApp

- Drawer  

## Buttons  
- RaisedButton 
- FloatingActionButton
- FlatButton  
- IconButton
- PopupMenuButton
- BUttonBar

## Input和Selections
- TextField  
- CheckBox
- Radio
- Switch
- Slider
- Date&Time Pickers

## 对话框  
- SimpleDialog  
- AlertDialog
- ButtonSheet
- ExpansionPanel  
- SnackBar  

## 信息展示  
- Image   
- Icon
- Chip
- Tooltip  
- DataTable  
- Card  
- LinearProgresssIndicator  
- Gridview

## Layout
- ListTile
- Stepper
- Diveider  

根据上面的控件，写了一个例子，如下图：  

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fvg9otscn0g30c20i5nb4.gif)

[代码地址](https://github.com/wangli135/flutter_demo/blob/master/lib/widgets_category.dart)

