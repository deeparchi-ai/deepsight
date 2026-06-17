# LLM Council 多模型配置实战指南

> 本文记录 DeepSight v2.3.0 Step 3c 跨模型审计的基础设施配置流程。
> 更新于 2026-06-10。

## 3c.0 路由决策速查（基础设施层）

> 本表只管"用几个厂商怎么接"。**审计严谨性规则——多样性度量、共识定义、裁决独立、降级对结论证据级的影响——见 [`council-protocol.md`](council-protocol.md)，以那份为准。**

按**独立模型族数 D**（按 model family，不是 base_url/厂商名；同族不同尺寸不计）路由：

| D（独立模型族） | 路由 | 效果 + 对结论证据级 |
|-----------|------|------|
| ≥3 | ✅ 正常 | 各角色错开不同模型族 → 完整交叉验证，可支撑 `[A]` |
| 2 | ⚠️ 部分 | 覆盖有限，结论证据级**封顶 `[B]`**，报告标注"双族审计" |
| 1 | ⚠️ 单族 | 审计员共享盲区，封顶 `[B]` 且标"单模型族·盲区风险高"，**不得宣称"已交叉验证"** |
| 0 | ❌ 跳过 | 增强单模型对抗替代，结论标"未经跨模型验证" |

## 配置步骤

### 1. 注册第二厂商 Provider

编辑 `~/.hermes/config.yaml`，在 `providers` 段注册（初始为 `{}`）：

```yaml
providers:
  kimi-coding-cn:
    base_url: https://api.moonshot.cn/v1
    key_env: KIMI_CN_API_KEY
```

**关键：** `key_env` 指向 `.env` 中的环境变量名，API key 不在 config 中硬编码。

### 2. 配置 Delegation（子 Agent 模型）

```yaml
delegation:
  model: kimi-k2.6
  provider: kimi-coding-cn
```

子 Agent 默认继承主模型。设置 delegation 后所有 delegate_task 子 Agent 使用替代模型。

### 3. 配置 Fallback（主模型故障切换）

```yaml
fallback_model:
  provider: kimi-coding-cn
  model: kimi-k2.6
```

### 4. 添加第三个厂商（达成 ≥3 正常路由）

按同样模式注册新 provider：

```yaml
providers:
  kimi-coding-cn:
    base_url: https://api.moonshot.cn/v1
    key_env: KIMI_CN_API_KEY
  anthropic:                    # 示例：直连 Anthropic
    base_url: https://api.anthropic.com/v1
    key_env: ANTHROPIC_API_KEY
```

或在 delegation 不可切多模型时，用 OpenRouter 一个 key 通吃：

```yaml
providers:
  openrouter:
    base_url: https://openrouter.ai/api/v1
    key_env: OPENROUTER_API_KEY
```

## 常见陷阱

### .env Key 损坏

症状：配置正确但 API 调用失败（403/401）。

原因：`.env` 中同一变量定义了两次，或被重复拼接。当前案例：第 55 行正确的 `KIMI_CN_API_KEY` 被第 417 行损坏值覆盖。

检查：`grep -n "KEY_NAME" ~/.hermes/.env` — 每个 key 只能出现一次。

修复：`sed -i 'LINE_NUMBERs/.*/# removed corrupted/' ~/.hermes/.env`

### Delegation 不生效

症状：子 Agent 仍使用主模型（`delegate_task` 返回的 `model` 字段未变）。

原因：Provider 未在 `providers` 段注册，或 gateway 未重启。

修复：
1. 确认 `providers` 段注册了新 provider（不能只是 `{}`）
2. 重启 Hermes gateway
3. 用 `delegate_task` 测试（子 Agent 返回的 `model` 应显示替代模型名）

### 测试 API 连通性

```bash
# 直接测试 Kimi
source ~/.hermes/.env
curl -s --max-time 15 "https://api.moonshot.cn/v1/models" \
  -H "Authorization: Bearer $KIMI_CN_API_KEY"

# 测试推理
curl -s --max-time 15 "https://api.moonshot.cn/v1/chat/completions" \
  -H "Authorization: Bearer $KIMI_CN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"kimi-k2.6","messages":[{"role":"user","content":"回复就绪"}],"max_tokens":10}'
```

## OpenRouter 免费模型（可用于 Council）

OpenRouter 免费层有多个不同厂商的免费模型，不需充值即可用于审计：

| 模型 ID | 厂商 | 上下文 |
|---------|------|--------|
| `moonshotai/kimi-k2.6:free` | Moonshot | 262K |
| `nvidia/nemotron-3-ultra-550b-a55b:free` | NVIDIA | 1M |
| `nvidia/nemotron-3-super-120b-a12b:free` | NVIDIA | 1M |
| `qwen/qwen3-coder:free` | Alibaba | 1M |
| `google/gemma-4-31b-it:free` | Google | 262K |
| `google/gemma-4-26b-a4b-it:free` | Google | 262K |
| `meta-llama/llama-3.3-70b-instruct:free` | Meta | 131K |
| `openai/gpt-oss-120b:free` | OpenAI | 131K |
| `z-ai/glm-4.5-air:free` | Zhipu | 131K |

