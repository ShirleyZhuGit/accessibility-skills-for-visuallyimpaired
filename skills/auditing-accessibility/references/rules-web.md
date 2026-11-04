# Web 基础读屏适配规则库

> 面向语义化 HTML、ARIA 与主流读屏（NVDA / VoiceOver），示例兼顾 React 写法，目标是让核心流程基本可用，不代表完整 WCAG 合规。
> 五大类 18 条规则，按「读什么 → 怎么读 → 状态 → 焦点 → 朗读行为」组织，与 rules-android.md / rules-ios.md 编号一一对应。
> Web 端第一原则：**能用原生元素就不用 ARIA**——`<button>`、`<a>`、`<input>`、`<dialog>`、`<details>` 自带角色、状态和键盘支持。
>
> 优先级定义：
> - **P0**：读屏用户无法完成核心操作（不可达、焦点陷阱、弹窗无法关闭）
> - **P1**：能用但语义缺失（无标签、状态不朗读、纯视觉信息无文字等价物）
> - **P2**：体验优化（碎片合并、符号转写、朗读冲突）

---

## 快速索引（先读这里）

| 类别 | 直接检查 | 高置信度时直接修 |
|---|---|---|
| 名称 | 图标按钮、重复按钮、符号计数、无 label 输入框、img 无 alt | 语义名称带业务对象，走项目的 i18n / 文案体系 |
| 分组 | 一个入口拆成多个 tabbable 元素 | 一个交互单元一个焦点，icon 加 aria-hidden |
| 状态 | 选中、禁用、展开、错误、进度只靠 class 切换 | 用原生元素或对应 aria 属性，不把状态全塞进名称 |
| 可达 | div/span 带 onClick、hover-only 操作、弹窗无关闭入口、outline:none | 换原生 button/a；确不能换才补 role+tabindex+键盘 |
| 反馈 | 操作结果和动态变化只有动画/图标 | role="status" / aria-live / role="alert" |

文案、复杂焦点、自绘节点、音频策略或业务行为不明确时不要猜，留下待办。以下详则仅在命中对应线索时读取。

---

## A. 标签规则 —— 读什么

### A1. 图标按钮必须有语义标签【P1】

只有图标、没有文字的可交互元素，必须提供描述**操作意图**的可访问名称。读屏应读出"删除""收藏""更多功能"，而不是什么都不读或读出 SVG 内容。

**首要原则：用原生元素**。可点击用 `<button>`，跳转用 `<a>`——`div` + `onClick` 天生没有角色、没有键盘支持（这是 D12 的 P0 问题）。

```html
<!-- ❌ 读屏什么都不读，或读出 svg 乱码 -->
<button class="close"><svg>...</svg></button>

<!-- ✅ -->
<button aria-label="关闭"><svg aria-hidden="true">...</svg></button>
```

```jsx
// ✅ React 同理，文案走项目 i18n
<button aria-label={t('close')} onClick={onClose}><CloseIcon aria-hidden /></button>
```

图片用 `alt`：信息图 `alt="描述"`，装饰图 `alt=""`（不要省略 alt 属性）。标签中**不要**包含"按钮""点击"等词——角色由元素语义自动朗读。

**审查线索**：`<button>` / `<a>` 内只有 svg / icon 字体且无 aria-label 和可见文本；`<img>` 无 alt 属性；aria-label 值是"图标""icon"等无意义词。

### A2. 标签带上作用对象【P1】

列表中重复出现的同类操作按钮，可访问名称要拼上上下文对象——读屏用户常用"元素列表"快捷导航，一排"删除 删除 删除"无法区分。

```jsx
// ❌ 每行都读"删除"
<button aria-label="删除">🗑</button>

// ✅ 读"删除 四级核心词汇"
<button aria-label={t('deleteItem', { name: book.name })}>🗑</button>
```

带序号的重复内容（例句、笔记）用"第 x 条"区分。

**审查线索**：列表渲染（map）中操作按钮的 aria-label 是写死的字符串常量。

### A3. 符号、缩写、格式化数字转成自然语言【P2】

读屏对符号的默认朗读往往是灾难：`|` 读"竖线"、`5/1653` 读"5 斜杠 1653"、`1x` 读"1 x"。用 `aria-label` 覆盖视觉文本：

