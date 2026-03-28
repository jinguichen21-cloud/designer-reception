---
name: designer-reception
description: 自动接待设计师各渠道的客户咨询，监控社媒消息并生成结构化设计Brief。当用户需要开启接单、接待客户、管理渠道、查看咨询、监控小红书抖音消息时使用。不要在品牌建设、设计出图、交付管理时触发。
---

## 你绝对不能做什么

- **禁止安装、配置、启动 MCP 服务器** — 不要执行"查找MCP""添加MCP""启动MCP"操作
- **禁止安装 Python 依赖、Playwright、创建虚拟环境**
- **禁止创建本地文件存数据** — 数据必须存入 AI 表格
- **禁止输出安装指南、配置清单、环境检查报告、替代方案列表**
- **禁止模拟执行** — 每一步都必须真正执行命令
- **禁止AI直接向社媒平台发送任何内容**（评论、私信、发帖）— AI 只生成回复建议，引导用户去平台手动回复
- 不要代替用户承诺价格、工期或交付内容
- 不要向客户透露用户使用 AI 接待的事实
- 不要编造设计师的过往案例或能力
- 不要删除或修改客户原始咨询消息

## 调用策略

钉钉能力优先用 `dws` CLI（Bash 执行），不可用时降级为 MCP 工具。首次失败后全局切换。

社媒监控通过 `browser-use`（`navigate_page` 等浏览器工具）实现，不依赖 social-inbox。

## 工具总览

### 钉钉能力

| 操作 | dws 命令（优先） | 降级 MCP 工具 |
|------|-----------------|--------------|
| 搜索 AI 表格 | `dws aitable base search --query <关键词>` | `search_bases(keyword)` |
| 创建 Base | `dws aitable base create --name <名称>` | `create_base(name)` |
| 获取 Base 详情 | `dws aitable base get --base-id <ID>` | `get_base(baseId)` |
| 创建数据表 | `dws aitable table create --base-id <ID> --name <名> --fields '<fields JSON>'` | `create_table(baseId, name, fields)` |
| 获取表详情(含 fieldId) | `dws aitable table get --base-id <ID>` | `get_tables(baseId)` |
| 新增记录 | `dws aitable record create --base-id <ID> --table-id <ID> --records '<JSON>'` | `create_records(baseId, tableId, records)` |
| 更新记录 | `dws aitable record update --base-id <ID> --table-id <ID> --records '<JSON>'` | `update_records(baseId, tableId, records)` |
| 查询记录 | `dws aitable record query --base-id <ID> --table-id <ID>` | `query_records(baseId, tableId)` |
| 获取文档根目录 | `dws doc space get-root` | `get_my_docs_root_dentry_uuid()` |
| 创建文档 | `dws doc file create --parent <UUID> --name <名称>` | `create_doc_under_node(parentDentryUuid, name)` |
| 写入文档 | `dws doc file write --id <UUID> --content <内容> --mode 0` | `write_content_to_document(targetDentryUuid, content, updateType)` |
| 搜索文档 | `dws doc file search --keyword <关键词>` | `list_accessible_documents(keyword)` |
| 获取用户信息 | `dws contact user get-self` | `get_current_user_profile()` |

### 社媒监控（browser-use 浏览器工具）

通过 agent 内置的 `browser-use` 浏览器工具访问社媒平台页面，提取客户咨询内容。

| 操作 | 工具 |
|------|------|
| 打开平台页面 | `navigate_page(url="...")` |
| 提取页面内容 | 读取页面 DOM / 截图分析 |
| 用户确认 | `ask_human` |

### 支持的社媒平台

| 平台 | 登录 URL | 巡检页面（需逐个访问） |
|------|---------|-----------|
| 小红书 | https://www.xiaohongshu.com | 通知: https://www.xiaohongshu.com/notification |
| 抖音 | https://www.douyin.com | 评论: https://creator.douyin.com/creator-micro/interactive/comment ，私信: https://creator.douyin.com/creator-micro/data/following/chat |
| 微博 | https://weibo.com | 消息: https://weibo.com/messages |
| 花瓣网 | https://huaban.com | 通知: https://huaban.com/notifications |

## 开启接单：依次执行以下步骤

**Step 1** — 初始化 AI 表格

执行 `dws aitable base search --query "设计团队管理" --format json`
- 找到 → 提取 baseId，执行 `dws aitable base get --base-id <ID> --format json` 获取 tableId
- 没找到 → 创建 Base + 3 张表（渠道管理、接待任务、咨询记录）

**Step 2** — 选择监控渠道

调用 `ask_human(question="请选择需要接待的渠道（可多选）：小红书、抖音、微博、花瓣网、其他")`

如果用户选择"其他"，继续 `ask_human` 收集平台名称和消息页 URL。

**Step 3** — 浏览器登录各渠道

对每个选定渠道，通过 `browser-use` 打开登录页让用户登录：

```
navigate_page(url="https://www.xiaohongshu.com")
```

打开后调用 `ask_human(question="请在浏览器中完成小红书登录，登录好后点确认")`

用户确认后，将渠道信息写入渠道管理表（状态=运行中）。

**逐个渠道重复此步骤**，直到所有选定渠道都登录完毕。

**Step 4** — 启动定时巡检

调用 `ask_human(question="请选择巡检频率：每5分钟 / 每10分钟 / 每15分钟 / 每30分钟")`

使用 agent 的定时任务能力（如 `use_cron`）创建轮询任务。

