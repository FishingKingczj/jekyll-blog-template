---
title: '腾讯 安卓开发实习 提前批一面'
layout: post
tags:
    - interview 
---
对比上午的直推面试要轻松一些，就是聊一个小时的话有点累，并且问题多且杂，回答的时候觉得自己基本都答上来了，但是在回顾到时候发现有很多回答得不正确，不全面，大多是自己有一定的印象但是并不是非常明晰的，最蛋疼的是面试官不会告诉你你错了而是让你自己讲下去...  
下面把这次面试的问题和知识点归纳一下。

<!--more-->

**目录**

* TOC
{:toc}

## 问题详解
### Bitcoin
这个比特币是我写在经验里的，但是事实上讲的并不是很好，主要有两个点：  
1. 比特币Net模块的机制细节：  
比特币的Net机制也就是P2P点对点传输的机制，主要流程为，当一个新的网络节点启动后，为了能够参与协同运作，它必须至少发现一个其他网络中的节点并与之建立连接。  
一个新启动的节点可以利用种子节点或节点引荐发现的新的节点，当节点与对等节点建立好连接后，首先要做的就是握手，节点向可见节点发送消息开始并握手  
完成握手协议后，新节点将会发送自己IP地址的addr消息给对等端peer，对等端peer收到以后又向与它连接的相邻节点发送addr消息，这样新节点的ip地址就会在P2P网络中广播出去。此外新节点还可以发送getaddr消息，要求对等端把自己知道的节点的IP地址发送过来。通过这种方式，新节点可以找到需要连接的对等节点。
2. 比特币的基本原理：  
主要有三部分：非对称加密技术、点对点传输技术、哈希现金算法机制  
非对称加密技术对应到比特币场景中就是比特币的地址和私钥  
点对点传输技术对应到比特币网络中就是利用点对点的技术，实现真正的去中心化
哈希现金算法机制被改造成了比特币的发行机制，用户贡献算力进行哈希运算，作为回报比特币网络就会将比特币赠予首个挖出区块的矿工，这就是比特币挖矿的原理

### Android 启动模式
这部分回答的时候顺口答了，之后才发现其实回答错了，但是也不知道面试官有没有听出来，挺丢人了...  
- standrad：标准模式，无论如何都在栈上加上启动的Activity  
- singleTop：栈顶复用，当栈顶和启动的Activity相同时不会启动新的Activity  
- SingleTask：栈内复用，当启动栈内存在的Activity时会将其之前的所有Activity*出栈*并使此Activity处于栈顶  
- singleInstance：单例模式，会在启动时新开一个栈加入Activity

### Java ArrayList和Vector的线程安全
ArrayList是线程不安全的，Vector才是线程安全的，Java希望Vector是线程安全的，而希望arraylist是高效的
Vector源码里对add操作加了同步锁，所以不会造成线程安全问题，但是Vector已经被弃用了，需要线程安全的List时可以使用  

```java
List<String> list1 = Collections.synchronizedList(new ArrayList<String>());
```

*另外：*  
- `Vector`, `HashTable`, `Properties`线程安全  
- `ArrayList`, `LinkedList`, `HashSet`, `TreeSet`, `HashMap`,`TreeMap`等都是线程不安全的  

### Java finalize()
`finalize()`是Object的protected方法，子类可以覆盖该方法以实现资源清理工作，GC在回收对象之前调用该方法。  
`finalize()`与C++中的析构函数不是对应的。C++中的析构函数调用的时机是确定的（对象离开作用域或delete掉），但Java中的finalize的调用具有不确定性  
不建议用finalize方法完成“非内存资源”的清理工作，但建议用于：
- 清理本地对象(通过JNI创建的对象)
- 作为确保某些非内存资源(如Socket、文件等)释放的一个补充：在finalize方法中显式调用其他资源释放方法

### Android 如何实现一个View/View的绘制
Android实现自定义控件有三种方式：
1. 继承现有控件，对其控件的功能进行拓展
2. 将现有控件进行组合，实现功能更加强大控件
3. 重写View实现全新的控件  

