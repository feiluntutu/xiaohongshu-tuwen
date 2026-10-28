# 快速开始

你可以直接使用飞轮网页产品，也可以把公开创作能力接入自己的 AI 工具。两种方式不要求同时完成。

[返回首页](../README.md) · [产品功能](product.md) · [API 摘要](api.md)

## 方式一：使用飞轮网页 SaaS

### 1. 注册并登录

打开[飞轮笔记官网](https://feilun.ltyfa.top/)，完成注册和登录。

### 2. 完善业务资料

进入素材库，先完善一个行业和产品档案，再按需要补充人设、知识库、内容洞察和视觉素材。

### 3. 确定创作方向

可以从以下入口开始：

- 在发现首页查看行业内容与选题灵感。
- 在内容策略中管理内容方向和每周节奏。
- 在竞品监控中研究同行账号和参考笔记。
- 在日历中直接添加或安排选题。

### 4. 制作与检查稿件

在写笔记中与 AI 完成选题、大纲、正文和配图，或将日历中的选题加入稿件。发布前仍需检查事实、表达、图片和平台规范。

### 5. 发布内容

需要自动发布时，在发布页绑定账号、选择人设并连接本机发布助手。也可以复制文案、下载图片后手动发布。

日历安排、加入稿件和连接助手本身都不等于自动发布，具体收费和发布操作以页面确认为准。

## 方式二：通过 Agent Skill 接入

### 推荐：使用线上三步接入页

1. 打开[飞轮 Skill API 文档](https://feilun.ltyfa.top/api-docs)。
2. 登录并创建 API Key，复制保存只显示一次的明文。
3. 将 Key 粘贴到第二步，点击“一键复制完整提示词”。
4. 把完整提示词发送给你信任、且具备命令执行环境的 AI 工具。

完整提示词会引导 AI 完成安装、登录、自检和第一次免费选题，不要求你逐条理解命令。

> 完整提示词包含 Key，不要发送到公开群聊、GitHub Issue 或不可信的工具。

### 手动安装

环境要求：**Node.js 20+**。

```bash
node --version
npx -y @feilunxhs/xhs init --agent auto
```

安装后，在 API 文档页创建 Key，然后执行：

```bash
npx -y @feilunxhs/xhs login
npx -y @feilunxhs/xhs doctor --agent
```

`doctor` 应能返回凭据来源、余额和 Skill 版本信息，并显示 `upgradeNeeded: false`。随后运行一次免费选题：

```bash
npx -y @feilunxhs/xhs coach '{"topic":"北京合租找房避坑","pageCount":5}'
```

成功结果应包含：

- `angles`：叙事角度候选。
- `audiences`：目标读者候选。
- `titleDirections`：标题方向候选。
- `suggestedGenre`：体裁建议。

首次验证只调用 coach，不需要生成大纲或图片，也不会扣除出图积分。

## 方式三：MCP 或直接调用 HTTP

### MCP

先按上方步骤完成本地 `login`，再将以下配置合并到支持 MCP 的 AI 工具中：

```json
{
  "mcpServers": {
    "feilun-xhs": {
      "command": "npx",
      "args": ["-y", "@feilunxhs/xhs", "mcp"]
    }
  }
}
```

MCP 配置中不要填写 Key，它会复用本地登录凭据。保存后完全重启 AI 工具，并检查是否出现飞轮的 12 个工具。

### HTTP

API Base URL：

```text
https://feilun.ltyfa.top
```

所有公开端点使用 `X-Api-Key` 鉴权。以下示例只演示健康检查，请将环境变量保存在可信的本地环境中：

```bash
curl https://feilun.ltyfa.top/api/v2/agent/health \
  -H "X-Api-Key: $FEILUN_API_KEY"
```

不要把真实 Key 直接写入代码或 Git。完整端点、参数和响应以[在线 API 文档](https://feilun.ltyfa.top/api-docs#advanced-http)为准。

## 首次接入后的建议

- 先调用 health 和 balance，确认凭据与余额。
- 再调用免费 coach，检查创作链路。
- 需要收费生成时，先阅读[安全与计费指南](security.md)。
- 大纲、出图和单页重出使用稳定的 `Idempotency-Key`。
- 出图提交后通过 record 查询异步进度。
- 遇到问题时先运行 `doctor --agent`，再查看[常见问题](faq.md)。
