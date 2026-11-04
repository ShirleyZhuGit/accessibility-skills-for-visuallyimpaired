# iOS 基础读屏适配规则库

> 面向 UIKit、SwiftUI 与 VoiceOver，目标是让核心流程基本可用，不代表完整无障碍合规。
> 五大类 18 条规则，按「读什么 → 怎么读 → 状态 → 焦点 → 朗读行为」组织，与 rules-android.md / rules-web.md 编号一一对应。
>
> 优先级定义：
> - **P0**：读屏用户无法完成核心操作（不可达、焦点陷阱、弹窗无法关闭）
> - **P1**：能用但语义缺失（无标签、状态不朗读、纯视觉信息无文字等价物）
> - **P2**：体验优化（碎片合并、符号转写、朗读冲突）

---

## 快速索引（先读这里）

| 类别 | 直接检查 | 高置信度时直接修 |
|---|---|---|
| 名称 | 图标操作、重复按钮、符号计数、输入框 | 语义名称带业务对象，使用 NSLocalizedString / String Catalog |
| 分组 | 一段信息被拆成多个焦点、容器吞掉子操作 | 只合并无独立操作的子元素 |
| 状态 | 选中、禁用、展开、错误、进度只靠视觉 | 优先用 traits（.selected/.notEnabled）等原生语义，不把状态全塞进名称 |
| 可达 | 自定义手势、装饰焦点、弹窗无关闭入口 | 复用同一业务 handler 提供等价操作 |
| 反馈 | 操作结果和动态变化只有动画/图标 | 使用 announcement 通知或原生错误/进度语义 |

文案、复杂焦点、自绘节点、音频策略或业务行为不明确时不要猜，留下待办。以下详则仅在命中对应线索时读取。

---

## A. 标签规则 —— 读什么

### A1. 图标按钮必须有语义标签【P1】

只有图标、没有文字的可交互元素，必须提供描述**操作意图**的 `accessibilityLabel`。VoiceOver 应读出"删除""收藏""更多功能"，而不是图片资源名或"按钮"。标签描述意图而非形态：星形图标读"收藏"不读"星形"。

```swift
// ❌ VoiceOver 读出图片资源名或什么都不读
let closeButton = UIButton()
closeButton.setImage(UIImage(named: "ic_close"), for: .normal)

// ✅
closeButton.accessibilityLabel = NSLocalizedString("accessibility_close", comment: "关闭")
```

```swift
// ❌ SwiftUI：纯图标按钮无标签
Button(action: onClose) { Image(systemName: "xmark") }

// ✅
Button(action: onClose) { Image(systemName: "xmark") }
    .accessibilityLabel(Text("accessibility_close"))
```

注意：标签中**不要**包含"按钮""点击"等词——控件角色由 traits 自动追加朗读（`.button`），写进标签会读成"关闭按钮 按钮"。

**审查线索**：`UIButton` / `UIBarButtonItem` 只 setImage 无 title 且无 accessibilityLabel；SwiftUI 可点击 `Image` 无 `.accessibilityLabel`；`Image(decorative:)` 误用在可交互元素上。

### A2. 标签带上作用对象【P1】

列表中重复出现的同类操作按钮，标签要拼上上下文对象，避免 VoiceOver 用户扫到一排无法区分的"删除"。

```swift
// ❌ 每个 cell 都读"删除"
deleteButton.accessibilityLabel = "删除"

// ✅ 读"删除 四级核心词汇"
deleteButton.accessibilityLabel = String(
    format: NSLocalizedString("accessibility_delete_item", comment: ""), book.name)
```

带序号的重复内容（例句、笔记）用"第 x 条"区分："第2条例句发音"、"第1条笔记"。

**审查线索**：UITableViewCell / SwiftUI List row 里操作按钮的 label 是写死的字符串常量。

### A3. 符号、缩写、格式化数字转成自然语言【P2】

VoiceOver 对符号的默认朗读往往是灾难：`|` 读"竖线"、`5/1653` 读"5 1653"、`1x` 读"1 x"。用 `accessibilityLabel` 转写视觉文本：

