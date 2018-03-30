# Android 性能优化

[TOC]



### 1.绘制优化

#### 1. 绘制原理

​	渲染流程CPU->驱动缓存队列->GPU.

​	首先需要知道一个概念：FPS （Frames Per Sendos）。表示每秒能够传递的帧数。理想情况下60FPS感觉不到卡顿，意味着每次绘制应该在16ms内完成。

​	但是Android 系统很有可能无法在16ms内完成复杂的界面渲染操作。Android系统每隔16ms就会发出同步信号，出发对UI的渲染，如果每次渲染成功，那么画面就会很流畅，如果没有渲染完成，就有可能完成卡顿现象。

​	为了解决卡顿现象，Android层面或者说各个端都采用的渲染机制

 -  垂直同步： 

    只有CPU把缓存区数据填充满了，才会通知GPU去取数据，效率很低；

    也有可能GPU还没绘制完，CPU已经把缓存区数据填满，通知GPU进行新的数据渲染，这个时候就会产生换面撕裂现象

 -  双缓存：采用两个缓存来存储cpu处理好的数据,保证每次渲染GPU渲染的数据都是一份完整的数据，不会出现数据没有处理完就被显示器拿去绘制，导致出现显示不完整，出现闪烁感，视觉上看上去像是慢慢绘制出来。

 -  三缓存：同时开启了垂直同步和双缓存之后 可能会出现同步时数据仍然没有准备好，浪费了一个同步时间，这是可以采取增加三缓存，加快数据准备效率。

#### 2.卡顿检测工具

#####2.1 Android 手机自带Profile GPU Rendering 分析渲染过程中各个步骤消耗时间。

