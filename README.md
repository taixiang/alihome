>原文链接：[https://mp.weixin.qq.com/s/GegMt7GDBCFVoUgFQWG3Sw](https://mp.weixin.qq.com/s/GegMt7GDBCFVoUgFQWG3Sw)

每次打开支付宝首页滑动，头部的伸缩动画甚是吸引人。于是自己决定动手来实现一个。    

无图言虚空，效果图：  

![](https://user-gold-cdn.xitu.io/2018/7/8/164782ba281bee30?w=480&h=289&f=gif&s=1489086)   

首先看到这种效果，第一反应就是`coordinatorLayout`布局，android studio新建项目时，可以直接新建个Scrolling Activity来查看demo效果。  

![](https://user-gold-cdn.xitu.io/2018/7/7/16472cb3bf607b71?w=855&h=500&f=jpeg&s=50655)   

官方自带的布局示例：   

![](https://user-gold-cdn.xitu.io/2018/7/7/164731a43c97bb42?w=480&h=348&f=gif&s=2036139)

#### gradle关联
```
implementation 'com.android.support:design:26.1.0'
```
简单介绍使用到的几个布局：    
#### coordinatorLayout
`coordinatorLayout`协调者布局，用来协调其子view并以触摸影响布局的形式产生动画效果的布局。`coordinatorLayout`是一个顶级布局。  
#### appBarLayout
`appBarLayout`主要给子布局配置属性`app:layout_scrollFlags`，5个属性值：  
`scroll`：所有想滚动出屏幕的view都需要设置这个属性值， 没有设置这个属性的view将被固定在屏幕顶部    
`enterAlways`：任意向下的滚动都会导致该view变为可见，启用快速“返回模式”   
`enterAlwaysCollapsed`：假设你定义了一个最小高度（minHeight）同时enterAlways也定义了，那么view将在到达这个最小高度的时候开始显示，并且从这个时候开始慢慢展开，当滚动到顶部的时候展开完。   
`exitUntilCollapsed`：当你定义了一个minHeight，此布局将在滚动到达这个最小高度的时候折叠。   
`snap`：当一个滚动事件结束，如果视图是部分可见的，那么它将被滚动到收缩或展开。例如，如果视图只有底部25%显示，它将折叠。相反，如果它的底部75%可见，那么它将完全展开。  

这里的属性可以组合使用查看效果。  
例如demo中自带的   
`app:layout_scrollFlags="scroll|exitUntilCollapsed"` 滑上去toolbar收缩在顶部：
![](https://user-gold-cdn.xitu.io/2018/7/7/164731c1208b4bc8?w=522&h=236&f=jpeg&s=42140)   

修改属性改成这样的：
`app:layout_scrollFlags="scroll|enterAlways"` 滑上去toolbar滑出屏幕：  
![](https://user-gold-cdn.xitu.io/2018/7/7/164731f4bd74035b?w=544&h=262&f=jpeg&s=55379)   

#### collapsingToolbarLayout
`collapsingToolbarLayout`可以作为`AppBarLayout`的子view，可以控制包含在其中的控件在滚动时的响应事件，子view可以是个可折叠的Toolbar，`app:layout_collapseMode`设置折叠模式。  
3种折叠模式：  
`off`：默认属性，布局将正常显示，无折叠行为。   
`pin`：折叠后，此布局将固定在顶部。   
`parallax`：折叠时，此布局也会有视差折叠效果。   
当其子布局设置了`parallax`模式时，我们还可以通过`app:layout_collapseParallaxMultiplier`设置视差滚动因子，值为：0~1。  


接下来，我们就使用以上的属性来完成demo  
#### 实现原理
1、`coordinatorLayout`嵌套`appBarLayout`   

2、`appBarLayout`的子view`collapsingToolbarLayout`设置属性   
`app:layout_scrollFlags="scroll|exitUntilCollapsed|snap"`    
让头部随着内容下拉而展开，随着内容上拉而收缩。  

3、`collapsingToolbarLayout`的子布局有两种，展开时显示的布局和`Toolbar`，其中`Toolbar`又包含了两种布局，展开时的和收缩时的。  
展开时，(扫一扫、付钱)的布局：
```
<include
    layout="@layout/include_open"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="50dp"
    app:layout_collapseMode="parallax"
    app:layout_collapseParallaxMultiplier="0.7" />
```
layout_marginTop="50dp"预留出toolbar的高度，避免布局重叠。  

toolbar里的两种布局：
```
<android.support.v7.widget.Toolbar
android:layout_width="match_parent"
android:layout_height="50dp"
app:contentInsetLeft="0dp"
app:contentInsetStart="0dp"
app:layout_collapseMode="pin">

<include
    android:id="@+id/include_toolbar_open"
    layout="@layout/include_toolbar_open" />

<include
    android:id="@+id/include_toolbar_close"
    layout="@layout/include_toolbar_close" />

</android.support.v7.widget.Toolbar>
```
toolbar里的两个布局，可以通过监听AppBarLayout的移动控制显示和隐藏。  

4、最下面的布局可以是NestedScrollView,一定要在布局中设置以下属性，这里我直接用的demo中的布局：
```
app:layout_behavior="@string/appbar_scrolling_view_behavior"
```
5、滑动过程中，各控件的透明度会有渐变的效果，这里采用类似遮罩的效果，每个include布局里都有个遮罩的view，在滑动过程中监听`appBarLayout`的`addOnOffsetChangedListener`，通过计算滑动的距离，逐渐改变透明度。
```
@Override
public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
    //垂直方向偏移量
    int offset = Math.abs(verticalOffset);
    //最大偏移距离
    int scrollRange = appBarLayout.getTotalScrollRange();
    if (offset <= scrollRange / 2) {//当滑动没超过一半，展开状态下toolbar显示内容，根据收缩位置，改变透明值
        toolbarOpen.setVisibility(View.VISIBLE);
        toolbarClose.setVisibility(View.GONE);
        //根据偏移百分比 计算透明值
        float scale2 = (float) offset / (scrollRange / 2);
        int alpha2 = (int) (255 * scale2);
        bgToolbarOpen.setBackgroundColor(Color.argb(alpha2, 25, 131, 209));
    } else {//当滑动超过一半，收缩状态下toolbar显示内容，根据收缩位置，改变透明值
        toolbarClose.setVisibility(View.VISIBLE);
        toolbarOpen.setVisibility(View.GONE);
        float scale3 = (float) (scrollRange  - offset) / (scrollRange / 2);
        int alpha3 = (int) (255 * scale3);
        bgToolbarClose.setBackgroundColor(Color.argb(alpha3, 25, 131, 209));
    }
    //根据偏移百分比计算扫一扫布局的透明度值
    float scale = (float) offset / scrollRange;
    int alpha = (int) (255 * scale);
    bgContent.setBackgroundColor(Color.argb(alpha, 25, 131, 209));
}
```
详细代码见  
github地址：[https://github.com/taixiang/alihome](https://github.com/taixiang/alihome)  

欢迎关注我的博客：[https://blog.manjiexiang.cn/](https://blog.manjiexiang.cn/)  
更多精彩欢迎关注微信号：春风十里不如认识你  
![image.png](https://upload-images.jianshu.io/upload_images/7569533-cfeb1f55473a2143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

有个「佛系码农圈」，欢迎大家加入畅聊，开心就好！  
![](https://user-gold-cdn.xitu.io/2018/7/8/16478099583f8ec2?w=185&h=250&f=jpeg&s=35412)   
过期了，可加我微信 tx467220125 拉你入群。