**限制：** 免费层 50 reqs/day。Council 偶尔跑一次足够。

**搭配策略：** 主推理用 DeepSeek（付费），审计用 OpenRouter 免费模型（不同厂商 = 不同训练数据 = 真交叉验证），delegation/fallback 用直连 Kimi（兜底）。

## 当前配置状态 (2026-06-10)

**Providers 已注册：** `deepseek` + `kimi-coding-cn` + `openrouter`

```yaml
# ~/.hermes/config.yaml
providers:
  kimi-coding-cn:
    base_url: https://api.moonshot.cn/v1
    key_env: KIMI_CN_API_KEY
  openrouter:
    key_env: OPENROUTER_API_KEY
delegation:
  model: kimi-k2.6
  provider: kimi-coding-cn
fallback_model:
  provider: openrouter
  model: google/gemma-4-31b-it:free
```

```
主推理:       DeepSeek V4 Pro        付费
Delegation:   Kimi K2.6 (直连)      付费
Fallback:     Gemma 4 31B (OR免费)   $0
OpenRouter:   余额 $0, Claude 需充值
```

## 实战验证过的 Council 阵容 (5厂商 → 3c.0 ✅)

| 角色 | 模型 | 厂商 | 成本 | 验证状态 |
|------|------|------|:--:|:--:|
| 主推理 | DeepSeek V4 Pro | DeepSeek | 付费 | ✅ |
| 唱反调者 | Kimi K2.6 (直连) | Moonshot | 付费 | ✅ |
| 局外人 | `google/gemma-4-31b-it:free` | Google | $0 | ✅ 实测通 |
| 第一性原理 | `openai/gpt-oss-120b:free` | OpenAI | $0 | ✅ 实测通 |
| 扩张主义者 | `nvidia/nemotron-3-ultra-550b-a55b:free` | NVIDIA | $0 | ✅ 实测通 |
| 执行者 | Kimi K2.6 (直连复用) | Moonshot | 付费 | ✅ |

## OpenRouter 免费模型实测 (2026-06-10 二次验证)

2026-06-10 初次实测后，同日二次验证发现模型可用性已变化：

| 模型 ID | 初次(上午) | 二次(下午) | 备注 |
|---------|:--:|:--:|------|
| `google/gemma-4-31b-it:free` | ✅ | ❌ 空响应 | 可能限流或已下架 |
| `nvidia/nemotron-3-ultra:free` | ✅ | ❌ 400 模型ID无效 | 模型ID可能已变更 |
| `openai/gpt-oss-120b:free` | ✅ | ✅ 但超时(60s+) | 可用但响应极慢 |
| `moonshotai/kimi-k2.6:free` | ⚠️ 429 | ⚠️ 429 | 持续限流 |
| `qwen/qwen3-coder:free` | ⚠️ 429 | ⚠️ 429 | 持续限流 |

**关键教训：** 免费模型可用性高度不稳定——同一天上午和下午就可能变化。Council 执行前必须用 curl 快速测试，不要依赖文档记录的可用状态。当前实际可用的独立厂商免费模型可能只有 0-1 个（GPT-OSS 超时），Council 执行时优先使用付费模型（Kimi 直连）作为替代厂商。

使用 `curl` 调用 `https://openrouter.ai/api/v1/chat/completions` 逐一测试：

| 模型 ID | 结果 | 备注 |
|---------|:--:|------|
| `google/gemma-4-31b-it:free` | ✅ | 262K ctx, 可靠 |
| `nvidia/nemotron-3-ultra:free` | ✅ | 1M ctx, 推理强 |
| `openai/gpt-oss-120b:free` | ✅ | 131K ctx |
| `moonshotai/kimi-k2.6:free` | ❌ 429 | 限流，偶发可用 |
| `qwen/qwen3-coder:free` | ❌ 429 | 限流，偶发可用 |
| `meta-llama/llama-3.3-70b:free` | ❌ 429 | 限流 |
| `google/gemma-4-26b-a4b-it:free` | ❌ 429 | 限流 |

**发现免费模型方法：** `curl https://openrouter.ai/api/v1/models | python3 -c "..."` 过滤 `prompt=0, completion=0`

**关键教训：** 免费模型 429 概率高，Council 执行时应先用 curl 快速测试可用性，只选当时通的。限流过了可能又通。

## Vision 模型配置修复

`auxiliary.vision.model: moonshot-v1-32k` 不支持图片输入。修复：
```bash
hermes config set auxiliary.vision.model moonshot-v1-128k-vision-preview
```
但 vision_analyze 调用仍可能缓存旧模型，需 gateway 重启生效。临时方案：直接用 Python+curl 调 Kimi vision API（base64 图片需写入文件后 `@file` 传参，命令行直接传会 Argument list too long）。

---

> ⚠️ **本文件的型号列表是易过期快照**（免费层可用性以小时计变化）。选型应按**能力声明**而非写死型号——见 [`council-capability-spec.md`](council-capability-spec.md)（厂商无关的审计席位要求 + 运行时选型流程）。严谨性规则见 [`council-protocol.md`](council-protocol.md)。
