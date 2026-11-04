# Android 基础读屏适配规则库

> 面向 XML Layout、Jetpack Compose 与 TalkBack，目标是让核心流程基本可用，不代表完整无障碍合规。
> 五大类 18 条规则，按「读什么 → 怎么读 → 状态 → 焦点 → 朗读行为」组织。
>
> 优先级定义：
> - **P0**：读屏用户无法完成核心操作（不可达、焦点陷阱、弹窗无法关闭）
> - **P1**：能用但语义缺失（无标签、状态不朗读、纯视觉信息无文字等价物）
> - **P2**：体验优化（碎片合并、符号转写、朗读冲突）

---

## 快速索引（先读这里）

| 类别 | 直接检查 | 高置信度时直接修 |
|---|---|---|
| 名称 | 图标操作、重复按钮、符号计数、输入框 | 语义名称带业务对象，使用字符串资源 |
| 分组 | 一段信息被拆成多个焦点、容器吞掉子操作 | 只合并无独立操作的子元素 |
| 状态 | 选中、禁用、展开、错误、进度只靠视觉 | 使用原生状态/范围语义，不把状态全塞进名称 |
| 可达 | 自定义触摸、装饰焦点、弹窗无关闭入口 | 复用同一业务 handler 提供等价操作 |
| 反馈 | 操作结果和动态变化只有动画/图标 | 使用结构化状态、live region 或错误语义 |

文案、复杂焦点、自绘节点、音频策略或业务行为不明确时不要猜，留下待办。以下详则仅在命中对应线索时读取。

---

## A. 标签规则 —— 读什么

### A1. 图标按钮必须有语义标签【P1】

只有图标、没有文字的可交互元素，必须提供描述**操作意图**的标签。读屏应读出"删除""收藏""更多功能"，而不是"按钮""图片"或什么都不读。标签描述意图而非形态：星形图标读"收藏"不读"星形"，垃圾桶读"删除"不读"垃圾桶图标"。

```xml
<!-- ❌ 读屏只读"按钮"，用户不知道是什么 -->
<ImageButton android:id="@+id/btn_close" android:src="@drawable/ic_close" />

<!-- ✅ -->
<ImageButton android:id="@+id/btn_close" android:src="@drawable/ic_close"
    android:contentDescription="@string/accessibility_close" />
```

```kotlin
// ❌ Compose：Icon 的 contentDescription 传 null 但外层可点击
IconButton(onClick = onClose) { Icon(Icons.Default.Close, contentDescription = null) }

// ✅
IconButton(onClick = onClose) {
    Icon(Icons.Default.Close, contentDescription = stringResource(R.string.accessibility_close))
}
```

注意：标签中**不要**包含"按钮""点击"等词——控件角色（role）由系统自动追加朗读，写进标签会读成"关闭按钮 按钮"。

**审查线索**：`ImageButton` / `ImageView`（有 clickListener）/ `IconButton` 缺 `contentDescription`；`contentDescription` 值是"图标""icon""按钮"等无意义词。

### A2. 标签带上作用对象【P1】

列表中重复出现的同类操作按钮，标签要拼上上下文对象，避免读屏用户扫到一排无法区分的"删除"。

```kotlin
// ❌ 列表里每一项都读"删除"
deleteBtn.contentDescription = "删除"

// ✅ 读"删除 四级核心词汇"
deleteBtn.contentDescription = context.getString(R.string.accessibility_delete_item, book.name)
```

同理："学习 + 书名"、"第 3 条评论，点击收藏"。带序号的重复内容（例句、笔记）用"第 x 条"区分。

**审查线索**：RecyclerView / LazyColumn 的 item 布局里，操作按钮的 contentDescription 是写死的字符串常量。

### A3. 符号、缩写、格式化数字转成自然语言【P2】

读屏引擎对符号的默认朗读往往是灾难：`|` 读"半角垂直线"、`5/1653` 读"5 1653"、`1x` 读"1 x"、`1.8%` 可能读"1.8 百分号"。标签中要用自然语言转写：

