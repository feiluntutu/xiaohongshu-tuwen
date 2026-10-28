# 常见问题

[返回首页](../README.md) · [快速开始](quickstart.md) · [在线自检](https://feilun.ltyfa.top/api-docs#self-check)

## 这个仓库包含飞轮 SaaS 源码吗？

不包含。本仓库只用于公开产品介绍、使用指南、演示素材和 API 接入说明。飞轮前端、后端、内部 Prompt、完整 Skill 内容和部署实现均不在仓库中。

## 一定要使用某一种 AI 工具吗？

不需要。你可以使用支持 Agent Skill 或 MCP 的 AI 工具，也可以直接通过 HTTP 调用。线上三步接入页提供完整提示词，不要求按工具品牌分别选择教程。

## 安装后 AI 工具找不到 Skill 怎么办？

确认使用 Node.js 20+，重新执行：

```bash
npx -y @feilunxhs/xhs init --agent auto
npx -y @feilunxhs/xhs doctor --agent
```

然后完全重启 AI 工具。如果工具在容器、远程环境或另一台设备运行，需要在实际运行环境中安装并登录。

## `login` 输入 Key 时看不到字符正常吗？

正常。密码式输入通常不会显示字符。粘贴完整 Key 后按回车即可。不要使用中文省略号、额外空格或不完整的 `v2k_` 前缀代替真实 Key。

## `doctor` 显示需要升级怎么办？

按照 `doctor` 输出和[在线文档](https://feilun.ltyfa.top/api-docs)中的最新命令更新公开 npm 包，再重新执行自检。不要为了绕过版本提示修改服务器地址或伪造版本号。

## MCP 中可以直接填写 API Key 吗？

不建议。先在 MCP 实际运行环境完成 `npx -y @feilunxhs/xhs login`，配置文件只保留 `npx ... mcp` 启动命令，让它复用本地凭据。

## 免费 coach 为什么返回 429？

免费 coach 限额是每账户每滚动 24 小时 30 次，同一账户的多个 Key 共享。查看响应中的 `retryAfterSec` 或 `Retry-After`，等待后再试，不要通过新建 Key 绕过账户限额。

## 大纲或出图请求超时后应该怎么办？

先保留原 `Idempotency-Key`，查询 record 和积分流水。若仍在处理，按照 `retryAfterSec` 等待并使用原键重试。不要直接更换幂等键创建新收费任务。

## 为什么出图请求已经返回，但还没有全部图片？

出图是异步任务。`202` 表示任务已受理，需要继续通过 `GET /records/:recordId` 查询每页进度和最终结果。

## 日历安排内容后会自动出图或发布吗？

不会。日历用于安排选题，加入稿件后再进行制作、审稿与发布确认。实际生成费用会在制作时提示，发布还需要相应账号、权限和确认操作。

## 不使用发布助手可以发布吗？

可以在稿件中复制文案、下载图片后手动发布。自动发布需要按页面指引绑定账号、选择人设并连接本机发布助手。

## 如何反馈文档或接入问题？

先运行：

```bash
npx -y @feilunxhs/xhs doctor --agent
```

再对照[在线自检指南](https://feilun.ltyfa.top/api-docs#self-check)。需要提交 GitHub Issue 时，只提供脱敏后的步骤和错误码，不要附带 Key、JWT、账号信息或用户稿件。
