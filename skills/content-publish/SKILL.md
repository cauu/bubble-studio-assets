---
name: content-publish
description: 将 twitter-digest 抓取的内容整理为可读性强的日报，发布到指定平台。
---

# Content Publish Skill

将 Twitter Digest 抓取结果整理为日报并发布到目标平台。

## 触发方式

- 自动：作为 twitter-digest 的 postProcess 触发
- 手动：「发布 TopicName 日报」

## 输入

读取最新的 twitter-digest 输出文件：
`笔记目录/日报/TopicName-YYYYMMDD/`

## 输出格式

整理为适合发布的日报格式：

```markdown
# Topic Daily - February 11, 2026

## 🔥 Today's Highlights

### 1. [话题标题]
简要描述这个话题的核心内容（2-3 句话）

**Related Tweets:**
- [@handle](https://x.com/handle/status/xxx): "推文摘要..."
- [@handle2](https://x.com/handle2/status/xxx): "推文摘要..."

---

### 2. [话题标题]
...

---

## 📊 Stats
- Accounts monitored: N
- Tweets collected: XX
- Topics identified: X
```

## 整理规则

1. **话题聚合**：将相关推文按话题分组，而非按账号罗列
2. **优先级排序**：重要话题在前（治理 > 技术更新 > 生态动态 > 社区活动）
3. **摘要精炼**：每个话题 2-3 句话概括，不要堆砌原文
4. **保留链接**：每条推文保留原文链接，方便读者深入
5. **多语言支持**：可生成中/英/日等多语言版本

## 发布接口示例

```bash
POST https://your-platform.example.com/api/posts
Header: Authorization: Bearer YOUR_API_KEY

Body:
{
  "category": "daily_digest",
  "status": "published",
  "content": {
    "type": "richText",
    "title": "Topic Daily - [Date]",
    "text": "[整理后的日报内容]",
    "language": "en"
  }
}
```

## 执行流程

1. 读取当日 twitter-digest 输出文件
2. 分析推文内容，识别热门话题
3. 按话题聚合推文
4. 生成日报格式（多语言）
5. **保存日报到笔记系统**
6. 调用发布平台 API
7. Git commit & push
8. 确认发布成功

## 笔记存档格式

日报同时保存到笔记系统：

```markdown
---
created: 2026-02-11
type: daily-digest
topic: TopicName
post_id: "xxx"
status: published
---

# Topic Daily - February 11, 2026

[日报正文内容]
```

## 错误处理

- 如果 twitter-digest 文件不存在，报错并终止
- 如果 API 调用失败，重试 3 次
- 记录发布结果到日志

## 自定义配置

使用前需要修改：
1. 发布平台 API 地址和密钥
2. 笔记目录路径
3. 日报分类和语言设置
