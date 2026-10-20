# Native Swift App Store Pre-Submission Audit

**面向用 Lovable / Bolt / v0 / Cursor 等工具生成、准备上架 App Store 的独立开发者。**

我提供一份**原生 Swift 视角的书面审计报告**（邮箱或 GitHub Issue 交付），在你点「提交审核」之前，标出最容易翻车的点——**不做代上架、不做拒审后专修**。

---

## 服务包含什么

¥349 一次，交付一份人工写的 App Store 提审前审计报告。工具能扫出来的（比如 release 包里的测试开关字符串）我会先跑一遍，但你付钱买的是下面这些需要人判断的部分。

| 模块 | 你拿到什么 |
|------|------------|
| 开关与远程配置的灰区判断 | 逐个看你的 Feature Flag、Remote Config、A/B 实验，按 Guideline 2.5.19 和 2.3.1 给出风险判断和处理建议。注意：这是审计解读，不是 Apple 原文；Apple 目前没有明确说哪些开关算违规。 |
| Notes for Review 代写 | 按 2.3.1(a) 的要求，帮你写好 App Review Information 里的 Notes for Review（中英各一版）：新功能说明、测试账号用法、非显而易见功能的进入路径。 |
| Paywall 录屏清单 | 告诉你每个 paywall 版本要录哪些画面、按什么顺序录、配什么文字说明。录屏由你自己完成。 |
| 被拒后的回复模板 | Resolution Center 回复和 App Review 申诉（appeal）的模板，按常见拒审条款分类，你填入自己的情况即可。 |
| 商店页检查 | 元数据是否提到 Android 等其他移动平台（2.3.10）；名称或描述是否用了 “For Kids” 一类词（按 2.3.8 为 Kids Category 保留）；内置浏览器是否影响年龄分级问卷里的 Unrestricted Web Access（2.3.6）。 |

交付物：一份 Markdown/PDF 报告，附按优先级排好的修改清单（P0 / P1 / P2）。付款后 2 天出初稿。

**不包含：** 代提审、代改代码、长期跟进。**不保证过审**——最终结果由 App Review 决定。报告是提审前的风险检查，不构成法律意见。

---

## Guideline 2.5.19：隐藏 / 休眠 / 未说明功能（2026-10-16 新增）

**条款原文（App Review Guidelines 2.5.19）：**

> Don’t include any hidden, dormant, or undocumented features in your app; your app’s functionality should be clear to end users and App Review. Attempting to hide undocumented features in your app binary, such as through code obfuscation or other methods, is grounds for removal of your app from the App Store and termination of your developer account.

**Apple Developer News（2026-10-16）补充：**

- 2.5.19 澄清对隐藏、休眠或未说明功能的既有政策，原属 Guideline 2.3.1。
- 禁止在 App Review 过程中隐瞒用途、过审后再改变行为（例如加入提审版本中不存在的功能）；违规可被下架并移出 Apple Developer Program。

> **标注：** 下列两项是基于条款原文与上述 News 的**审计解读**，不是 Apple 原文逐条列举。远程配置 / Feature Flag、以及按审核账号 / IP / 时区 / TestFlight 等切换功能，条款未点名，但属「hidden, dormant, or undocumented features」与 bait-and-switch 的常见落地场景。中文译本 Apple 称将在一个月内提供。

### 检查项 A｜远程配置 / Feature Flag 开关披露

| | 内容 |
|--|------|
| **检查点** | ① 列出所有服务端下发开关（如 Firebase Remote Config、自建配置接口、A/B 平台），逐个标注影响的功能、默认值、当前线上值。② 确认每个受控功能已在提审二进制中完整存在，且审核期间对 App Review 可见、可走通；开关只用于灰度 / 降级 / 实验，不用于上线未审功能。③ 检查是否存在「默认关闭、过审后再打开」且与元数据（描述、截图、分类、年龄分级）不一致的功能。④ 检查 App Review Notes 是否说明受远程配置控制的主要功能及触发方式。 |
| **风险说明** | 过审后通过开关启用审核时不可见、或与元数据不符的功能，属于 2.5.19 禁止的 hidden / dormant / undocumented features，也是 Apple 点名的 bait-and-switch；后果可能不止拒审，还包括下架与终止开发者账号。 |
| **修改建议** | 建立开关清单（功能、用途、提审时状态）；提审时面向用户的功能全部打开，或给 App Review 可开启路径。线上新增面向用户的功能走新版本提审，不靠开关上线。元数据与年龄分级问卷覆盖所有可被开启的功能。 |

