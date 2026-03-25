---
name: skill-review
description: >
  Skill 生命周期审计与部署的完整流程编排。当 inbox/ 目录出现新的 SKILL.md 文件、
  用户想评估一个 skill 是否值得安装、需要判断某个 skill 的安全性或工作区适配性时，
  必须触发此 skill。自动编排以下完整流程：
  (1) 用 Explore 子代理读取 skill 文件，
  (2) 调用 skill-vetter 安全审计，
  (3) 执行四维分析（设计意图/代码质量/可执行性/适配性），
  (4) 调用 value-first-gate 价值评估（含正反驳斥），
  (5) GO/NO-GO 部署决策，
  (6) 安装到 ~/.claude/skills/ 或归档，并写入 value-review.md，
  (7) 调用 inbox-triage 清理 inbox。
  触发词：评估这个 skill、这个 skill 值得装吗、审计 skill、vet this skill、
  review this skill、inbox 有新 skill、help me evaluate a skill、skill 安全吗。
  不要等用户手动编排每个步骤——只要有 skill 评估需求就主动触发完整流程。
---

# Skill Review — 生命周期审计与部署

## 适用场景

- `inbox/` 目录出现新的 SKILL.md（或 .skill 文件）
- 用户提供了 skill 路径，需要判断是否安装
- 需要重新审计已安装 skill 的安全性或适配性

## 总体流程

```
Phase 1: 定位文件      → Explore 子代理读取，提取关键元信息
Phase 2: 安全审计      → skill-vetter（风险门控）
Phase 3: 四维分析      → 设计意图 / 架构质量 / 可执行性 / 工作区适配
Phase 4: 价值评估      → value-first-gate（含第一性原理驳斥）
Phase 5: 决策          → 综合 Phase 2-4，输出 GO / REVISE / NO-GO
Phase 6: 执行          → 安装或归档，追加 value-review.md
Phase 7: 清理          → inbox-triage
```

每个 Phase 完成后方可进入下一步，产出写入 `value-review.md`（追加）。

---

## Phase 1 — 定位 Skill 文件

1. 若用户未指定路径，扫描 `inbox/`（排除 README.md），找到 SKILL.md 或 *.skill 文件
2. 若 inbox 为空，报告"inbox 无待审文件"并终止
3. 使用 **Explore 子代理**读取完整内容，提取：
   - `name`、`description`（frontmatter）
   - 文件类型：纯 Prompt / 含脚本（scripts/）/ 依赖外部 API
   - 总行数、有无附属资源目录（references/ assets/ scripts/）
   - 触发描述覆盖的场景

> Explore 子代理针对文件读取优化，可以在不污染主上下文的情况下完整摄取 skill 内容，
> 为后续分析提供干净的信息基础。

---

## Phase 2 — 安全审计（skill-vetter）

调用 `/skill-vetter`，提交 SKILL.md 全文。

提取报告中的关键字段：
- **风险等级**：🟢 LOW / 🟡 MEDIUM / 🔴 HIGH / ⛔ EXTREME
- **RED FLAGS**：列出发现的任何危险信号
- **所需权限**：文件 / 网络 / 命令

**风险门控规则：**

| 风险等级 | 行动 |
|---------|------|
| 🟢 LOW | 继续 Phase 3 |
| 🟡 MEDIUM | 继续，但在最终报告中标注注意事项 |
| 🔴 HIGH | 暂停，告知用户，**需要 human 显式批准**才能继续 |
| ⛔ EXTREME | 立即终止，记录拒绝原因，跳至 Phase 6（归档）→ Phase 7 |

---

## Phase 3 — 四维分析

基于 Phase 1 提取的内容逐一分析，每项输出一句话结论：

### 3.1 设计意图
这个 skill 解决什么具体问题？面向什么用户场景？
触发描述是否清晰、不歧义、不会误触发或漏触发？

### 3.2 架构与代码质量
- 流程是否有清晰的阶段划分和终止条件？
- 若含代码（scripts/）：安全性、可读性、依赖是否明确？
- 有无硬编码路径、缺失降级方案、过度耦合等反模式？