| 屏幕显示 | ❌ 默认朗读 | ✅ 标签转写 |
|---------|-----------|-----------|
| `查询 \| 中英文单词` | 查询 竖线 中英文单词 | 点击查询中英文单词 |
| `31/1674词` | 31 斜杠 1674 词 | 共1674个词，已学31个 |
| `1x` / `0.7x` | 1 x | 1倍速 / 0.7倍速 |

```html
<span aria-label="共1674个词，已学31个">31/1674词</span>
```

另外两条：
- 纯装饰符号（分隔符、装饰 emoji）加 `aria-hidden="true"`。
- 音标（如 `/ˈɪndɪkeɪt/`）不能让读屏逐字符读；加 `aria-hidden` 并保证发音按钮可用。

**审查线索**：文本包含 `/`、`|`、`%`、`x` 后缀、`0/30` 计数格式且无 aria-label。

### A4. 输入框要有编程可关联的标签【P1】

每个输入框必须有**编程可关联**的标签——`<label for>` 或 `aria-label` / `aria-labelledby`。placeholder 不能替代 label（输入后消失、部分读屏不读）。文案要语义化——"输入6位验证码"优于"验证码"。

```html
<!-- ❌ 只有 placeholder -->
<input placeholder="手机号" />

<!-- ✅ -->
<label for="phone">输入中国大陆手机号</label>
<input id="phone" type="tel" autocomplete="tel" />
```

错误提示用 `aria-describedby` 关联 + `aria-invalid="true"`，不要只靠红字。

**审查线索**：`<input>` / `<textarea>` 无关联 label 且无 aria-label；错误提示只靠颜色和位置暗示。

---

## B. 合并规则 —— 一次读一个完整意思

### B5. 碎片元素合并为一个焦点单元【P2】

视觉上是"一个东西"、DOM 里是多个元素的组合，交互时应作为一个单元。**只合并无独立操作的子元素**。典型场景：

- **icon + 下方文字**（入口卡片）：一个 `<a>`/`<button>` 包住 icon（aria-hidden）和文字，而不是两个 tabbable 元素
- **标题 + 数值**（设置项一行）：整行一个交互元素，可访问名称自动拼成"每日新词 10"

```html
<!-- ✅ 一个链接一个焦点，icon 不单独朗读 -->
<a href="/review">
  <svg aria-hidden="true">...</svg>
  <span>语境回顾</span>
</a>
```

注意 Web 与移动端的差异：**不要为了合并把大段正文塞进 aria-label**——正文保持原生 DOM 结构，读屏用户靠标题/段落/landmark 导航；合并只用于原子交互单元。

**审查线索**：相邻的 icon 链接 + 文字链接指向同一目标（Tab 要按两次）；一个入口卡片拆成多个 tabbable 元素。

### B6. 焦点可见且点击目标一致【P0/P2】

焦点可见性必须保留——**禁止全局 `outline: none`** 而不提供 `:focus-visible` 替代样式（键盘用户会完全迷路，P0）。同类按钮的可点击区域应一致，点击目标不小于 24×24 CSS px。

**审查线索**：CSS 里 `outline: none` / `outline: 0` 无配套 `:focus-visible` 样式；同一列表并列按钮 hit area 不一致。

---

## C. 状态规则 —— 把眼睛看到的状态说出来

### C7. 选中态要朗读【P1】

tab、单选项、筛选标签、倍速选项等，选中时读屏要读出"已选定"。颜色高亮对读屏用户不存在。用对属性，各有归属，不把状态塞进名称：

```html
<!-- tab 用 aria-selected -->
<button role="tab" aria-selected="true">今日任务</button>
<!-- 开关型按钮用 aria-pressed -->
<button aria-pressed="true">0.7倍速</button>
<!-- 导航当前页用 aria-current -->
<a aria-current="page" href="/words">单词</a>
<!-- 原生单选/复选自带选中语义，优先用 -->
<input type="radio" checked />
```

**审查线索**：`.active` / `.selected` 类名切换但无对应 aria 属性变化；div 自绘 tab。

### C8. 禁用态要朗读，并说明恢复条件【P1】

原生 `disabled` 会让元素**不可聚焦**——读屏用户可能根本发现不了这个按钮。表单提交按钮建议用 `aria-disabled="true"`（可聚焦、可朗读、代码拦截点击），恢复条件用 `aria-describedby` 关联：

