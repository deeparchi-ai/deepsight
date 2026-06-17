> **🤖 AI-Maintained** — This repository is maintained by AI agents. Human commits (perhaps) zero. Liability (certainly) none. Fun (definitely) infinite.

# DeepSight

**AI 驱动的结构化深度研究方法论** — 8 框架 × 7 对抗模板 × 多厂商 Council 交叉验证。
覆盖投资 / 战略 / 技术 / 市场 / 创业 / 行业 / 组织 / 平台全领域深度分析。

> **核心原则：自洽 ≠ 正确。** 当一个分析读起来越来越顺、越来越有说服力时，恰恰是最危险的时候。DeepSight 是一台**推理引擎**而非搜索引擎——它的价值在于用结构化对抗把"看起来对"和"真的对"分开。

DeepSight 不是程序，而是一套 **Agent 技能（skill）**：以提示词/流程的形式驱动 Claude 等 Agent，按固定的多阶段、对抗式流程产出可追溯、分级标注的研究结论。

---

## 适用 / 不适用

**适用**：深度分析、跨领域推理、产业研究、技术路线对比、战略判断、物理/工程可行性评估——需要因果推理而非事实检索的场景。

**不适用**：简单事实查询、实时新闻、纯代码编写、纯数据统计，或用户明确要求"搜索"。

---

## 方法论概览

```
Phase 0  路由   ── 轻量 or 深度模式？→ 从 8 套框架选 1 套
Phase 1  深研究 ── 框架四步递进；每步含 约束追问 + 搜索 + 对抗审查 + 冲突检测 + 证据标注
Phase 2  审计   ── 可选：多厂商 Council 交叉验证（单模型自审的升级版）
Phase 3  交付   ── 结构化报告（10 章 + Mermaid 配图 + 决策刹车）
```

### 八套框架（Phase 0 按领域信号选一）

| 框架 | 信号词 | 四步结构 |
|------|--------|---------|
| 投资研究 | 估值、护城河、安全边际 | 赛道 → 竞争格局 → 商业质量 → 投资判断 |
| 战略转型 | 企业战略、数字化转型 | 战略意图 → 市场洞察 → 创新焦点 → 业务设计 |
| 技术评估 ★ | 技术路线、TRL、量产 | 技术全景 → 核心挑战 → 玩家深评 → 场景路线图 |
| 市场机会 ★ | TAM、赛道选择、进入策略 | 市场数据 → 机会筛选 → 可行性 → 进入策略 |
| 创业评估 | MVP、PMF、融资 | 问题验证 → 方案评估 → 市场验证 → 财务模型 |
| 行业研究 ★ | 产业链、竞争格局、趋势 | 行业全景 → 价值链 → 竞争动态 → 趋势预判 |
| 组织设计 | 组织架构、变革管理 | 组织诊断 → 架构选项 → 人才激励 → 变革路线图 |
| 平台/生态 | 网络效应、多边市场 | 平台定义 → 网络效应 → 竞争动态 → 赢家通吃判定 |

★ = 高频框架。详见 [`skills/research/deepsight/references/framework-catalog.md`](skills/research/deepsight/references/framework-catalog.md)。

### 七套对抗模板（Phase 1 对抗审查按问题类型选）

工程/技术 · 市场/商业/战略 · 社会科学/政策 · 市场洞察/PESTEL · 地缘/供应链安全 · 通用决策（Council 五角色）· 期望校准。
详见 [`.../references/adversarial-templates.md`](skills/research/deepsight/references/adversarial-templates.md)。

### 证据分级 + 决策刹车

每条结论强制标注证据等级：

- `[A-验证]`：搜索 + 推理双重支持，**外部独立来源可查证**（利益相关方自我确认最高只能 `[B]`）
- `[B-推理]`：推理链产出、逻辑自洽，但未经独立外部验证
- `[C-推断]`：从 B 级推导的次级结论，或 Fermi 估算 / 经验类推

强制规则：`[B]` 占比 > 70% → 退回重标；`[C]` 目标 20–35%。**决策刹车**：`[C]` 级结论禁止直接进入行动建议，降级为"待外部验证"并附翻盘条件。

### 多厂商 Council 交叉验证（Phase 2，可选）

当 `[C]` 占比 > 30%（强制）或 20–30% + 真金白银决策（建议）时触发：用**不同厂商的模型**做外部交叉验证，而非主模型自审。五个盲审角色（唱反调者 / 第一性原理 / 扩张主义者 / 局外人 / 执行者）各自只拿到"原始问题 + 结论摘要"，识别 ≥3 位独立提出的共识批评，再裁决分歧、修正证据等级。配置见 [`.../references/council-model-setup.md`](skills/research/deepsight/references/council-model-setup.md)。

---

## 两种模式

| 模式 | 适用 | 流程 |
|------|------|------|
| **轻量** | 简单对比、快速判断、单维度 | 2 轮约束追问 + 1 轮对抗审查 + 证据标注 → 直接输出 |
| **深度** | 跨领域、真金白银决策、多步骤 | 完整四步 + 可选 Council → 结构化报告 |

默认轻量；当问题涉及 ≥2 个独立维度、用户要求"深度分析"、或结论用于投资/采购/战略决策时升级为深度。

---

## 怎么用

DeepSight 以 **Agent 技能**形式运行，没有 CLI：

1. 让你的 Agent（Claude Code / 兼容 [agentskills.io](https://agentskills.io) 标准的运行时）加载 [`skills/research/deepsight/SKILL.md`](skills/research/deepsight/SKILL.md) 作为技能。
2. 抛出需要深度推理的问题；Agent 按上述 Phase 0–3 流程执行。
3. 可选基础设施：搜索（Bocha + SearXNG，搜不到不影响主流程）；Council（≥3 个不同模型族，详见配置文档）；报告交付（结构化 consulting-report 模板）。

跨会话状态由 [`skills/productivity/session-bootstrap`](skills/productivity/session-bootstrap) + `task_registry.json` + `progress.log` 维护。

---

## 仓库结构

```
skills/
  research/deepsight/        # 方法论本体
    SKILL.md                 #   主流程（Phase 0–3）
    references/              #   框架目录 / 对抗模板 / 约束链 / Council 配置 / 报告管线 / 脑暴法
  productivity/session-bootstrap/   # 跨会话编排（harness 模式）
task_registry.json           # 自我编排状态 + 版本演进
progress.log                 # 进度日志
```

---

## 状态

当前 **v2.5**。本仓库由 AI agent 维护。改进项以 GitHub Issues（`P0`/`P1`/`P2` 标签）跟踪——尤其欢迎对 Council 交叉验证严谨性（反同质化度量、共识裁决独立性）的贡献。

## License

[MIT](LICENSE) © 2026 DeepArchi (Kuang Mi)
