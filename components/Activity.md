## android 常用组建 -- Activity ##
### 一、Activity 生命周期 ###

![activit的生命周期](http://developer.android.com/images/activity_lifecycle.png)

我们知道activity有四种状态：

- 运行状态：activity处于栈顶，在屏幕前台显示。


- 暂停状态：activity失去焦点，但是依然可见。（如栈顶的Activity是透明的或者栈顶Activity并不是铺满整个手机屏幕）


- 停止状态：activity被其他activity完全覆盖，此时此Activity对用户不可见。


- 销毁状态：从停止状态被系统从内存中删除。


Activity实例是由系统自动创建，并在不同的状态期间回调相应的方法。一个最简单的完整的Activity生命周期会按照如下顺序回调：onCreate -> onStart -> onResume -> onPause -> onStop -> onDestroy。称之为entire lifetime。

当执行onStart回调方法时，Activity开始被用户所见（也就是说，onCreate时用户是看不到此Activity的，那用户看到的是哪个？当然是此Activity之前的那个Activity），一直到onStop之前，此阶段Activity都是被用户可见，称之为visible lifetime。

当执行到onResume回调方法时，Activity可以响应用户交互，一直到onPause方法之前，此阶段Activity称之为foreground lifetime。

在实际应用场景中，假设A Activity位于栈顶，此时用户操作，从A Activity跳转到B Activity。那么对AB来说，具体会回调哪些生命周期中的方法呢？回调方法的具体回调顺序又是怎么样的呢？

开始时，A被实例化，执行的回调有A:onCreate -> A:onStart -> A:onResume。

当用户点击A中按钮来到B时，假设B全部遮挡住了A，将依次执行A:onPause -> B:onCreate -> B:onStart -> B:onResume -> A:onStop。

此时如果点击Back键，将依次执行B:onPause -> A:onRestart -> A:onStart -> A:onResume -> B:onStop -> B:onDestroy。

**在Android中，有两个按键在影响Activity生命周期需要注意，即Back键和Home键。**

1、此时如果按下Back键，系统返回到桌面，并依次执行A:onPause -> A:onStop -> A:onDestroy。

2、此时如果按下Home键（非长按），系统返回到桌面，并依次执行A:onPause -> A:onStop。由此可见，Back键和Home键主要区别在于是否会执行onDestroy。
### 二、 Activity 的任务栈和管理任务栈 ###

Task是一个Activities的收集器，专门收集用户操作交互所打开的Activity。这些Activities都被安排在一个回收栈back stack中，安排的顺序和它们打开的顺序一致。即先打开的安排在最底部，最后一个打开的安排在顶部。一个Task任务是一个完整的单元，它可以运行于后台。当用户开启一个新的任务Task，或者通过HOME键跳到主屏幕时，这个任务就处于后台运行，不过这个Task包含所有的Activities都处于Stopped状态。但back stack维护着这个任务的完整数据，只是简单的被其他task替换焦点而已。在同一时刻，允许多个任务Task运行于后台，但当系统资源不足，需要结束掉某些状态为Stopped的任务时，Activities的状态数据将被丢失。

由于Activities在Task中并不支持重新排列，因此当你的一个Activity在一次应用中打开多次，那么每次都创建一个新的Activity实例，而不是引用旧的Activity实例。也就是说，一个Activity在你的应用中会多次实例化，当然你也可以通过其他方式使得同一Activity在一个任务中只实例化一次。这涉及下面的Activity的四种启动模式。



##### 我们来看看<activity>元素的launchMode属性定义的四种启动模式： #####

- **"standard"(默认启动模式)**：
standard是默认的启动模式，即如果不指定launchMode属性，则自动就会使用这种启动模式。这种启动模式表示每次启动该Activity时系统都会为创建一个新的实例，并且总会把它放入到当前的任务当中。声明成这种启动模式的Activity可以被实例化多次，一个任务当中也可以包含多个这种Activity的实例。

- **"singleTop"**：
这种启动模式表示，如果要启动的这个Activity在当前任务中已经存在了，并且还处于栈顶的位置，那么系统就不会再去创建一个该Activity的实例，而是调用栈顶Activity的onNewIntent()方法。声明成这种启动模式的Activity也可以被实例化多次，一个任务当中也可以包含多个这种Activity的实例。

- **"singleTask"**：
这种启动模式表示，activity只会在任务栈里面存在一个实例。如果要激活的activity，在任务栈里面已经存在，就不会创建新的activity，而是复用这个已经存在的activity，调用 onNewIntent() 方法，并且清空这个activity任务栈上面所有的activity。
- **"singleInstance"**：
单一实例，整个手机操作系统里面只有一个实例存在。不同的应用去打开这个activity共享公用的同一个activity。

##### 除了在launchMode中配置外，还有一些因素影响到任务栈的管理 #####
#### 1、配置intent的flag ####

- FLAG_ACTIVITY_NEW_TASK ：
效果同singleTask

- FLAG_ACTIVITY_SINGLE_TOP：
效果同singleTop

- FLAG_ACTIVITY_CLEAR_TOP：
这个模式是没有属性配置支持的。在这种模式下，如果启动一个已经存在于当前Task任务的Activity，那么Task顶部所有的Activity将被销毁，并且为将要启动的Activity新建一个Activity实例，存放在task的back stack的顶部。

#### 2、处理affinity ####
affinity可以用于指定一个Activity更加愿意依附于哪一个任务，在默认情况下，同一个应用程序中的所有Activity都具有相同的affinity，所以，这些Activity都更加倾向于运行在相同的任务当中。当然了，你也可以去改变每个Activity的affinity值，通过<activity>元素的taskAffinity属性就可以实现了。

affinity主要有以下两种应用场景：

- 当启动一个Activity的Intent包含了FLAG_ACTIVITY_NEW_TASK标识.
在默认情况下，新建一个Activity，会调用startActivity()方法进入Task当中。它放置到和启动它的Activity相同的back stack。但是，如果启动的Intent包含了FLAG_ACTIVITY_NEW_TASK标识，系统将为这个Activity寻找一个不同的Task。通常是新建一个新的Task。但是也未必全是这样，如果存在一与之相同taskAffinity定义的Task，那么这个Activity将运行在那里，否则新建Task。

- 当一个Activity的属性allowTaskReparenting设置为true时.
举个例子说明这个属性的作用：
假设一个选择城市查看天气的Activity是一个旅游应用程序的一部分。这个Activity与这个应用中的其他Activity有相同affinity,并且设置了这个属性为true。当你的其他Activity启动这个查看天气的Activity时，它和你的Activity属于同一个Task。但是，当你的旅游应用程序再次展现在前端时，这个查看天气的Activity会重新分配到旅游应用程序的Task中，并显示天气情况
#### 3、清理任务栈 ####

如果一个用户离开一个Task很长时间，系统会清理这个Task，当然除了跟Activity。当用户回来时，只剩下这个跟Activity，其他的都被销毁了。
以下属性可以设置并改变这种行为方式：
 
- alwaysRetainTaskState：如果在Task的跟Activity中设置这个属性为true，默认的行为将不再发生，所有Activity的状态数据将永久保存。
- clearTaskOnLaunch：如果在Task的跟Activity中设置这个属性为true，当用户离开或者回到这个Task时，都会清理除了跟Activity之外的Activity。
- finishOnTaskLaunch：类似clearTaskOnLaunch ，但只在一个Activity中生效，而不是整个Task，它可以清理任何的Activity，包括跟Activity。你为当期会话保存Activity，当用户回来或者离去，都不将再恢复呈现。

### 三、Activity 启动方式 ###
可参考 [http://blog.csdn.net/daiyelang/article/details/8649134](http://blog.csdn.net/daiyelang/article/details/8649134 "intent 显示和隐示启动activity")

### 四、Activity 启动流程 ###






参考：

- http://developer.android.com/
- http://www.cnblogs.com/lwbqqyumidi/p/3769113.html
- http://blog.csdn.net/chaoyue0071/article/details/43791909
