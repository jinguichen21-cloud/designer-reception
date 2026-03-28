# designer-reception

自动接待设计师各渠道的客户咨询，监控社媒消息并生成结构化设计 Brief。

## 快速开始

1. 安装 MCP: dingtalk-table, dingtalk-doc, dingtalk-robot
2. 安装内置 social-inbox MCP: `cd social-inbox-mcp && uv sync && uv run playwright install chromium`
2. 对 AI 说："帮我开启自动接单" 或 "监控小红书消息"
3. 选择渠道 → 扫码登录 → 自动监控 → 生成 Brief

## 核心能力

- 多渠道接待（钉钉/小红书/抖音/微博等）
- 社媒消息自动监控和通知
- AI 智能回复建议
- 咨询记录沉淀到 AI 表格
- 自动生成结构化设计 Brief

## 依赖

- dingtalk-table/doc/robot: 数据存储和通知
- social-inbox: 社媒平台监控（内置于 ./social-inbox-mcp/）
