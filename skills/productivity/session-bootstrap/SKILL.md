---
name: session-bootstrap
description: "Session 启动协议 — 基于 Anthropic Harness 模式，在每个新 session 开始时自动恢复上下文、读取任务清单、报告当前状态和建议下一步。解决跨 session 状态丢失问题。"
version: 1.0.0
metadata:
  based_on: Anthropic Harness for long-running agents
  triggers: session_start
  auto_load: true
---

# Session Bootstrap

> **触发条件：** 每个新 session 开始时自动执行。如果用户有明确的紧急指令，跳过 bootstrap 直接响应。

> **设计来源：** Anthropic 的两篇 Harness 博客（"Effective harnesses for long-running agents" 2025.11 + "Harness design for long-running application development" 2026.3），核心思想是 **feature list + progress log + git 做跨 session 状态传递**。

## 触发方式

Bootstrap 有两种触发方式：

1. **自动触发**：每个新 session 开始时，由 SOUL.md 指令自动触发
2. **手动触发**：用户在任何时候发送 `/bootstrap` 或「bootstrap」（独立成句），立即执行以下 checklist

两种触发执行完全相同的流程。

---

## 启动 Checklist

执行以下步骤。**不要跳过任何步骤，但每一步应该快速完成（< 30 秒）。**

### Step 1: 读取状态文件

用 `read_file` 读取以下文件：

1. **`~/.hermes/task_registry.json`** — 任务清单（Anthropic 的 feature_list 等价物）
2. **`~/.hermes/progress.log`** — 最后 5 行进度日志

```
read_file ~/.hermes/task_registry.json
read_file ~/.hermes/progress.log offset=-5  # 或用 search_files 读最后几行
```

### Step 2: 识别状态

从 task_registry.json 中提取：

- **in_progress 任务**（`status: "in_progress"`）：需要继续的任务
- **高优先级 pending 任务**（`priority ≤ 2` 且 `status: "not_started"`）
- **近期完成的任务**（`status: "completed"` + 最近 `last_session` 的任务）

### Step 3: 加载相关 Skills

根据 in_progress 任务类型，自动加载对应 skill：

| 任务 category | 应加载的 skill |
|--------------|---------------|
| `consulting` | `blm-strategy-consulting`, `consulting-report`, `deeparchi-writing` |
| `content` | `consulting-report`, `deeparchi-writing`, `contentmill` |
| `data_collection` | `bank-procurement`, `deeparchi` |
| `infrastructure` | `hermes-agent` |
| `development` | `subagent-driven-development`, `plan` |

### Step 4: 输出 Bootstrap 报告

用以下格式输出简洁的状态报告：

```
📋 Session Bootstrap — 状态恢复

🔄 进行中的任务：
  [hermes-bootstrap] Hermes Session Bootstrap 基础设施搭建
    → 当前步骤: step_2/5 (创建 progress.log ✅ → 创建 skill ⏳)

📌 待处理（高优先级）：
  [blm-jutong-strategy] 聚通云 BLM 战略咨询报告
  [deeparchi-whitepaper-cubesandbox] 白皮书: Agent沙箱技术对比

🆕 新任务？如有新任务请告知，或直接继续上述任务。

💡 建议：先完成 [hermes-bootstrap]，再启动 [deeparchi-whitepaper-cubesandbox]（素材已齐备）
```

### Step 5: 等待用户指令

输出报告后等待用户指示。不要主动开始执行任务。

---

## Session 结束时更新

在 session 结束前（用户说"结束"/"先这样"或对话自然终止时），执行以下操作：

1. **更新 task_registry.json**：修改当前任务的 step 状态，标记完成/继续
2. **追加 progress.log**：写入本 session 完成的事项摘要
3. **询问是否需要 git commit**（如果涉及代码/文档变更）

```
追加到 progress.log:
[2026-06-11 08:XX +08:00] [session_id] COMPLETED — (本session完成的事项)
```

---

## 文件格式规范

### task_registry.json

```json
{
  "version": "1.0",
  "convention": "Anthropic Harness pattern",
  "tasks": [
    {
      "id": "unique-task-id",
      "category": "consulting|development|research|content|data_collection|infrastructure",
      "title": "任务标题",
      "status": "not_started|in_progress|completed|blocked",
      "priority": 1,
      "steps": [
        {"name": "步骤名", "status": "not_started|in_progress|completed|blocked"}
      ],
      "created": "ISO_TIMESTAMP",
      "last_session": "ISO_TIMESTAMP (optional)",
      "notes": "自由文本"
    }
  ],
  "last_updated": "ISO_TIMESTAMP"
}
```

### progress.log

```
# Hermes Session Progress Log
[ISO_TIMESTAMP] [SESSION_ID] ACTION — detail
```

SESSION_ID 可以从当前上下文中获取（如果没有，使用 "unknown"）。

---

## 注意事项

- **不要在执行中修改 task_registry.json** 的 `last_updated` 以外的字段——只在任务步骤真正完成/变化时才更新
- **Bootstrap 本身不拆分为 task_registry 中的任务**——它是基础设施，不是用户任务
- **如果用户在新 session 中直接发指令**（如"帮我查一下XX"），跳过 bootstrap 直接响应
- **不要为每个对话创建任务**——只跟踪需要跨 session 的复杂任务（3+ 步骤）

## 参考

- `references/anthropic-harness-methodology.md` — Anthropic Harness 原始博客的方法论提炼，包含两篇文章的完整分析、四种失败模式、Session 启动 checklist 设计原则
