# Release Audit — 巡视记录

> 巡视日期：2026-07-07

## 矛盾

audit 的职责是"发现和契约不符合的地方"。但当前 audit 检查的是"每个 artifact 各自是否存在"，而不是"artifact 之间是否一致"。

一个 tag 存在、一个 Release 存在、一个 CHANGELOG 存在——三个独立的"存在"不能保证发布是完整的。真正的问题是**三者之间缺乏因果约束**：publish 命令原本是按顺序创建这三者的，但 AI 可以绕过 publish 独立操作其中任何一个，打破因果关系。

## 本质

这不是一个 audit 检查项够不够的问题，而是**流程约束力的问题**。

- publish 创建 artifact 链：CHANGELOG → tag → Release
- AI 绕过 publish 后，artifact 链断裂
- audit 只能发现断裂，无法防止断裂
- 要防止断裂，需要工具在 design 层面假设使用者会绕过它，而不是假设使用者会遵守流程

## 反脆弱视角

反脆弱的 audit 不是"发现问题后报告"，而是"发现问题后能自动收敛到正确状态"：

| 问题 | 反脆弱修复 |
|------|-----------|
| 缺 CHANGELOG | 从 tag 和 git log 补全（修改 tag 会改变事实源，需谨慎） |
| 缺 Release | 直接 `gh release create` 补充，不改变任何事实源 |
| 缺 tag | 无法自动修复，tag 是事实源，需人类确认 |

## 追问

- 如果 AI 可以绕过 publish，那 publish 存在的意义是什么？
- 是否应该让 publish 成为创建 CHANGELOG/tag/Release 的唯一入口，让绕过它的操作都被 audit 标记为"非受控状态"？
- "非受控状态"是否应该阻断后续的 CI/CD 流程？