| 屏幕显示 | ❌ 默认朗读 | ✅ 标签转写 |
|---------|-----------|-----------|
| `查询 \| 中英文单词` | 查询 半角垂直线 中英文单词 | 点击查询中英文单词 |
| `5/1653词` | 5 1653 词 | 共1653个词，已学5个 |
| `已学1.8%` | 已学 1.8 百分号 | 已学百分之1.8 |
| `1x` / `0.7x` | 1 x | 1倍速 / 0.7倍速 |
| `新学词 0/5` | 新学词 0 5 | 新学词共5个，已学0个 |

另外两条：
- 纯装饰符号（分隔线、装饰 emoji、来源角标）不应朗读——设为对无障碍不可见。
- 音标（如 `/ˈɪndɪkeɪt/`）不能让读屏按字母逐个读；音标元素的标签设为朗读单词发音本身的语义（如"单词发音"），或直接排除出朗读并保证发音按钮可用。

**审查线索**：文本包含 `/`、`|`、`%`、`x` 后缀、`0/30` 计数格式且没有单独设置 contentDescription。

### A4. 输入框要有提示标签和编辑状态【P1】

输入框必须暴露：用途（hint/label）、当前编辑状态、光标位置。placeholder 文案要语义化——"输入6位验证码"优于"验证码"。

```xml
<!-- ✅ hint 会被 TalkBack 朗读 -->
<EditText android:hint="输入中国大陆手机号" android:inputType="phone" />
```

```kotlin
// ✅ Compose 用 label/placeholder，TextField 自带编辑语义
OutlinedTextField(value = phone, onValueChange = { phone = it },
    label = { Text("输入中国大陆手机号") })
```

**审查线索**：`EditText` / `TextField` 无 `hint` / `label`；hint 文案只有名词没有动作提示。

---

## B. 合并规则 —— 一次读一个完整意思

### B5. 碎片元素合并为一个焦点单元【P2】

视觉上是"一个东西"、代码里是多个 View 的组合，要合并成一个无障碍焦点，一次读完整个意思。典型场景：

- **icon + 下方文字**（金刚位入口）：合成一个焦点，读文字标签
- **词性 + 释义**、**标题 + 数值**（设置项一行）：合成一行读"每日新词 10"
- **多段计数**（"今日复习 0/30"）：合成"今日复习0个，共30个"

```xml
<!-- ✅ XML：静态信息组使用读屏聚焦，不改变键盘焦点 -->
<LinearLayout
    android:screenReaderFocusable="true">
    <TextView android:text="@string/daily_new_words" />
    <TextView android:text="10" />
</LinearLayout>
```

```kotlin
// ✅ Compose：mergeDescendants 合并子节点朗读
Row(modifier = Modifier.semantics(mergeDescendants = true) {}) {
    Text("每日新词"); Text("10")
}
// 可点击的 Row（clickable）默认已合并，无需重复处理
```

**审查线索**：视觉成组的相邻 TextView / Icon 各自可聚焦；一行信息要扫 3 次才能听完。

### B6. 焦点框范围完整且同类一致【P0/P2】

同类按钮的可点击区域和焦点框应一致：不能列表里一个按钮框住整行、另一个只框住小图标。焦点框太小还会导致低视力用户看不清当前位置。上下两行属于同一语义的内容（如单词卡的上下两段）应框为一体。

**审查线索**：同一列表 item 中并列按钮的尺寸/padding/touch target 不一致；`minWidth/minHeight` 小于 48dp。

---

## C. 状态规则 —— 把眼睛看到的状态说出来

### C7. 选中态要朗读【P1】

tab、单选项、筛选标签、倍速选项等，选中时读屏要读出"已选定 + 选项名"。颜色高亮对读屏用户不存在。

```xml
<!-- XML：优先用系统 selected 状态，TalkBack 自动读"已选定" -->
view.isSelected = true
<!-- 只有系统状态无法表达自定义状态时才补 stateDescription，避免重复朗读 -->
ViewCompat.setStateDescription(view, customState)
```

```kotlin
// ✅ Compose：selected 语义
Modifier.semantics { selected = isSelected }
// 或用 Tab / FilterChip 等自带 selected 语义的组件
```

