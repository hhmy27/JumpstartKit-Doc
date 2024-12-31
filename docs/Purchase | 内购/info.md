# 介绍

本篇文章会教你如何设置内购，以及如何使用 JumpstartKit 处理内购

代码使用的是 StoreKit 2，这是苹果官方的最新内购框架，少一个第三方依赖，就少一份风险。

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

StoreKit 不会为消耗品保存购买凭证，只会为非消耗品保存购买凭证。

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

订阅：月度会员、年度会员

为你的月度订阅，提供 3 天免费试用，为年度订阅，提供 7 天免费试用

这也是大多数 App 的常见用法

## 价格方案

月度会员假设是$1，那么年度会员应该是$12 的 50%～ 80%折扣，而终身会员，一般都是 1.5 倍到 2 倍的价格。

比如说某个 App：

-   月度会员 $1
-   年度会员 $6
-   终身会员 $12

这个价格就比较合理

或者你可以将月度会员作为锚点，引导用户购买年度会员、终身会员：

-   周度/月度会员 $9.9
-   年度会员 $100

像这种情况会把周度/月度会员设置得很贵，但是提供试用，真正引导用户购买的是年度会员

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

如果你是第一次提交，那么内购和 App 是同时审核，之后的审核都是分开的

所以在你的 App 通过审核之后，后续修改内购，可能会出现这种情况：App 改好代码，上线了，但是内购还没有审核通过，导致无法购买，一定要确认内购审核通过。

但是在测试的时候，就是我们还没有提交的时候，我们是可以通过沙盒环境获取到你所有的内购配置的，方便我们进行测试

## 网络并不稳定，需要兜底

只要涉及到网络，就没有可靠的

我早期完全依赖苹果服务器返回的验证，判断用户是否是会员

结果有小部分用户，告诉我，已经购买了会员，但是进入 App 还是显示非会员

为此，我在 JumpstartKit 中增加了兜底策略，避免这个问题。

当用户购买了会员后，我们会在本地存储会员状态，只有过期后，才会重新请求苹果服务器，判断是否是会员

这样有效避免验证失败的问题

## 试用策略，只能享受一次

如果用户享受了一次试用，但是你第二次仍然展示可以试用，属于错误引导，应该避免这个问题。

JumpstartKit 中也有对应的判断方法，避免这个问题

# 使用 JumpstartKit 处理内购

JumpstartKit 中处理内购和订阅的工具是`IAPManager`

## 注入代码

在 App 启动时注入 IAPManager：

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

1. 创建一个名为 ProductList.plist 的属性列表文件
2. 设置为字典类型
3. 添加你的内购项目标识符，例如：
    - Pro Monthly: com.yourapp.pro.subscription.monthly
    - Pro Annual: com.yourapp.pro.subscription.annual
    - Pro Lifetime: com.yourapp.pro.lifetime

这里会决定给用户展示何种内购项目

## IAPManager 核心功能

### 1. 主要属性

-   `purchasedProducts`: 用户已购买的产品列表
-   `storeProducts`: App Store 中可用的产品列表
-   `isPremiumUser`: 判断用户是否是会员
-   `isUserPurchased`: 用户购买状态（存储在 UserDefaults）

### 2. 产品管理

-   `requestProducts()`:
    -   作用：从 App Store 获取可用产品列表
    -   使用：应用启动时自动调用
    -   特点：会自动对产品进行排序

### 3. 购买功能

-   `purchase(_ product: Product)`:
    -   作用：处理产品购买请求
    -   返回：`StoreKit.Transaction?`

### 4. 订阅管理

-   `checkAndUpdateSubscriptionStatus()`:

    -   作用：检查并更新订阅状态
    -   特点：包含 7 天的缓存机制，避免频繁检查

-   `purchaseSuccessSubscriptionStatus()`:

    -   作用：购买成功后更新订阅状态
    -   使用：购买完成后调用

-   `isEligibleForFreeTrial()`:
    -   作用：检查产品是否符合免费试用资格
    -   使用：展示试用选项前调用

### 5. 购买恢复

-   `restorePurchases()`:

    -   作用：恢复用户之前的购买
    -   特点：会验证商品是否在当前 plist 列表中
    -   返回：是否存在有效购买

-   `getPurchaseHistory()`:
    -   作用：获取带有购买类型的完整购买历史
    -   返回：交易记录和购买类型的元组数组

### 6. 状态更新

-   `updateCustomerProductStatus()`:
    -   作用：更新用户已购买产品状态
    -   特点：
        -   自动过滤已退款商品
        -   检查订阅是否过期
        -   处理非消耗品和永久会员

## 使用示例

### 购买商品

```swift
private func performPurchase() {
    guard let product = selectedProduct else { return }
    isLoading = true
    Task {
        do {
            let transaction = try await iapManager.purchase(product)
            if transaction != nil {
                await iapManager.purchaseSuccessSubscriptionStatus()
                // 处理购买成功
            }
        } catch {
            // 处理错误
        }
        isLoading = false
    }
}
```

### 恢复购买

```swift
private func performRestore() {
    isLoading = true
    Task {
        do {
            let hasPurchases = try await iapManager.restorePurchases()
            if hasPurchases {
                await iapManager.purchaseSuccessSubscriptionStatus()
                // 处理恢复成功
            }
        } catch {
            // 处理错误
        }
        isLoading = false
    }
}
```

## 注意事项

1. IAPManager 在初始化时会自动：

    - 从 plist 加载产品列表
    - 开始监听交易更新
    - 请求产品信息
    - 更新用户产品状态

2. 订阅状态检查有缓存机制：

    - 对于已购买用户，7 天内不重复检查
    - 可以减少对 App Store 的请求频率

3. 新增商品时：

    - 必须更新 ProductList.plist
    - 确保商品 ID 在 plist 中存在

4. 测试时可使用 `clearStoredData()` 清除购买状态
