# designer-reception 完整 API 参考

## dingtalk-table MCP

工具参数与 designer-branding 相同，参见 branding-api.md。额外工具：

### get_tables
- **参数**: `baseId` (必填)
- **返回**: `{ tables: [{ tableId, name }] }`

### update_records
- **参数**: `baseId` (必填), `tableId` (必填), `records` (必填)
- **格式**: `[{ "id": "rec001", "fields": { "状态": "已回复" } }]`
- **返回**: `{ success: true }`

## dingtalk-robot MCP

### 机器人批量发送单聊消息
> 工具名为钉钉 MCP 原始命名

- **参数**:
  - `robotCode` (必填): 机器人 Code
  - `userIds` (必填): 用户 ID 数组
  - `msgType` (必填): "text" / "markdown"
  - `msgContent` (必填): 消息内容 JSON
- **返回**: `{ success: true }`

## dingtalk-doc MCP

参见 branding-api.md，工具相同。
