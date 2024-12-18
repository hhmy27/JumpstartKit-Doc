# 介绍

通知对 App 来说非常重要，大多数 App 都会要求申请通知权限，来发送 App 相关的通知。

这里我会介绍如何用 JumpstartKit 来管理通知

# 使用 JumpstartKit 管理通知

JumpstartKit 中，和通知有关的代码有如下 5 个模块

-   App 通知相关的代码
    `NotificationConfig`
    `NotificationError`
    `NotificationManager`
-   通知音效相关的代码
    `NotificationSoundManager`
    `NotificationSoundType`

## 1. NotificationConfig

通知配置结构体，用于定义单个通知的所有属性。

### 属性

```swift
struct NotificationConfig {
    let id: String               // 通知唯一标识符
    let title: String            // 通知标题
    let body: String             // 通知内容
    let soundType: NotificationSoundType   // 通知声音类型
}

```

SwiftUI 支持三种类型的通知，有时间间隔通知、固定时间通知、位置通知（需要申请权限）

根据你的需要，你可以修改扩展 NotificationConfig，设置对应的通知类型，和具体时间、位置

## 2. NotificationError

通知系统可能出现的错误类型。

```swift
enum NotificationError: Error {
    case permissionDenied    // 未获得通知权限
    case invalidConfig       // 配置无效
    case scheduleFailed      // 调度失败
}
```

根据你的需要在这里扩展

## 3. NotificationManager

通知管理器，负责通知的核心功能。

### 关键方法

```swift
class NotificationManager: ObservableObject {
    static let shared: NotificationManager    // 单例访问点

    // 请求通知权限
    func requestPermission() async -> Bool

    // 验证自定义声音文件
    func verifyCustomSound() -> Bool

    // 调度通知
    func scheduleNotifications(config: NotificationConfig) async throws

    // 移除指定前缀的通知
    // 某些行为可能要取消特定的通知
    func removeNotifications(identifierPrefix: String) async
}
```

## 4. NotificationSoundManager

通知声音管理器，负责声音相关功能。

### 关键方法

```swift
class NotificationSoundManager: ObservableObject {
    static let shared: NotificationSoundManager    // 单例访问点

    // 当前选中的声音类型
    @Published var selectedSound: NotificationSoundType

    // 播放声音预览，可以给用户展示特定的声音
    func playPreviewSound(_ sound: NotificationSoundType)
}
```

## 5. NotificationSoundType

通知声音类型枚举。

```swift
enum NotificationSoundType: String, CaseIterable {
    case Demo

    var displayName: LocalizedStringKey    // 本地化显示名称
}
```

在这里面添加你的通知音效

## 使用方法

### 基本设置

```swift
// 在合适的时机请求通知权限
// 比如说onboarding页面中
let hasPermission = await NotificationManager.shared.requestPermission(
```

### 创建和调度通知

```swift
// 创建通知配置
let config = NotificationConfig(
    id: "reminder_123", // 根据业务决定
    title: "你的App名字", // 根据业务决定
    body: "该喝水了", // 根据业务决定
    soundType: .arpeggio, // 自定义音效
    times: [.init(hour: 9, minute: 0)] // 根据你的需求，决定通知时间，可以修改Config扩展
)

// 调度通知
try await NotificationManager.shared.scheduleNotifications(config: config)

```

### 音效管理

```swift
// 播放声音预览
NotificationSoundManager.shared.playPreviewSound(.arpeggio)

// 更改选中的声音
NotificationSoundManager.shared.selectedSound = .magic
```

## 上传音效

Xcode 需要的音效文件是 m4r 格式，直接添加到 Xcode 项目里面即可，不需要特殊的配置

推荐一个音效相关的网站：
https://notificationsounds.com/

可以直接下载 m4r 的资源，免费
