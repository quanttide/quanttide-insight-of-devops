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

## 改进方案

`collect_git_log` 的比较基准选择逻辑需要区分场景：

| 发布类型 | 示例 | 比较基准 | 原因 |
|---------|------|---------|------|
| 预发布晋级 | beta.8 → rc.1 | 上一个任意 tag | 无新提交，占位即可 |
| 预发布递增 | rc.1 → rc.2 | 上一个任意 tag | 有新提交 |
| 正式版发布 | rc.2 → 0.10.0 | **上一个正式版 tag** | 需汇总整个预发布周期 |

### 具体改法

在 `ensure_changelog` 中新增分支：当目标版本为正式版（无 prerelease 后缀）且最新 tag 是同一 base 版本的预发布 tag 时，将比较基准改为上一个 base 版本不同的正式版 tag。

```rust
fn get_comparison_tag(version: &str, all_tags: &[String]) -> Option<String> {
    let target = parse_semver(version);
    // 如果目标是正式版，且最新 tag 是同版本的预发布
    if target.prerelease.is_empty() {
        let latest = all_tags.iter()
            .filter_map(|t| parse_semver_stripping_scope(t))
            .max_by(|a, b| a.cmp(b))?;
        if latest.prerelease.is_empty() {
            return Some(latest.to_string()); // 最新就是正式版，正常比较
        }
        // 最新是预发布，往上找上一个不同 base 的正式版
        // 需要排除同 base 版本的所有预发布 tag
    }
    // 预发布版本维持原有逻辑
    all_tags.iter().max_by(|a, b| semver_cmp(a, b)).cloned()
}
```

### CHANGELOG 生成方式的扩展

即使找到正确的比较基准，`git log v0.9.3..HEAD` 的输出长达 96 条提交，无法直接作为 CHANGELOG 条目。LLM 生成的概括性描述也偏单薄。

更合适的做法是：**从 CHANGELOG 已有的预发布条目汇总**，而非从 git log 重新生成。即读取 `[0.10.0-alpha.1]` 到 `[0.10.0-rc.2]` 的所有条目，由 LLM 合并去重后生成正式版条目。这样既利用了已有人工审核的内容，又避免了 LLM 对 git log 过度概括。
