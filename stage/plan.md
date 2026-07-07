# 计划阶段 — 巡视记录

> 巡视日期：2026-07-07
> 巡视范围：qtcloud-devops-cli v0.10.0-beta.7

## 意图

让 AI 接管日常维护工作：人类指导 AI 发现问题 → 更新 ROADMAP 和 TODO → doctor/audit 修复审计 → 进入编码阶段。

## 差距

### 1. ROADMAP 状态与代码实际完成情况不同步

以下条目已实现但 ROADMAP 仍标记 `[ ]`：

| ROADMAP 条目 | 实现证据 |
|-------------|---------|
| `plan clean` 同时清理 ROADMAP 和 TODO | `run_plan_clean` 遍历 `["ROADMAP.md", "TODO.md"]`；TODO 对应节已 [x] |
| `plan audit` 三项结构检查 | `plan_audit` 实现路径检查/粒度检查/孤儿检查；TODO 对应节已 [x] |
| `plan doctor` LLM prompt 覆盖 ROADMAP + TODO | `edit_llm` 按文件名切换 prompt；TODO 对应节已 [x] |
| 补充 `release/audit.rs`、`contract.rs` 测试 | commit `4e3005f` 已补全 |

**根因**：当前工作流缺少 TODO [x] → ROADMAP [x] 的自动同步步骤。TODO 标记完成时，ROADMAP 对应条目不会自动推进，需要手动或通过 `plan clean` 之后统一更新。

### 2. 工具缺陷（已修 vs 未修）

| 缺陷 | 状态 |
|------|------|
| `extract_line_paths` 不支持 `:N` 行号 | ✅ `4e3005f` 已修 |
| `CATEGORIES` 缺少 `### Refactor` | ✅ `4e3005f` 已修 |
| `plan audit` 不支持 scope — 无法审计子模块内规划文件 | ❌ 未修 |
| `determine_submodule_status` 长参数重构不完整（plan edit 改写但代码不完整，已 revert） | ❌ 需重新做 |
| ROADMAP 2 个孤儿条目（L8 code 命令重新设计、L22 子模组指针未更新） | ❌ 未修 |

### 3. 缺乏统一的"问题收集"入口

当前只有 `qtcloud-code review` 一个输入源。build/test/release 等阶段的发现缺乏集中收集到规划文件的机制。

### 4. 进度虚报

ROADMAP 0/16 完成，但实际约 4 项已完成。需批量状态更新后才能判断编码阶段准入条件。
