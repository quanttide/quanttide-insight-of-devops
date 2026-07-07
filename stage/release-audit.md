# Release Audit — 巡视记录

> 巡视日期：2026-07-07

## 意图

AI 特别喜欢绕过预设的 Skill 和 CLI 自己乱跑。所以 audit 需要能够发现和契约不符合的地方，并且从反脆弱的角度给出修复建议。

常见的问题有：
- 只有 tag 和 release，没有 CHANGELOG
- 只有 CHANGELOG 和 tag，没有 release

根本原因是，devops-release 命令和 CLI 无法面对 AI 制造的海量不遵守流程随意跳过去的混乱。

## 当前能力

`release audit` 检查 6+1 项，但每一项只做"有没有"的判断，不做"三者是否一致"的聚合检查。

## 差距

### 1. 缺少 artifact 完整性检查

| 组合 | 当前判定 | 应有判定 |
|------|---------|---------|
| CHANGELOG ✅ + Tag ✅ + Release ✅ | 通过（三项各自通过） | 正常 |
| CHANGELOG ❌ + Tag ✅ + Release ✅ | 通过（Tag 和 Release 各自通过） | **⚠️ 缺少 CHANGELOG** |
| CHANGELOG ✅ + Tag ✅ + Release ❌ | 通过（CHANGELOG 和 Tag 各自通过） | **⚠️ 未创建 Release** |

audit 需要一张聚合矩阵，而不是逐项独立检查。

### 2. 缺少反脆弱修复建议

| 发现问题 | 当前行为 | 应有建议 |
|---------|---------|---------|
| 缺少 CHANGELOG | 报错 | "CHANGELOG 缺失，补充后 tag 会改变事实源，建议用 `release publish --force` 重新发布" |
| 缺少 GitHub Release | 报错 | "Release 缺失，可直接补充 `gh release create <tag>`" |
| 缺少 tag | 报错 | "Tag 缺失，建议用 `release publish` 统一发布而非手动打 tag" |

### 3. 工具本身无法应对海量混乱

devops-release 的每个子命令都假设使用者会按流程走。但 AI 不按流程走——它可能同时绕过多步、同时产生多个不一致。当前的工具没有"收敛"能力，无法自动发现并修复批量产生的混乱。

## 改进方向

### 短期：artifact 完整性矩阵

新增一项 audit 检查，输出：

```
CHANGELOG: ✅   Tag: ✅   Release: ✅   → 正常
CHANGELOG: ❌   Tag: ✅   Release: ✅   → 建议通过 publish --force 补充
CHANGELOG: ✅   Tag: ✅   Release: ❌   → 建议 gh release create 补充
```

### 中期：audit → 自动修复链路

引入 `release audit --fix`，对可自动修复的问题（如缺少 Release）自动执行修复，无需人类介入。

### 长期：反脆弱设计

工具应当在 design 层面假设使用者会绕过程序，而不是假设使用者会遵守流程。每个步骤都应该能在任意状态下安全重入，并自动收敛到正确状态。
