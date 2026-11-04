---
name: auditing-accessibility
description: |
  审查或修复存量 Android（XML Layout / Jetpack Compose）、iOS（UIKit / SwiftUI）或 Web（HTML / React / Vue）项目的基础读屏无障碍问题。
  当用户明确要求无障碍适配、读屏修复、TalkBack / VoiceOver 检查或 a11y audit 时触发；不在正常 UI 开发过程中自动触发。
  触发词：无障碍适配、读屏修复、TalkBack、VoiceOver、a11y audit、accessibility
  Audit or fix basic screen-reader accessibility issues in existing Android (XML Layout / Jetpack Compose), iOS (UIKit / SwiftUI) or Web (HTML / React / Vue) projects.
  Use when the user explicitly asks for accessibility support, screen-reader fixes, a TalkBack / VoiceOver check or an a11y audit; do not trigger automatically during normal UI development.
  Trigger words: accessibility, a11y audit, screen reader, TalkBack, VoiceOver, NVDA, accessible labels
---

# 基础读屏适配（Android / iOS / Web）

目标是让读屏（TalkBack / VoiceOver / NVDA）用户能够理解并完成核心操作，不代表完整无障碍合规。面向个人开发者：不安装新工具，不先输出长计划，不要求用户理解无障碍 API。

## 先判断用户意图

- 用户说“检查、审查、看看有什么问题”：只读扫描，输出问题，不修改文件。
- 用户说“适配、修复、直接改、帮我做好”：直接修改高置信度问题，不逐项等待确认。
- 当前工作区能确定项目与范围时直接开始；只有存在多个无关项目且无法判断目标时才提一个问题。

## 执行

1. 自动识别目标模块及平台技术栈（Android XML / Compose、iOS UIKit / SwiftUI、Web 框架）；检查现有改动，避免覆盖无关内容。
2. 按平台读对应规则文件顶部“快速索引”——[rules-android.md](references/rules-android.md) / [rules-ios.md](references/rules-ios.md) / [rules-web.md](references/rules-web.md)（18 条规则三份编号一致）；发现对应问题时再读取该条规则，不必全文加载。
3. 优先搜索图标操作、列表 item、选中/禁用/展开状态、颜色信息、自定义触摸、弹窗和动态反馈。
4. 直接修复能从事件处理、资源名、相邻文本和数据模型确定意图的问题，并沿用项目的字符串资源与组件模式。
5. 以下问题不猜：不明图标文案、复杂自绘节点、焦点策略、视图层级重构、自动播放策略、可能改变业务流程的修改。列入“未自动修改”，最多 5 条。
6. 不新增 Lint、Espresso、Accessibility Test Framework、axe、eslint 插件或其他依赖。项目已有明显的最小构建/测试/lint 命令时运行；否则只做静态复查并如实说明。

## 输出契约

保持简短：

```text
已完成：修复 N 处（标签 N / 状态 N / 可达性 N / 反馈 N）
验证：<运行的现有命令及结果，或“未发现可直接运行的命令”>
未自动修改：<最多 5 条；没有则写“无”>
读屏冒烟：核心路径能走通 / 控件读得懂 / 状态会变化 / 弹窗能关闭 / 操作有结果
```

不要输出合规评分，不要声称编译通过等于读屏（TalkBack / VoiceOver）实测通过。