**Step 5** — 定时巡检循环体（由定时任务触发）

每轮巡检执行以下步骤：

```
# 1. 逐个访问已配置渠道的消息页
navigate_page(url="https://www.xiaohongshu.com/notification")
# 提取页面中的新评论、@提及、私信内容

navigate_page(url="https://creator.douyin.com/creator-micro/interactive/comment")
# 提取页面中的新评论

navigate_page(url="https://creator.douyin.com/creator-micro/data/following/chat")
# 提取页面中的新私信

# 2. 与 AI 表格已有记录对比去重
dws aitable record query --base-id <ID> --table-id <咨询记录表ID> --format json

# 3. 对每条新咨询：
#    a. AI 基于知识库生成回复建议（标记【AI建议回复】）
#    b. 写入咨询记录表
#    c. 通知用户有新咨询，给出回复建议，引导用户去相应平台回复

# 4. 如发现意向客户 → ask_human 确认 → 创建 Brief
```

**Step 6** — 处理新咨询并写入表格

**写入前必须清洗数据，严格遵守以下规则：**

1. **客户名称** — 必须是对方的真实昵称/用户名，不能是序号、ID、"unknown" 或无意义文本

2. **咨询内容** — 必须且只能是**客户发送的消息**，不能包含设计师自己发的回复

3. **一条记录 = 一个客户** — 不要把不同客户的消息混入同一条记录

4. **写入前用 ask_human 确认** — 展示即将写入的数据让用户确认：
   > 即将录入咨询记录：
   > - 客户名称：xxx
   > - 渠道：douyin
   > - 咨询内容：xxx（客户原话）
   > 确认录入？

确认后执行（先用 `dws aitable table get` 获取 fieldId）：
```
dws aitable record create --base-id <ID> --table-id <ID> --records '[{"cells":{"<客户名称fieldId>":"<客户真实昵称>","<渠道fieldId>":"<platform>","<咨询内容fieldId>":"<仅客户发送的消息>","<回复内容fieldId>":"【AI建议回复】...","<状态fieldId>":"待跟进"}}]'
```

**Step 7** — 意向客户转设计任务

客户表达明确设计意向时：
- 调用 `ask_human` 确认是否创建 Brief
- 创建结构化 Brief 文档：`dws doc space get-root` → `dws doc file create --parent <UUID> --name "Brief-{客户名}"` → `dws doc file write`
- 引导用户使用 designer-studio 进行设计实施

## 其他操作

**用户说"查看咨询/客户消息":**
→ 通过 `navigate_page` 访问对应平台消息页
→ 或查历史: `dws aitable record query --base-id <ID> --table-id <ID> --format json`

**用户口述客户咨询:**
→ 从用户消息中提取: 客户名称、渠道、咨询内容 → 生成回复建议 → 写入 AI 表格

**用户说"创建 Brief":**
→ `dws doc space get-root` → `dws doc file create` → `dws doc file write`

**用户说"停止接单":**
→ 停止定时巡检任务，更新渠道管理表状态为"已停用"

**关键区分**: designer-reception="接待+生成Brief" vs designer-studio="设计实施"

## 上下文传递

| 操作 | 从返回中提取 | 用于 |
|------|-------------|------|
| dws aitable base search / create | **baseId** | 所有 table 操作 |
| dws aitable base get / table get | **tableId**, **fieldId** | record create, record query |
| navigate_page + 页面提取 | 客户名称、咨询内容 | 写入咨询记录表 |
| dws doc file create(Brief) | **dentryUuid** = briefId | dws doc file write |
| dws doc space get-root | **rootDentryUuid** | dws doc file create --parent |

## 数据格式

dws record create/update 使用 `--records` 参数，每条记录用 `cells` 包装，key 是 **fieldId**（不是字段名）。

使用前必须先通过 `dws aitable table get --base-id <ID> --format json` 获取字段列表，拿到每个字段的 `fieldId`。

```bash
# [正确] 用 fieldId 作为 key
dws aitable record create --base-id <ID> --table-id <ID> --records '[{"cells":{"fldXXX1":"张三","fldXXX2":"xiaohongshu","fldXXX3":"想做Logo","fldXXX4":"待跟进"}}]'

# [错误] 用字段名作为 key
dws aitable record create --base-id <ID> --table-id <ID> --records '[{"cells":{"客户名称":"张三"}}]'

# [错误] 用 fields 代替 cells
dws aitable record create --base-id <ID> --table-id <ID> --records '[{"fields":{"fldXXX":"张三"}}]'
```

**降级 MCP 工具**（`create_records`）使用 `fields` + 字段名，与 dws 不同：
```json
create_records(baseId, tableId, records=[{"fields": {"客户名称": "张三", "渠道": "xiaohongshu"}}])
```

## 错误处理

- dws 命令返回认证错误 → 提示用户运行 `dws auth login`
- `navigate_page` 页面加载失败 → 提示用户检查网络连接
- 浏览器登录超时 → 重新执行 `navigate_page` 让用户登录
- 页面内容提取为空 → 可能页面结构变化，提示用户手动查看

**禁止建议用户调整系统权限。禁止输出替代方案列表。**

## 详细参考

- [references/table-schema.md](./references/table-schema.md) — AI 表格存储结构
- [references/reception-api.md](./references/reception-api.md) — 完整参数说明
- [references/error-codes.md](./references/error-codes.md) — 错误码表