主要是方法3，继承View或ViewGroup并重写绘制方法实现的自定义控件：
1. 在`onMeasure()`方法中，测量自定义控件的大小，使自定义控件能够自适应布局各种各样的需求
2. 在`onDraw()`方法中，利用Canvas与Paint来绘制要显示的内容
3. 在`onLayout()`方法中来确定控件显示位置。
4. 在`onTouchEvent()`方法处理控件的触摸事件。

下面是例子  
**构造函数**  

```java
/**
  * 在new的时候调用
  */
public MyView(Context context) {
    this(context, null);
}

/**
  * 在布局中使用
  */
public MyView(Context context, @Nullable AttributeSet attrs) {
    this(context, attrs, 0);
}

/**
  * 布局layout中调用,但是会有style
  */
public MyView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
}
```

**`onMeasure()`(非必须)**  

自定义view的测量方法，返回View的实际大小  
```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    //布局的宽高都由这个方法指定
    //指定控件的宽高,需要测量
    //获取宽高的模式
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
    /**
      * MeasureSpec.AT_MOST : 在布局中指定 wrap_content
      * MeasureSpec.EXACTLY : 在布局中指定确切的值 100dp   match_parent fill_parent
      * MeasureSpec.UNSPECIFIED : 尽可能的大,很少能用到，ListView, ScrollView 在测量子布局的时候会用UNSPECIFIED
      */
    //获取宽高的大小
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);
    // ... 进行操作重新计算width和height
    setMeasuredDimension(newwidth, newheight);
}
```

**`onDraw()`使用canvas/paint等工具绘制View**

```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // 使用canvas/paint等工具绘制View
}
```

**`onLayout()`(ViewGroup)**

```java
// layout的作用是ViewGroup用来确认子元素的位置
public void layout(int l, int t, int r, int b) {
  // 初始化mLeft mRight mBottom mTop四个值，这一步完成后，view在父容器中的位置就确定下来了
   boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
  // 接着会调用onLayout方法，在onLayout中去循环遍历子元素，确定子元素的位置
  onLayout(changed, l, t, r, b);
}

/**
当ViewGroup的位置被确定后，它在onLyaout中会遍历所有的子元素并调用其layout方法，在layout方法中又会调用onLayout方法并调用每一个子view的layout()方法，把之前在onMeasure()里保存下来的它们的位置作为参数传进去，然它们进行自我布局
*/
/**
  * 实现一个子View上下居中，分隔20dp排布的ViewGroup
  * @param changed 当前View的大小和位置改变了 
  * @param l,t,r,b 
  * 放置父控件的矩形可用空间（除去margin和padding的空间）
  * 的左上角的left、top以及右下角right、bottom值
  */
@Override  
protected void onLayout(boolean changed, int l, int t, int r, int b) {
  int padding = 20;
  // 保存到父View左侧的距离
  int mLeft = padding;
  
  // 循环所有子View
  for (int i = 0; i < getChildCount(); i++) {  
    View child = getChildAt(i);  
    // 取出当前子View长宽  
    int childViewHeight = childView.getMeasuredHeight();
    int childViewWidth = childView.getMeasuredWidth();
    // 让子View在竖直方向上显示在父View中间位置
    int mTop = (t + b - childViewHeight) / 2;
    // 调用layout给每一个子View设定位置l,t,r,b
    childView.layout(mLeft, mTop, mLeft + childViewWidth, mTop
        + childViewHeight);
    // 改变下一个子View到父View左侧的距离
    mLeft += childViewWidth;
  }  
}  
```

getLeft(), getTop(), getRight(), getBottom  
getWidth() = getRight() - getLeft()  
getHeight() = getBottom() - getTop()  
*`getWidth/Height()`和`getMeasuredWidth/Height()`的区别：*   
`getWidth()`, `getLeft()`等这些函数是View相对于其父View的位置。而`getMeasuredWidth()`, `getMeasuredHeight()`是测量后该View的实际值  
实际上在当屏幕可以包裹内容的时候，他们的值是相等的，只有当View超出屏幕后，才能看出他们的区别：`getMeasuredHeight()`是实际View的大小，与屏幕无关，而`getHeight`的大小此时则是屏幕的大小。当超出屏幕后，`getMeasuredHeight()`等于`getHeight()`加上屏幕之外没有显示的大小