**App Review Notes 参考写法：**

```
This version uses server-side configuration for gradual rollout and A/B testing of [功能名]. All features are enabled for the review account. No functionality is hidden from App Review.
```

### 检查项 B｜审核环境检测代码

| | 内容 |
|--|------|
| **检查点** | ① 搜索客户端与服务端是否按以下条件切换功能：审核 / 演示账号、IP 或地区、设备时区或语言、TestFlight 或沙盒收据（`sandboxReceipt`、StoreKit 环境）、首次安装时间、「审核中」版本标记等。② 检查是否有代码混淆、加密字符串、动态拼接路由或 WebView 跳转等用于隐藏上述逻辑。③ 区分合理用途：沙盒检测仅用于内购收据验证、地区判断仅用于合规或内容版权，且不得因此改变审核员看到的功能范围。 |
| **风险说明** | 对 App Review 展示一套功能、对普通用户展示另一套，是审核期间隐瞒用途的典型做法。2.5.19 明确将通过代码混淆等方式隐藏未说明功能列为下架与终止开发者账号的理由。（**解读：** 条款未逐一列举 IP / 时区 / TestFlight 等检测方式，但属于 “other methods” 的合理适用范围。） |
| **修改建议** | 删除以「是否在审核」为条件的功能分支，审核环境与线上用户看到同一套功能。沙盒 / TestFlight 判断只保留在收据验证等技术用途。确有地区差异（合规或版权）时，在 App Review Notes 中主动说明差异与原因。审核账号与普通账号权限一致；如需预置数据，在 Notes 中说明。 |

*本模块为提审前风险检查，不保证过审，不构成法律意见。*

---

## Guideline 2.5.19: Hidden, dormant, or undocumented features (added 2026-10-16)

English version of the section above.

**Guideline text (App Review Guidelines 2.5.19):**

> Don’t include any hidden, dormant, or undocumented features in your app; your app’s functionality should be clear to end users and App Review. Attempting to hide undocumented features in your app binary, such as through code obfuscation or other methods, is grounds for removal of your app from the App Store and termination of your developer account.

**Apple Developer News (2026-10-16):**

- 2.5.19 clarifies the existing policy on hidden, dormant, or undocumented features. That policy was previously covered under Guideline 2.3.1.
- Apps must not hide their purpose during App Review and then change behavior after approval (for example, by adding features that were not in the submitted version). A violation can lead to removal from the App Store and removal from the Apple Developer Program.

> **Label:** The two checklists below are an **audit interpretation** of the guideline text and the News note above. They are not a line-by-line list from Apple. Remote Config / feature flags, and switching features by review account, IP, timezone, TestFlight, or similar signals, are not named in the guideline. They are common ways the “hidden, dormant, or undocumented features” rule and the bait-and-switch guidance show up in shipping apps. Apple says translations of the updated guidelines will be available within one month.

### Checklist A | Remote Config / feature-flag disclosure

| | Details |
|--|------|
| **What we check** | ① List every server-controlled switch (for example Firebase Remote Config, a custom config API, or an A/B platform). For each one, record the feature it affects, the default, and the current live value. ② Confirm each controlled feature is fully present in the binary under review, and that App Review can see it and complete the flow. Flags are only for gradual rollout, kill switches, or experiments, not for shipping features that were never reviewed. ③ Look for features that stay off by default and are turned on after approval, when that disagrees with the metadata (description, screenshots, category, age rating). ④ Check that App Review Notes describe the main remotely controlled features and how to trigger them. |
| **Risk** | Turning on, after approval, a feature that was invisible during review or that disagrees with the metadata is the hidden / dormant / undocumented behavior 2.5.19 prohibits, and the bait-and-switch pattern Apple named. The result can go beyond rejection: removal from the App Store and termination of the developer account. |
| **What to change** | Keep a switch inventory (feature, purpose, state at submission). At submission, leave user-facing features on, or give App Review a way to turn them on. Ship new user-facing features in a new version, not by flipping a flag. Metadata and the age-rating questionnaire must cover every feature that can be enabled. |

**Sample App Review Notes:**

```
This version uses server-side configuration for gradual rollout and A/B testing of [feature]. All features are enabled for the review account. No functionality is hidden from App Review.
```

### Checklist B | Review-environment detection

