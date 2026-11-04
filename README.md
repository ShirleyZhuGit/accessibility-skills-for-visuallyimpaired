# accessibility-skills

Agent skills that make your app usable with screen readers (TalkBack / VoiceOver / NVDA) — built for solo developers and small teams. No new tooling, no long audit plan: point the agent at your project and let it fix the obvious issues.

[中文说明](#中文说明)

## Install

```bash
npx skills add ShirleyZhuGit/accessibility-skills
```

Works with Claude Code, Cursor, Codex, Copilot and any other agent supported by [skills.sh](https://www.skills.sh/).

## Skills

### `auditing-accessibility`

Audit or fix basic screen-reader accessibility in an existing codebase.

| Platform | Stack | Rules |
|----------|-------|-------|
| Android | XML Layout, Jetpack Compose, TalkBack | [rules-android.md](skills/auditing-accessibility/references/rules-android.md) |
| iOS | UIKit, SwiftUI, VoiceOver | [rules-ios.md](skills/auditing-accessibility/references/rules-ios.md) |
| Web | Semantic HTML, ARIA, React / Vue | [rules-web.md](skills/auditing-accessibility/references/rules-web.md) |

The same 18 rules on every platform, grouped into five areas: **names** (every control has a readable label), **grouping** (related items are read as one), **state** (selected / disabled / expanded are announced), **reachable focus** (nothing important is skipped or trapped), and **spoken feedback** (changes are announced).

**Fix mode** — the agent edits high-confidence issues directly and lists at most five things it was unsure about:

```text
Do a basic accessibility pass on this project. Fix anything you are sure about.
```

**Report mode** — nothing is changed:

```text
Only check this project for screen-reader issues. Do not modify code.
```

The skill only runs when you explicitly ask for accessibility work; it does not interfere with normal UI development.

## Scope

The goal is that a screen-reader user can understand and complete the core flows. This is not a full WCAG compliance audit.

## License

MIT

---

## 中文说明

面向个人开发者和小团队的读屏无障碍 skill（TalkBack / VoiceOver / NVDA）。不装新工具、不出长计划，直接让 AI 修掉能确定的问题。

安装：

```bash
npx skills add ShirleyZhuGit/accessibility-skills
```

目前包含一个 skill：`auditing-accessibility`，审查或修复存量 Android / iOS / Web 项目的基础读屏问题。三个平台共用同一套 18 条规则（名称 / 分组 / 状态 / 可达焦点 / 朗读反馈）。

直接修复：

```text
帮我给这个项目做基础无障碍适配，能确定的直接改。
```

只看报告：

```text
只检查这个项目的读屏问题，不要修改代码。
```

目标是让核心流程对读屏用户基本可用，不等于完整无障碍合规。
