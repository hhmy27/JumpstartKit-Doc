# 介绍

有些时候，你希望在 App 里面支持多个图标，比如说节日限定、会员专属等，这也是一个卖点。那么这里会教你怎么上传多个图标，然后使用 JumpstartKit 来设置图标切换

# 上传图标

首先图标一定要符合设计规范，要注意的是不能带有透明度通道

## 导入

我推荐你使用 Export Opaque PNG 这个 figma 插件导出没有透明度的图标。我曾以为 figma 自带这个功能，导致我浪费了几个小时，都没有导出成功，希望你不要再踩这个坑。

导出之后，需要你在 Xcode 的 Asset 里面添加 iOS > New iOS App Icon 来添加图标

iOS 18 之后，可以配置 dark、Tinted 这两种图标来适应不同的主题，这里根据你的需要配置。

## 配置

选择你的 Target > App Icons and Launch Screen，这里面有三个选项：

-   App Icon，填写你默认的 Icon 名字
-   App Icons Source，勾选 include all app icon assets，不然无法识别图标
-   Launch Screen File，开屏图片

这又有一个坑，如果你不知道，起码要花三个小时来解决

因为 iOS17+不再支持在 Image 中渲染 App Icon，所以你需要在 Asset 中，选择 New Image Set 来添加每个 icon 对应的 Image Set 资源

> https://forums.developer.apple.com/forums/thread/757162

我一般习惯在 Image Set 的图片后面加上 LOGO，用来区分资源类型

## JumpstartKit

## AppIcon Enum

根据需要编辑`AppIcon`枚举，用来表示支持的 Icon

```swift
enum AppIcon: String, CaseIterable {
		case ...
    var previewImage: String {
        return rawValue + "LOGO"
    }
}
```

接下来我们会在`AppIconViewModel` 中负责处理切换 Icon 的逻辑

## AppIconViewModel

用来管理切换逻辑的

### 属性

```swift
static let shared                      // 单例实例
@Published var currentIcon: AppIcon?   // 当前选中的图标
let availableIcons: [AppIcon]         // 可用图标列表
```

### 方法

```swift
private init()

// 切换应用图标
// Parameters:
//   - icon: 目标图标
func changeAppIcon(to icon: AppIcon)

```

## AppIconPickerView

JumpstartKit 还会给你提供一个图标选择界面的 SwiftUI 视图，用来和 viewModel 配合，完成 Icon 切换工作

### 属性

```swift
@Environment(\.dismiss)          // 环境变量，用于关闭视图
@StateObject private var viewModel   // 图标管理视图模型
columns: [GridItem]             // 网格布局配置

```

使用方法：

```swift
// 展示图标选择器
AppIconPickerView()

// 在视图中调用切换图标
AppIconViewModel.shared.changeAppIcon(to: .yourIcon)

```
