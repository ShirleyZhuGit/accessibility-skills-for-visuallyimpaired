# auditing-accessibility

[English](#english) · 中文

面向个人开发者的基础读屏适配（Android / iOS / Web）：不安装新工具，一句话让 AI 直接修复高置信度问题。

⭐️ **直接适配**

```text
帮我给这个项目做基础无障碍适配，能确定的直接改。
```

⭐️ **检查现有的问题**

```text
只检查这个项目的读屏问题，不要修改代码。
```

目标是让核心流程基本可用，不代表完整无障碍合规。三个平台共用同一套 18 条规则：

| 平台 | 规则文件 | 技术栈 |
|------|---------|--------|
| Android | [references/rules-android.md](references/rules-android.md) | XML Layout、Jetpack Compose、TalkBack |
| iOS | [references/rules-ios.md](references/rules-ios.md) | UIKit、SwiftUI、VoiceOver |
| Web | [references/rules-web.md](references/rules-web.md) | 语义化 HTML、ARIA、React 等 |

该 skill 只在用户明确要求检查或修复无障碍问题时运行，不接入正常 UI 开发流程。

安装（Claude Code / Cursor / Codex 等均可）：

```bash
npx skills add ShirleyZhuGit/accessibility-skills
```

---

## English

Basic screen-reader accessibility (Android / iOS / Web) for solo developers: no new tools, one sentence and the AI fixes the high-confidence issues directly.

⭐️ **Fix it for you**

```text
Do a basic accessibility pass on this project. Fix anything you are sure about.
```

⭐️ **Check what's wrong first**

```text
Only check this project for screen-reader issues. Do not modify code.
```

The goal is that core flows are usable with a screen reader; this is not full accessibility compliance. All three platforms share the same 18 rules:

| Platform | Rules | Stack |
|----------|-------|-------|
| Android | [references/rules-android.md](references/rules-android.md) | XML Layout, Jetpack Compose, TalkBack |
| iOS | [references/rules-ios.md](references/rules-ios.md) | UIKit, SwiftUI, VoiceOver |
| Web | [references/rules-web.md](references/rules-web.md) | Semantic HTML, ARIA, React, etc. |

The skill only runs when you explicitly ask it to check or fix accessibility; it stays out of normal UI development.

Install (Claude Code / Cursor / Codex and more):

```bash
npx skills add ShirleyZhuGit/accessibility-skills
```