**`OnTouchEvent()`触摸事件**

```java
/**
  * 处理用户交互
  */
@Override
public boolean onTouchEvent(MotionEvent event) {
  switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN:
      // ... 
      break;
    case MotionEvent.ACTION_MOVE:
      // ... 
      break;
    case MotionEvent.ACTION_UP:
      // ... 
      break;
    default:
      break;
  }
  return super.onTouchEvent(event);
}
```

**自定义属性**

```xml
<resources>
  <!--name 自定义View的名字 TextView-->
  <declare-styleable name="TextView">
    <!-- name 属性名称
      format 格式: string 文字 color 颜色 dimension 宽高,字体大小
      integer 数字 reference 资源(drawable)-->
    <attr name="text" format="string" />
    <attr name="textColor" format="color" />
    <attr name="textSize" format="dimension" />
    <attr name="maxLength" format="integer" />
    <attr name="background" format="reference|color" />
    <!-- 枚举 -->
    <attr name="inputType">
      <enum name="number" value="1" />
      <enum name="text" value="2" />
      <enum name="password" value="3" />
    </attr>
  </declare-styleable>
</resources>
```

### Android 开发生态/第三方库
- Retrofit：Retrofit 是目前 Android 最流行的 Http Client 库之一
- Picasso：Picasso 是一款图片缓存库。
- EventBus：EventBus 是 Android 事件管理总线, 使用它可以替代 BroadCast, BroadCastReceiver, Handler 在 Activity, Fragment, Service, 线程之间传递消息, 大大简化了事件传递逻辑。
- ZXing：ZXing 提供了多个平台的二维码/条形码扫描解决方案。
- LeakCanary：LeakCanary 是一款检测内存泄露工具, 帮助你在开发阶段方便的检测出内存泄露的问题, 使用起来非常简单方便。
- ButterKnife：ButterKnife 是 View 注入框架, 使用它为了简写很多 findViewById 代码, 同时还支持 View 的一些事件处理函数。
- MPAndroidChart：MPAndroidChart 是一款强大的 Android 图表库, 支持各种各样图表显示。
- UltimateRecyclerView：UltimateRecyclerView 包括了下拉刷新, 加载更多, 多种动画, 空数据提示, 拖动排序, 视差处理, 工具栏渐变, 滑动删除, 自定义floating button, 多种刷新效果, scrollbar, sticky header, 多 layout 支持等等元素。
- Logger：Logger 是一个简单, 漂亮, 强大 Android 打印日志库。
- Acra：Acra 是一个能够让 Android 应用自动将崩溃报告以谷歌文档电子表的形式进行发送的库, 旨在当应用发生崩溃或出现错误行为时, 开发者可以获取到相关数据。

### Android 跨平台框架Flutter
Flutter是Google的移动UI框架，基于Dart语言。  
Flutter组件采用现代响应式框架构建，这是从React中获得的灵感，中心思想是用组件(widget)构建你的UI。组件描述了在给定其当前配置和状态时他们显示的样子。当组件状态改变，组件会重构它的描述(description)，Flutter会对比之前的描述，以确定底层渲染树从当前状态转换到下一个状态所需要的最小更改。  
Flutter与RN的区别之处在于Flutter 中，UI组件和渲染器已经从平台中集成到用户的应用程序中。没有系统UI组件可以操作，所以RN虚拟控件树的地方现在是真实的控件树。Flutter渲染UI控件树并将其绘制到平台画布上。  
Flutter与其它大多数框架不同之处在于，Flutter 既不使用 WebView，也不使用操作系统的原生控件。相反，Flutter使用自己的高性能渲染引擎来绘制 widget。Flutter使用 C、C ++、Dart 和 Skia（2D渲染引擎）构建。

