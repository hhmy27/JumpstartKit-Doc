# 介绍

本篇文章介绍如何设置多主题，以及如何使用 JumpstartKit 一键切换主题。

你会看到很多 App 支持深色、浅色、跟随系统三种主题设置，有些用户喜欢深色模式，而有些用户会在特定时间切换成深色模式，此时如果你能自动响应这个变化，对用户来说是一个不错的体验。

支持多种主题，能够让你的 App 更人性化。

我把主题分为两种，纯色主题和渐变主题，其中纯色主题可以通过 Xcode 的 Asset Catalog 来设置，而渐变主题，需要一些技巧，我也会在这篇文章中分享

## 纯色主题

**添加颜色**

通过设置 Xcode 中的 Asset，可以自定义颜色，自动兼容浅色、深色两种模式

在 Xcode 中，选择 Asset，右键选择 New Color Set，就能添加一个新的颜色，此时你可以重命名这个颜色，例如`headerBackground`，根据颜色的含义决定。

设置颜色的时候，你会发现，它有两种主题，分别是 Any Appearance，以及 Dark，前者就是浅色的状态，后者是深色的状态，同时你还可以对颜色设置透明度。

我建议遵循这个方法来为你的项目添加颜色，因为它能自动的识别当前颜色主题，你不需要再写判断逻辑了。

**应用颜色**

添加好了颜色之后，在需要用到的时候，使用`Color(”headerBackground”)` 来调用即可。

如果你想设置某个视图的背景颜色，通过下面的代码就能设置了

```swift
.background(Color("headerBackground"))
```

## 渐变主题

**添加颜色**

很遗憾，Asset 中的 Color 并不支持渐变，但是我们可以通过添加多个颜色，来拼接成为渐变颜色

比如说，你可以添加多个渐变颜色，然后通过`extension` 来扩展 LinearGradient，获得一个`backgroundGradient`

```swift
extension LinearGradient {
    static var backgroundGradient: LinearGradient {
        LinearGradient(
            stops: [
                .init(color: Color("GradientTop"), location: 0),
                .init(color: Color("GradientMiddle"), location: 0.51),
                .init(color: Color("GradientBottom"), location: 1)
            ],
            startPoint: .top,
            endPoint: .bottom
        )
    }
}
```

进一步的，你需要一个视图和一个视图修饰符来快速应用这个渐变颜色

```swift
struct BackgroundGradientView: View {...}
```

在需要使用背景颜色的时候，通过这个方法设置

```swift
.background(GradientBackgroundView())
```

## SwiftUI 的主题判断

SwiftUI 可以通过 colorSchema 来判断当前主题

> 文档：https://developer.apple.com/documentation/swiftui/colorscheme

```swift
@Environment(\.colorScheme) private var colorScheme

var body: some View {
    Text(colorScheme == .dark ? "Dark" : "Light")
}
```

你可以根据它来进行主题判断，或者设置成对应的颜色

但是这样管理会很混乱，JumpstartKit 中提供了`ThemeManager`来帮助管理主题

# 使用 JumpstartKit 管理主题

`ThemeMode` 负责管理当前的主题

`ThemeManager` 负责管理切换处理主题的逻辑

## ThemeMode

```swift
enum ThemeMode: String, CaseIterable {
    case light
    case dark
    case system
    ...
}
```

对应三种主题，system 指的是跟随系统

## ThemeManager

```swift
class ThemeManager: ObservableObject {
	@AppStorage("themeMode") var themeMode: ThemeMode = .system
	...
}
```

## 使用方法

首先是注入

```swift
@main
struct YourApp: App {
		// 1. 初始化
    @StateObject private var themeManager = ThemeManager()

    var body: some Scene {
        WindowGroup {
              ContentView()
                  .environmentObject(themeManager) // 2. 注入
                  .preferredColorScheme(themeManager.themeMode.colorScheme) // 3.设置当前的颜色偏好
            }
        }
    }
}
```

其次是在视图中使用

```swift
// 1. 获取
@EnvironmentObject var themeManager: ThemeManager

// 2. 获取当前主题
themeManager.themeMode.colorScheme

// 3. 设置当前主题，例如通过picker来设置
themePicker(
    selection: $themeManager.themeMode,
    options: ThemeMode.allCases,
)
```

通过 themeManager，你可以轻松的管理主题的状态，一键切换主题
