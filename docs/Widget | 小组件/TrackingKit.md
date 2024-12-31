# 介绍

Widget Kit 和 Core Data 的联动，其实有一个很大的问题

就是 Widget 完全是由系统调度的，何时刷新，何时获取数据，全都是由系统决定的

而 Widget 提供的`WidgetCenter.shared.reloadAllTimelines()` 这个刷新方法，也是告诉希望，我们希望立即刷新所有时间线，但是是否刷新，由系统决定

那么这就会有延迟问题

比如用户通过小组件计数器进行打卡，进入 App 后，发现 App 内的计数器没有更新

或者反过来在 App 打卡，小组件没有更新

这都是非常糟糕的体验

而看到这篇文章的你非常幸运，我被这个问题折磨了整整一周，我都没有找到解决方案，直到后面我发现了一个 Kit 能够完美解决这个问题

所以如果你考虑添加交互式小组件，务必看完这批文章

# PersistentHistoryTrackingKit

https://github.com/fatbobman/PersistentHistoryTrackingKit

赞美开发者，提供了这样一款工具

如果你想知道原理，作者的文章提供了详细的解释：

https://fatbobman.com/zh/posts/persistent-history-tracking-in-swiftdata/

让我用一句话总结，TrackingKit 能够让 App 和小组件互相监听数据的变化，当数据变化的时候，立即更新数据

## 配置 TrackingKit

首先你要到导入 TrackingKit 到项目中，如何导包看官方文档就好：

https://developer.apple.com/cn/documentation/xcode/adding_package_dependencies_to_your_app/

然后在 PersistenceController 启动的时候进行配置

```swift
import CoreData
import PersistentHistoryTrackingKit

struct PersistenceController {
    static let shared = PersistenceController()
    // 1. 声明historyTrackingKit属性
    var historyTrackingKit: PersistentHistoryTrackingKit?

    let container: NSPersistentCloudKitContainer

    init() {
        container = NSPersistentCloudKitContainer(name: "YourApp")

        // 2. 配置持久化存储描述
        let desc = container.persistentStoreDescriptions.first!

        // 3. 开启持久化历史跟踪
        desc.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
        desc.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)

        // 4. 设置当前作者标识
        let currentAuthor = getCurrentAuthor()
        container.viewContext.transactionAuthor = currentAuthor

        // 5. 初始化UserDefaults用于存储时间戳
        let userDefaults = UserDefaults(suiteName: "group.YourApp")!

        // 6. 初始化TrackingKit
        self.historyTrackingKit = PersistentHistoryTrackingKit(
            container: self.container,
            currentAuthor: currentAuthor,  // 当前作者标识
            allAuthors: ["App", "Widget"],  // 所有需要同步的作者标识
            userDefaults: userDefaults,  // 用于存储最后同步时间戳
            cleanStrategy: .none,  // 清理策略
            logLevel: 3,  // 日志级别
            autoStart: true  // 是否自动开始监听
        )

        container.loadPersistentStores { (storeDescription, error) in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        }
    }

    // 7. 用于区分是否在Widget环境中
    func isInWidget() -> Bool {
        guard let extesion = Bundle.main.infoDictionary?["NSExtension"] as? [String: String] else {
            return false
        }
        guard let widget = extesion["NSExtensionPointIdentifier"] else { return false }
        return widget == "com.apple.widgetkit-extension"
    }

    // 8. 获取当前环境的作者标识
    func getCurrentAuthor() -> String {
        return isInWidget() ? "Widget" : "App"
    }
}
```

其中`getCurrentAuthor`和 App Group 的配置尤为重要

当你添加小组件之后，你需要通过 getCurrentAuthor 来获取当前环境的作者是 App 还是 Widget

如果你没有做这一步，Tracking 是不会生效的，这也是我一开始踩的坑，最后咨询作者才顺利解决这个问题

然后关于 TrackingKit 的配置：

```swift
    self.historyTrackingKit = PersistentHistoryTrackingKit(
        container: self.container,
        currentAuthor: currentAuthor,  // 当前作者标识
        allAuthors: ["UCan", "UCanWidgetExtension"],  // 所有需要同步的作者标识
        userDefaults: userDefaults,  // 用于存储最后同步时间戳
        cleanStrategy: .none,  // 清理策略
        logLevel: 3,  // 日志级别
        autoStart: true  // 是否自动开始监听
    )
```

其中要注意的是清理策略，这里开启 CloudKit 之后，必须要设置为.none，永远不清理，不然 CloudKit 会删除本地数据：

https://fatbobman.com/zh/posts/persistenthistorytracking/ 作者文章原话

> **CoreData 会自动处理和清除 CloudKit 同步产生的 Transaction，但是如果我们不小心删除了尚没被 CoreData 处理的 CloudKit Transaction，可能会导致数据库同步错误，CoreData 会清空当前的全部数据，尝试从远程重新加载数据。**

所以建议始终都设置为 none 就好了

## 验证

如果你配置正确，那么你可以结合我们之前关于交互式小组件的文章，验证一下 App 更新和 Widget 更新是否同步，如果同步了，那么就说明搞定了，软件的品质再上一个台阶 👏