| 屏幕显示 | ❌ 默认朗读 | ✅ 标签转写 |
|---------|-----------|-----------|
| `查询 \| 中英文单词` | 查询 竖线 中英文单词 | 点击查询中英文单词 |
| `31/1674词` | 31 1674 词 | 共1674个词，已学31个 |
| `已学1.8%` | 已学 1.8 百分号 | 已学百分之1.8 |
| `1x` / `0.7x` | 1 x | 1倍速 / 0.7倍速 |

另外两条：
- 纯装饰符号（分隔线、装饰 emoji、来源角标）不应朗读——`isAccessibilityElement = false` / `.accessibilityHidden(true)` / `Image(decorative:)`。
- 音标（如 `/ˈɪndɪkeɪt/`）不能让 VoiceOver 逐字符读；音标元素排除出朗读并保证发音按钮可用，或标签设为"单词发音"。

**审查线索**：文本包含 `/`、`|`、`%`、`x` 后缀、`0/30` 计数格式且没有单独设置 accessibilityLabel。

### A4. 输入框要有提示标签和编辑状态【P1】

输入框必须暴露用途；编辑状态和光标位置 `UITextField` / SwiftUI `TextField` 原生支持，不要用自定义绘制输入框丢掉这些语义。placeholder 文案要语义化——"输入6位验证码"优于"验证码"。

```swift
// ✅ placeholder 会被 VoiceOver 朗读；也可显式设 label
phoneField.placeholder = "输入中国大陆手机号"
phoneField.accessibilityLabel = "输入中国大陆手机号"
```

```swift
// ✅ SwiftUI
TextField("输入中国大陆手机号", text: $phone)
```

**审查线索**：`UITextField` / `TextField` 无 placeholder 且无 accessibilityLabel；placeholder 只有名词没有动作提示；自绘输入控件。

---

## B. 合并规则 —— 一次读一个完整意思

### B5. 碎片元素合并为一个焦点单元【P2】

视觉上是"一个东西"、代码里是多个 view 的组合，要合并成一个无障碍焦点，一次读完整个意思。**只合并无独立操作的子元素**——容器内还有独立按钮时不要整体合并（会吞掉子操作）。典型场景：

- **icon + 下方文字**（金刚位入口）：合成一个焦点，读文字标签
- **词性 + 释义**、**标题 + 数值**（设置项一行）：合成一行读"每日新词 10"
- **多段计数**（"今日复习 0/30"）：合成"今日复习0个，共30个"

```swift
// ✅ UIKit：容器成为唯一无障碍元素（子元素均无独立操作时）
containerView.isAccessibilityElement = true
containerView.accessibilityLabel = "每日新词 10"
```

```swift
// ✅ SwiftUI：combine 合并子节点朗读
HStack { Text("每日新词"); Text("10") }
    .accessibilityElement(children: .combine)
// Button / NavigationLink 默认已合并内部内容，无需重复处理
```

**审查线索**：视觉成组的相邻 UILabel / Image 各自可聚焦；一行信息要扫 3 次才能听完；容器 `isAccessibilityElement = true` 但内部还有独立按钮（吞操作）。

### B6. 焦点框范围完整且同类一致【P0/P2】

同类按钮的焦点框（`accessibilityFrame`）应一致：不能列表里一个按钮框住整行、另一个只框住小图标。上下两行属于同一语义的内容应框为一体。自绘内容用 `UIAccessibilityElement` 显式圈定。

**审查线索**：同一列表 cell 中并列按钮的 frame / hit area 不一致；触达区域小于 44×44pt。

---

## C. 状态规则 —— 把眼睛看到的状态说出来

### C7. 选中态要朗读【P1】

tab、单选项、筛选标签、倍速选项等，选中时 VoiceOver 要读出"已选定 + 选项名"。颜色高亮对读屏用户不存在。优先用原生 trait，不把状态塞进标签（避免重复朗读）。

```swift
// ✅ UIKit：.selected trait，VoiceOver 自动读"已选定"
speedButton.accessibilityTraits.insert(.selected)   // 选中时
speedButton.accessibilityTraits.remove(.selected)   // 取消时
```

```swift
// ✅ SwiftUI
Text("1倍速")
    .accessibilityAddTraits(isSelected ? .isSelected : [])
// 或用 Picker / Toggle 等自带选中语义的组件
```

**审查线索**：用颜色/背景变化表示选中，但代码里没有 `.selected` trait 变更；自绘 tab / segmented 控件。

### C8. 禁用态要朗读，并说明恢复条件【P1】

