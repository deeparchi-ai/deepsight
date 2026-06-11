# 跨模型交叉验证基础设施配置

> DeepSight Step 3c.0 的落地操作指南。记录如何添加第二个模型厂商、验证生效、以及 OpenRouter 定价参考。

## 前置诊断

```bash
# 1. 检查当前模型厂商
grep -A 3 "^model:" ~/.hermes/config.yaml
grep "^providers:" ~/.hermes/config.yaml
grep -A 3 "^delegation:" ~/.hermes/config.yaml
grep -A 3 "^fallback_model:" ~/.hermes/config.yaml

# 2. 检查可用 API keys
grep -E "KIMI|OPENROUTER|DEEPSEEK" ~/.hermes/.env | grep -v "^#"

# 3. 直接测试 API 连通性
source ~/.hermes/.env 2>/dev/null
curl -s --max-time 10 "https://api.moonshot.cn/v1/models" \
  -H "Authorization: Bearer $KIMI_CN_API_KEY"
```

## 添加第二个模型厂商

### 方案 A：已有厂商 API Key（如 Kimi）

**步骤 1：确保 .env 中有正确的 API key**

```bash
# ~/.hermes/.env
KIMI_BASE_URL=https://api.moonshot.cn/v1
KIMI_CN_API_KEY=sk-xxxx  # 从 https://platform.kimi.ai 获取
```

⚠️ 如果 .env 中有重复/损坏的 key（如多次拼接），用 `sed` 删除损坏行。

**步骤 2：在 config.yaml 注册 provider**

```yaml
# ~/.hermes/config.yaml
providers:
  kimi-coding-cn:
    base_url: https://api.moonshot.cn/v1
    key_env: KIMI_CN_API_KEY
```

**步骤 3：设置 delegation 模型**

```bash
hermes config set delegation.provider kimi-coding-cn
hermes config set delegation.model kimi-k2.6
```

**步骤 4：设置 fallback 模型（可选，主模型挂了自动切）**

```yaml
fallback_model:
  provider: kimi-coding-cn
  model: kimi-k2.6
```

### 方案 B：OpenRouter（统一网关，一个 key 通吃）

**获取 API Key：** https://openrouter.ai/keys → 创建 key → 充预付额度

**常用模型价格（per 1M tokens）：**

| 模型 | 输入 | 输出 | 上下文 |
|------|:---:|:----:|:-----:|
| `anthropic/claude-sonnet-4.6` | $3 | $15 | 1M |
| `google/gemini-2.5-pro` | $1.25 | $10 | 1M |
| `google/gemini-2.5-flash` | $0.15 | $0.60 | 1M |
| `anthropic/claude-haiku-4.6` | $1 | $5 | 200K |

平台费 5.5%（Pay-as-you-go），无最低消费。

**配置方式：**

```bash
# ~/.hermes/.env
OPENROUTER_API_KEY=sk-or-v1-xxxx

# ~/.hermes/config.yaml
# fallback_model 或 providers 中用 provider: openrouter
```

### ⚠️ 关键陷阱：delegation 需要重启 gateway

修改 `delegation.provider` 和 `delegation.model` 后，子 Agent **不会**热加载配置——继续用主模型。

**验证方法：**

```bash
# 跑一个烟雾测试 delegate_task，检查返回的 model 字段
# 如果 model 仍是主模型（如 deepseek-v4-pro）→ 未生效
# 如果 model 显示 kimi-k2.6 → 已生效
```

**解决方法：** 重启 Hermes gateway（`hermes gateway restart` 或等价命令）。

## 3c.0 路由速查

| 替代厂商数 | 路由 | 审计效果 |
|:---------:|------|---------|
| ≥3 | ✅ 正常 | 5角色错开分配不同厂商，完整交叉验证 |
| 1-2 | ⚠️ 降级 | 5审计员均独立于主模型，但共享同一替代厂商盲区 |
| 0 | ❌ 跳过 | 增强对抗性审查替代，报告标注置信度偏差 |

**当前环境（DeepSeek + Kimi）：** 1 替代厂商 → ⚠️ 降级执行。

**加 OpenRouter 后（DeepSeek + Kimi + Claude + Gemini）：** 3 替代厂商 → ✅ 正常执行。
