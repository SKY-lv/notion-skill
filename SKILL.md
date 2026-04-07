---
name: notion-skill
description: "Notion API 集成助手。操作 Notion 数据库、页面、块。触发词：notion、数据库、笔记。"
metadata: {"openclaw": {"emoji": "📝"}}
---

# Notion Skill

## 功能说明

与 Notion API 深度集成，操作数据库、页面、块。

## 前置配置

1. 在 [Notion Developers](https://www.notion.so/my-integrations) 创建集成
2. 获取 Internal Integration Token
3. 分享数据库/页面给集成

## 核心工具

### 1. 创建页面

```javascript
async function createPage(databaseId, properties, content) {
  const response = await fetch('https://api.notion.com/v1/pages', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer ' + process.env.NOTION_TOKEN,
      'Notion-Version': '2022-06-28',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      parent: { database_id: databaseId },
      properties: properties,
      children: content ? [{ object: 'block', type: 'paragraph', paragraph: { rich_text: [{ type: 'text', text: { content } }] } }] : []
    })
  });
  return response.json();
}
```

### 2. 查询数据库

```javascript
async function queryDatabase(databaseId, filter, sorts) {
  const response = await fetch(`https://api.notion.com/v1/databases/${databaseId}/query`, {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer ' + process.env.NOTION_TOKEN,
      'Notion-Version': '2022-06-28',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ filter: filter || {}, sorts: sorts || [] })
  });
  return response.json();
}
```

### 3. 更新页面属性

```javascript
async function updatePage(pageId, properties) {
  const response = await fetch(`https://api.notion.com/v1/pages/${pageId}`, {
    method: 'PATCH',
    headers: {
      'Authorization': 'Bearer ' + process.env.NOTION_TOKEN,
      'Notion-Version': '2022-06-28',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ properties })
  });
  return response.json();
}
```

### 4. 添加块内容

```javascript
async function appendBlocks(pageId, children) {
  const response = await fetch(`https://api.notion.com/v1/blocks/${pageId}/children`, {
    method: 'PATCH',
    headers: {
      'Authorization': 'Bearer ' + process.env.NOTION_TOKEN,
      'Notion-Version': '2022-06-28',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ children })
  });
  return response.json();
}
```

## 常用属性类型

| 类型 | 格式 |
|------|------|
| title | `{ title: [{ text: { content: "标题" } }] }` |
| rich_text | `{ rich_text: [{ text: { content: "文本" } }] }` |
| number | `{ number: 42 }` |
| select | `{ select: { name: "选项" } }` |
| multi_select | `{ multi_select: [{ name: "A" }, { name: "B" }] }` |
| date | `{ date: { start: "2026-04-01" } }` |
| checkbox | `{ checkbox: true }` |
| url | `{ url: "https://example.com" }` |
| email | `{ email: "test@example.com" }` |
| phone_number | `{ phone_number: "+86 13800000000" }` |

## 常用过滤条件

```javascript
// 按标题过滤
{ property: "Name", title: { contains: "关键词" } }

// 按select过滤
{ property: "Status", select: { equals: "Done" } }

// 按日期过滤
{ property: "Date", date: { after: "2026-04-01" } }

// 组合过滤
{ and: [
  { property: "Status", select: { equals: "Active" } },
  { property: "Priority", select: { equals: "High" } }
]}
```

## 常见用例

### 日程管理
- 创建每日任务页面
- 自动归档已完成项目
- 生成周报汇总

### 知识库
- 批量创建笔记模板
- 自动分类整理
- 搜索和检索

### CRM
- 客户跟进记录
- 销售漏斗管理
- 自动发送提醒

## 错误处理

```javascript
if (!response.ok) {
  const error = await response.json();
  throw new Error(`Notion API Error: ${error.message}`);
}
```

## 限制

- 每个请求不超过 30 秒
- 批量操作建议 100 条/批
- Rate limit: 3 requests/second
