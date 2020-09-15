# UI

## 一、Image

### 1.1 正确示范

```dart
CachedNetworkImage(
                          imageUrl: data?.avatar ?? "",
                          placeholder: (context, url) {
                            return Image(
                              height: 32,
                              width: 32,
                              image: AssetImage("image/init_personal_icon.png"),
                            );
                          },
                          width: 32,
                          height: 32,
                          fit: BoxFit.cover,
                        ),
```

***

### 1.2 错误情况

+ 1.2.1、 版本号错误

+ 1.2.2、 可访问非图片url，如https://flutter.dev/docs

![image-20200907154908621](/uploads/b8c9c5428719aaf70578e45c9b8197b7/image-20200907154908621.png)
+ 1.2.3、 不可访问url，如随便一个字符串

![image-20200907154530650](/uploads/95a1c2ff8fab8ac00289f344e8942b80/image-20200907154530650.png)


***

### 1.3 部分代码用处

+ 1.3.1 **placeholder**:  

&emsp;&emsp;当没有图片或者在加载图片的时候显示指定的图片，或者动画效果。

+ 1.3.2 **fit: BoxFit.cover**:  

&emsp;&emsp;让图像自适应填充。

+ 1.3.3 **CachedNetworkImage**:  

&emsp;&emsp;`CachedNetworkImage` 与 `Image.network` 都是从网络请求加载图片。但是`cached_network_image` 这个库实现了图片缓存加载和载入效果而 `Image.network`记载图片没有任何的缓存，每次都是重新加载时间比较慢，同时没有任何的动画效果。

&emsp;&emsp;`CachedNetworkImage`多用于需要从网络拉取图片的情景。

+ 1.3.4 **errorWidget**:

  &emsp;&emsp;看官方文档吧，很简单的https://pub.dev/packages/cached_network_image。

## 二、UI与原生界面的交互


### 2.1&emsp;&emsp;Dart -> Native(UI传递数据给原生界面)

```dart
final methodChannel = MethodChannel("HomeFlutterPage");
```

  在实际开发中  MethodChannel() 里传入的字符串参数和自己的page名相同。

  这段代码的用处是告诉原生界面要你调用什么函数。在这个例子中，你告诉原生界面你要调用的函数叫 `HomeFlutterPage`,`HomeFlutterPage`作为通道的唯一标志符,用于区分不同的通道调用。



```dart
methodChannel.invokeMethod("loginStateResult", false);
```

拿到MethodChannel对象后,通过调用其`invokeMethod()`方法用于向平台发起一次调用.在`invokeMethod()`方法中会将一次方法调中的方法名method (在这里是 `loginStateResult`) 和方法参数arguments (在这里是`false`) 封装为MethodCall对象,然后使用MethodCodec对其进行二进制编码,最后通过`BinaryMessages.send()`发起平台方法调用请求.



```dart
await _methodChannel.invokeMethod("nimUserInfo", {"account": account.account});
```

在这里方法参数arguments是{"account": account.account}。是告诉原生界面名为account的变量的值，为UI这边的account.account的值。



###  2.2&emsp;&emsp;Native -> Dart(原生界面传递数据给UI)

```dart
_methodChannel.setMethodCallHandler(
  (call) async {
    if (call.method == "nimTeamInfoUpdate") {
      // 更新im群信息
      String teamInfoJsonSource = call.arguments;

      if (teamInfoJsonSource != null) {
        ImTeamInfoEntity imTeamInfo = ImTeamInfoEntity().fromJson(jsonDecode(teamInfoJsonSource));
        groupMemberList = imTeamInfo.members;

        // 更新群成员
        for (final account in imTeamInfo.members) {
          await _methodChannel.invokeMethod("nimUserInfo", {"account": account.account});
        }
      }
      setState(() {});

    } else if (call.method == "nimUserInfoUpdate") {
      // 更新im用户信息
      String userInfoJsonSource = call.arguments;

      if (userInfoJsonSource != null) {
        ImUserInfoEntity imUserInfo =
        ImUserInfoEntity().fromJson(jsonDecode(userInfoJsonSource));
        userInfoMap[imUserInfo.account] = imUserInfo;
      }
      generateSectionList();
    }

    return null;
  },
);
```

通过call.method和call.arguments可以获得原生界面的方法名和传递过来的参数。

**坑：一开始将原生 MethodChannel 写到initState外面，会导致 Flutter 没收到请求**

因为 Flutter 是在 initState 里面去 setMethodCallHandler 的，而 debug 模式下可能 Flutter 还没加载完成，这个时候发送消息，Flutter 就可能没收到。

## 三、组装数据

### 3.1  UI是用数据驱动的

>数据的组装最好是在page页面完成，然后在传入Widget页面里。

>建议把dart的语法糖、Map、List、for循环的各种操作仔细阅读。