按钮置灰时读"已停用"（`isEnabled = false` 自动加 `.notEnabled` trait），恢复条件写进标签："同意协议并绑定，请先输入手机号和验证码"。

```swift
submitButton.isEnabled = false
submitButton.accessibilityLabel = "同意协议并绑定，请先输入手机号和验证码"
```

**审查线索**：`isEnabled=false` 的提交按钮无附加说明；用 alpha 变灰假装禁用（trait 无变化，读屏完全无感知）。

### C9. 展开/收起态要朗读【P1】

可展开元素读"展开可查看更多 xx"，已展开读"已展开"。方向箭头（chevron）本身不传达任何信息。

```swift
// ✅ 状态用 accessibilityValue，标签保持稳定
sectionHeader.accessibilityLabel = "单词书分类"
sectionHeader.accessibilityValue = expanded ? "已展开" : "已折叠"
```

```swift
// ✅ SwiftUI：DisclosureGroup 自带展开语义；自定义控件用 value
.accessibilityValue(expanded ? "已展开" : "已折叠")
```

**审查线索**：chevron 旋转 + 子视图 isHidden 切换，但标签和 value 固定不变。

### C10. 业务状态 + 可执行动作一起读【P1】

绑定开关、订阅状态等，把"当前状态"和"点击后会发生什么"一起说清："已绑定微信，换绑或解绑微信"、"尚未绑定QQ，绑定QQ"。

**审查线索**：同一个按钮在不同状态下图片变了但 accessibilityLabel 不变（或没有）。

### C11. 纯视觉信息要有文字等价物【P1】

只靠视觉传达的信息，读屏用户会完全丢失。逐项检查：

- **颜色表达对错**：答错标红 → 朗读补"选择错误，单词+释义"；答对标绿 → 读"正确"
- **UIPageControl** → 原生自带"第1页，共2页"；自绘指示器必须补 label + `accessibilityValue`
- **进度** → 优先保留 `UIProgressView` / SwiftUI `ProgressView` 原生数值语义，不要用固定标签覆盖
- **红点角标** → "消息中心，有3条新消息"（数字进标签或 `accessibilityValue`）

**审查线索**：仅通过 `textColor` / `tintColor` 区分状态的逻辑分支；自绘 page indicator / badge 无标签。

---

## D. 可达性与焦点规则 —— 能到、能回、顺序对

### D12. 所有可交互元素必须可被焦点选中【P0】

"看得见点不着"是最高优先级 bug。常见原因：

- `UITapGestureRecognizer` 挂在无语义视图上，`isAccessibilityElement` 未设 true、无 `.button` trait
- Canvas/CALayer 自绘内容没有创建 `UIAccessibilityElement` 虚拟节点
- 父视图 `isAccessibilityElement = true` 吞掉了内部按钮（合并过度）
- SwiftUI 里 `onTapGesture` 挂在无语义的 `Image` / `Rectangle` 上，没有配套 `.accessibilityAddTraits(.isButton)` + label

反向同样成立：**纯装饰元素不得抢焦点**——装饰图设 `isAccessibilityElement = false` / `.accessibilityHidden(true)` / `Image(decorative:)`。

**审查线索**：`UITapGestureRecognizer` / `onTapGesture` 目标视图无 accessibility 配置；装饰性 UIImageView 可聚焦。

### D13. 页面切换要有上下文，不随意强推焦点【P1】

进入新页面后应让用户知道所处页面，同时保持逻辑阅读顺序。不要一律把焦点抢到主按钮，否则会跳过标题和上下文。优先保证页面标题存在且带 `.header` trait，页面切换用 `.screenChanged` 通知（argument 传 nil 让系统按默认顺序定位）；只有明确业务场景需要时才把 argument 指向特定元素。

```swift
titleLabel.accessibilityTraits.insert(.header)
UIAccessibility.post(notification: .screenChanged, argument: nil)
```

**审查线索**：页面无可识别标题 / 无 `.header` trait；代码把 `.screenChanged` argument 一律指向某个按钮；打开页面后焦点跳过必要上下文。

### D14. 弹窗焦点管理：能进、能关、能回【P0】

