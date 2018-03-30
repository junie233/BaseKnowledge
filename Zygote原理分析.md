# Andorid Zygote 分析

[TOC]



### 1.概览

Android 系统中存在着两大世界：

- Java世界 Google SDK均是针对该世界，这个世界中所运行的程序都是基于虚拟机的java程序
- Native 世界 ：用Nactive语言如c,c++ 开发

疑问点：

- Android 是基于Linux内核构建，那么Java世界是什么时候构造出来的？
- 每个程序都需要有一个进程，但是我们开发工程中很少涉及到进程，那么应用程序的进程是在什么时候创建的呢？
- 我们经常使用到的系统Service，是在什么时候初始化好的

### 2.Android 启动流程

![](http://img.blog.csdn.net/20151111170920935?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



### 3.Android Zygote 分析

​	首先明确下Zygote的含义，Zygote中文为“受精卵”，顾名思义是整个Android 系统的起点，也可以认为它是“女娲”，塑造了整个Android 系统。

​	Zygote名字的定义来源。Zygote的起始名字叫“app_process”，这个名字是在`Android.mk`文件中定义的。但是在运行过程中，app_process通过Linux系统的pctrl系统调用，将自己的名字修改为“zygote”。因此通过ps命令在运行时看到的其实是“zygote”。

​	Zygote是在Linux的init进程启动后根据init.rc文件中的配置所创建的。首先看下zygote也就是app_process最初所对应的源文件：App_Main.cpp

```c++
App_Main.cpp

int main( ){
       AppRuntime runtime;
       ...
       if (zygote) {
              runtime.start("com.android.internal.os.ZygoteInit",
                startSystemServer ? "start-system-server" : "");
       }
}
```



#### AppRunTime分析

​	AppRunTime由AndroidRunTime派生出来，start其实是调用了AndroidRunTime中的方法



![](http://img.blog.csdn.net/20150802153752007?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

​	

```c++
[-->AndroidRuntime.cpp]

void AndroidRuntime::start(const char*className, const bool startSystemServer){

    //className的值是"com.android.internal.os.ZygoteInit"
    //startSystemServer的值是true

    char*slashClassName = NULL;
    char*cp;
    JNIEnv* env;
    blockSigpipe();//处理SIGPIPE信号
     ......
    constchar* rootDir = getenv("ANDROID_ROOT");


    //① 创建虚拟机
    if(startVm(&mJavaVM, &env) != 0)
        goto bail;
    //②注册JNI函数
    if(startReg(env) < 0) 
        goto bail;
      
   stringClass = env->FindClass("java/lang/String");
    //创建一个有两个元素的String数组，即Java代码 String strArray[] = new String[2]
   strArray = env->NewObjectArray(2, stringClass, NULL);
   classNameStr = env->NewStringUTF(className);
   //设置第一个元素为"com.android.internal.os.ZygoteInit"
   env->SetObjectArrayElement(strArray, 0, classNameStr);
   startSystemServerStr = env->NewStringUTF(startSystemServer ?"true" : "false");
   //设置第二个元素为"true"，注意这两个元素都是String类型，即字符串。
   env->SetObjectArrayElement(strArray, 1, startSystemServerStr);

   slashClassName = strdup(className);
   startClass = env->FindClass(slashClassName);
    ......
    //找到ZygoteInit类的static main函数的jMethodId。
   startMeth = env->GetStaticMethodID(startClass, "main","([Ljava/lang/String;)V");

	//③调用
   env->CallStaticVoidMethod(startClass,startMeth, strArray);
}
```

- 1.startVm 创建虚拟机

  主要配置虚拟机的一系列参数，如JNI方法检查，虚拟机的堆大小等。然后调用相关方法创建java虚拟机

- 2.startReg

  注册Jni的一些函数，如log等

- 3.CallStaticVoidMethod

  最终JNI调用到对应的类com.android.internal.os.ZygoteInit中的方法main函数



#### Welcome to Java World

```java
[-->ZygoteInit.java]

public static void main(String argv[]) {
  try { 
       SamplingProfilerIntegration.start();
       //①注册Zygote用的socket
       registerZygoteSocket();
       //②预加载类和资源
       preloadClasses();
       preloadResources();
       ......

      //我们传入的参数满足if分支
      if (argv[1].equals("true")) {
          startSystemServer();//③启动system_server进程
       }
      // ZYGOTE_FORK_MODE被定义为false，所以满足else的条件
       if(ZYGOTE_FORK_MODE) {
            runForkMode();
       }else {
          runSelectLoopMode();//④zygote调用这个函数
       }
       closeServerSocket();//关闭socket
       
  }catch (MethodAndArgsCaller caller) {

       caller.run();//⑤很重要的caller run函数，以后分析
  }catch (RuntimeException ex) {
       closeServerSocket();
       throw ex;
   }
     .....
}
```

1. 注册Zygote用的socket

  ​这个sock谁是客户端？在哪里处理消息？

2. 预加载类和资源

    - 在framework/base/preloaded-classes文件中定义了预加载类，这个文件声明了有一千多个类需要预加载，每一个类加载时间不得超过1250毫秒
    - preloadResources 加载framework-res.apk中的文件，我们在开发过程中使用到的com.android.R.xxxx 就是在这个时候被加载的。

3. **启动system_server进程**

   ​	这个方法会去创建Android系统中的常驻进程system_server进程，是Android framework的核心，如果system_server进程死掉了，那么Zygote也会自杀。

   ```java
   [-->ZygoteInit.java]    
   private static boolean startSystemServer() {
   int pid;
       try {
         //把上面字符串数组参数转换成Arguments对象。具体内容请读者自行分析。
         parsedArgs = new ZygoteConnection.Arguments(args);
         int debugFlags = parsedArgs.debugFlags;
         //fork一个子进程，看来，这个子进程就是system_server进程。
         pid = Zygote.forkSystemServer(  // Native fork
           parsedArgs.uid,parsedArgs.gid,
           parsedArgs.gids,debugFlags, null);
       }catch (IllegalArgumentException ex) {
         throw new RuntimeException(ex);
       }
        if(pid == 0) {  这里pid =0
          //system_server进程的工作
          handleSystemServerProcess(parsedArgs);
        }
   }
   ```

   ​

4. 有求必应等待请求  `runSelectLoopMode()`

   ​	开启一个while循环做socke

   ```
   [-->ZygoteInit.java]

   private static void runSelectLoopMode()throws MethodAndArgsCaller {
          while (true) {
   		...
   		if (index == 0) {
                //如有一个客户端连接上，请注意客户端在Zygote的代表是ZygoteConnection
                  ZygoteConnection newPeer = acceptCommandPeer();
                  peers.add(newPeer);
                  fds.add(newPeer.getFileDesciptor());
              } else {
                  boolean done;
                 //客户端发送了请求，peers.get返回的是ZygoteConnection
                 //后续处理将交给ZygoteConnection的runOnce函数完成。
                  done = peers.get(index).runOnce();
           }
       }
   ```


#### 小结：

![](http://images.cnitblog.com/blog/563439/201309/25164254-eadc0c7979b74628aefad305b4b4f75a.png)





#### System_Server

​	System_server  创建后便开始执行他的使命

```java
if(pid == 0) {  这里pid =0
       //system_server进程的工作
       handleSystemServerProcess(parsedArgs);
}
```

```java
[–>RuntimeInit.java]

public static final void zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)throws ZygoteInit.MethodAndArgsCaller {
 	......
 	//native层初始化
 	nativeZygoteInit();
    ......
    //调用com.android.server.SystemServer.main方法
    invokeStaticMain(args.startClass, args.startArgs, classLoader);
    ...... 
 }
```

1. nativeZygoteInit是一个native函数，主要用来与Binder机制建立连接，这样SystemServer就能够使用Binder了。
2. **invokeStaticMain**中是通过抛出异常来启动SystemServer中的main方法的。Zygote的main方法会截获这个异常，并调用caller.run();来启动SystemServer.main。





至此了System_server 的mian函数，在这个main函数做了那些处理：

```
System_server.java

public static void main(String[] args) {
        new SystemServer().run();
    }
private void run() {
    .....
 	init1();//  调用native方法，创建系统服务，最后通过jni调用init2，加载各种重要service
}
```





System_server  小结

![](http://img.blog.csdn.net/20150802153830848?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



- 在AndroidRuntime   里创建虚拟机，并跳转到java层的ZygoteInit
- ZygoteInit中创建socket做消息处理，预加载资源，创建system_server进程
- 调用system_server 的main  加载系统各种服务





###4.Zygote的分裂

​	我们上面讲创建了一个Socket，那么谁会给他发消息呢？

​	我们在启动一个Activity时，如果Activity所在的进程不存在如何创建进程？

​	在ActivityManagerService中，如果进程不存在，回去尝试连接Zygote的Socket，并发送创建进程信息。

在上文的Socket处理消息中，会调用runOnce（），fork一个子进程并最终会调用到`RunTimeInit`的``invokeStaticMain()`,

​	这个方法我们之前是用它加载进程的Main方法。同样的道理，这里也是会去调用到ActivityThread的Main函数，做一些初始化处理

​	![](http://img.blog.csdn.net/20150802154008866?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)







### 5.Interesting

- 虚拟机heapSize的大小

  从前面看到每一个虚拟机的heapSize大小都是从文件固定好的，如果我们改动了这个文件，那么所有的虚拟机都会变化成配置的大小。一般厂商都会修改这个配置。

- 开机速度优化

  - 前面讲到在Zygote初始化过程中预加载一千多个class，消耗不少时间


  - 开机时会对系统内的所有apk进行扫描获取信息
  - System_server 创建各种Service ，花费时间

  讨论第一个问题，其实，这些类的预加载是可以去掉的，因为如果在使用的时候发现没有还是会去加载的，但是就会破坏Android通过fork机制创建进程的本意了。

- WatchDog

  - 单例

  - 开启线程while循环去检测各种Service的状态，主要是判断有没有发生死锁，如果异常会去重启System_Server

    主要是ActivityManagerService  PowerManagerService  WindowManagerService

    在他们的构造函数内，会把自己加入到WatchDog的检测队列中

    ​

  ​

