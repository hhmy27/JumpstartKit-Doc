# 介绍

在前面的文章中，我们介绍了 Widget 的基础知识、以及 iOS 17+的 Widget 配置方法

本篇文章中，我们会介绍如何添加桌面 Widget 以及锁屏 Widget

## Widget Bundle

还是以计数器作为例子

其实你添加 Widget Target 之后，会自动地生成一个`WidgetBundle`

它是类似这样的形式

```swift
@main
struct CounterWidgetBundle: WidgetBundle {
    var body: some Widget {
	    interactiveCounterWidget()
	    todayCounterWidget()
    }
```

当你填写 Widget 后，系统就会自动的添加到 Widget 选择列表里面，让用户进行添加

## Widget 信息配置

配置是在 Widget 里面的

如果你想指定某个 Widget 为锁屏小组件，只需要这个声明：

`.supportedFamilies([.accessoryCircular, .accessoryRectangular, .accessoryInline])`

这里支持的是三种类型的 Widget，分别是圆形，矩形和标题行三个类型

![image.png](https://i0.wp.com/swiftsenpai.com/wp-content/uploads/2022/12/create-lock-screen-widget-families.png?w=722&ssl=1)

图片来源：https://swiftsenpai.com/development/create-lock-screen-widget/

```swift
@main
struct CounterWidget: Widget {
    var body: some WidgetConfiguration {
        IntentConfiguration(
            kind: "com.yourapp.counterwidget",
            intent: CounterWidgetConfigurationIntent.self,
            provider: CounterWidgetProvider()
        ) { entry in
            CounterWidgetEntryView(entry: entry)
        }
        // 支持哪些系列的Widget
        .configurationDisplayName("Counter Widget")
        .description("Display your counter on home screen or lock screen")
        // 配置主屏幕Widget支持的尺寸
        .supportedFamilies([.systemSmall, .systemMedium])
        // 配置锁屏Widget支持的位置
        .supportedFamilies([.accessoryCircular, .accessoryRectangular, .accessoryInline])
    }
}
```

上面的代码中，表示 CounterWidget，同时支持了桌面小组件和锁屏小组件

如果你的 Widget 比较简单，那么你可以同时支持多个类型的 Widget，在 Widget View 中通过当前的 widget 类型来展示不同的视图：

```swift
@Environment(\.widgetFamily) var family

    var body: some View {
        Group {
            switch family {
            case .accessoryRectangular:
                // 矩形锁屏小组件
                accessoryRectangularView
            case .accessoryInline:
                // 单行锁屏小组件
                accessoryInlineView
            default:
                // 主屏幕小组件
                homeScreenWidget
            }
        }
    }
```

如果你的 Widget 很复杂，那么可以单独实现，不需要混在一起写。
