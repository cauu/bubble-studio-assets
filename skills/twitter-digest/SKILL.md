---
name: twitter-digest
description: 定期抓取指定 Twitter 账号的推文，过滤低质量内容，智能判断相关性，生成主题摘要并存入笔记系统。支持每日/每周/每月频率，可手动触发或 cron 自动执行。
---

# Twitter Digest Skill

抓取 Twitter 推文，过滤、分析、归档到笔记系统。

## 前置依赖

- Nitter 实例（自建或公共）
- 笔记系统（如 Obsidian vault）
- Git（用于同步）

## 触发方式

- 手动：「抓取 [主题名] 推文」「运行 twitter digest [主题]」
- 自动：cron job 分批次执行

## 配置文件

主题配置：`skills/twitter-digest/topics.json`

```json
{
  "ExampleTopic": {
    "handles": ["user1", "user2", "..."],
    "frequency": "daily",
    "triggerTime": "22:00",
    "description": "示例主题日报",
    "postProcess": ["content-publish"]
  }
}
```

## 分批执行流程

### 批次任务（Batch 1-N）

每个批次负责抓取约 50 个 handles：

1. 读取 `topics.json` 获取 handles 列表
2. 根据批次索引确定抓取范围
3. 使用 Nitter 实例抓取推文
4. 解析 HTML，提取推文内容和真实 status ID
5. **暂存到** `笔记目录/tweet抓取/.staging/TopicName-YYYYMMDD-batchN.json`
6. **更新进度** `笔记目录/tweet抓取/.staging/progress.json`

### 汇总任务（Process & Publish）

1. 读取所有 `.staging/TopicName-YYYYMMDD-batch*.json`
2. 合并所有推文，去重
3. 应用过滤规则
4. AI 判断相关性
5. 按五个维度组织内容：
   - 🔥 今日要点（整体概览）
   - ⚖️ 治理讨论
   - 💬 社区声音
   - 🛠️ 技术更新
   - 📰 项目新闻
6. 每个板块包含总结 + 相关推文链接
7. 生成多语言版本（如 zh/en/ja）
8. 写入最终文档
9. 执行 postProcess
10. 清理 `.staging/` 目录

## 暂存文件格式

### progress.json

```json
{
  "date": "2026-02-12",
  "topic": "ExampleTopic",
  "batches": {
    "batch1": { "status": "done", "handles": 50, "tweets": 25, "completedAt": "2026-02-12T22:14:00Z" },
    "batch2": { "status": "done", "handles": 50, "tweets": 18, "completedAt": "2026-02-12T22:29:00Z" },
    "batch3": { "status": "running", "handlesCompleted": 30, "tweets": 12 },
    "batch4": { "status": "pending" }
  },
  "lastHandle": "someuser",
  "lastIndex": 130
}
```

### batch.json

```json
{
  "batch": 1,
  "date": "2026-02-12",
  "topic": "ExampleTopic",
  "indexRange": [0, 49],
  "tweets": [
    {
      "handle": "user1",
      "content": "推文内容...",
      "statusId": "1234567890123456789",
      "url": "https://x.com/user1/status/1234567890123456789",
      "timestamp": "2026-02-11T14:32:00Z",
      "isReply": false,
      "isRetweet": false,
      "isQuote": true,
      "hasMedia": false,
      "replyTo": null
    }
  ],
  "success": ["user1", "user2"],
  "failed": ["user3", "user4"]
}
```

## 过滤规则

### 硬规则过滤

| 类型 | 处理方式 |
|------|----------|
| 转发（Retweet） | ❌ 过滤 |
| 纯 emoji | ❌ 过滤 |
| 字数 < 10 | ❌ 过滤 |
| 普通回复 | ⚠️ 仅保留字数 > 50 的回复 |
| 引用推文 | ✅ 保留 |
| 带媒体推文 | ✅ 保留（即使文字少） |

### 相关性判断（智能模式）

通过 AI 判断：这条推文对关注该主题的人是否有价值？

- 直接相关：明确讨论主题内容
- 间接相关：宏观政策、监管动态、行业趋势
- 社区讨论：有价值的观点交流、技术讨论
- 不相关：个人生活、无关话题 → 过滤

## Nitter 抓取与解析

### 抓取命令

```bash
curl -s "http://your-nitter-instance:8080/search?f=tweets&q=from:USERNAME&since=YYYY-MM-DD&until=YYYY-MM-DD" \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  --max-time 30
```

### 解析要点

从 HTML 中提取：
- `tweet-content` 类：推文正文
- `tweet-link` 或 `a[href*="/status/"]`：真实 status ID
- `tweet-date`：发布时间
- `retweet-header`：判断是否转发
- `replying-to`：判断是否回复
- `quote`：判断是否引用
- `attachments`：判断是否有媒体

**关键：确保提取真实的 status ID 用于生成正确的链接**

## 反爬策略

1. 每个 handle 间隔 8-15 秒（随机）
2. 带 User-Agent 头
3. 失败退避：遇到 429 等待 30-60 秒
4. 连续失败 3 次暂停 5 分钟
5. 分批执行，批次间有间隔

## Cron 时间表示例

| 时间 (UTC) | 任务 |
|------------|------|
| 22:00 | Batch 1 (handles 1-50) |
| 22:15 | Batch 2 (handles 51-100) |
| 22:30 | Batch 3 (handles 101-150) |
| 22:45 | Batch 4 (handles 151-200) |
| 23:55 | Process & Publish |

## 输出格式

### 目录结构

```
笔记目录/日报/TopicName-YYYYMMDD/
├── zh.md  (中文)
├── en.md  (English)
└── ja.md  (日本語)
```

### 日报模板

```markdown
# Topic 生态日报 | YYYY-MM-DD

> 📅 覆盖时间：YYYY-MM-DD ~ YYYY-MM-DD
> 📊 数据来源：N 个 Twitter 账号 | M 条推文保留

---

## 🔥 今日要点

整体概览，2-3 句话总结当日最重要的事件。

---

## ⚖️ 治理讨论

治理相关内容的总结。

**相关推文：**
- [@user1](https://x.com/user1/status/xxx): 推文描述
- [@user2](https://x.com/user2/status/xxx): 推文描述

---

## 💬 社区声音

社区讨论和观点的总结。

**相关推文：**
- ...

---

## 🛠️ 技术更新

技术相关更新的总结。

**相关推文：**
- ...

---

## 📰 项目新闻

项目动态和新闻的总结。

**相关推文：**
- ...

---

*本日报由 AI 自动生成，基于 Twitter 公开数据整理*
```

## 断点恢复

如果某个批次失败：
1. 检查 `progress.json` 确定失败位置
2. 手动触发：「继续抓取 TopicName 从 batch3」
3. 系统会从 `lastIndex` 继续执行

## 手动命令

- 「抓取 TopicName 推文」→ 立即执行完整抓取
- 「继续抓取 TopicName」→ 从断点继续
- 「重新处理 TopicName」→ 用已有暂存数据重新生成文档
- 「清理 TopicName 暂存」→ 删除 .staging 目录

## 自定义配置

使用前需要修改：
1. Nitter 实例地址（在 TOOLS.md 中配置）
2. 笔记目录路径
3. topics.json 中的 handles 列表
4. Cron job 中的模型名称
