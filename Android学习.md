## 2016/8/12 15:33:42  自定义View
### 自定义view基础（一）
摘录自[博客地址](http://blog.csdn.net/lfdfhl/article/details/51324275)

* **Configuration**
* **ViewConfiguration**
* **GestureDetector**
* **VelocityTracker**
* **Scroller**
* **ViewDragHelper**


&nbsp;&nbsp;&nbsp;&nbsp;以上的工具封装了我们经常会用到的方法，比如拖拽View，计算滑动速度，View的滚动，手势处理等等。如果我们自己去实现这些方法会比较繁琐，而且容易出一些bug。所以，作为自定义View系列教程的开端，先介绍一下这些常用的工具，以便在后续的学习和工作中使用。

> Configuration  
 
	Configuration用来描述设备的配置信息。 
	比如用户的配置信息：locale和scaling等等 
	比如设备的相关信息：输入模式，屏幕大小， 屏幕方向等等 

> ViewConfiguration  

	ViewConfiguration提供了一些自定义控件用到的标准常量，比如尺寸大小，滑动距离，敏感度等等。 
> GestureDetector 

	手势监听

> VelocityTracker  

	VelocityTracker用于跟踪触摸屏事件（比如，Flinging及其他Gestures手势事件等）的速率。

> Scroller  

	scrollTo()和scrollBy()

> ViewDragHelper

	在项目中很多场景需要用户手指拖动其内部的某个View，此时就需要在onInterceptTouchEvent()
    和onTouchEvent()这两个方法中写不少逻辑了，比如处理：拖拽移动，越界，多手指的按下，加速
	度检测等等。 



## 2016/8/12 16:20:12 自定义View
### 自定义view基础（二）
摘录自[博客地址](http://blog.csdn.net/lfdfhl/article/details/51347818)

#### MeasureSpec基础知识

1. MeasureSpec封装了父布局传递给子View的布局要求。 
2. MeasureSpec可以表示宽和高 
3. MeasureSpec由size和mode组成

> MeasureSpec高2位代表SpecMode即测量模式，低30位为SpecSize规格大小

```java
//获取SpecSize  
int specSize = MeasureSpec.getSize(measureSpec)	  
//获取specMode  
int specMode = MeasureSpec.getMode(measureSpec)
//当然，也可以通过这两个值生成新的MeasureSpec
int measureSpec=MeasureSpec.makeMeasureSpec(size, mode);
```

>> SpecMode一共有三种  


* MeasureSpec.EXACTLY模式表示：父容器已经检测出子View所需要的精确大小；
* MeasureSpec.AT_MOST模式表示：父容器未能检测出子View所需要的精确大小，但是指定了一个可用大小即specSize,在该模式下，View的测量大小不能超过SpecSize；
* MeasureSpec.UNSPECIFIED这种模式一般用作Android系统内部，或者ListView和ScrollView等滑动控件，父容器不对子View的大小做限制。

## 2016/8/22 14:01:26 自定义View
自定义View的流程 measure->layout->draw
**measure:** 测量view的宽和高；
**layout:** 确定View在父容器中的位置；
**draw:** 将View绘制在屏幕上； 
performTraversals会依次调用performMeasure、performLayout和performDraw三个方法；
具体的：
performMeasure->measure->onMeasure


## 2016/8/19 11:19:10 事件分发机制
#### android事件分发（一） 
http://blog.csdn.net/guolin_blog/article/details/9097463

##### 1.onTouch和onTouchEvent有什么区别，又该如何使用？

答：从源码中可以看出，这两个方法都是在View的dispatchTouchEvent中调用的，onTouch优先于onTouchEvent执行。如果在onTouch方法中通过返回true将事件消费掉，onTouchEvent将不会再执行。
另外需要注意的是，onTouch能够得到执行需要两个前提条件，第一mOnTouchListener的值不能为空，第二当前点击的控件必须是enable的。因此如果你有一个控件是非enable的，那么给它注册onTouch事件将永远得不到执行。对于这一类控件，如果我们想要监听它的touch事件，就必须通过在该控件中重写onTouchEvent方法来实现。

##### 2.为什么给ListView引入了一个滑动菜单的功能，ListView就不能滚动了？

如果你阅读了Android滑动框架完全解析，教你如何一分钟实现滑动菜单特效 这篇文章，你应该会知道滑动菜单的功能是通过给ListView注册了一个touch事件来实现的。如果你在onTouch方法里处理完了滑动逻辑后返回true，那么ListView本身的滚动事件就被屏蔽了，自然也就无法滑动(原理同前面例子中按钮不能点击)，因此解决办法就是在onTouch方法里返回false。

##### 3.为什么图片轮播器里的图片使用Button而不用ImageView？

提这个问题的朋友是看过了Android实现图片滚动控件，含页签功能，让你的应用像淘宝一样炫起来 这篇文章。当时我在图片轮播器里使用Button，主要就是因为Button是可点击的，而ImageView是不可点击的。如果想要使用ImageView，可以有两种改法。第一，在ImageView的onTouch方法里返回true，这样可以保证ACTION_DOWN之后的其它action都能得到执行，才能实现图片滚动的效果。第二，在布局文件里面给ImageView增加一个android:clickable="true"的属性，这样ImageView变成可点击的之后，即使在onTouch里返回了false，ACTION_DOWN之后的其它action也是可以得到执行的。

#### android事件分发（二） 
http://blog.csdn.net/guolin_blog/article/details/9153747 

1.Android事件分发是先传递到ViewGroup，再由ViewGroup传递到View的。

2.在ViewGroup中可以通过onInterceptTouchEvent方法对事件传递进行拦截，onInterceptTouchEvent方法返回true代表不允许事件继续向子View传递，返回false代表不对事件进行拦截，默认返回false。

3.子View中如果将传递的事件消费掉，ViewGroup中将无法接收到任何事件。



## 2016/8/22 9:16:52 IPC
#### IPC（Inter-Process Communication）进程间通信
* Bundle
* 文件共享
* AIDL
* Messenger
* ContentProvider
* Socket

##### Android中的多进程模式

正常情况下，在Android中的多进程指的是一个应用中存在多个进程的情况。在Android中只有一种方法，就是给四大组件（Activity、Service、Receiver、ContentProvider）在AndroidMenifest中指定android:process属性。
```xml
android:process=":remote"
```
查看进程信息：
```cmd
adb shell ps 或者是adb shell ps | grep packagename
```
 
**使用多线程会造成以下几方面的问题：**
* 静态成员和单例模式完全失效；
* 线程同步机制完全失效；
* SharePreferences的可靠性下降；
* Application会多次创建。

##### IPC基础知识

1. Serializable接口，implements Serializable serialVersionUID，自定生成hash值或者指定1L，很大程度上避免反序列化失败；  
2. Parcelable接口；
3. **Binder**，实现了IBinder接口

##### Android中的IPC方式
1. Bundle，传输的数据必须能够序列化，比如基本类型、实现了Serializable接口、Parcellable接口的对象以及一些Android支持的特殊对象。
2. 使用文件共享，可以使用文本文件、XML等读/写双方约定的数据格式，存在并发读/写的问题。系统对sharepreferences文件的读/写有一定的缓存策略，即在内存中会有一份sp文件的缓存，因此在多进程的模式下，系统对它的读/写就变得不可靠，当面对高并发的读/写访问，sp有很大几率会丢失数据，因此，**不建议在进程间通信使用SharePreferences**。
3. 使用Messenger，轻量级的IPC方案，底层实现是AIDL。
4. 书签《Android开发艺术探索》P80















































  


