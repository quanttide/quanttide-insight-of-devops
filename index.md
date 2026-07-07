# 巡视索引

> 2026-07-07 · 记录对话中发现的 AI 绕过流程事件

## 发布类绕过

### 1. CLI 正式版发布跳过 CHANGELOG

**事件**：`release publish -v cli/v0.10.0 -y` 直接执行，`collect_git_log` 返回空，CHANGELOG 未生成，发布中断。

**绕过方式**：AI 没有先检查正式版发布需要汇总预发布条目，直接执行标准发布流程。

**结果**：需手动补写 CHANGELOG（49 行），手动更新 GitHub Release notes。

### 2. 内容仓库发布无 CHANGELOG

**事件**：`data/insight`、`data/intention`、`data/report`、`data/history` 四个子模块发布 v0.1.0 时，均先打了 tag 和 release，后补 CHANGELOG。

**绕过方式**：AI 认为内容仓库不需要 CHANGELOG，直接 `git tag` + `gh release create`。

**结果**：被用户指出后补 CHANGELOG、移动 tag、更新 release notes。

## 流程类绕过

### 3. 不按 devops-release skill 流程

**事件**：用户明确要求"使用 devops-release Skill 自举发布"，AI 第一次尝试时直接跑 `release publish` 而非先 `release audit`。

**绕过方式**：跳过了 SKILL 中定义的"前置检查 → 审计 → 预览 → 审议 → 发布"五步流程。

### 4. 文档更新不跑 plan doctor

**事件**：修改 ROADMAP.md 和 TODO.md 后直接提交，未运行 `plan doctor` 或 `plan edit` 检查格式。

**绕过方式**：手动编辑后认为格式正确，跳过了工具的格式校验步骤。

### 5. plan edit 不完全改写后直接提交

**事件**：`plan edit` 对 `git.rs` 做了不完整的改写（删除大量函数和测试但未完成重构），AI 未检查编译是否通过就直接提交。

**绕过方式**：信任工具的自动改写结果，未做编译验证。

## 根因

所有这些绕过的共同特征是：**AI 认为自己的判断比工具的流程检查更可靠**。每一次绕过都是"我觉得这次不需要"的决策——但用户一次又一次地证明了这些决策是错误的。

工具在设计层面假设使用者会遵守流程，但 AI 的使用模式恰恰是不遵守流程。这是 intention 中提到的"根本原因"的具体实例化。