### C++ 模板  
[C++ 模板](http://www.runoob.com/w3cnote/c-templates-detail.html)

- 函数模板

```c++
template <class T> void swap(T& a, T& b)
{
  // do sth
}
swap(2, 3);
```

- 类模板

```c++
template <class T> class A
{
  public: 
    T a;
    T b;
    T hy(T c, T &d);
};

A<int> m;
```

- 模板的形参
  - 非类型模板形参
  - 类型模板形参
  - 模板模板形参
- 模板实例化
  - 显示实例化
  - 隐式实例化
  - 具体化
- 实参推演

### C++ 内存（堆栈/野指针/内存管理）
- 内存分配方式
  - 从静态存储区域分配。内存在程序编译的时候就已经分配好，这块内存在程序的整个运行期间都存在。例如全局变量，static 变量。
  - 在栈上创建。在执行函数时，函数内局部变量的存储单元都可以在栈上创建，函数执行结束时这些存储单元自动被释放。栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
  - 从堆上分配。程序在运行的时候用 malloc 或 new 申请任意多少的内存，程序员自己负责在何时用 free 或 delete 释放内存。  
  动态内存的生存期由程序员决定，使用非常灵活，但如果在堆上分配了空间，就有责任回收它，否则运行的程序会出现内存泄漏，频繁地分配和释放不同大小的堆空间将会产生堆内碎块。
- 内存泄漏  
  申请的堆内存使用完毕后没有释放掉。程序运行时间越长，占用内存越多，最终用尽全部内存，程序崩溃。由程序申请的一块内存，且没有任何一个指针指向它，这块内存就泄漏了。
- 野指针  
  野指针不是NULL指针，是指向垃圾内存的指针。if语句对野指针的判断是无法预测的。野指针的成因主要有两种：
  - 指针变量没有被初始化。任何指针变量刚被创建时不会自动成为NULL指针，它的缺省值是随机的。所以，指针变量在创建的同时应当被初始化设置为NULL，或指向合法的内存。
  - 指针被free或者delete之后，没有置为NULL。
- 悬垂指针  
  多个指向同一块内存的指针，对其中一个指针执行了free或者delete并置为NULL了，但其他指向该块内存的指针并没有被置为NULL。  
  这类情况可以使用指针计数的方式（智能指针）进行管理。
- 内存管理
  - 检查内存分配是否成功
  - 分配之后初始化再使用
  - new-delete和malloc-free且使用次数必须相同
  - free或delete后，指针即时设为NULL
- 资源管理
  - RAII  
  本质是用对象代表资源，把管理资源的任务转化为管理对象的任务，将资源的获取和释放与对象的构造和析构对应起来，从而确保在对象的生存期内资源始终有效，对象销毁时资源一定会被释放。
  - 智能指针

### CS 进程与静态类变量
父进程通过fork来复制出一个子进程的副本，根据原理，子进程拥有父进程的一份完整数据拷贝。  
同时由于fork时按复制数据太耗时，子进程在刚被fork出来的时候，读取的其实是父进程中的内存数据，这时候静态变量是共享的  
但是， 当父子进程中的一方对内存进行写入操作时，就会触发写时复制机制，这个变量将在子进程中产生一个新的空间来存放，于是变量不再共享。  
所以，父进程和子进程中的变量是不可以被共享的，在程序员和进程的角度来看，每个进程的内存空间都是属于它自己的。

### Net TCP流量控制
[TCP流量控制与拥塞控制](http://blog.chinaunix.net/uid-26275986-id-4109679.html)

- **原因**  
如果发送者发送数据过快，接收者来不及接收，那么就会有分组丢失。为了避免分组丢失，控制发送者的发送速度，使得接收者来得及接收，这就是流量控制。流量控制根本目的是防止分组丢失，它是构成TCP可靠性的一方面。  
- **实现**  
由滑动窗口协议（连续ARQ协议）实现。滑动窗口协议既保证了分组无差错、有序接收，也实现了流量控制。主要的方式就是接收方返回的重复确认（ACK）中会包含自己的接收窗口的大小，并且利用大小来控制发送方的数据发送。  
- **特殊情况死锁**  
当发送者收到了一个*窗口为0*的应答，发送者便停止发送，等待接收者的下一个应答。但是如果这个*窗口不为0的应答*在传输过程丢失，发送者一直等待下去，而接收者以为发送者已经收到该应答，等待接收新数据，这样双方就相互等待，从而产生死锁。  
为了避免流量控制引发的死锁，TCP使用了持续计时器。每当发送者收到一个零窗口的应答后就启动该计时器。时间一到便主动发送报文询问接收者的窗口大小。若接收者仍然返回零窗口，则重置该计时器继续等待；若窗口不为0，则表示应答报文丢失了，此时重置发送窗口后开始发送，这样就避免了死锁的产生。

### Net TCP拥塞控制
拥塞控制是TCP避免网络拥塞的算法，是互联网上主要的一个拥塞控制措施。它使用一套基于线增积减模式的多样化网络拥塞控制方法来控制拥塞  
TCP会为每条连接维护一个“拥塞窗口”来限制可能在端对端间传输的未确认分组总数量。TCP在一个连接初始化或超时后使用一种“慢启动”机制来增加拥塞窗口的大小。它的起始值一般为最大分段大小（Maximum segment size，MSS）的两倍，虽然名为“慢启动”，初始值也相当低，但其增长极快：当每个分段得到确认时，拥塞窗口会增加一个MSS，使得在每次往返时间（round-trip time，RTT）内拥塞窗口能高效地双倍增长。  
当拥塞窗口超过慢启动阈值（ssthresh）时，算法就会进入一个名为“拥塞避免”的阶段。在拥塞避免阶段，只要未收到重复确认，拥塞窗口则在每次往返时间内线性增加一个MSS大小
- **拥塞窗口**  
拥塞窗口（congestion window）是任何时刻内确定能被发送出去的字节数的控制因素之一，是阻止发送方至接收方之间的链路变得拥塞的手段。他是由发送方维护，通过估计链路的拥塞程度计算出来的，与由接收方维护的接收窗口大小并不冲突。  
当一条连接创建后，每个主机独立维护一个拥塞窗口并设置值为连接所能承受的MSS的最小倍数，之后的变化依靠线增积减机制来控制，这意味如果所有分段到达接收方和确认包准时地回到发送方，拥塞窗口会指数增大，直到发生超时或者超过一个称为“慢启动阈值（ssthresh）”的限值。如果发送方到达这个阈值时，每收到一个新确认包，拥塞窗口只按照线性速度增加自身值的倒数。 
当发生超时的时候，慢启动阈值降为超时前拥塞窗口的一半大小、拥塞窗口会降为1个MSS，并且重新回到慢启动阶段。  
系统管理员可以设置窗口最大限值，或者调整拥塞窗口的增加量，来对TCP调优。
![拥塞控制](http://t1.aixinxi.net/o_1d5jukliagm23uhcb7oh76qa.jpg-w.jpg)  
- 快重传+快恢复  
快重传：上面的重传机制都是等到超时还未收到接收方的回复，才开始进行重传。而快重传的设计思路是：如果发送方收到 3 个重复的接收方的重复确认，就可以判断有报文段丢失，此时就可以立即重传丢失的报文段，而不用等到设置的超时时间到了才开始重传，提高了重传的效率。  
快恢复：上面的拥塞控制会在网络拥塞时将拥塞窗口降为 1，重新慢开始，这样存在的一个问题就是网络无法很快恢复到正常状态。快恢复中出现拥塞时，拥塞窗口只会降低到新的慢开始门阀值（12），而不会降为 1，然后直接开始进入拥塞避免加法增长。  
![快恢复](http://t2.dt8.co/o_1d5jugvsrjom1j56puc1uq6a6fa.jpg-w.jpg)  

### 其他问题
- 介绍自己的项目
- C++ 多态
- Java Object方法  
  `HashCode();wait();notify();equals();getClass();toString();clone();finalize();`
- Java Synchronize锁
- CS 死锁的原理
- CS 进程
- Android 四大组件
- Android Handler
- RN 原理
- AL 冒泡排序
- AL 链表翻转

## 总结
- 千万不要觉得自己已经会了，很多知识真的只是你“觉得”你会了而已。
