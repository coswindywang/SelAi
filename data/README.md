# SelAi 数据存储目录

此目录用于存储用户的生活日志数据。

## 目录结构

```
data/
├── logs/              # 原始日志文件（Markdown 格式）
│   └── YYYY/
│       └── MM/
│           └── YYYY-MM-DD.md
├── index.db           # SQLite 索引数据库（自动生成）
├── config.json        # 用户配置文件
└── exports/           # 导出数据目录
```

## 日志文件格式

每条日志以 Markdown 格式存储，包含 YAML Front Matter：

```markdown
---
date: 2026-11-03T08:00:00+08:00
type: daily_log
tags: [health, work]
mood: 7
---

## 今日记录

今天的内容...
```

## 数据主权

- 所有数据存储在本地，不会自动上传到云端
- 你可以随时导出或删除所有数据
- 数据格式为标准的 Markdown 和 JSON，不依赖特定软件
