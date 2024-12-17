# 介绍

本篇文章会教你如何设置内购，以及如何使用 JumpstartKit 处理内购

代码使用的是 StoreKit 2，这是苹果官方的最新内购框架

# 内购类型

苹果的内购分为两种

-   App 内购买项目（In App Purchase，简称 IAP）
    -   对应的是一次性付费的物品
-   订阅
    -   对应的是自动续费的物品

## IAP

IAP 又分为消耗品和非消耗品两种类型

消耗品：例如游戏里面的宝石、金币，这些物品往往可以重复购买

非消耗品：例如 App 中终身会员、特定小组件、特定手表表盘，不能重复购买

**那么这里有一个陷阱，新手很容易踩：**

StoreKit 不会为消耗品保存购买凭证，只会为消耗品保存购买凭证。

如果你把终身会员，设置成消耗品，那么当你想要查询用户是否购买过终身会员，此时是查不到的，这对用户来说体验非常糟糕，已经购买了会员，但是却不是会员身份，只能申请退款

因此，务必设置好 IAP 产品的类型。

## 订阅（Auto-Renewable subscriptions）

订阅就很好理解了，比如说月度订阅、年度订阅等，到期了会自动刷新

苹果规定，所有的订阅必须属于某个群组，然后再给这个群组添加订阅选项

那么为 App 添加订阅，步骤就是：

1. 创建订阅群组
2. 在订阅群组中添加订阅选项
3. 设置订阅选项的价格、持续时长

特别的，我们经常会看到订阅提供了 3 天免费试用，7 天免费试用，也是在这里设置的：

1. 点击特定的订阅选项
2. 找到订阅价格，点击+号，选择「创建推介促销优惠」，进行创建
3. 选择上架的国家、时间、类型
4. 选择免费，周期有 3 天、一周、两周到一年等

当你配置完毕后，用户就可以免费试用会员了，是一个很常见的促销政策

# 推荐方案

## 内购配置

IAP（非消耗型）：1 个终身会员

订阅：月度会员+年度会员

为你的月度订阅，提供 3 天免费试用，为年度订阅，提供 7 天免费试用

## 价格方案

月度会员假设是$1，那么年度会员应该是$12 的 50%～ 80%折扣，而终身会员，一般都是 1.5 倍到 2 倍的价格。

比如说某个 App：

月度会员 $1

年度会员 $6

终身会员 $12

这个价格就比较合理

# 在代码中获取内购项目

创建一个 StoreKit Configuration File

随后打开文件，点击左下角同步按钮，你应该能看到所有配置的选项

这个文件的结构是：

**App**

-   Configuration Settings
-   App Policies

In-App Purchases

-   你创建的多个内购项目

Auto-Renewable subscriptions

-   你创建的多个订阅项目

这个文件不用编辑，直接同步就好

# ⚠️  注意事项

## 内购和 App 的审核是分开的

如果你是第一次提交，那么内购和 App 是同时审核

之后的审核都是分开的

所以可能会出现这种情况：App 改好代码，上线了，但是内购还没有审核通过，导致无法购买

一定要确认内购审核通过

但是在测试的时候，就是我们还没有提交的时候，我们是可以通过沙盒环境获取到你所有的内购配置的

## 网络并不稳定

只要涉及到网络，就没有可靠的

我早期完全依赖苹果服务器返回的验证，判断用户是否是会员

结果有小部分用户，告诉我，已经购买了会员，但是进入 App 还是显示非会员

为此，我在 JumpstartKit 中增加了兜底策略，避免这个问题。

当用户购买了会员后，我们会在本地存储会员状态，只有过期后，才会重新请求苹果服务器，判断是否是会员

这样有效避免验证失败的问题

## 试用策略，只能享受一次

如果用户享受了一次试用，但是你第二次仍然展示可以试用，属于错误引导，当用户退款的时候，对你来说是没有优势的，应该避免这个问题。

JumpstartKit 中也有对应的判断方法。

# 使用 JumpstartKit 处理内购

JumpstartKit 中处理内购的是`IAPManager`

## 注入代码

在 App 启动的时候，通过下面的代码就能注入`IAPManager`

```swift
struct YourApp: App {
    @StateObject private var iapManager = IAPManager()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(iapManager)
        }
    }
}
```

## 配置内购项目

创建一个 Property List，就是字典，命名为 ProductList，用来管理你的内购项目

比如说我创建了三个内购项目，KV 分别是

-   Pro Monthly: com.tutorial.yourapp.pro.subscription.monthly
-   Pro Annual: com.tutorial.yourapp.pro.subscription.annual
-   Pro Lifetime: com.tutorial.yourapp.pro.lifetime

## IAPManager 介绍

用于处理用户内购，功能包括：产品查询、购买处理、购买恢复、订阅状态判断、兜底逻辑等

### 1. 产品管理

-   `requestProducts()`:
    -   作用：从 App Store 获取可用产品列表
    -   使用场景：应用启动时或需要刷新产品列表时调用
-   `updateCustomerProductStatus()`:
    -   作用：更新用户已购买产品的状态
    -   使用场景：购买完成后或需要刷新购买状态时调用

### 2. 购买功能

-   `purchase(_ product: Product)`:
    -   作用：发起产品购买
    -   使用场景：用户点击购买按钮时调用
-   `isPurchased(_ product: Product)`:
    -   作用：检查特定产品是否已购买
    -   使用场景：需要验证产品购买状态时调用
-   `hasEverPurchased(_ productId: String)`:
    -   作用：检查指定产品 ID 是否曾经购买过
    -   使用场景：检查历史购买记录时使用

### 3. 订阅管理

-   `checkAndUpdateSubscriptionStatus()`:
    -   作用：检查并更新订阅状态
    -   使用场景：定期检查订阅状态或应用启动时调用
-   `purchaseSuccessSubscriptionStatus()`:
    -   作用：购买成功后更新订阅状态
    -   使用场景：成功完成购买后调用
-   `isEligibleForFreeTrial()`:
    -   作用：检查用户是否符合免费试用资格
    -   使用场景：展示试用选项前调用

### 4. 购买恢复

-   `getPurchaseHistory()`:
    -   作用：获取用户的购买历史记录
    -   使用场景：需要查看历史购买记录时调用
-   `restorePurchases()`:
    -   作用：恢复用户之前的购买
    -   使用场景：用户更换设备或重新安装应用时调用

### 5. 工具方法

-   `checkVerified()`:
    -   作用：验证交易结果的有效性
    -   使用场景：内部使用，验证购买交易
-   `checkPurchaseType()`:
    -   作用：检查购买类型（普通购买、兑换码等）
    -   使用场景：处理不同类型的购买时使用
-   `checkForActivePurchases()`:
    -   作用：检查是否有活跃的购买
    -   使用场景：验证用户权限时使用

### 6. 调试功能

-   `clearStoredData()`:
    -   作用：清除存储的购买数据
    -   使用场景：仅用于测试环境，清除购买状态
