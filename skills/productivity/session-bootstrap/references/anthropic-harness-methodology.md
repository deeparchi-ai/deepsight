# Anthropic Harness 方法论 — 原始资料与设计原则

> 来源：Anthropic Engineering Blog 两篇文章的完整分析。这些是 session-bootstrap skill 的理论基础。

## 文章一：Effective harnesses for long-running agents (2025.11.26)

**作者：** Justin Young  
**来源：** https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents

### 核心问题

Agent 跨多个 context window 工作时，每个新 session 都没有前一个 session 的记忆。就像轮班工程师，每个新来的人不知道上一个人做了什么。

### 两种 Agent 角色

| 角色 | 职责 | 何时运行 |
|------|------|---------|
| **Initializer Agent** | 搭建环境骨架：feature_list.json、init.sh、初始 git commit、progress 文件 | 仅第一次 session |
| **Coding Agent** | 每次 session 读取 artifacts → 挑一个 feature → 实现 → 验证 → 提交 → 更新 progress | 每次后续 session |

### 四个关键组件

1. **Feature List (JSON)**: 从用户 prompt 展开的完整功能清单，每项标记 `passes: true/false`。用 JSON 而非 Markdown，因为模型更不容易意外修改 JSON 文件。
2. **Progress File**: 每次 session 结束时的进度摘要，让下一个 session 的 agent 能快速了解状态。
3. **Git + 描述性 commit**: 可回滚的代码状态。Agent 被要求 commit 进度，以便出错时用 `git revert` 恢复。
4. **init.sh**: 一键启动开发环境的脚本，让 agent 不用花时间摸索如何跑起来。

### 四种失败模式与解法

| 失败模式 | 解法 |
|---------|------|
| Agent 过早宣布整个项目完成 | Feature list 明确标出所有功能，Agent 必须逐个完成 |
| Agent 留下 buggy/未文档化的状态 | 结束前 commit + progress update；下个 session 先跑 init.sh 验证 |
| Agent 标记 feature 完成但未经测试 | 强制使用端到端测试工具（如 Puppeteer MCP 浏览器自动化） |
| Agent 花时间摸索如何运行 | init.sh 包含完整启动指令 |

### Session 启动 checklist（每个 Coding Agent 的标准流程）

```
1. pwd — 确认工作目录
2. git log --oneline -20 — 查看最近改动
3. read claude-progress.txt — 读进度摘要
4. read feature_list.json — 选一个未完成的 feature
5. ./init.sh — 启动环境
6. 跑基本端到端测试 — 验证环境没坏
7. 开始实现选中的 feature
```

### 关键洞察

- **Incremental progress**: 每次只做一个 feature，做完提交
- **Clean state before exit**: 结束前必须让代码处于"可以合并到 main"的状态
- **Self-verify**: Agent 必须自己验证 feature 通过，不能假设自己写对了
- **多 Agent 架构仍然开放**: 文章结尾提到"还不清楚单个通用 agent 好还是多 agent 架构更好"

---

## 文章二：Harness design for long-running application development (2026.03.24)

**来源：** https://www.anthropic.com/engineering（engineering 页面可见，标题为 "Harness design for long-running application development"，但因 Cloudflare 防护无法获取全文）

### 从工程页面的上下文推断

这篇文章是 11 月文章的**升级版**，从"经验总结"走向"工程方法论"。发布时间紧邻 "Auto mode" 文章（次日），表明 Anthropic 正在将 Harness 从「经验模式」升级为「可编程的自动化模式」。

### 与 Dynamic Workflow 概念的关联

杨博士视频提到的 "Dynamic Workflow" 是 Harness 的第三次认知升级：

| 阶段 | 范式 | 时间 |
|------|------|------|
| v1 权限白名单 | 静态规则 | 2025 前 |
| v2 Hooks 拦截 | 可编程规则 | 2025 |
| v3 Auto Mode + Harness | 自动决策 + 静态 harness | 2026.03 |
| v4 Dynamic Workflow | **上下文感知的动态约束** | 2026.06 |

---

## 对 session-bootstrap 的设计启示

| Anthropic 原则 | session-bootstrap 实现 |
|---------------|----------------------|
| Feature List 做共享状态 | `task_registry.json` — JSON 格式任务清单 |
| Incremental progress | 每次 session 只推进一个 step |
| Clean state before exit | Session 结束前更新 task_registry + progress.log |
| Session 启动 checklist | Bootstrap Step 1-2: 读 task_registry → 读 progress → 报告状态 |
| Self-verify before marking done | 完成任务前验证，只标记真正完成的 step |
| 多 Agent 架构 | 不同 skill/Agent 读同一份 task_registry，各取所需 |

## 关键差异

Anthropic 的 Harness 是为 **Claude Agent SDK 的代码编写场景** 设计的，强调 git + 测试 + 环境管理。  
session-bootstrap 是为 **Hermes 的通用任务场景** 设计的，强调任务状态追踪 + 跨 session 上下文恢复 + skill 自动加载。