http://codingdict.com/article/21992

https://dart.cn/guides/language/language-tour#operators




## 四、





## 五、Container中的Text居中显示

### 5.1 错误情况
```dart
// 错误代码
Container(
  width: 120,
  color: Colors.red,
  alignment: Alignment.center,
  child: Text(
    "123",
    style: TextStyle(
      color: Colors.black,
      fontSize: 16,
    ),
  ),
),
```

![image-20200907155911371](/uploads/1465f32bcfb017d11a8c30ce74804591/image-20200907155911371.png)


- `alignment`：这个属性针对的是Container内child的对齐方式，也就是容器子内容的对齐方式，并不是容器本身的对齐方式。在Container中给这个属性赋值会使得它实际的宽高变成各种约束条件下的最大值。

&emsp;&emsp;原因：在Container中添加alignment属性或者child是Align时，Container若没有限制宽/高，会在横/纵向铺满其父组件。正确的方法是使用Text中的textAlign属性

### 5.2 正确示范
```dart
// 正确代码
Container(
  width: 120,
  color: Colors.red,
  child: Text(
    "123",
    style: TextStyle(
      color: Colors.black,
      fontSize: 16,
    ),
    textAlign: TextAlign.center,
  ),
),
```
- `textAlign`：输入框内编辑文本在水平方向的对齐方式。

## 六. ListView.builder常见问题

### 6.1 当ListView.builder作为Column的直接子项时

#### 6.1.1 错误情况

```dart
Column(
  children: <Widget>[
    buildAppBar(),
    ListView.builder(
      itemCount: 20,
      itemBuilder: ...,
    )
  ]
)
```

![image-20200908101732593](/uploads/242b46db4e97b9675893994d973a5cd5/image-20200908101732593.png)

&emsp;&emsp;原因：根据错误信息，原因是ListView此时是一个没有限制高度的viewport

#### 6.1.2 改进1.0

```dart
Column(
  children: <Widget>[
    buildAppBar(),
    ListView.builder(
      shrinkWrap: true,
      itemCount: 20,
      itemBuilder: ...,
    )
  ]
)
```

- `shrinkWrap`：该属性表示是否根据子组件的总长度来设置`ListView`的长度，默认值为`false` 。默认情况下，`ListView`的会在滚动方向尽可能多的占用空间。当`ListView`在一个无边界(滚动方向上)的容器中时，`shrinkWrap`必须为`true`。

&emsp;&emsp;当把`ListView.builder`的`shrinkWrap`属性置为`true`后，编译器不再提示以上错误，但是当`ListView`中的内容超过屏幕高度后，底部会报溢出错误。

![image-20200908104523458](/uploads/d8beac6f6ce0db33c883db3020d45091/image-20200908104523458.png)

#### 6.1.3 正确代码

```dart
Expanded(
  child:Column(
    children: <Widget>[
      buildAppBar(),
      ListView.builder(
        shrinkWrap: true,
        itemCount: 20,
        itemBuilder: ...,
      )
    ]
  )
)
```

- `Expanded`：一般作为`Column`、`Row`或`Flex`的子组件，以下内容摘自文档

Using an Expanded widget makes a child of a Row, `Column`, or `Flex` expand to fill the available space along the main axis (e.g., horizontally for a `Row` or vertically for a Column). If multiple children are expanded, the available space is divided among them according to the flex factor.

An `Expanded` widget must be a descendant of a `Row`, Column, or `Flex`, and the path from the Expanded widget to its enclosing `Row`, Column, or `Flex` must contain only `StatelessWidgets` or StatefulWidgets (not other kinds of widgets, like `RenderObjectWidgets`).

https://api.flutter.dev/flutter/widgets/Expanded-class.html

## 七. AppBar的左中右布局

### 7.1 错误写法

```dart
Container(
  height: 44,
  ...,
  child: Stack(
    children:<Widget>[
      Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children:<Widget>[
          Container(
            height: 44,
            width: 40,
            child: ...,
          ),
          Container(
            alignment: Alignment.centerRight,
            padding: EdgeInsets.only(right: 10),
            child: Text(
              "邀请攻略",
              ...,
            ),
          )
        ]
      ),
      Center(
        child: Text(
          "邀请好友",
          ...,
          textAlign: TextAlign.center,
        ),
      )
    ]
  )
)
```

### 7.2 正确示范

```dart
Container(
  alignment: Alignment.center,
  height: 44,
  ...,
  child: Row(
    children: <Widget>[
      Container(
        alignment: Alignment.centerLeft,
        margin: EdgeInsets.only(left: 16.5),
        height: 44,
        width: 40,
        child: ...,
      ),
      Expanded(
        child: Text(
          "邀请好友",
          ...,
          textAlign: TextAlign.center,
        ),
      ),
      Container(
        alignment: Alignment.centerRight,
        padding: EdgeInsets.only(right: 10),
        child: Text(
          "邀请攻略",
          ...,
        ),
      )
    ],
  ),
)
```

