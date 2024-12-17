# Design System 设计系统文档

## 概述

Design System 一系列预设好的设计标准，包括间距、圆角、排版、下划线、设置选项、阴影效果和动画时间等设计元素。

你肯定不想整个 App 里面魔法数字满天飞吧？重构会非常的痛苦。

在项目初期，就使用预设好的 DesignSystem，可以有效的统一 UI 风格，保证用户体验。

## 核心组件

```swift
struct DesignSystem {
    enum Spacing { ... }        // 间距定义
    enum CornerRadius { ... }   // 圆角定义
    enum Typography { ... }     // 字体定义
    enum Underline { ... }     // 下划线定义
    enum Settings { ... }      // 通用设置
    enum Shadow { ... }        // 阴影效果
    enum Animation { ... }     // 动画时间
}
```

## 用法：

```swift
// 使用间距
padding(.horizontal, DesignSystem.Spacing.regular)

// 使用圆角
.cornerRadius(DesignSystem.CornerRadius.medium)

// 使用字体
.font(DesignSystem.Typography.buttonText)

// 使用阴影
.shadow(color: DesignSystem.Shadow.medium, radius: DesignSystem.Shadow.mediumRadius)
```