**审查线索**：用颜色/背景变化表示选中，但代码里没有 `isSelected` / `stateDescription` / `selected` 语义；自绘 tab 控件。

### C8. 禁用态要朗读，并说明恢复条件【P1】

按钮置灰时，读屏应读出"已停用"，标签最好带上恢复条件："同意协议并绑定，已停用，请先输入手机号和验证码"。

```kotlin
// isEnabled=false 系统会读"已停用"，但恢复条件要写进标签
button.isEnabled = false
button.contentDescription = "同意协议并绑定，请先输入手机号和验证码"
```

**审查线索**：`isEnabled=false` / `enabled = false` 的提交按钮，无附加说明；用 alpha 变灰假装禁用（读屏完全无感知）。

### C9. 展开/收起态要朗读【P1】

可展开元素读"展开可查看更多 xx"，已展开读"已展开，收起 xx"。方向箭头（⬆️⬇️、`>`）本身不传达任何信息。

```kotlin
// ✅ Compose
Modifier.semantics { expand { /* ... */ ; true } }
// 或直接把状态写进标签
arrow.contentDescription = if (expanded) "收起单词书分类" else "展开可查看更多单词书分类"
```

**审查线索**：旋转箭头/chevron 图标控制 visibility 切换，但标签固定不变。

### C10. 业务状态 + 可执行动作一起读【P1】

绑定开关、订阅状态等，把"当前状态"和"点击后会发生什么"一起说清："已绑定微信，换绑或解绑微信"、"尚未绑定QQ，绑定QQ"。

**审查线索**：同一个按钮在不同状态下 icon 变了但 contentDescription 不变（或没有）。

### C11. 纯视觉信息要有文字等价物【P1】

只靠视觉传达的信息，读屏用户会完全丢失。逐项检查：

- **颜色表达对错**：答错标红 → 朗读要补"选择错误，单词+释义"；答对标绿 → 读"正确"
- **页面指示器小圆点** → "第一页，共两页"
- **进度条** → 优先保留原生 `ProgressBar` 范围语义；自定义 Compose 进度使用 `progressBarRangeInfo`，不要用 `contentDescription` 覆盖默认数值
- **红点角标** → "消息中心，有3条新消息"（数字进标签）

**审查线索**：仅通过 `setTextColor` / `tint` 区分状态的逻辑分支；`TabLayout`/`PageIndicator` 无标签；Badge 组件。

---

## D. 可达性与焦点规则 —— 能到、能回、顺序对

### D12. 所有可交互元素必须可被焦点选中【P0】

"看得见点不着"是最高优先级 bug。常见原因：

- 自定义 View 未设 `focusable`，或触摸事件写在 `onTouchEvent` 而没有 `performClick`
- 可点击区域是 Canvas 自绘/图表的一部分，没有虚拟节点（`ExploreByTouchHelper`）
- 元素被无障碍层级遮挡（父容器 `importantForAccessibility="noHideDescendants"` 误伤）
- Compose 里 `pointerInput` 手势没有配对的 `onClick` 语义

反向同样成立：**纯装饰元素不得抢焦点**——装饰图、背景、分隔线设 `importantForAccessibility="no"`（XML）；Compose 的装饰 `Icon`/`Image` 使用 `contentDescription = null`。只有确定要清除整个节点及后代语义时才用 `clearAndSetSemantics {}`。

**审查线索**：`onTouchEvent` / `GestureDetector` / `pointerInput` 无对应 click 语义；装饰性 `ImageView` 有默认焦点。

### D13. 页面切换要有上下文，不随意强推焦点【P1】

进入新页面后应让用户知道所处页面，同时保持逻辑阅读顺序。不要一律把焦点抢到主按钮，否则会跳过标题和上下文。优先使用 Activity 标题、无障碍 heading 或 pane title；只有明确业务场景需要时才单独设计焦点位置。

```kotlin
ViewCompat.setAccessibilityPaneTitle(rootView, getString(R.string.learning_complete))
```

