# 介绍

这篇文章会介绍如何增加多语言支持，并且如何在 App 中切换到对应语言

## 多语言支持

有时候你需要给 App 增加多语言支持，常见的是英文、简体中文、繁体中文、日语等。

那么你需要新建一个 String Catalog

> 官方文档：https://developer.apple.com/documentation/xcode/localizing-and-varying-text-with-a-string-catalog

这是 WWDC23 带来新的国际化方案，它的文件格式是.xcstrings，相比传统的 .strings 和 .stringsdict 文件有很大改进，不要再创建多个.strings 或者.stringsdict 文件了，同时是背后 JSON 格式构建的，适合版本化管理。

想要支持多语言，现在只需要两步：

1. 选择 project > info > Localization，添加对应的语言，比如说简体中文是 Chinese, Simplified
2. 在 String Catalog 中，增加对应语言

启动项目，Xcode 会自动把所有的文案生成到 String Catalog 中，然后在里面进行翻译即可

建议任何时候都以英文作为主语言，方便国际化

## 注意事项

**无法识别翻译文案**

Xcode 只会识别 `Text(”Hello World”)` 这样的文案，把`”Hello World”`作为一个待翻译的文案

如果你的字符串是传入参数，而不是写死的，Xcode 就不会识别，你需要把字符串类型改成 LocalizedStringResource 或者 LocalizedStringKey，Xcode 才能识别出来。

> LocalizedStringResource 是 iOS16 推出的，LocalizedStringKey 是 iOS13 推出的
> 两个官方文档：
> https://developer.apple.com/documentation/foundation/localizedstringresource
> https://developer.apple.com/documentation/swiftui/localizedstringkey

**翻译后仍然显示英文**

这里又有一个坑，有时候你使用 LocalizedStringResource 翻译完毕后，在模拟器中发现并没有翻译过来，还是展示英文，当你确定 String Catalog 含有你翻译的文本，那么你可以使用真机进行测试，或者打包 testflight，在真机上查看是否翻译完毕

# JumpstartKit 切换多语言

如果你需要让用户自行选择语言，那么你就需要这个功能了

JumpstartKit 中提供了`LanguageManager`、`LocalizationModifier`、`Language` 这三个工具来帮助你切换多语言

## LanguageManager

管理语言切换的逻辑

```swift
class LanguageManager: ObservableObject {
    // 属性
    var language: Language              // 当前选择的语言
    var bundle: Bundle                  // 当前使用的语言包

    // 方法
    func updateLocalization()           // 更新本地化设置
}
```

## Language 枚举

用来定义支持的语言列表

```swift
enum Language {
    // 属性
    var displayName: LocalizedStringKey  // 获取当前语言的显示名称
    var localeIdentifier: String        // 获取实际使用的语言代码
}
```

其中有一个比较特殊的枚举值是`system`，表示当前手机系统的语言设置

如果你不确定 localeIdentifier 的只，你可以通过下面的方法获取准确的 id，比如说简体中文，应该是 zh_Hans

```swift
Locale.availableIdentifiers.forEach { identifier in
    let locale = Locale(identifier: identifier)
    let languageName = locale.localizedString(forIdentifier: identifier) ?? "Unknown"
    print("\(identifier) - \(languageName)")
}
```

我一般支持这几种语言：

```swift
    case english = "en"  // 英文
    case simplifiedChinese = "zh_Hans"  // 简体中文
    case traditionalChinese = "zh_Hant"  // 繁体中文
    case russian = "ru"  // 俄语
    case korean = "ko"  // 韩语
    case japanese = "ja"  // 日语
    case system = "system"  // 跟随系统
```

如果你需要切换的话，你可以继续添加对德语、法语、西班牙语等欧洲语言的支持。

## LocalizationModifier

用于实时切换界面语言的 SwiftUI 修饰符

### 使用方法

1. 初始化语言管理器：

```swift
// 在App中初始化
@StateObject private var languageManager = LanguageManager()
```

2. 添加修饰符

```swift
// 在ContentView根视图注入languageManager，同时添加修饰符
ContentView()
	.environmentObject(languageManager)
	.modifier(LocalizationModifier(language: languageManager.language))
```

3. 在视图中，可以通过下面的方法来切换多语言

```swift
// 获取环境变量
@EnvironmentObject private var languageManager: LanguageManager

// 在特定组件中设置当前语言
PickerLanguage($languageManager.language)
```
