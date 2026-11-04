# auditing-accessibility

面向个人开发者的基础读屏适配（Android / iOS / Web）：不安装新工具，一句话让 AI 直接修复高置信度问题。

```text
帮我给这个项目做基础无障碍适配，能确定的直接改。
```

如果只想看报告：

```text
只检查这个项目的读屏问题，不要修改代码。
```

目标是让核心流程基本可用，不代表完整无障碍合规。规则库为同一套 18 条规则的三个平台版本（编号一致，改规则请多份副本同步）：

| 平台 | 规则文件 | 技术栈 |
|------|---------|--------|
| Android | [references/rules-android.md](references/rules-android.md) | XML Layout、Jetpack Compose、TalkBack |
| iOS | [references/rules-ios.md](references/rules-ios.md) | UIKit、SwiftUI、VoiceOver |
| Web | [references/rules-web.md](references/rules-web.md) | 语义化 HTML、ARIA、React 等 |

该 Skill 只在用户明确要求检查或修复无障碍问题时运行，不接入正常 UI 开发流程。

安装（Claude Code / Cursor / Codex 等均可）：

```bash
npx skills add ShirleyZhuGit/accessibility-skills
```
