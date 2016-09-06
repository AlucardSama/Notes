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