# UIKit - App and Environment

管理生命周期时间以及app的UI界面，并获得有关app运行的特性和环境的信息

#### Overview

在iOS13及之后，用户可以同时创造并管理app用户界面的多个实例，并且通过app的开关在界面之间进行转换。在iPad，用户也可以肩并肩展示多个实例。每一个UI的实例展现不同的内容或用不同的方式展现相同的内容。例如，用户可以在日历app中显示一个具体的日子，或者一整个月份。

UIKit通过使用trait collections对现有环境进行细节的交流。trait collections反映了设备设置、界面设置、用户偏好的组合。例如，你使用这些特性去侦测暗黑模式在当前视图中是否被激活。当你想要基于当前环境下去自定义界面的内容时，可以调用UIView或UIViewController对象的trait collection。当你想要某个对象获得这些特性改变的通知时，让这个类去遵守UITraitEnvironment协议。



#### 我遇到的一些问题

Q: applicationDidEnterBackground are not called on iOS 13?

A: New projects created by Xcode 11 are configured to use the new scene-based lifecycle. The "old" app delegate lifecycle methods are not called in apps that uses scenes. There are similar lifecycle methods in the scene delegate that are called instead. WWDC 2019 212: Introducing Multiple windows on iPad discusses this (and much more).

**基于界面的app会调用SceneDelegate里面的类似函数而不会调用AppDelegate内的方法**



## Life Cycle

### Managing Your App‘s Life Cycle

当你的app在前台或者后台或者处理其他一些重要的与系统相关的事件时，向系统发送通知。

#### Overview

app的当前状态决定了它可以或不可以做某些事情。例如，当一个用户在使用一个前台app，它的优先级会高于各种系统资源，包括CPU。相反，由于后台app并不展现在屏幕上，其必须尽可能执行越少的任务，最好是什么也不做。当app在状态之间改变时，应该相对应的调整app的行为。

当app的状态改变时，UIKit会通过调用以下的合适的委托对象的方法通知你：

