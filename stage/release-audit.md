# Release Audit — 巡视记录

> 巡视日期：2026-07-07

## 意图

> audit 需要能够发现和契约不符合的地方。常见问题：只有 tag 和 release 没有 CHANGELOG、只有 CHANGELOG 和 tag 没有 release。（`intention/stage/release.md`）

## 当前能力

`release audit` 目前检查 6+1 项：

| 检查项 | 发现问题 |
|--------|---------|
| 版本号格式 | ❌ 不发现 artifact 完整性 |
| 配置文件一致性 | ❌ 不检查 release/tag/CHANGELOG 三者是否齐全 |
| CHANGELOG | ✅ 检查是否存在版本记录 |
| 工作区状态 | ✅ |
| 标签冲突 | ✅ |
| 远程可达性 | ✅ |
| GitHub Release | ✅ 检查是否已发布 |

## 差距

意图中的两个典型问题当前 audit 无法发现：

**只有 tag 和 release，没有 CHANGELOG**
- `precheck_version_changelog` 只在 CHANGELOG.md 存在时检查条目
- 如果 CHANGELOG.md 不存在，报错信息不同，但不构成统一的"artifact 完整性"检查

**只有 CHANGELOG 和 tag，没有 release**
- `release audit` 中 `audit_github_release` 检查 tag 是否已发布为 GitHub Release
- 但 audit 结果中没有聚合这几个 artifact 的状态

## 改进方向

引入 **artifact 完整性检查**，统一检测 CHANGELOG、tag、GitHub Release 三者是否齐全：

| 组合 | 判定 |
|------|------|
| CHANGELOG ✅ + Tag ✅ + Release ✅ | 正常 |
| CHANGELOG ❌ + Tag ✅ + Release ✅ | ⚠️ 缺少 CHANGELOG |
| CHANGELOG ✅ + Tag ❌ + Release ❌ | ⚠️ 未发布 |
| CHANGELOG ✅ + Tag ✅ + Release ❌ | ⚠️ 未创建 Release |
| CHANGELOG ❌ + Tag ✅ + Release ❌ | ⚠️ 只有 tag |
| CHANGELOG ❌ + Tag ❌ + Release ✅ | ❌ 不可能发生（release 依赖 tag） |

这需要在 `release audit` 中新增一项检查，读取 CHANGELOG/tag/GitHub Release 并输出聚合状态矩阵。

## 与 publish 的关系

release publish 自动生成 CHANGELOG → 创建 tag → 创建 GitHub Release，所以正常执行 publish 不会出现不一致。不一致通常来自：

- AI 绕过 publish 手动操作（如直接 `gh release create`、手动编辑 CHANGELOG 后忘记打 tag）
- 发布中断（如超时后部分步骤成功、部分失败）
