---
name: twitter-digest
description: 定期抓取指定 Twitter 账号的推文，过滤低质量内容，智能判断相关性，生成主题摘要并存入 Obsidian。支持每日/每周/每月频率，可手动触发或 cron 自动执行。
---

# Twitter Digest Skill

抓取 Twitter 推文，过滤、分析、归档到 Obsidian。

## 前置依赖

- Nitter 实例（自建或公共）
- Obsidian vault（本地目录）
- Git（用于同步）

## 触发方式

- 手动：「抓取 [主题名] 推文」「运行 twitter digest [主题]」
- 自动：cron job 按配置的 triggerTime 触发

## 配置文件

主题配置：`skills/twitter-digest/topics.json`

```json
{
  "MyTopic": {
    "handles": ["user1", "user2", "user3"],
    "frequency": "daily",
    "triggerTime": "23:00",
    "description": "主题描述"
  }
}
```

- `handles`: Twitter 用户名（不带 @）
- `frequency`: `daily` | `weekly` | `monthly`
- `triggerTime`: UTC 时间，cron 触发时刻

## 抓取规则

### 时间窗口

- daily: 前一天 triggerTime → 今天 triggerTime
- weekly: 过去 7 天
- monthly: 过去 30 天

### 过滤规则（硬规则）

以下推文直接过滤：
1. 转发（Retweet）
2. 回复（Reply）
3. 纯 emoji（无实质文字）
4. 字数 < 20

### 相关性判断（智能模式）

通过 AI 判断：这条推文对关注该主题的人是否有价值？

- 直接相关：明确讨论主题内容
- 间接相关：宏观政策、监管动态、行业趋势等影响该主题的内容
- 不相关：个人生活、无关话题 → 过滤

## 反爬策略

**必须严格遵守以下策略：**

1. **启动随机延迟**：任务启动后先 sleep 0-60 分钟（随机），分散请求时间
2. **请求间隔随机化**：每个 handle 之间间隔 8-15 秒（随机），不要固定间隔
3. **User-Agent 伪装**：
   ```bash
   curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
   ```
4. **失败退避**：
   - 遇到 429 或超时：等待 30-60 秒再重试
   - 连续失败 3 次：暂停整个任务 5 分钟
5. **分批抓取**：如果 handles 超过 50 个，分批执行，批次之间休息 5 分钟

## 抓取流程

1. 读取 `topics.json` 获取主题配置
2. **随机等待 0-60 分钟**
3. 对每个 handle：
   - 调用 Nitter 搜索接口
   - **随机间隔 8-15 秒**
   - 失败重试（带退避）
4. 应用硬规则过滤
5. AI 判断相关性，过滤不相关内容
6. 为保留的推文打标签
7. 生成摘要
8. 写入 Obsidian 文件
9. Git commit & push

## Nitter 抓取

```bash
# 搜索用户推文（带 User-Agent）
curl -s "http://YOUR_NITTER_INSTANCE/search?f=tweets&q=from:USERNAME" \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  --max-time 30

# 按时间范围
curl -s "http://YOUR_NITTER_INSTANCE/search?f=tweets&q=from:USERNAME&since=2026-02-10&until=2026-02-11" \
  -H "User-Agent: Mozilla/5.0 ..." \
  --max-time 30
```

解析 HTML 中的 `tweet-content` 提取推文内容。

## 输出格式

文件路径：`YOUR_OBSIDIAN_VAULT/tweet抓取/YYYY-MM/[主题]-[频率]-YYYYMMDD.md`

```markdown
---
created: 2026-02-11
topic: MyTopic
frequency: daily
period: 2026-02-10 23:00 UTC → 2026-02-11 23:00 UTC
handles_total: 50
handles_success: 48
handles_failed: 2
tweets_raw: 150
tweets_kept: 25
status: partial
---

## 摘要

本期要点：
- xxx
- xxx

## 推文

### @user1

> 推文内容...
>
> [原文](https://x.com/user1/status/xxx) · 2026-02-10 14:32 UTC

#标签1 #标签2

---

## 抓取记录

✅ 成功: 48 个 handles
❌ 失败: 2 个 handles
- @xxx: timeout (重试 3 次)
- @yyy: 429 rate limit
```

## Cron 配置示例

```json
{
  "name": "MyTopic Twitter Digest",
  "schedule": { "kind": "cron", "expr": "0 23 * * *", "tz": "UTC" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "执行 MyTopic 主题的 Twitter 抓取任务。首先随机等待 0-60 分钟，然后按照 twitter-digest skill 流程执行。",
    "model": "your-preferred-model"
  }
}
```

## 错误处理

- 单个 handle 失败不影响其他 handle
- 失败记录写入文档底部
- 不自动补抓，用户手动触发

## 手动命令

- 「抓取 [主题] 推文」→ 立即执行抓取
- 「补抓 [主题] @xxx」→ 单独抓取某个失败的 handle
- 「添加主题 xxx」→ 交互式添加新主题配置
- 「列出所有主题」→ 显示 topics.json 内容
- 「测试抓取 [主题] 5」→ 随机选 5 个 handle 测试

## 自定义配置

使用前需要修改：
1. Nitter 实例地址
2. Obsidian vault 路径
3. topics.json 中的 handles 列表
4. Cron job 中的模型名称
