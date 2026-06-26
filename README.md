# AI Daily Assistant

基于 Hermes Agent + 飞书 构建的智能日报系统，自动采集、筛选、总结 AI/Agent 领域技术内容，沉淀到飞书知识库并推送通知。

## 目录结构

```
├── doc/
│   ├── architecture.md                          # 系统架构文档
│   ├── practice.md                              # 实践总结（含运行数据、难点、反思）
│   ├── evaluation.md                            # 评估报告（含 8 张实拍证据截图）
│   ├── ai-daily-assistant-architecture.html     # 系统架构图 (浏览器打开)
│   └── images/                                  # 证据截图
│       ├── wiki-directory.jpg                   # 飞书 Wiki 左侧目录树
│       ├── wiki-structure.jpg                   # 飞书 Wiki 完整目录结构
│       ├── tech-report.jpg                      # 技术日报总结文档
│       ├── learning-report.jpg                  # 学习日报总结文档
│       ├── dashboard-overview.png               # Dashboard 系统概览
│       ├── architecture-diagram.png             # 系统架构图
│       ├── cron-log-evidence.png                # Cron 本地执行日志
│       └── feishu-push-records.jpg              # 飞书 Bot 每日推送记录
├── .gitignore
└── README.md
```

## 核心能力

| 功能 | 说明 |
|------|------|
| 📡 多源采集 | RSS 14 源 + 微信公众号 2 个 + Exa 语义搜索 4 组 |
| 🧠 AI 筛选 | 双层过滤（脚本关键词 + Agent 语义判断），区分学习类 vs 时讯类 |
| 📝 智囊团分析 | 8 角色（Musk/Jobs/Buffett/Karpathy 等）多视角观点生成 |
| 📄 飞书归档 | 转存文档 + 总结文档双备份，沉淀到飞书 Wiki 知识库 |
| 🔔 卡片通知 | 飞书交互卡片推送，含「📄 飞书原文」直达按钮 |
| 🔄 自进化 | 48h 自学会话自动优化 skills/memory，记录成长日志 |