```html
<!-- ✅ 可聚焦的禁用态 + 恢复条件 -->
<button aria-disabled="true" aria-describedby="submit-hint">同意协议并绑定</button>
<p id="submit-hint">请先输入手机号和验证码</p>
```

**审查线索**：只用 CSS 类置灰、点击空操作（无 disabled 也无 aria-disabled）；禁用提交按钮无任何恢复说明。

### C9. 展开/收起态要朗读【P1】

可展开元素用 `aria-expanded`，读屏自动读"已展开/已折叠"。方向箭头（chevron）加 `aria-hidden`。原生方案 `<details>/<summary>` 自带展开语义，优先用。

```html
<button aria-expanded="false" aria-controls="book-list">
  单词书分类 <svg aria-hidden="true">...</svg>
</button>
<div id="book-list" hidden>...</div>
```

**审查线索**：箭头旋转 + 内容 display 切换，但触发按钮无 aria-expanded。

### C10. 业务状态 + 可执行动作一起读【P1】

绑定开关、订阅状态等，把"当前状态"和"点击后会发生什么"一起说清：

```jsx
<button aria-label={bound ? "已绑定微信，换绑或解绑微信" : "尚未绑定微信，绑定微信"}>
  <WechatIcon aria-hidden />
</button>
```

**审查线索**：同一个按钮不同状态下 icon 颜色变了但可访问名称不变。

### C11. 纯视觉信息要有文字等价物【P1】

只靠视觉传达的信息，读屏用户会完全丢失。逐项检查：

- **颜色表达对错**：答错标红 → 用视觉隐藏文本（sr-only）补"选择错误"；对错不能只靠红绿
- **轮播/分页小圆点** → 每个圆点按钮 `aria-label="第1页，共2页"`，当前页加 `aria-current="true"`
- **进度** → 优先 `<progress>` 原生数值语义；自绘用 `role="progressbar"` + `aria-valuenow/min/max`，不要用固定标签覆盖
- **红点角标** → sr-only 文本"有3条新消息"放进按钮内

```html
<!-- 视觉隐藏但读屏可读（项目通常已有 sr-only / visually-hidden 工具类，沿用） -->
<button>消息中心<span class="sr-only">，有3条新消息</span></button>
```

**审查线索**：仅通过 color / background class 区分状态的逻辑分支；badge / indicator 组件无文本等价物。

---

## D. 可达性与焦点规则 —— 能到、能回、顺序对

### D12. 所有可交互元素必须可被焦点选中【P0】

"看得见点不着"（键盘/读屏无法到达）是最高优先级 bug。常见原因：

- `div` / `span` + onClick：无角色、无 tabindex、无键盘事件 → **优先换 `<button>`**；确实不能换时补齐三件套 `role="button"` + `tabindex="0"` + Enter/Space 键盘处理，复用同一业务 handler
- 鼠标悬停才出现的操作（hover 显示的删除按钮）：键盘聚焦时也要显示（`:focus-within`）
- 正数 tabindex 打乱顺序（禁用）；`tabindex="-1"` 误加在交互元素上

反向同样成立：**纯装饰元素不得进入焦点/朗读流**——装饰性 svg / img 加 `aria-hidden="true"`；视觉不可见但残留在 DOM 的内容（收起的抽屉、轮播的隐藏页）用 `hidden` 或 `inert`，防止读屏读到"幽灵内容"。

**审查线索**：可点击的 div / span；hover-only 操作；正数 tabindex；离屏内容未 hidden / inert。

### D13. 页面切换要有上下文，不随意强推焦点【P1】

让用户知道所处页面，同时保持逻辑阅读顺序，不要一律把焦点抢到主按钮：

- 每页有唯一 `<h1>`，标题层级不跳跃；landmark（`<main>` / `<nav>`）完整
- 提供"跳到主内容"链接（skip link）作为页面第一个可聚焦元素
- SPA 路由切换读屏不会自动播报——更新 `document.title`，并把焦点移到新页面的 `<h1>`（`tabindex="-1"` + `.focus()`）或用 live region 播报页面名

```jsx
// ✅ SPA 路由切换后
useEffect(() => { document.title = pageTitle; headingRef.current?.focus(); }, [pathname]);
<h1 tabIndex={-1} ref={headingRef}>学习完成</h1>
```