**审查线索**：页面标题无法被识别；代码用 `TYPE_VIEW_FOCUSED` 模拟无障碍焦点；打开页面后焦点跳过必要上下文。

### D14. 弹窗焦点管理：能进、能关、能回【P0】

- 弹窗打开：焦点进入弹窗内部，弹窗外内容不可被读屏访问
- 弹窗必须提供可聚焦的"关闭"途径——视觉交互是"点弹窗外任意处关闭"时，读屏用户无法执行，必须补显式关闭按钮或让遮罩层可聚焦且标签为"关闭"
- 弹窗关闭：焦点回到触发它的元素

```kotlin
// BottomSheet / 自定义弹窗：给遮罩补上关闭语义
scrim.contentDescription = "关闭"
scrim.isFocusable = true
```

**审查线索**：BottomSheetDialog / PopupWindow / 自定义遮罩，关闭方式只有"点击外部区域"；弹窗内无任何 focusable 的关闭控件。

### D15. 遍历顺序符合视觉和逻辑顺序【P1】

读屏线性遍历应保持意义和操作顺序。优先调整布局/语义结构；XML 必要时使用 `accessibilityTraversalAfter/Before`。Compose 必要时先给父容器设置 `isTraversalGroup = true`，再给可被读屏选中的子节点设置 `traversalIndex`。

**审查线索**：ConstraintLayout 中 View 声明顺序与视觉顺序差异大；浮层按钮插在布局末尾但视觉在中间。

---

## E. 朗读行为规则 —— 别打架

### E16. 避免 App 音频与读屏朗读重叠【P2】

App 自带音频（单词发音、句子朗读）与读屏朗读同时出声会完全混掉。规则：

- 不要为了避免重叠把有意义的正文移出无障碍树；用户仍需重新阅读它
- 优先取消自动播放或提供清晰的播放/暂停入口；改变自动播放策略属于产品行为时列为待办
- 播放类页面进入时，避免"系统读页面标题 + App 自动放音频"抢声道
- 双击朗读发音的控件，其标签不要设计成会触发二次朗读的长文本

**审查线索**：页面 onResume 自动播放音频且同一内容也在可聚焦文本里；发音按钮的标签包含单词本身。

### E17. 动态内容变化要主动播报【P1】

不移动焦点的前提下，内容变化（消息数、计时、操作结果 toast）要让读屏用户听到：

```xml
<!-- XML：liveRegion -->
<TextView android:accessibilityLiveRegion="polite" />
```

```kotlin
// Compose
Modifier.semantics { liveRegion = LiveRegionMode.Polite }
// 错误使用 TextView.setError / 节点 error 语义；页面变化使用 paneTitle。
// 不再推荐 Android 16 已废弃的 announceForAccessibility。
```

**审查线索**：Toast/Snackbar 自定义实现；操作成功只有视觉动效反馈。

### E18. 操作提示交给 hint，不塞进标签【P2】

"双击即可执行""双击并按住可拖动"这类操作方式提示，系统会基于控件角色自动追加，不要写进 contentDescription。自定义手势才需要自定义 action：

```kotlin
// ✅ 自定义操作用 AccessibilityAction，TalkBack 会提示"双击执行 收藏"
ViewCompat.addAccessibilityAction(view, "收藏") { _, _ -> collect(); true }
```

```kotlin
// Compose
Modifier.semantics { customActions = listOf(CustomAccessibilityAction("收藏") { collect(); true }) }
```

**审查线索**：contentDescription 里出现"点击""双击""可拼写，单指双击即可执行"等操作说明文字。

---

## 附：验收清单（TalkBack 实测）

修复完成后建议按此清单人工验证（自动化无法覆盖真实朗读体验）：

1. 开启 TalkBack，闭眼完成一条核心用户路径（注册→首页→核心功能→返回）
2. 每个可点元素：能聚焦、朗读有意义、角色正确
3. 每个选中/禁用/展开状态切换：朗读跟着变
4. 每个弹窗：能进、能关、能回
5. 音频场景：无双重发音
6. 只靠听力能知道自己在第几页、进度多少、操作是否成功