### 3.3 Agent 可执行性
- 哪些阶段 Claude Code Agent 可以完全自动化？哪些需要 human 介入？
- 与当前工具链（Bash、Glob、Grep、MCP 工具）的兼容性如何？
- 若含"独立第三方评估"等要求，操作路径是否明确可行？

### 3.4 工作区适配性
从四个维度评分（各 1-5 星）：

| 维度 | 评分 | 说明 |
|------|------|------|
| 触发频率 | ★-★★★★★ | 预计每月触发次数 |
| 用户角色匹配 | ★-★★★★★ | 与用户背景/工作方式的契合度 |
| 工具链整合 | ★-★★★★★ | 与现有 skill 栈的协同程度 |
| 泛化价值 | ★-★★★★★ | 可跨项目/场景复用的程度 |

---

## Phase 4 — 价值评估（value-first-gate）

调用 `/value-first-gate`，评估主题为：**"是否值得将 [skill-name] 部署到当前工作区"**

执行完整流程（Problem Framing → First-Principles Check → 6 维度打分 → 决策）。

**完成后补充正反综合：**
- **支持部署的最强论点**（来自 Phase 3 分析）
- **反对部署的最强驳斥**（来自 value-first-gate 第一性原理）
- **综合判断**（1-2 句话，结论优先）

---

## Phase 5 — 部署决策

综合 Phase 2-4，输出决策卡片：

```
┌──────────────────────────────────────────┐
│ Skill: [name]                            │
│ 安全等级:  🟢 LOW / 🟡 MEDIUM             │
│ value-gate: GO / REVISE / NO-GO (XX/30)  │
│ 适配评分:  ★★★★☆                          │
│ ─────────────────────────────────────── │
│ 最终决策:  ✅ 安装 / ⚠️ 条件安装 / ❌ 归档 │
└──────────────────────────────────────────┘
```

**决策规则：**
- 安全 ≤ 🟡 且 value-gate = GO 且 适配 ≥ 3★ → **安装**
- 安全 = 🟡 或 value-gate = REVISE → **条件安装**（列出前提，等用户确认）
- 安全 ≥ 🔴 或 value-gate = NO-GO → **归档**

等待用户确认后进入 Phase 6。

---

## Phase 6 — 执行

### 决策为"安装"

1. 创建安装目录：`~/.claude/skills/{skill-name}/`
2. 复制 SKILL.md 及所有附属资源到安装目录
3. 创建 `_meta.json`：
   ```json
   {
     "name": "{skill-name}",
     "installed_date": "YYYY-MM-DD",
     "source": "inbox / [来源描述]",
     "risk_level": "LOW",
     "value_gate_score": 0,
     "verdict": "GO",
     "notes": "安装原因一句话"
   }
   ```
4. 追加记录到 `value-review.md`（含：决策摘要、安全等级、value-gate 分数、安装路径）

### 决策为"归档"

1. 追加记录到 `value-review.md`（含：拒绝理由、安全等级、value-gate 分数）
2. 不创建任何安装目录

---

## Phase 7 — 清理（inbox-triage）

调用 `/inbox-triage`：

- 自动扫描 + 分类：已评估的 skill 文件标注 `已安装` 或 `已归档`
- **等待用户确认**分拣计划后再执行文件移动
- 验证 inbox/ 最终仅剩 README.md

---

## 每次运行的总结卡片

```markdown
## Skill Review 完成

| 项目 | 结果 |
|------|------|
| Skill 名称 | [name] |
| 安全审计 | 🟢 LOW |
| value-first-gate | GO / NO-GO（XX/30）|
| 四维适配 | ★★★★☆ |
| 最终决策 | ✅ 已安装 / ❌ 已归档 |
| value-review.md | ✅ 已追加 |
| inbox 清理 | ✅ 已完成 |
```

---

## 降级处理

| 场景 | 处理方式 |
|------|---------|
| skill-vetter 未加载 | 手动执行 `skill-install-protocol.md` 检查清单 |
| value-first-gate 未加载 | 使用内嵌简化 6 维度表格手动打分（0-5 per 维度）|
| inbox-triage 未加载 | 手动移动文件并报告最终 inbox 状态 |
| 安装路径权限不足 | 告知用户，提供手动安装命令 |