**审查线索**：SPA 无路由焦点管理、切页后焦点残留在旧位置；无 skip link；页面无 `<h1>`。

### D14. 弹窗焦点管理：能进、能关、能回【P0】

- **优先用原生 `<dialog>` + `showModal()`**：自带焦点圈定、Esc 关闭、背景不可交互
- 自定义弹窗必须做齐四件事：打开时焦点移入弹窗；焦点圈定在弹窗内（focus trap，背景加 `inert`）；`Esc` 可关闭 + 显式关闭按钮（"点遮罩关闭"对键盘/读屏用户不可用）；关闭后焦点回到触发元素
- 容器加 `role="dialog"` + `aria-modal="true"` + `aria-labelledby` 指向标题

```jsx
// ✅ 原生方案
<dialog ref={ref} aria-labelledby="dlg-title">
  <h2 id="dlg-title">倍速设置</h2>
  ...
  <button onClick={() => ref.current.close()}>关闭</button>
</dialog>
```

**审查线索**：自定义 modal 无 focus trap / 无 Esc 处理 / 关闭后焦点丢回 body；点遮罩是唯一关闭方式；背景内容未 inert 仍可 Tab 到。

### D15. 遍历顺序符合视觉和逻辑顺序【P1】

Tab 顺序 = DOM 顺序，应与视觉阅读顺序一致。用 CSS（`order`、`row-reverse`、grid 重排）改变视觉顺序会造成"视觉在前、焦点在后"的错位——优先改 DOM 顺序而不是 CSS 重排。禁用正数 tabindex。

**审查线索**：CSS `order` / `row-reverse` 用于交互区域；tabindex 大于 0。

---

## E. 朗读行为规则 —— 别打架

### E16. 避免页面音频与读屏朗读重叠【P2】

自动播放的音频（单词发音、句子朗读）与读屏朗读同时出声会完全混掉：

- 不要为了避免重叠把有意义的正文移出无障碍树；用户仍需重新阅读它
- 优先取消自动播放（本身违反 WCAG 1.4.2）或提供清晰的播放/暂停入口；改变自动播放策略属于产品行为时列为待办
- 双击朗读发音的控件，其可访问名称不要包含单词本身

**审查线索**：`autoplay` 属性 / 路由 effect 里 `audio.play()`；发音按钮的 aria-label 包含单词本身。

### E17. 动态内容变化要主动播报【P1】

不移动焦点的前提下，内容变化（操作结果 toast、计数、加载完成）要让读屏用户听到——用 live region：

```html
<!-- ✅ toast 容器常驻 DOM，内容变化时自动播报 -->
<div role="status">已加入生词本</div>
<!-- 紧急错误用 role="alert" -->
<div role="alert">网络错误，请重试</div>
```

注意：live region 容器必须**先于内容存在于 DOM**，动态插入整个容器可能不播报。

**审查线索**：自定义 toast / snackbar 组件无 role="status" / aria-live；加载态只有 spinner 动画。

### E18. 操作提示交给语义，不塞进标签【P2】

"可点击""按 Enter 执行"这类操作方式提示，读屏基于角色自动追加，不要写进 aria-label。补充说明用 `aria-describedby`；复杂交互（拖拽排序等）提供键盘等价操作并说明用法：

```html
<!-- ✅ 名称与操作说明分离 -->
<button aria-describedby="fav-hint">第1条笔记</button>
<span id="fav-hint" hidden>按 Enter 收藏</span>
```

**审查线索**：aria-label 里出现"点击""可点击""按此按钮"等操作说明文字。

---

## 附：验收清单（键盘 + 读屏实测）

修复完成后建议按此清单人工验证（自动化工具只能覆盖约三成问题）：

1. **纯键盘**走完一条核心用户路径：Tab 顺序合理、焦点始终可见、无焦点陷阱、Esc 能关弹窗
2. 开启读屏（NVDA / VoiceOver）闭眼重走一遍：每个可点元素能聚焦、朗读有意义、角色正确
3. 每个选中/禁用/展开状态切换：朗读跟着变
4. 每个弹窗：能进、能关、能回
5. 音频场景：无双重发音
6. 只靠听力能知道自己在第几页、进度多少、操作是否成功
7. 附加：项目已有 axe / Lighthouse / eslint-plugin-jsx-a11y 时跑一遍清零；不新增依赖
