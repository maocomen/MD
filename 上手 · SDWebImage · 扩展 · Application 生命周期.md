# 上手 · SDWebImage · 扩展 · Application 生命周期





AppDelegate 会在程序启动时执行一些其他的任务：

- 查看启动配置字典（launchOptions）来确定应用程序启动的原因

  ``` objective-c
  - (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions NS_AVAILABLE_IOS(6_0);
  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions NS_AVAILABLE_IOS(3_0);

  这两个方法提供了一个字典 launchOptions，它包含表明应用程序启动原因的键。
  ```

- 确定是否应该恢复状态。如果应用程序之前保存了视图的控制器的状态，则当下面这个方法返回为 YES 的时候才会将这些视图控制器恢复到之前的状态

  ``` objective-c
  - (BOOL) application:(UIApplication *)application shouldRestoreApplicationState:(NSCoder *)coder NS_AVAILABLE_IOS(6_0);
  ```

- 为应用注册远程通知。为了接收远程通知（也称为推送通知），应用程序必须通过下面方法去向远程通知服务注册，通常是在启动时执行此方法

  ```objective-c
  - (void)registerForRemoteNotifications NS_AVAILABLE_IOS(8_0);
  ```

- 打开一个发送你应用的 URL。如果有一个要打开的 URL，系统会调用下面这个方法。你也可以通过查看 launch option 字典来判断是否有要打开的 URL。你必须通过将 CFBundleURLTypes 键添加应用程序的 Info.plist 文件来声明你应用程序支持的 URL 类型。有关注册和处理 URL 的更多信息，可以参阅  [App Programming Guide for iOS](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072)

  ```objective-c
  - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options NS_AVAILABLE_IOS(9_0); 
  ```

- 为你应用程序提供 Root Window，正常来说，由 Xcode 提供的 AppDelegate 已经实现了 window 属性，除非你要定制应用程序的窗口，不然你不用做任何事情

下面方法中的 option 字典包含应用程序启动时重要的信息，这个字典里面的 Key 会告诉你程序启动的原因，并且给你一个相应调整启动程序的机会。例如，如果你的程序是因为点击一个远程推送而打开的，你可能需要跳转页面来展示这个通知相关的数据。有关程序启动原因列表，可以参阅 [Launch Options Keys](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/launch_options_keys) 

```objective-c
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions NS_AVAILABLE_IOS(6_0);
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions NS_AVAILABLE_IOS(3_0);
```

### Managing State Transitions

AppDelegate 主要工作之一就是相应程序的状态改变。对于每次发生状态改变，系统都会调用 AppDelegate 里面对应的方法。表一列举了应用程序可能的状态，图一显示应用程序如何从一个状态转换成另外一个状态。

表一、App 状态

| State 状态             | Description 描述                                             |
| ---------------------- | ------------------------------------------------------------ |
| Not running 非运行状态 | 应用程序尚未启动或者被用户或者系统终止                       |
| Inactive 非活跃状态    | 应用程序在前台运行，但是没有接收到事件（尽管如此，程序也有可能正在执行其它代码）。程序通常只会当转换成其他状态时，短暂的处于这个非活跃状态<br />当进入这种状态时，程序应该将自己置于静止状态，并期望很快能进入后台状态或者活跃状态 |
| Active 活跃状态        | 应用程序正在前台运行并且接受事件，这个是前台应用程序正常的模式<br />应用程序在前台状态是没有特殊的限制的，前台状态就是要对响应用户的事件 |
| Background 后台状态    | 应用程序正在执行代码，但是在屏幕上不可见。当用户退出程序时，系统会暂时将程序状态改为后台状态。在其他时候，系统可能会启动程序到后台状态（或者唤醒被挂起的程序），并给它时间来处理特殊的任务。例如，系统可能会唤起应用程序，以便它可以处理后台下载、位置事件、远程通知和其他类型的事件 |
| Suspended 挂起         | 应用程序在内存中，但是不执行代码。系统会将后台状态且没有任何待完成任务的程序转换成挂起状态。系统随时会清除挂起状态的程序，而不会唤醒它们为其他程序腾出空间 |

图一 iOS App 状态转变

![mage-20180328014339](/var/folders/l3/fvc9fyz96yb28xg6g_jqc0qr0000gn/T/abnerworks.Typora/image-201803280143395.png)









### Responding to Notifications and Events

### Topics

### Initializing the App

### Responding to App State Changes and System Events

### Managing App State Restoration

### Downloading Data in the Background

### Handling Remote Notification Registration

### Continuing User Activity and Handling Quick Actions

### Interacting With WatchKit

### Interacting With HealthKit

### Opening a URL-Specified Resource

### Disallowing Specified App Extension Types

### Handling SiriKit Intents

### Handling CloudKit Invitations

### Managing Interface Geometry

### Providing a Window for Storyboarding



## Relationships

