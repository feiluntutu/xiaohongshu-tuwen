# 公开 Skill API 摘要

飞轮 Skill API 将选题、大纲、改稿、出图和记录查询能力开放给外部 AI 工具与自动化系统。

[返回首页](../README.md) · [快速开始](quickstart.md) · [完整在线文档](https://feilun.ltyfa.top/api-docs)

> 本文用于快速理解公开能力。端点参数、实时报价和最新响应示例以[在线 API 文档](https://feilun.ltyfa.top/api-docs)为准。

## 基本信息

- Base URL：`https://feilun.ltyfa.top`
- 鉴权请求头：`X-Api-Key: v2k_...`
- 数据格式：JSON
- 公开端点：12 个
- API 与网页登录 JWT 不互通，公开端点只使用 API Key。

## 端点总览

| 分类 | 方法与路径 | 用途 |
| --- | --- | --- |
| 元信息 | `GET /api/v2/agent/health` | 健康检查、余额与 Skill 版本信息 |
| 计费 | `GET /api/v2/agent/balance` | 查询当前积分余额 |
| 计费 | `GET /api/v2/agent/ledger` | 查询 Skill API 积分流水 |
| 选题 | `POST /api/v2/agent/coach` | 生成角度、读者和标题方向候选 |
| 大纲 | `POST /api/v2/agent/outlines` | 创建大纲 record |
| 改稿 | `PUT /api/v2/agent/outlines/:recordId` | 整组替换分页内容 |
| 改稿 | `PATCH /api/v2/agent/outlines/:recordId/pages/:index` | 修改指定分页 |
| 记录 | `GET /api/v2/agent/records/:recordId` | 查询 record、分页与出图进度 |
| 稿件 | `POST /api/v2/agent/records/:recordId/to-note` | 出图完成后查询生成的稿件 |
| 出图 | `POST /api/v2/agent/images` | 确认并提交整篇异步出图 |
| 出图 | `POST /api/v2/agent/images/regenerate` | 按页报价并重出指定图片 |
| 出图 | `POST /api/v2/agent/images/cancel` | 取消出图任务 |

## 推荐调用顺序

```text
health / balance
    ↓
coach（免费选题）
    ↓
outlines（创建收费大纲，携带幂等键）
    ↓
PUT / PATCH outlines（免费改稿，携带最新版本号）
    ↓
images（明确确认报价后提交异步出图）
    ↓
GET records/:recordId（查询进度）
    ↓
POST records/:recordId/to-note（完成后获取稿件）
```

单页重出使用 `images/regenerate`；需要停止任务时使用 `images/cancel`。

## 免费与计费边界

- 健康检查、余额、流水、改稿和查询免费。
- coach 免费，每账户每滚动 24 小时 30 次，同账户多个 Key 共享额度。
- 大纲、整篇出图和单页重出按接口实时报价计费。
- 收费操作必须先明确确认，不建议在首次接入测试中自动执行。
- 出图采用异步任务，返回 `202` 不代表所有图片已经生成完成。

## 幂等约定

以下端点必须携带 `Idempotency-Key`：

- `POST /outlines`
- `POST /images`
- `POST /images/regenerate`

规则：

1. 同一次逻辑操作始终使用同一个幂等键。
2. 相同幂等键和相同参数可以安全重试或重放结果。
3. 相同幂等键用于不同参数会返回 `409 IDEMPOTENCY_CONFLICT`。
4. 请求仍在处理时返回 `429 IDEMPOTENCY_IN_PROGRESS`，按 `retryAfterSec` 或 `Retry-After` 等待。
5. 网络中断后先查询 record 与积分流水，不要直接更换幂等键重新收费。

## 乐观锁

改稿和出图可能使用 `expectedRevision` 防止覆盖其他客户端已经完成的修改。发生版本冲突时：

1. 重新读取 record。
2. 基于最新 `revision` 检查内容。
3. 再提交修改或确认出图。

不要用强制覆盖代替版本检查。

## HTTP 示例

### 健康检查

```bash
curl https://feilun.ltyfa.top/api/v2/agent/health \
  -H "X-Api-Key: $FEILUN_API_KEY"
```

### 免费选题

```bash
curl -X POST https://feilun.ltyfa.top/api/v2/agent/coach \
  -H "X-Api-Key: $FEILUN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"topic":"北京合租找房避坑","pageCount":5}'
```

### 查询积分流水

```bash
curl "https://feilun.ltyfa.top/api/v2/agent/ledger?limit=50" \
  -H "X-Api-Key: $FEILUN_API_KEY"
```

## 常见错误

| 状态或错误码 | 含义 | 建议处理 |
| --- | --- | --- |
| `400 BAD_INPUT` | 缺少参数或参数格式不正确 | 按在线文档检查请求体 |
| `400 IDEMPOTENCY_REQUIRED` | 收费端点缺少幂等键 | 为本次逻辑操作生成稳定幂等键 |
| `401 INVALID_API_KEY` | Key 无效 | 重新检查完整 Key 和本地登录凭据 |
| `403 API_KEY_REVOKED` | Key 已停用或吊销 | 创建新 Key 并重新登录 |
| `402 INSUFFICIENT_BALANCE` | 积分不足 | 查询余额并补充积分，不要自动重复收费请求 |
| `409 REVISION_CONFLICT` | record 已被其他操作更新 | 重新读取最新 revision |
| `409 IDEMPOTENCY_CONFLICT` | 同键对应了不同参数 | 恢复原请求参数；新业务使用新键 |
| `429 RATE_LIMITED` | 免费 coach 达到账户限额 | 按响应中的等待时间重试 |
| `429 IDEMPOTENCY_IN_PROGRESS` | 相同请求仍在处理 | 保留原幂等键并等待 |
| `429 DRAW_BUSY` | 当前稿件已有出图任务 | 查询进度并等待 |

更多错误和自助排查步骤见[在线接入自检](https://feilun.ltyfa.top/api-docs#self-check)与[安全指南](security.md)。
