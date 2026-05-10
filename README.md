# Remote Config 运维说明

本仓库保存产品远程配置。SQL Studio 后端会同时请求 GitHub 和 Gitee raw 地址，选择最快可用源，并在本地缓存成功读取的配置。

## SQL Studio 配置

当前配置文件：

```text
sql-studio/config.json
```

线上 raw 地址：

```text
https://raw.githubusercontent.com/wangdashuaihenshuai/config/refs/heads/main/sql-studio/config.json
https://raw.giteeusercontent.com/hahwang/config/raw/main/sql-studio/config.json
```

### 配置示例

```json
{
  "version": 1,
  "activation": {
    "enabled": true,
    "server_url": "https://keygenix-sdk-api.wangdashuaihenshuai.work",
    "application_id": "7572ad9e-4c33-4d33-a18b-ba280b67fefa"
  },
  "premium": {
    "enabled": true,
    "daily_limits": {
      "ai_generate": 20,
      "mcp_tool_call": 50
    },
    "limit_message": "今日免费额度已用完，请激活后继续使用高级功能。"
  }
}
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `version` | 配置版本号，当前为 `1`。 |
| `activation.enabled` | 是否启用激活能力。为 `false` 时，高级功能不做激活和额度限制。 |
| `activation.server_url` | Keygenix SDK API 地址，不要以 `/` 结尾。 |
| `activation.application_id` | Keygenix 中 SQL Studio 应用的 ID。 |
| `premium.enabled` | 是否启用高级功能免费额度限制。为 `false` 时 AI/MCP 不限额。 |
| `premium.daily_limits.ai_generate` | 未激活用户每天可调用 `/api/ai/generate` 的次数；缺省或小于等于 0 时使用后端默认值 `20`。 |
| `premium.daily_limits.mcp_tool_call` | 未激活用户每天可调用 MCP `tools/call` 的次数；缺省或小于等于 0 时使用后端默认值 `50`。 |
| `premium.limit_message` | 超额后返回给前端展示的提示文案。 |

### 配置策略

- 基础数据库浏览、连接、查询功能不受远程配置限制。
- 远程配置只控制激活服务地址和高级功能额度。
- Ed25519 公钥不放在远程配置中，必须编译进客户端或后端代码，避免远程配置被篡改后替换验签公钥。
- SQL Studio 本地有远程配置缓存；所有远程源失败时优先使用缓存。
- 如果远程配置和缓存都不可用，SQL Studio 默认允许高级功能，不限制额度。
- SQL Studio 默认每 4 小时轮询一次远程配置，也可以通过后端接口 `POST /api/license/refresh-config` 手动刷新。

### 修改和发布流程

1. 修改配置文件。

```bash
cd /Users/wanghaha/code/sql_studio_group/config
vi sql-studio/config.json
```

2. 校验 JSON。

```bash
jq empty sql-studio/config.json
```

3. 查看变更。

```bash
git diff -- sql-studio/config.json
```

4. 提交并推送到 GitHub 和 Gitee。

```bash
git add sql-studio/config.json README.md
git commit -m "Update SQL Studio remote config"
git push origin main
git push cn-origin main
```

5. 验证 raw 地址可以访问并返回新配置。

```bash
curl -fsSL https://raw.githubusercontent.com/wangdashuaihenshuai/config/refs/heads/main/sql-studio/config.json | jq .
curl -fsSL https://raw.giteeusercontent.com/hahwang/config/raw/main/sql-studio/config.json | jq .
```

6. 需要立即生效时，在 SQL Studio 后端调用刷新接口。

```bash
curl -X POST http://127.0.0.1:8080/api/license/refresh-config
```

如果线上后端启用了 API Token，刷新请求需要带上对应认证头。
