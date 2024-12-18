# 介绍

现在很多 App 在第一次下载，都会有 Onboarding 页面，App 可以在 Onboarding 流程中完成很多工作，比如说介绍产品特点、获取用户信息、请求权限等工作。常见的小技巧也有 Onboarding 完成后立即展示付费墙，而不是 App 本身。

是否需要 Onboarding，根据你自己的喜好来决定，JumpstartKit 提供了一套 Onboarding 流程，帮助你串起多个 Onboarding 页面，你可以在每个页面中完成你想要的工作。

# 使用 JumpstartKit 管理 Onboarding

常见的 Onboarding 是由多个页面组成的，在 JumpstartKit 中，有两个工具负责管理：

-   `OnboardingPageType` 定义 Onboarding 页面，以及发生顺序
-   `OnboardingManager` 用来管理页面之前的调度切换
-   `OnboardingContainerView` 用来管理多个页面的展示

## OnboardingPageType

这是一个枚举，可以在这里定义所有页面、页面 id、下一个页面、前一个页面等操作

一般情况下 Onboarding 页面不会太多，所以我们这里手动组织一下页面关系就好。

```swift
enum OnboardingPageType: Identifiable {
    case welcomePage
    // ...

    var id: String {
        switch self {
        case .welcomePage: return "welcome"
        // ...
        }
    }

    var nextPage: OnboardingPageType? {
        switch self {
        case .welcomePage: return nil // 根据业务决定
        // ...
        }
    }

    var previousPage: OnboardingPageType? {
        switch self {
        case .welcomePage: return nil
        // ...
        }
    }

    var isFirstPage: Bool {
        previousPage == nil
    }

    var isLastPage: Bool {
        nextPage == nil
    }

    var pageIndex: Int {
        switch self {
        case .welcomePage: return 0
        // ...
        }
    }

    static var totalPages: Int {
        1 //
    }

    static func page(at index: Int) -> OnboardingPageType? {
        switch index {
        case 0: return .welcomePage
        default: return nil
        }
    }
}
```

## OnboardingManager

Manger 是用来管理翻页、移动到上一页的动作

```swift
class OnboardingManager: ObservableObject {
    @Published var currentPage: OnboardingPageType
    @AppStorage("hasCompletedOnboarding") private var hasCompletedOnboarding = false

    init(initialPage: OnboardingPageType = .welcomePage) {
        self.currentPage = initialPage
    }

		// 移动到下一页
    func moveToNextPage() {...}

    // 更新到上一页
    func moveToPreviousPage() {...}

    // 获取当前页码和总页数
    var pageNumbers: (current: Int, total: Int) {...}
}

```

## OnboardingContainerView

在这里可以管理多个页面

```swift
struct OnboardingContainerView: View {
    @StateObject private var manager = OnboardingManager()

    var body: some View {
        ZStack {
            switch manager.currentPage {
            case .welcomePage:
                WelcomePageView(manager: manager)
                    .transition(.opacity)
            case .introPage:
                IntroPageView(manager: manager)
                    .transition(.opacity)
            case .featurePage:
                FeaturePageView(manager: manager)
                    .transition(.opacity)
            case .notificationPermissionPage:
                NotificationPermissionPageView(manager: manager)
                    .transition(.opacity)
            }
        }
        .animation(.easeInOut, value: manager.currentPage)
    }
}
```

通过这个方式，可以管理和组织多个页面，并且自动地完成翻页。

## 使用方法

在 ContentView 中添加以下代码，用来控制 Onboarding

```swift
struct ContentView: View {
    @AppStorage("hasCompletedOnboarding") private var hasCompletedOnboarding = false

    var body: some View {
        if !hasCompletedOnboarding {
            OnboardingContainerView()
        } else {
            // 正常的App页面
            MainAppView()
        }
    }
}
```