![](http://img.blog.csdn.net/20170611210804816?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcDEwNjc4Njg2MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#####2.2  DDMS Systraces （Capture Ssystem wide trace using Androidsystrace）

我们可以看到此界面的Frame的圆圈会显示三种颜色，绿，黄，红。一般红色就是有问题的，要优化的地方。

![](http://img.blog.csdn.net/20161210120509775?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGZyZWVtYW4yMDA4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



#####2.3 DDMS中的TraceView  分析每一个方法的执行时间（Method Profiling)

![数据展示](http://img.blog.csdn.net/20161231162545995?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQyMjQ0MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



**startMethodTracing  stopMethodTracing**



#### 3.布局优化

##### 3.1 布局过于复杂

​	Android 每一个页面的测量，显示和绘制都是通过递归来完成，遍历的时间与树的高度h相关，复杂度为O(h);如果层级过深，每增加一层就会增加更多的页面显示时间。因此应该尽量避免页面的层级。

 -  **页面层级检测工具 Hierarchy Viewer**

     注：1.只能连接开发版手机或者模拟器   

     ​	 2.Debug模式下无法使用

    ![img](http://upload-images.jianshu.io/upload_images/1897639-61839f18387848d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    ​	![img](http://img.blog.csdn.net/20161023104353419)



- 解决布局过于复杂方案：

  - 合理使用RelativeLayout和LinearLayout

  - 合理使用Merge  

     Android 加载Merge标签时，直接其内部子元素添加到merge标签的Parent中，这样就可以减少一层层级（比如自定义的标题栏）

##### 3.2 提高布局加载速度 

 - 减少界面复杂度（同上）
 - 使用ViewStub，延迟加载

##### 3.3 过度绘制 

​	过度绘制是指 屏幕上的某个像素点在同一帧的时间内被绘制了多次。在多层次重叠的UI结构中，如果不可兼得UI也在做绘制操作，就会导致统一像素点过度绘制，浪费CPU和GPU资源，导致性能消耗。

​	Android提供了测量Overdraw的选项，在开发者选项－调试**GPU过度绘制（Show GPU Overdraw）**，打开选项就可以看到当前页面Overdraw的状态，就可以观察屏幕的绘制状态。

![img](https://upload-images.jianshu.io/upload_images/1187237-9d24d83236255c0a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/382)

 -  合理使用LinearLayout和RelativeLayout

 -  去掉其他不必要的背景

    如果我们给一个布局设置了背景，但是其内部一个控件的宽度和高度是match_parent,那么控件的背景色必然会覆盖布局的背景色。这个时候就可以考虑分别给控件设置背景色，减少过度绘制。

#### 4.合理刷新机制

- 控制刷新频率

  如下载进度条刻度是100，没有必要变化1%就刷新一次，会增加UI刷新负担。

- 避免后台线程刷新影响

  如Listview滚动时，可以暂时停止listView上的UI操作，待滑动完毕再进行刷新操作

- 减少刷新区域

  在自定义View中，如果仅仅时某一小部分布局或者控件需要刷新，则没有必要刷新全局

  ```java
  局部更新方法：
  invalidate(Rect dirty);
  invalidate(int left.int top,int right,int bottom)
  ```





### 2.内存优化

####1.内存管理机制

​	参见Java内存管理机制

   **优化内存的意义	：**

 - 系统在内存回收时会对系统性能造成什么影响？
 - 明明有一定的剩余内存，为什么会发生OOM？
 - 内存占用高对应用存活有什么影响？



- GC在工作时，所有的其他工作线程都会被暂停，包括负责绘制的UI线程，并且在不同区域的内存释放速度存在差异，但是不管在哪个区域，都是要等到这次GC执行完成才会继续原来的线程。

  如果短时间内触发了大量的GC，那么就会影响到应用的渲染工作。在一个16ms时间内，如果发生GC，那么必然会压缩渲染的时间，进而有可能导致渲染无法完成，造成卡顿，影响应用流畅性。

- 如果内存在某一个时间段内达到了内存空间的阀值或者频繁地出现毛刺，在这个峰值时，正好需要申请一块较大的内存（如加载图片），但是堆内存空间不足，这个时候就会发生OOM。因此OOM的主要原因就是申请的内存仍旧无法满足程序此次需要的内存。

- 如果后台应用占用了较高的内存，导致系统内存紧张，这时候系统会很大概率杀死应用。

  ​

####2.内存分析工具

##### 2.1 memory monitor

![img](https://upload-images.jianshu.io/upload_images/1836169-265f44d2132adce7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

#####2.2 heapviewer

​	可以大致的分析各种数据类型所占用的内存

![这里写图片描述](http://img.blog.csdn.net/20150924221517594)



| free                           | 空闲的对象               |
| ------------------------------ | ------------------- |
| data object                    | 数据对象,类类型对象，最主要的观察对象 |
| class object                   | 类类型的引用对象            |
| 1-byte array(byte[],boolean[]) | 一个字节的数组对象           |
| 2-byte array(short[],char[])   | 两个字节的数组对象           |
| 4-byte array(long[],double[])  | 4个字节的数组对象           |
| non-Java object                | 非Java对象             |



##### 2.3 .Allocation tracker

![这里写图片描述](http://img.blog.csdn.net/20150926132245307)

![这里写图片描述](http://img.blog.csdn.net/20150926170512491)



#### 3.优化类型

##### 3.1 内存泄漏

 **定义：**

![img](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3398482026,1341694023&fm=27&gp=0.jpg)

原因：

- 资源对象未及时关闭
- 注册对象未及时注销
- 静态变量大量持有数据对象
- ....

##### 3.2 内存溢出

​	一般内存溢出都是由于内存泄漏导致的内存居高不下，无法申请到足够的内存，因此应该重点管理好内存使用，减少内存泄漏

##### 3.3 内存监控

LeackCanary 

（感兴趣可以看源码...）

##### 3.4 图片内存优化

图片占用内存 = 图片长度 * 图片宽度 * 单位像素占用字节数

- 设置位图格式

  - RGB888(int)：R、G、B分量各占8位
  - RGB565(short)：R、G、B分量分别占5、6、5位
  - RGB555(short)：RGB分量都用5位表示（剩下的1位不用）
  - ARGB8888(int)：A、R、G、B分量各占8位
  - ARGB4444(short)：A、R、G、B分量各占4位

- inSampleSize 

  默认为1，图片大小 = 原图片大小 / inSampleSize的平方

  如16*16 像素的图片，设置inSampleSize为2，那么在水平和竖直方向上会每隔2个像素点取一个像素点作展示，这样最终下来图片只有 8 * 8像素。

- inScaled, inDensity, inTargetDensity 调整图片大小

  inScaled 为true才可进行下面调整

  options.inDensity表示的是bitmap所使用的像素密度。

  options.inTargetDensity表示的是目标Bitmap即将被画到屏幕上的像素密度（每英寸有多少个像素）

- 具图加载

  BitmapRegionDecoder 分块显示（参考新浪微博大图浏览）

- 三级缓存

### 3.稳定性优化

#### 3.1 代码质量检测

- CI静态检测

- Lint代码检查

- ```
  abortOnError false
  ```

#### 3.2崩溃检测

##### 3.2.1 Crash检测

​	一般是因为代码上的空指针等问题导致

 -  Java 崩溃监测方案

    通过注册自定义的UncaughtExceptionHandler 来捕获异常

- Native 崩溃统计

  Linux 在发生崩溃时会发出Crash信号，可以通过在JNI层注册信号，这样在发生崩溃是就可以获取到崩溃信号，同时查找dump文件进行崩溃分析处理

  一般我们采用开源的linux捕获工具来捕获异常

  - coffeecatch  代码量较少，腾讯bugly在使用
  - Google breakpad  跨平台兼容各大平台，需要做相应裁剪，输入法崩溃SDK在使用 

##### 3.2.2  ANR检测

#####    - Activity的无响应时长是5秒.

- Service        20S
- BroadCastReceiver   10S

检测方案：通过观察者模式检测**data/anr/traces.txt**,当发生ANR时，该文件会被写入相关信息，可以捕捉到ANR信息。

##### 3.2.3  卡顿检测

```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        msg.target.dispatchMessage(msg);

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
```

​	可以通过自定义LogPrinter 检测UI执行时间，并根据时间做相应卡顿等级划分

​	PS：其他方案待补充。

### 方式4： 利用Choreographer.FrameCallback监控卡顿  [kɒriˈɒgrəfi]

doFrame 回调方法

//待完善

#### 3.3启动优化

- 非必要操作延迟执行
- 开屏动画处理加载逻辑

可以追溯到startActivity  ，说到底是一个Activity的加载问题

1.页面优化

2.耗时的后台操作延迟启动，懒加载，引擎等



####3.4耗电优化   

三大模块 ： 显示，网络 ，cpu   ->方案

#### 3.5 后台进程优先级

- 注册静态广播

- 前台页面

  通知栏/悬浮窗

- 前台服务

- 关联系统账号系统

#### 3.6 安装包优化

- 资源压缩

- 冗余代码

- 代码/资源混淆

  ​













