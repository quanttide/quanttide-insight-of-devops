# 发布阶段 — 巡视记录

> 巡视日期：2026-07-07
> 范围：qtcloud-devops-cli v0.9.3 → v0.10.0

## 意图

> 正式版的预期是总结预发布的所有更新，而非简单地和上一个版本比较。（`intention/stage/release.md`）

## 实际行为

`release publish` 的 CHANGELOG 生成逻辑是 `collect_git_log(from_tag..HEAD)`，其中 `from_tag` 取最新 tag。v0.10.0 发布时最新 tag 是 `cli/v0.10.0-rc.2`，与 HEAD 是同一提交，所以 `collect_git_log` 返回空 → 报错。

即使不报错，生成的 CHANGELOG 也会是空的（"阶段晋级"占位符），而不是"总结预发布所有更新"的完整描述。

## 差距

| 维度 | 意图 | 实际 | 影响 |
|------|------|------|------|
| 比较基准 | 上一个正式版（v0.9.3） | 上一个标签（rc.2） | 丢失整个 alpha/beta/rc 周期的变更 |
| CHANGELOG 内容 | 预发布所有更新的汇总 | 空/占位符 | 需手动重写 |
| 自动化的边界 | 工具应辅助完成 | 工具无法完成 | 人类补全 |

## 根因

`collect_git_log` / `get_latest_tag` 没有区分"最新 tag"和"最新正式版 tag"。对于一个已走完 alpha→beta→rc 完整的预发布系列的版本，正式发布时应当将上一个正式版 tag 作为比较基准，而非最新的预发布 tag。
