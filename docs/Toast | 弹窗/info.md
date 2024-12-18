# 介绍

有些时候，你希望通过弹窗来提醒用户，比如说操作失败，或者引导提示，那么你会需要一个全局的 ToastView 来展示相关信息

# 使用 JumpstartKit 管理弹窗

关于弹窗的代码有：

`ToastManager`

`ToastView`

# ToastManager

全局 Toast 管理器，负责控制 Toast 的显示状态和内容。

包括动画效果、本地化

```swift
// 调用此方法，会控制Toast展示对应message
func showToast(_ message: LocalizedStringKey)
```

# ToastView

一个极简风格的 ToastView，用来提示用户弹窗信息，支持国际化

# 使用方法

配置

```swift
@main
struct YourApp: App {
		// 1. 初始化
    @StateObject private var toastManager = ToastManager()

    var body: some Scene {
        WindowGroup {
            ZStack {
                ContentView()
                    .environmentObject(toastManager) // 2. 注入

								// 3. 在ZStack中添加ToastView
                if toastManager.isPresented {
                    ToastView(
                        message: toastManager.message,
                        isPresented: $toastManager.isPresented
                    )
                }
            }
        }
    }
}
```

在视图中，比如说点击按钮，触发弹窗

```swift
struct NoteView: View {
    @EnvironmentObject var toastManager: ToastManager

    var body: some View {
        Button(action: {
            toastManager.showMessage("Hello, world!")
        }) {
            Text("Show Toast")
        }
    }
}
```