- In iOS 13 and later, use [`UISceneDelegate`](apple-reference-documentation://hsd-omBXXP) objects to respond to life-cycle events in a scene-based app.
- In iOS 12 and earlier, use the [`UIApplicationDelegate`](apple-reference-documentation://hsIWiQNXtw) object to respond to life-cycle events. 

> Note
>
> If you enable scene support in your app, iOS always uses your scene delegates in iOS 13 and later. In iOS 12 and earlier, the system uses your app delegate.

#### Respond to Scene-Based Life-Cycle Events

如果app支持多个界面，UIKit会为每个界面分别传递生命周期事件。一个界面代表当前运行设备的app UI的一个实例。用户可以为每个app创建多个不同的界面，每个界面可能在不同的状态下执行。例如，一个界面可能在前台同时另一个界面在后台或者被挂起。

> Important
>
> Scene support is an opt-in feature. To enable basic support, add the [`UIApplicationSceneManifest`](apple-reference-documentation://hs6xjucOII)key to your app’s `Info.plist` file as described in [Specifying the Scenes Your App Supports](apple-reference-documentation://ts3262272). 

下面的图片展现了界面之间的状态转换。当用户或者系统向你的app请求一个新的界面，UIKit会创建它并使其属于unattached状态。用户请求的界面会快速移动至前台，而系统请求的界面通常会进入后台以对某个事件进行处理。例如，系统可能会请求一个界面以处理一个定位事件。当用户关闭了UI，UIKit会将相关连的界面移动至background状态并最终移至suspended状态。UIKit可以在任何时间解除一个background或suspended界面的联系并取回它们占用的资源，并将该界面移动至unattached状态。

![](https://docs-assets.developer.apple.com/published/61283402a3/024b99c5-4ab6-4ee0-bb41-6e6426ec6a64.png)

使用界面转换去执行下列任务：

- 当UIKit连接到app的界面时，配置界面的初始UI并载入界面所需要的数据。
- 当转换到foreground-active状态时，配置UI并准备与用户进行交互。See [Preparing Your UI to Run in the Foreground](apple-reference-documentation://ts2922735).
- 离开foreground-active状态时，保存数据并逐渐暂停app的行为。See [Preparing Your UI to Run in the Background](apple-reference-documentation://ts2922738).
- 进入background状态时，结束重要的任务，释放尽可能多的内存，并准备app快照。See [Preparing Your UI to Run in the Background](apple-reference-documentation://ts2922738).
- 界面失去连接时，清除与界面关联的所有共享资源。
- 对于与界面相关的事件，你必须通过[`UIApplicationDelegate`](apple-reference-documentation://hsIWiQNXtw) 对象去回应app的启动。see [Responding to the Launch of Your App](apple-reference-documentation://ts2922731).

#### Respond to App-Based Life-Cycle Events （这一节的内容应该用不到）

在iOS12之前，以及对于不支持界面的apps，UIKit会将生命周期时间传递到[`UIApplicationDelegate`](apple-reference-documentation://hsIWiQNXtw)对象。app的委托会管理app的所有窗口，包括那些展示在不同屏幕上的窗口。这导致app的状态转换会影响app的整个UI，包括外部显示的内容。

下面的图片展示了设计app委托对象的状态转移。在启动之后，系统将app放置在inactive或background状态，取决于UI是否将要在屏幕上显示。如果在前台，系统会将app自动转换为active状态。之后状态会在active和background之间波动直到app被terminated。

![](https://docs-assets.developer.apple.com/published/c63cd35863/4d403429-fa30-4706-863f-5e3617ee21d0.png)

使用app转换去执行以下的任务：

- 启动时，初始化app的数据结构以及UI。See [Responding to the Launch of Your App](apple-reference-documentation://ts2922731).
- 激活时，停止配置UI并准备与用户进行交互。See [Preparing Your UI to Run in the Foreground](apple-reference-documentation://ts2922735).
- 失活时，存储数据并逐渐暂停app的行为。See [Preparing Your UI to Run in the Background](apple-reference-documentation://ts2922738).
- 进入background状态时，结束重要的任务，释放尽可能多的内存，并准备app快照。See [Preparing Your UI to Run in the Background](apple-reference-documentation://ts2922738).
- 终结时，应立即停止所有的任务并且释放所有相关的资源。See [`applicationWillTerminate(_:)`](apple-reference-documentation://hs4ku9mfiz).

#### Respond to Other Significant Events

对于处理生命周期事件，app必须准备处理下面表格所列出的事件。使用[`UIApplicationDelegate`](apple-reference-documentation://hsIWiQNXtw)对象进行处理。在某些情况下，你要使用通知进行处理，而允许你响应app的其他部分。

| Memory warnings                              | Received when your app’s memory usage is too high. Reduce the amount of memory your app uses; see [Responding to Memory Warnings](apple-reference-documentation://ts2928954). |
| -------------------------------------------- | ------------------------------------------------------------ |
| Protected data becomes available/unavailable | Received when the user locks or unlocks their device. See [`applicationProtectedDataDidBecomeAvailable(_:)`](apple-reference-documentation://hs0Gavo8Aw) and [`applicationProtectedDataWillBecomeUnavailable(_:)`](apple-reference-documentation://hspwWdQQ7F). |
| Handoff tasks                                | Received when an [`NSUserActivity`](apple-reference-documentation://hs1XnsIq8s) object needs to be processed. See [`application(_:didUpdate:)`](apple-reference-documentation://hs0Hju5j6O). |
| Time changes                                 | Received for several different time changes, such as when the phone carrier sends a time update. See [`applicationSignificantTimeChange(_:)`](apple-reference-documentation://hsyJsfMCKe). |
| Open URLs                                    | Received when your app needs to open a resource. See [`application(_:open:options:)`](apple-reference-documentation://hsxenBWBG5). |



### Responding to the Launch of Your App

#### Overview

当用户触碰app的图标时app就会开始启动。如果app请求某些具体的事件，系统可能会将你的app在后台启动以对这些事件进行处理。对于一个基于界面的app，类似的，取决于你的某一个界面在前台或者执行一些任务，系统会在不同的状态下启动app。

所有的app有一个相关连的进程，由[`UIApplication`](apple-reference-documentation://hsZC8VWkhJ)对象表示。apps也拥有一个app委托对象，其遵循[`UIApplicationDelegate`](apple-reference-documentation://hsIWiQNXtw)协议，这个协议是为了响应这个进程内发生的重要事件。即使是基于界面的app也使用app委托去管理诸如启动或者终结的基本事件。在启动时，UIKit自动创建一个`UIApplication`对象以及app委托。然后app的main event loop开始。

#### Provide a Launch Storyboard

当用户第一次在设备上开启app时，系统首先展示app的启动故事板直到app准备好展示它的UI。展示启动故事板向用户确保你的app正在启动。如果app使用的初始化时间十分短暂，启动故事板只会展示很短的时间。

Xcode提供一个可以自定义的默认启动故事板，也可以根据需求添加更多的启动故事板。

1. Open your project in Xcode.
2. Select File > New > New File.
3. Add a Launch Screen resource to your project.

为你的启动故事板添加视图，并使用自动布局去限制视图的大小和位置以使得视图能够适应底层的环境。

> Important
>
> In iOS 13 and later, always provide a launch storyboard for your app. Don’t use static launch images.

#### Initialize Your App's Data Structures

将app的初始化代码放进下面的方法中：

- [`application(_:willFinishLaunchingWithOptions:)`](apple-reference-documentation://hssMx3XNbq)
- [`application(_:didFinishLaunchingWithOptions:)`](apple-reference-documentation://hs_UpR7RSf)

UIKit在app启动周期的开始调用这些函数，使用它们以：

- 初始化app的数据结构
- 核实app拥有其运行所需的资源
- 当你的app第一次启动时，进行所有的一次性设置(one-time setup)。例如，安装模板或在可写的目录下的用户可修改文件。See [Performing One-Time Setup for Your App](apple-reference-documentation://ts2922734). 
- 链接至app使用的重要的服务。例如链接Apple Push Notification服务如果app支持远程推送。
- 检查启动选项字典以得到关于你的app为什么被启动的信息。See [Determine Why Your App Was Launched](apple-reference-documentation://ts2922731#2922740).

对于不基于界面的app，UIKit在启动时自动加载默认的用户界面。使用`application(_:didFinishLaunchingWithOptions:)`方法去对界面在显示前进行改变。（注：我目前的app无用）

#### Move Long-Running Tasks off the Main Thread

当用户启动app时，应当尽可能缩短启动的时间。UIKit直到[`application(_:didFinishLaunchingWithOptions:)`](apple-reference-documentation://hs_UpR7RSf)方法返回后才会开始展示app的页面。在这个方法内或[`application(_:willFinishLaunchingWithOptions:)`](apple-reference-documentation://hssMx3XNbq)方法执行耗时长的任务会使得你的app在用户视角显得十分的缓慢。由于后台执行时间有限，在后台启动时快速返回是十分重要的。

将对app初始化不重要的任务移除出启动时间序列，例如：

- 延迟不需要立即使用的特性的初始化过程
- 将重要的、长时间的任务移除出app的主线程。例如，将这些任务异步地在全局分配队列(global dispatch queue)上执行。

#### Determine Why Your App Was Launched

当UIKit启动app时，它会传递启动选项字典到[`application(_:willFinishLaunchingWithOptions:)`](apple-reference-documentation://hssMx3XNbq)和[`application(_:didFinishLaunchingWithOptions:)`](apple-reference-documentation://hs_UpR7RSf)方法以通知为什么你的app被启动了。字典的键值表示了需要立即执行的重要任务。例如，键值可能反映了用户在其他地方开启了app并想要继续app任务的行为。要经常检查这个字典的内容以合理应对不同的情况。

> Note
>
> For a scene-based app, examine the options passed to the [`application(_:configurationForConnecting:options:)`](apple-reference-documentation://hsXCIRc8BC) method to determine why your scene was created. 

下面的代码展示了app委托的方法处理后台位置更新的情况。当位置键值存在时，app会立即开始更新而不是推迟处理这个事件。

```swift
class AppDelegate: UIResponder, UIApplicationDelegate, 
               CLLocationManagerDelegate {
    
   let locationManager = CLLocationManager()
   func application(_ application: UIApplication,
              didFinishLaunchingWithOptions launchOptions:
              [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
       
      // If launched because of new location data,
      //  start the visits service right away.
      if let keys = launchOptions?.keys {
         if keys.contains(.location) {
            locationManager.delegate = self
            locationManager.startMonitoringVisits()
         }
      }
       
      return true
   }
   // other methods…
}
```

系统并不会包含你的app不支持相关特性的键值。例如，系统不会包含[`remoteNotification`](apple-reference-documentation://hshfaLsyDm)键值如果你的app不支持远程推送。

对于启动选项键值，请查看[`UIApplication.LaunchOptionsKey`](apple-reference-documentation://hsIqAZX-cg)。

#### About the App Launch Sequence

启动app包含许多复杂的步骤，其中大多数步骤UIKit会自动进行处理。在启动序列中，UIKit会调用app委托的方法使得你可以执行自定义的任务。下图阐述了app初始化的步骤：

![](https://docs-assets.developer.apple.com/published/52c7b459e7/76e68c08-6b09-4bac-8a00-44df7a097a43.png)

1. The app is launched, either explicitly by the user or implicitly by the system.
2. The Xcode-provided `main` function calls UIKit's [`UIApplicationMain(_:_:_:_:)`](apple-reference-documentation://hswqXjGwiK) function.
3. The [`UIApplicationMain(_:_:_:_:)`](apple-reference-documentation://hswqXjGwiK) function creates the [`UIApplication`](apple-reference-documentation://hsZC8VWkhJ) object and your app delegate.
4. UIKit loads your app's default interface from the main storyboard or nib file.
5. UIKit calls your app delegate's [`application(_:willFinishLaunchingWithOptions:)`](apple-reference-documentation://hssMx3XNbq) method.
6. UIKit performs state restoration, which calls additional methods of your app delegate and view controllers.
7. UIKit calls your app delegate's [`application(_:didFinishLaunchingWithOptions:)`](apple-reference-documentation://hs_UpR7RSf) method .

当初始化结束后，系统使用界面委托或app委托去展示UI并管理app的生命周期。

#### Performing One-Time Setup for Your App

这部分的内容参见文档，由于我的app中不使用就没有细看。

#### Preserving Your App's UI Across Launches

同上。



### UIApplication

iOS app的控制、协调中心。

```swift
class UIApplication : UIResponder
```

#### Overview

每个app只拥有一个`UIApplication`的实例（少数情况下可能是该类的子类）。当app启动时，系统会调用[`UIApplicationMain(_:_:_:_:)`](apple-reference-documentation://hswqXjGwiK)函数；在app的其他任务中，这个函数创建一个[Singleton](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Singleton.html#//apple_ref/doc/uid/TP40008195-CH49) `UIApplication`对象（Singleton：不论系统请求多少次，只会返回同一个实例）。因此你可以通过调用访问[`shared`](apple-reference-documentation://hssxDFtL7y)类方法访问这个对象。（个人理解：整个app环境下只会有这一个该类的实例，如果想访问这个实例调用类的该方法即可）

这个对象所扮演的主要角色是对初始化以及即将发生的用户时间进行处理。其通过控制对象（[`UIControl`](apple-reference-documentation://hsfkjoOZkS)类的实例）派送行为消息给合适的目标对象。应用对象维护了一系列的开启视窗([`UIWindow`](apple-reference-documentation://hsmNIu8_YZ)对象)并且通过这些视窗可以获得app的任意[`UIView`](apple-reference-documentation://hsdIxI1kkd)对象。

`UIApplication`类定义了一个遵循[`UIApplicationDelegate`](apple-reference-documentation://hsIWiQNXtw)协议的委托并且必须拥有协议定义的一些方法。应用对象会通知委托一些重要的运行时发生的事件，例如，app启动、低内存警告、app终止。这使得app可以合适地处理这些事件。

app可以通过[`openURL(_:)`](apple-reference-documentation://hsIXImZyJa)方法合作处理某些资源，比如邮件或图片文件。例如，app调用这个方法并使用email的url作为参数会导致邮件app打开并显示信息。

类内的许多API能使你管理设备的行为：

- 临时暂停后续的触碰事件 ([`beginIgnoringInteractionEvents()`](apple-reference-documentation://hs8rhpzP0d))
- 注册远程推送([`registerForRemoteNotifications()`](apple-reference-documentation://hscdbGTKvU))
- 触发 undo-redo UI ([`applicationSupportsShakeToEdit`](apple-reference-documentation://hs2TSlwiBR))
- 判断是否有已安装的app处理对应的URL([`canOpenURL(_:)`](apple-reference-documentation://hssXuTJxu1))
- **延长app执行时间使得app可以在后台结束一个任务**([`beginBackgroundTask(expirationHandler:)`](apple-reference-documentation://hslLABkVG8), [`beginBackgroundTask(withName:expirationHandler:)`](apple-reference-documentation://hsBnf57zab))
- 调度和取消本地通知([`scheduleLocalNotification(_:)`](apple-reference-documentation://hsdiTIvQCm), [`cancelLocalNotification(_:)`](apple-reference-documentation://hsajcqo-dq))
- 协调远端控制事件 ([`beginReceivingRemoteControlEvents()`](apple-reference-documentation://hsNj-NJ4VZ), [`endReceivingRemoteControlEvents()`](apple-reference-documentation://hsxEsl_ntW))
- 执行app层面状态恢复任务(methods in the [Managing the State Restoration Behavior](apple-reference-documentation://hsZC8VWkhJ#1657552) task group)



### beginBackgroundTask(expirationHandler:)

#### Declaration

```swift
func beginBackgroundTask(expirationHandler handler: (() -> Void)? = nil) -> UIBackgroundTaskIdentifier
```

#### Parameters

```swift
handler
```

在后台剩余时间到达0时所调用的函数。用这个函数去清理并标记后台任务的终结。如果没有显式终结任务会导致app的终结。系统会和主线程同步调用这个函数并暂时阻塞app的挂起。

#### Return Value

为新的后台任务返回独一无二的标识符。你必须将这个值传递到[`endBackgroundTask(_:)`](apple-reference-documentation://hsTqIAib9T)方法以终结后台任务。如果返回[`invalid`](apple-reference-documentation://hsmfn5TKFj)表示无法在后台运行。

#### Discussion

这个方法为你的app请求额外的后台执行时间。如果遗留的未完成任务对用户体验有不好的影响可以调用这个方法。例如，在写数据之前调用这个方法以防止操作进行时app被系统挂起。对于需要更长时间的后台任务请参见[BackgroundTasks](apple-reference-documentation://csbackgroundtasks)

在开始你的任务之前尽可能早的调用这个方法，最好在app真正进入后台之前。这个方法为你的app异步请求任务断言（task assertion）。如果在app即将被挂起之前调用这个方法，在断言被许可之前系统有可能挂起你的app。例如，不要在[`applicationDidEnterBackground(_:)`](apple-reference-documentation://hs3L5RGJXG)方法的末尾调用这个函数并期望app可以继续运行。

每次调用这个方法必须匹配另一个调用`endBackgroundTask(_:)`。在后台运行的app只有有限的时间去执行任务（可以通过[`backgroundTimeRemaining`](apple-reference-documentation://hsRQD2xNXO)属性获得最大的后台运行剩余时间）。如果在时间结束之前没有调用`endBackgroundTask(_:)`，app会被系统终结。如果你提供了handler参数，系统在终结之前会调用这个函数。

你可以在app的任意执行阶段调用这个方法。你可以多次调用这个方法来标记不同后台任务的开始。然而每个任务必须被分别终止。你可以通过这个方法返回的值来识别给定的任务。



### UIApplicationDelegate

#### Declaration

```swift
protocol UIApplicationDelegate
```

#### Overview

app的委托对象管理app的共享行为。app委托是app的高效根对象，并且其与[`UIApplication`](apple-reference-documentation://hsZC8VWkhJ)结合共同管理一些系统的交互。UIKit在创建app时就会创建委托对象。

使用委托对象以处理以下的任务：

- 初始化核心数据结构
- 配置app界面
- 响应app外部生成的通知，比如低内存警告、下载完毕通知等
- 响应目标为app本身的事件，不具体到app的界面、视图和视图控制器
- 注册app所需的服务，如远程推送服务

更多的信息参见[Responding to the Launch of Your App](apple-reference-documentation://ts2922731).



以下为该协议下的一些重要的函数：

### application(_:didFinishLaunchingWithOptions:)

**通知委托app的启动即将结束并且app即将开始运行**

#### Declaration

```swift
optional func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool
```

#### Parameters

- `application`

  The singleton app object.

- `launchOptions`

  A dictionary indicating the reason the app was launched (if any). The contents of this dictionary may be empty in situations where the user launched the app directly. For information about the possible keys in this dictionary and how to handle them, see *Launch Options Keys*.

#### Return Value

`false` if the app cannot handle the URL resource or continue a user activity, otherwise return `true`. The return value is ignored if the app is launched as a result of a remote notification.

### application(_:configurationForConnecting:options:)

**当创建一个新的界面时返回对应的配置信息给UIKit**

#### Declaration

```swift
optional func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration
```

#### Parameters

- `application`

  The singleton app object.

- `configurationForConnecting`

  The session object associated with the scene. This object contains the initial configuration data loaded from the app's `Info.plist` file, if any. 

- `options`

  System-specific options for configuring the scene. 

#### Return Value

The configuration object containing the information needed to create the scene.

### application(_:didDiscardSceneSessions:)

**通知委托用户关闭了一个或多个界面**

#### Declaration

```swift
optional func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>)
```

#### Parameters

- `application`

  The singleton app object.

- `sceneSessions`

  The session objects associated with the discarded scenes.



### Scenes

同时对app UI的多个实例进行管理，并引导资源到合适的UI实例

#### Overview

UIKit通过[`UIWindowScene`](apple-reference-documentation://hsyhR5D2LB)对象管理app UI的每个实例。一个界面(scene)包含窗口以及视图控制器以展示UI的一个实例。每一个界面也有对应的[`UIWindowSceneDelegate`](apple-reference-documentation://hs0leiBqpu)对象以用来协调UIKit与app的交互。不同的界面同时运行并共享app进程空间的相同内存。因此一个app在同时可能拥有多个视图以及对应的多个视图委托。



### Preparing Your UI to Run in the Foreground

#### Overview

使用前台转换以准备app即将出现在屏幕上的UI。app通常为了响应用户的行为从而转移至前台。例如，当用户点击app图标时，系统会启动app并进入app前台。使用前台转换更新app的UI，并获得资源、开启需要的服务以处理用户的请求。

所有的状态转移导致UIKit会发送通知给合适的委托对象：

- In iOS 13 and later—A [`UISceneDelegate`](apple-reference-documentation://hsd-omBXXP) object.
- In iOS 12 and earlier—The [`UIApplicationDelegate`](apple-reference-documentation://hsIWiQNXtw) object.

你可以同时支持两种委托对象，但UIKit通常只使用可行的界面委托对象。UIKit只会通知即将进入前台的界面相关连的界面委托。关于如歌配置界面的支持，参见 [Specifying the Scenes Your App Supports](apple-reference-documentation://ts3262272).

#### Update Your App’s Data Model when Entering the Foreground

在启动时，系统在将app转换至前台之前会先将其置为inactive状态。调用app启动时机触发的方法以处理需要的任务。对于处于后台的app，UIKit通过调用以下的一种方法并将app转移到inactive状态：

- For apps that support scenes—The [`sceneWillEnterForeground(_:)`](apple-reference-documentation://hseYUNYT8l) method of the appropriate scene delegate object. 
- For all other apps—The [`applicationWillEnterForeground(_:)`](apple-reference-documentation://hsHg58nGAc) method.

当从后台转移至前台时，使用这些方法从磁盘加载资源并从网络上取得数据。

对于如何在启动时准备好你的app，参见 [Responding to the Launch of Your App](apple-reference-documentation://ts2922731).

#### Configure Your User Interface and Initial Tasks at Activation

系统在展示app UI之前会立即将app转移至active状态。激活期(activation)可以配置你的app UI以及运行行为，包括：

- 展示app的窗口，如果需要的话
- 更改当前可见的视图控制器，如果需要的话
- 更新数据的值以及视图、控制的状态
- 展示控制以继续一个暂停的游戏
- 开始或继续任意的用来执行任务的分配队列
- 更新数据源对象
- **开启定时器**

将你的配置代码放入以下方法中的一种：

- For a scene-based UI—The [`sceneDidBecomeActive(_:)`](apple-reference-documentation://hs4Kv221iP) method of the appropriate scene delegate object. 
- For all other apps—The [`applicationDidBecomeActive(_:)`](apple-reference-documentation://hs6Kg5eqZX) method of your app delegate object. 

激活期也可以用于在向用户展示UI之前终结UI的触控（？）。不要在激活期方法内执行可能阻碍激活的代码。提前确保app拥有所需要的东西。**例如，如果在app外数据会经常改变，在app转入前台之前使用后台任务从网络获得对应的更新，否则你就要在更新UI时异步获得这些改变。**

#### Start UI-Specific Tasks when Your View Appears

当激活期的方法返回之后，UIKit会可视化你建立的窗口。它也会通知相关的视图控制器这些视图即将出现。使用视图控制器的[`viewWillAppear(_:)`](apple-reference-documentation://hs3rt_SgrT)方法以执行对UI的最终更新，例如：

- 开始用户界面动画
- 如果可以自动播放，开始播放视频文件
- 以全帧率开始展示游戏的图像以及沉浸式内容

不要在上面的方法里尝试展示一个不同的视图控制器或者向用户界面制造大规模的变化。当视图控制器出现在屏幕上时，用户界面应当已经准备好进行显示了。



### Preparing Your UI to Run in the Background

#### Overview

app可能由于许多原因而转移至后台：当用户离开一个前台的app，在UIKit挂起这个app之前其会暂时进入background状态；系统也有可能直接将app启动在后台或将一个被挂起的app转移至后台并给它时间执行重要的任务。

当app位于后台时，其应当处理尽可能少的任务，最好什么任务也不处理。如果app曾经处于前台，使用后台转换以停止相关的任务并释放任何共享的资源。如果app进入后台处理一个重要的事情，其应当尽快对其进行处理并离开。

所有的状态转移会使UIKit向合适的委托对象发送通知：

- In iOS 13 and later—A [`UISceneDelegate`](apple-reference-documentation://hsd-omBXXP) object.
- In iOS 12 and earlier—The [`UIApplicationDelegate`](apple-reference-documentation://hsIWiQNXtw) object.

你可以同时支持两种委托对象，但是UIKit只会使用适用的委托对象。UIKit只会通知正在进入后台的界面相关连的界面委托。

#### Quiet Your App upon Deactivation

系统会因为许多原因使app失活(deactivate)：当用户离开前台时，系统在将app转移至后台之前会立即使app失活；当系统需要临时中断app时也会使app失活，例如显示系统的警告，当用户解除弹窗时，系统会重新激活app。

在失活时，UIKit会调用下列app方法中的一种：

- For apps that support scenes—The [`sceneWillResignActive(_:)`](apple-reference-documentation://hspiva1DVT) method of the appropriate scene delegate object. 
- For all other apps—The [`applicationWillResignActive(_:)`](apple-reference-documentation://hsjPUcq-Zh) method of the app delegate object.

在失活期保存用户的数据并通过暂停主要工作将你的app置入quiet状态，具体的：

- 存储用户数据、关闭打开的文件
- 暂停分配、操作队列
- 不要调度任何需要执行的任务
- **解除正在运行的计时器**
- 自动暂停游戏运行
- 不要提交任何即将被处理的Metal工作（？）
- 不要提交任何新的OpenGL命令

#### Release Resources upon Entering the Background

当app转移至后台时，释放app的内存以及相关的共享资源。对于一个从前台转移至后台的app，释放内存是十分重要的。前台比内存、其他系统资源都拥有更高的优先级，系统会通过终结后台app使得前台的app有充足的资源去运行。即使你的app不在前台，你也应当进行检查，确保后台的app占有尽可能少的资源。

进入后台时，UIKit会调用以下方法的一种：

- For apps that support scenes—The [`sceneDidEnterBackground(_:)`](apple-reference-documentation://hsH3bUC2WJ) method of the appropriate scene delegate object. 
- For all other apps—The [`applicationDidEnterBackground(_:)`](apple-reference-documentation://hs3L5RGJXG) method of the app delegate object.

在后台转换时，执行以下的任务：

- 弃置直接从文件读取的图像或媒体文件
- 弃置可以从硬盘重新创造或重新加载的位于内存中的大型对象
- 释放照相机资源以及其他共享硬件资源的访问
- 隐藏app用户界面的敏感信息，如密码
- 解除警告和一些其他临时的界面
- 关闭与系统数据库的连接
- 解除与Bonjour服务的注册以及关闭其他与这些服务相关的listening socket
- 确保Metal命令缓冲区已经被调度，参见[Preparing Your Metal App to Run in the Background](apple-reference-documentation://ts3542444).
- 确保先前提交的OpenGL命令已经结束

你不需要弃置从app的asset catalog中加载的命名图像。类似的，你不需要释放遵循[`NSDiscardableContent`](apple-reference-documentation://hsnVx1bA-q)协议的对象或[`NSCache`](apple-reference-documentation://hs3dlYnTwl)对象。系统会对这些对象进行自动清理。

确保当app转移至后台时其不占有任意的系统共享资源。如果app在转移至后台后持续访问这些资源（如照相机、共享系统数据库），系统会终结你的app并释放这些资源。如果使用系统框架去访问一种资源，请检查对应框架的文档以获得更多的信息。

#### Prepare Your UI for the App Snapshot

在app进入后台并且委托的方法返回了，UIKit会对当前用户界面进行一次快照。系统会将这个图案显示在app转换器中（那个双击home键或者底端上划出现的东西），当app重新回到前台时，这个图案也会被短暂地显示。

app UI一定不能包含任何敏感的用户信息，比如密码、信用卡号等。如果界面包含这些信息的话，应当在app进入后台时移除。并且，解除警告、临时界面以及系统视图控制器以遮挡app当中的内容。快照展示了app的界面并且应该对用户时=是可辨别的。当app返回至前台时，你可以适当地恢复这些数据和视图。

> Note
>
> 对于支持状态保存和恢复的app，系统在委托的方法返回时之前立即开始保存过程。移除这些敏感的数据也会防止这些信息被保存在app的保存档案之中。更多请参见[Preserving Your App's UI Across Launches](apple-reference-documentation://ts2928651).

#### Respond to Important Events in the Background

app在进入后台时通常不会获得额外的执行时间。然而，UIKit可以为支持以下时间敏感服务的app提供额外的执行时间

- 通过Airplay的语音交流或Picture in Picture video
- 定位服务
- Voice over IP（？）
- 通过外部配件的交流
- 通过蓝牙配件的交流
- 来自服务器的常规更新
- 苹果推送通知的支持 

如果app需要支持这些后台特性，在Xcode的Background Modes capability开启。每一个后台任务拥有不同的需求；参见合适的框架以获得更多信息。



### Updating Your App with Background App Refresh

**根据系统所给的机会在后台获取内容并更新app界面**（可能用不到app中）

#### Overview

后台app刷新（Background App Refresh）可以让你的app在后台周期运行从而对其内容进行更新。新闻类、社交媒体类app可以使用这个方法使得app的内容可以实时更新。在app需求之前在后台下载数据能够最小化用户启动app并显示数据所需要的加载时间。

为了支持后台app刷新，做以下几步：

1. Enable the background fetch capability in your app ().
2. Call the [`setMinimumBackgroundFetchInterval(_:)`](apple-reference-documentation://hstBdC_FfY) method of [`UIApplication`](apple-reference-documentation://hsZC8VWkhJ) at launch time.
3. Implement the [`application(_:performFetchWithCompletionHandler:)`](apple-reference-documentation://hsygyIaZ9p) method in your app delegate.

当系统调用app委托的[`application(_:performFetchWithCompletionHandler:)`](apple-reference-documentation://hsygyIaZ9p)方法，配置一个[`URLSession`](apple-reference-documentation://hsCh_qyx_t)对象去下载新数据。系统会等到网络以及电源状态良好的时候才会开始下载，因此你应当迅速地取回充足数量的数据。当你不想要更新app时，调用完成处理器并提供一个可能包含无可行数据的结果。

> Important
>
> 请及时调用完成处理器并返回一个准确的结果，这可以帮助系统决定在未来你的app可以获得多少执行时间。如果app的更新时间过长，系统会更不频繁地调度你的app以节约电量

下面的代码展示了如何请求并处理后台更新。Xcode项目可以允许background fetch并且app可以在启动时每小时请求更新。当app获得了执行时间，它会检查是否有新的数据，若有则将数据取出。

```swift
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions:
                 [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
   // Override point for customization after application launch.
        
   // Fetch data once an hour.
   UIApplication.shared.setMinimumBackgroundFetchInterval(3600)

   // Other initialization…
   return true
}
    
func application(_ application: UIApplication, 
                 performFetchWithCompletionHandler completionHandler:
                 @escaping (UIBackgroundFetchResult) -> Void) {
   // Check for new data. 
   if let newData = fetchUpdates() {
      addDataToFeed(newData: newData)
      completionHandler(.newData)
   }
   completionHandler(.noData)
}
```

如果没有app后台更新会对用户体验造成很大的影响的话，你可以检查`UIApplication`的[`backgroundRefreshStatus`](apple-reference-documentation://hso0JSKeg4)属性去判断这个特性是否可行。用户可以在设置里关闭后台更新。如果你已经提示用户去开启这个特性，尊重用户的选择并不要提示第二次。



### Extending Your App's Background Execution Time

**当app移动至后台时确保重要任务的完成（十分重要！！！！）**

#### Overview

为你的app延长后台执行时间可以确保你有充足的时间去执行重要的任务。对于需要更多时间的任务请参见[BackgroundTasks](apple-reference-documentation://csbackgroundtasks)

当你的app移动至后台时，系统会调用app委托的[`applicationDidEnterBackground(_:)`](apple-reference-documentation://hs3L5RGJXG)方法，这个方法拥有五秒钟的执行时间，然后返回。当这个方法返回之后，系统会将app置入suspended状态。对于大多数app，5秒足够执行重要的任务，但如果你需要更多的时间，你可以请求UIKit去延长app的运行时间。

你可以通过调用[`beginBackgroundTask(withName:expirationHandler:)`](apple-reference-documentation://hsBnf57zab)方法延长app的运行时间。调用这个方法可以给你额外的时间去执行重要的任务（你可以通过[`backgroundTimeRemaining`](apple-reference-documentation://hsRQD2xNXO)属性查询最大可行的后台时间）。当你结束任务时，调用[`endBackgroundTask(_:)`](apple-reference-documentation://hsTqIAib9T)方法从而通知系统。如果你没有及时调用这个方法，系统会终结你的app。

> Note
>
> Don’t wait until your app moves to the background to call the [`beginBackgroundTask(withName:expirationHandler:)`](apple-reference-documentation://hsBnf57zab) method. Call the method before performing any long-running task.

下面的代码展示了如何配置后台任务从而app可以花费超过5秒的时间将自己的数据存储到服务器中。`beginBackgroundTask(withName:expirationHandler:)`方法会返回一个标识符，这个标识符需要被传递给[`endBackgroundTask(_:)`](apple-reference-documentation://hsTqIAib9T)方法

```swift
func sendDataToServer( data : NSData ) {
   // Perform the task on a background queue.
   DispatchQueue.global().async {
      // Request the task assertion and save the ID.
      self.backgroundTaskID = UIApplication.shared.
                 beginBackgroundTask (withName: "Finish Network Tasks") {
         // End the task if time expires.
         UIApplication.shared.endBackgroundTask(self.backgroundTaskID!)
         self.backgroundTaskID = UIBackgroundTaskInvalid
      }
            
      // Send the data synchronously.
      self.sendAppDataToServer( data: data)
            
      // End the task assertion.
      UIApplication.shared.endBackgroundTask(self.backgroundTaskID!)
      self.backgroundTaskID = UIBackgroundTaskInvalid
   }
}
```

> Note
>
> The `beginBackgroundTask(withName:expirationHandler:)` method cannot be called from an app extension. To request extra execution time from your app extension, call the [`performExpiringActivity(withReason:using:)`](apple-reference-documentation://hsPaZwtMoH) method of `ProcessInfo` instead.
>



### About the Background Execution Sequence

学习当你的app移动至后台时按照什么顺序执行你的自定义代码

#### Overview

app可能通过多个不同的初始点之一进入后台。系统事件可能导致一个被挂起的app返回后台或者一个没有运行的app被直接运行于后台。当另一个app启动了或者当用户按了Home键，一个前台app会转换至后台。

![](https://docs-assets.developer.apple.com/published/0896abf42f/a668c5de-d033-423e-8aea-6100f72c3378.png)

#### Handle Background Events

对于支持Background Modes的app，系统会将app启动或保持在后台以处理与这些模式有关的事件。例如，系统会启动或保持app以响应地理位置的更新或者执行后台消息获取。

如果当事件出现时app并不在运行，系统会启动app并将其移动至后台，并遵循以下的顺序：

1. The system launches the app and follows the initialization sequence described in [About the App Launch Sequence](apple-reference-documentation://ts2922733).
2. UIKit calls the app delegate's [`applicationDidEnterBackground(_:)`](apple-reference-documentation://hs3L5RGJXG) method.
3. UIKit delivers the event that caused the launch.
4. The app's snapshot is taken.
5. The app may be suspended again.

如果你的app位于内存之中并且时间出现时被挂起，系统会在后台继续运行你的app，并遵循以下的顺序：

1. The system resumes the app.
2. UIKit calls the app delegate's [`applicationDidEnterBackground(_:)`](apple-reference-documentation://hs3L5RGJXG) method.
3. UIKit delivers the event that caused the launch. 
4. The app's snapshot is taken.
5. The app may be suspended again.

#### Transition from the Foreground

当另一个app启动了或者用户回到了手机主页面，前台app会被移动至后台，并遵循以下的顺序：

1. The user exits the running app.
2. UIKit calls the app delegate's [`applicationWillResignActive(_:)`](apple-reference-documentation://hsjPUcq-Zh) method. 
3. UIKit calls the app delegate's [`applicationDidEnterBackground(_:)`](apple-reference-documentation://hs3L5RGJXG) method.
4. The app's snapshot is taken.
5. The app may be suspended again.