- 弹窗打开：`accessibilityViewIsModal = true`，让 VoiceOver 无法访问弹窗外内容
- 弹窗必须提供可聚焦的"关闭"途径——视觉交互是"点弹窗外任意处关闭"时，VoiceOver 用户无法执行，必须补显式关闭按钮，并实现两指 Z 字形退出手势：

```swift
// ✅ 支持 VoiceOver 标准退出手势（两指 Z 字）
override func accessibilityPerformEscape() -> Bool {
    dismiss(animated: true)
    return true
}
// 遮罩层也可补关闭语义
scrimView.isAccessibilityElement = true
scrimView.accessibilityLabel = "关闭"
```

- 弹窗关闭：焦点回到触发它的元素（`.screenChanged` argument 指向触发按钮）。原生 `UIAlertController` / SwiftUI `.sheet` 已处理，自定义浮层才需要手工做。

**审查线索**：自定义弹窗/半屏浮层无 `accessibilityViewIsModal`；关闭方式只有"点击外部区域"；未实现 `accessibilityPerformEscape`。

### D15. 遍历顺序符合视觉和逻辑顺序【P1】

VoiceOver 线性遍历应保持意义和操作顺序。优先调整视图结构；必要时用容器的 `accessibilityElements` 数组显式指定顺序：

```swift
containerView.accessibilityElements = [titleLabel, priceLabel, buyButton]
```

```swift
// SwiftUI 必要时
.accessibilitySortPriority(2)  // 数值大的先读
```

**审查线索**：视图添加顺序与视觉顺序差异大；浮层按钮最后 addSubview 但视觉在中间。

---

## E. 朗读行为规则 —— 别打架

### E16. 避免 App 音频与读屏朗读重叠【P2】

App 自带音频（单词发音、句子朗读）与 VoiceOver 朗读同时出声会完全混掉。规则：

- 不要为了避免重叠把有意义的正文移出无障碍树；用户仍需重新阅读它
- 优先取消自动播放或提供清晰的播放/暂停入口；改变自动播放策略属于产品行为时列为待办
- 页面进入时避免"VoiceOver 读页面标题 + App 自动放音频"抢声道
- 双击朗读发音的控件，其标签不要包含单词本身（否则读标签一遍、发音一遍）

**审查线索**：viewDidAppear 自动播放音频且同一内容也在可聚焦文本里；发音按钮的标签包含单词本身。

### E17. 动态内容变化要主动播报【P1】

不移动焦点的前提下，内容变化（消息数、操作结果 toast）要让读屏用户听到：

```swift
// ✅ 一次性播报
UIAccessibility.post(notification: .announcement, argument: "已加入生词本")
// 布局变化伴随焦点调整用 .layoutChanged
```

```swift
// ✅ SwiftUI（iOS 17+）
AccessibilityNotification.Announcement("已加入生词本").post()
```

**审查线索**：自定义 toast / HUD 组件无任何 accessibility 通知；操作成功只有视觉动效反馈。

### E18. 操作提示交给 hint，不塞进标签【P2】

"双击即可执行"这类操作方式提示，VoiceOver 基于 traits 自动追加，不要写进 accessibilityLabel。补充性操作说明用 `accessibilityHint`；自定义手势用 custom actions：

```swift
// ✅ hint 与 label 分离
noteCell.accessibilityLabel = "第1条笔记"
noteCell.accessibilityHint = "双击收藏"

// ✅ 自定义操作，VoiceOver 用户上下滑选择
noteCell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(name: "收藏") { _ in collect(); return true }
]
```

```swift
// ✅ SwiftUI
.accessibilityHint("双击收藏")
.accessibilityAction(named: "收藏") { collect() }
```

**审查线索**：accessibilityLabel 里出现"点击""双击""单指双击即可执行"等操作说明文字。

---

## 附：验收清单（VoiceOver 实测）

修复完成后建议按此清单人工验证（自动化无法覆盖真实朗读体验）：

1. 开启 VoiceOver，闭眼完成一条核心用户路径（注册→首页→核心功能→返回）
2. 每个可点元素：能聚焦、朗读有意义、traits 正确
3. 每个选中/禁用/展开状态切换：朗读跟着变
4. 每个弹窗：能进、能关（含两指 Z 字手势）、能回
5. 音频场景：无双重发音
6. 只靠听力能知道自己在第几页、进度多少、操作是否成功
7. 附加：Xcode Accessibility Inspector 跑一遍 Audit
