# accessibility-skills

[English](#english) · 中文

这是面向个人开发者的客户端（Android / iOS / Web）基础无障碍适配 skill，可以让你不安装任何工具，一句话 + 一点点 token，就能让视障人士轻松使用你的应用！

## 为什么无障碍很重要

全球有近 3 亿视力障碍者，包括大约 4000 万全盲者和 2 亿多低视力人士，他们通过手机自带的读屏功能来朗读网页和 App。

所有 Android 和 iOS 设备都有这个功能，你可以在「系统设置 → 无障碍」中打开旁白（VoiceOver）或 TalkBack 体验一下。

但读屏功能无法朗读未经标记的图片和 icon。比如页面上有一个垃圾桶图标，代码没做特殊处理的话，读屏只能念出「图标 1」，用户根本不知道这是删除。各类 App 里遍布的弹窗、一模一样的列表元素，也给读屏软件带来了很多困扰。

很多开发者没有意识到无障碍适配的重要性，或者想适配，但不知道怎么做。

## 如何使用

通过命令安装：

```bash
npx skills add ShirleyZhuGit/accessibility-skills
```

或者干脆把这个仓库地址扔给你的 AI，告诉它「安装这个」。Claude Code、Cursor、Codex 等都支持。

安装后，目前有 2 个功能：

⭐️ **直接适配**

```text
帮我给这个项目做基础无障碍适配，能确定的直接改。
```

⭐️ **检查现有的问题**

```text
只检查这个项目的读屏问题，不要修改代码。
```

三个平台共用同一套 18 条规则（名称 / 分组 / 状态 / 可达焦点 / 朗读反馈）：

| 平台 | 技术栈 | 规则 |
|------|--------|------|
| Android | XML Layout、Jetpack Compose、TalkBack | [rules-android.md](skills/auditing-accessibility/references/rules-android.md) |
| iOS | UIKit、SwiftUI、VoiceOver | [rules-ios.md](skills/auditing-accessibility/references/rules-ios.md) |
| Web | 语义化 HTML、ARIA、React / Vue | [rules-web.md](skills/auditing-accessibility/references/rules-web.md) |

## 靠得住吗

这个 skill 是我和无障碍适配专业测试团队合作后整理的细节，可以满足基础的读屏适配，并且经过真实用户测试。

但也只是基础读屏适配，小开发者够用了，不是完整的无障碍合规。

如果你是视障人士，或者熟悉无障碍开发，欢迎提出意见！

这个世界也许有点坏，但我还想做个好人。

## License

MIT

---

## English

An agent skill that gives your Android / iOS / Web app basic screen-reader accessibility. Built for solo developers: no new tools to install, one sentence and a handful of tokens, and people with visual impairments can actually use your app.

### Why accessibility matters

Nearly 300 million people worldwide live with a visual impairment, including about 40 million who are blind and over 200 million with low vision. They rely on the screen reader built into their phone to read web pages and apps aloud.

Every Android and iOS device has one. Try it yourself: go to Settings → Accessibility and turn on VoiceOver or TalkBack.

But a screen reader cannot describe an image or icon that was never labeled. Take a trash-can icon: if the code does nothing special, the screen reader just says "icon 1", and the user has no idea it means delete. The pop-ups and identical list items that fill most apps confuse screen readers too.

Many developers never realize accessibility matters, or want to do it but don't know where to start.

### How to use

Install with one command:

```bash
npx skills add ShirleyZhuGit/accessibility-skills
```

Or just paste this repo URL to your AI and tell it to install it. Works with Claude Code, Cursor, Codex and more.

Once installed, there are two things it can do:

⭐️ **Fix it for you**

```text
Do a basic accessibility pass on this project. Fix anything you are sure about.
```

⭐️ **Check what's wrong first**

```text
Only check this project for screen-reader issues. Do not modify code.
```

All three platforms share the same 18 rules, covering names, grouping, state, reachable focus and spoken feedback:

| Platform | Stack | Rules |
|----------|-------|-------|
| Android | XML Layout, Jetpack Compose, TalkBack | [rules-android.md](skills/auditing-accessibility/references/rules-android.md) |
| iOS | UIKit, SwiftUI, VoiceOver | [rules-ios.md](skills/auditing-accessibility/references/rules-ios.md) |
| Web | Semantic HTML, ARIA, React / Vue | [rules-web.md](skills/auditing-accessibility/references/rules-web.md) |

### Can I trust it?

I put these rules together after working with a professional accessibility testing team. They cover basic screen-reader support and have been tested with real users.

That said, it is basic screen-reader support: enough for a small developer, not full accessibility compliance.

If you are visually impaired, or you know accessibility development well, I would love to hear your feedback!

The world may be a bit broken, but I still want to be one of the good ones.

### License

MIT
