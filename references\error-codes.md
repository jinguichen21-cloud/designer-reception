# designer-reception 错误码表

| 错误码 | 场景 | 原因 | 修复建议 |
|--------|------|------|---------|
| RC_001 | social-inbox MCP 未配置 | 内置 MCP Server 未初始化 | cd social-inbox-mcp && uv sync && uv run playwright install chromium |
| RC_002 | login_platform 超时 | 用户未扫码或 cookie 过期 | 提示重新扫码 |
| RC_003 | knowledgeBaseId 不存在 | 用户未完成品牌建设 | 引导先使用 designer-branding |
| RC_004 | start_watching interval < 30 | 轮询间隔太短 | 设置 interval >= 30 秒 |
| RC_005 | check_notifications 无数据 | 平台无新通知 | 正常状态，继续监控 |
| RC_006 | send_reply 失败 | 平台限制或登录过期 | 重新登录后重试 |
| RC_007 | create_base 失败 | dingtalk-table MCP 未配置 | 提示安装 MCP |
| RC_008 | 平台不支持 | 当前仅支持小红书和抖音 | 提示支持的平台列表 |
| RC_009 | 机器人发消息失败 | robotCode 无效或 MCP 未配置 | 检查 dingtalk-robot 配置 |