&emsp;&emsp;原因：`Column`、`Row`中一般用`Expanded`来扩展以填充主轴上的可用空间

## 八、快捷键

### 8.1 option+Enter

在flutter嵌套结构中，`option + Enter`快捷键可以快捷添加或删除修改某个widget。

首先单击选择目标widget，然后按`option + Enter`键，弹出可进行的操作，单击选项执行相应操作。

![截屏2020-09-09_下午3.12.20](/uploads/d2199b4c0568f5bed1409d6a78752586/截屏2020-09-09_下午3.12.20.png)

Wrap with widget...：将目标widget放到任意widget中。

Wrap with Center：将widget放到Center部件中，使其居中。

Wrap with Column（Row）：一次性将一个或多个widget放在Column（Row）中。

Wrap with Padding：将widget添加一层Padding。

Wrap with StreamBuilder：将widget放在Streambuilder中。

Swap with child：将widget与其子widget交换。

Remove this widget：将widget作为最外层部件移除。

光标选中的widget为Row或Column时，Move widget up（down）：将widget在Row或Column中的顺序往左右或上下调整。

光标选中的是类名时，可出现Convert class into `mixin`、Conver to `StatelessWidget` 和Convert to `StatefulWidget`选项。

***

### 8.2 创建新的StatefulWidget或StatelessWidget

输入`stful`可快捷创建一个StatefulWidget类与其State类，需要重写的build和createState方法也会被同时创建。

输入`stless`可快捷创建一个Stateless类，与其需要重写的build方法。

***

### 8.3 其他

+ 选择整个widget的代码部分

点击选择widget，按下`option + up`快捷键可选中widget从起始到结束位置的所有代码。

+ 代码格式化并对齐

`option + command + l`

+ 删除未使用的import 

`ctrl + option+o`

+ 搜索查找替换

`command + f`：当前文件搜索

`command + shift + f`：全局查找

`command + o`：全局搜索整个项目的类、文件、关键字。

`command + r`：当前文件搜索并替换

`command + shift + r`：全局替换

+ 抽取代码为单独的widget

`command + option + w`

+ 抽取代码为单独的方法

`command + option + m`

+ 查看抽象类的实现

`command + option + b`

+ 上下移动代码

`option + shift + up/down`

+ 热重载(hot reload)

`command + \`

+ 热重启(hot restart)

`command + option + \`

+ 运行项目

`control + r`

+ 删除行

`command + delete`

+ 注释与取消注释,效果`/**/`

`command + option + /`

+ if后面自动加`(){ }`

`command + shift + Enter`

+ 快速生成模版代码块

`command + j`

+ Surround with快速调出if,for,try…catch,while等环绕代码

`command + option + t`：需要选中需要嵌套的代码块

+ 快速导入头文件

`option + Enter`

## 九、json转model类

### 9.1 当json内容为Map

```json
{
        "uid": 3,
        "avatar": "https://c-ssl.duitang.com/uploads/item/201704/03/20170403154201_xMhaZ.thumb.400_0.jpeg",
        "name": "滴滴司机",
        "nimAccount": "5d4c29c22abe40a198c0f4bd5297c540"
}
```

+ 新建一个可以将json转换的modle类，添加`JsonConvert`的功能

  New -> `JsonToDartBeanAction` -> 输入model名称和Json文本 -> Make

  ```dart
  class RgSlaveEntity with JsonConvert<RgSlaveEntity> {
    int uid;
    String avatar;
    String name;
    String nimAccount;
  }
  ```

+ 将json转换为一个modle类

  ```dart
  RgSlaveEntity().fromJson(json);
  ```

***

### 9.2 当json内容为List

```json
{
		[{
        "uid": 2,
        "avatar": "https://c-ssl.duitang.com/uploads/item/201907/28/20190728002855_grzes.thumb.400_0.jpg",
        "name": "厨师",
        "nimAccount": "05af6f617e2c4583840f4a8b68862fc2"
    }, {
        "uid": 3,
        "avatar": "https://c-ssl.duitang.com/uploads/item/201704/03/20170403154201_xMhaZ.thumb.400_0.jpeg",
        "name": "滴滴司机",
        "nimAccount": "5d4c29c22abe40a198c0f4bd5297c540"
    }]
}
```

+ 使用`JsonConvert`类中的方法`fromJsonAsT`

  `fromJsonAsT`方法可以通过泛型的方法来获得Entity实例。当泛型为List<A>类型时，可以解析返回对应的List。

  ```dart
  List<RgSlaveEntity> rgSlaveEntityList = JsonConvert.fromJsonAsT<List<RgSlaveEntity>>(json);
  ```

  