| | Details |
|--|------|
| **What we check** | ① Search the client and the server for branches that change features based on: a review or demo account, IP or region, device timezone or language, TestFlight or a sandbox receipt (`sandboxReceipt`, StoreKit environment), first-install time, or an “under review” version flag. ② Check for code obfuscation, encrypted strings, dynamically assembled routes, or WebView redirects used to hide that logic. ③ Separate legitimate uses: a sandbox check only for in-app purchase receipt validation, and a region check only for compliance or content rights. Those checks must not change the feature set the reviewer sees. |
| **Risk** | Showing one feature set to App Review and another to ordinary users is the usual way to hide a purpose during review. 2.5.19 explicitly treats hiding undocumented features through code obfuscation or other methods as grounds for removal and termination of the developer account. (**Interpretation:** the guideline does not list IP, timezone, or TestFlight checks one by one. Those sit inside a fair reading of “other methods.”) |
| **What to change** | Remove feature branches that key off “are we in review?”. The review environment and production users should see the same features. Keep sandbox / TestFlight checks only for technical uses such as receipt validation. If a regional difference is real (compliance or rights), explain the difference and the reason in App Review Notes. The review account should have the same permissions as a normal account. If you preload data, say so in the Notes. |

*This module is a pre-submission risk check. It does not guarantee approval and is not legal advice.*

---

## What's included

¥349 (about $49) per audit. You get a hand-written App Store pre-submission report. I run the usual automated checks first (like scanning a release build for leftover test switch strings), but what you're paying for is the judgment calls below.

| Area | What you get |
|------|--------------|
| Feature flags & remote config | I go through your feature flags, Remote Config, and A/B tests and tell you where the risk is under Guidelines 2.5.19 and 2.3.1, and what to do about it. This is my reading as an auditor, not Apple's wording — Apple hasn't said exactly which switches cross the line. |
| Notes for Review, written for you | Notes for Review in App Review Information, in English and Chinese, written to meet 2.3.1(a): what's new, how to use the demo account, and how to reach anything that isn't obvious. |
| Paywall recording checklist | For each paywall variant: which screens to record, in what order, and what to write alongside it. You do the recording. |
| Rejection reply templates | Templates for Resolution Center replies and App Review appeals, grouped by common rejection guidelines. Fill in your specifics. |
| Product page check | Mentions of Android or other mobile platforms in metadata (2.3.10); terms like “For Kids” that are reserved for the Kids Category (2.3.8); whether an in-app browser changes your Unrestricted Web Access answer on the age rating questionnaire (2.3.6). |

Deliverable: a Markdown/PDF report plus a prioritized fix list (P0 / P1 / P2). First draft within 2 days of payment.

**Not included:** submitting for you, changing your code, ongoing support. **No approval guarantee** — App Review makes the call. This is a pre-submission risk review, not legal advice.

---

## 定价

**¥349 / 次**（约等于 $49，固定价小包）

适合：首版或大改版后、第一次提审前。  
加急或超大工程可另议，下单前先发 IPA/工程说明或 TestFlight 链接评估是否接单。

---

## 样例报告（脱敏摘要）

完整样例见本仓库 [Issue #1](../../issues/1)（或仓库 Issues 置顶帖）。摘要如下：

> **结论：** 暂不建议直接提审。发现 **2 个 P0**、**3 个 P1**。  
> **P0-1（4.2）：** 主流程依赖未登录 WebView，原生侧无独立核心功能 → 高概率被判壳。  
> **P0-2（隐私）：** 使用相机但 `Info.plist` / Privacy Manifest 用途说明与实际功能不符。  
> **P1：** 截图含「ChatGPT」字样易触发商标/误导；后台音频 entitlement 未使用；缺少账号删除入口文案（若有账号体系）。  
> **建议路径：** 先补一个离线可用的原生主功能 → 对齐隐私文案 → 再提审。

---

## 怎么找我

1. 在本仓库开 [Issue](../../issues/new)，标题写：`Audit request: <你的 App 名>`  
2. 或通过 GitHub 联系：[CoderKingXY](https://github.com/CoderKingXY)

请附上：App 一句话介绍、是否已有 TestFlight、主要技术栈（纯原生 / RN / Flutter / WebView）、是否含登录与内购。

---

## 关于作者

10 年 iOS（Swift / Objective-C），独立交付过完整 App。本服务只做**上架前书面审计**，帮你用原生经验避开第一轮常见拒审。

---

*本仓库仅作作品集与样例展示。价格与范围以沟通确认为准。*
