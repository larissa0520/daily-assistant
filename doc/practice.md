# 基于 Hermes Agent 的个人 AI 学习助手 — 实践总结

## 📌 项目定位

基于 Hermes Agent 平台 + 飞书（Feishu/Lark）构建的个人 AI 日报系统，自动采集、筛选、总结 AI/Agent 领域技术内容，沉淀到飞书知识库并推送通知。

---

## 一、实际效果

### 运行概况

| 维度 | 数值 |
|------|------|
| 启动时间 | 2026年4月（技术日报）/ 5月（学习日报） |
| 运行频率 | 每日 08:00 技术日报 → 09:00 学习日报 → 09:30 Dashboard 概览 |
| 额外周期 | 自学升级 cron 每 48h 运行一次 |
| 累计 cron 执行 | ~143 次（含失败重跑） |
| 成功 Agent 运行 | 48 次（技术日报 15 次 + 学习日报 33 次） |
| 累计处理文章 | ~294 篇（技术日报 ~191 篇 + 学习日报 ~103 篇） |
| 飞书文档创建 | ~50+ 篇（含总结文档、转存文档、wiki 页面等） |
| 飞书卡片推送 | 5+ 次成功，近期大部分自动执行（卡片 + 文档双通道） |
| 代码规模 | 20+ Python 文件，~5,000 行 |
| 配置的 RSS 源 | 14 个（早期 19 个，移除 5 个死源后） |
| 学习日报附加源 | 3 个 Anthropic 补采 + 2 个微信公众号 + 4 组 Exa 语义搜索 |

### 典型结果样例

**学习日报（6月18日）采集的 8 篇文章：**
```
1. 深入解析：AI Agent、Harness 与 Hermes Agent 的核心差异与选型指南 - 云栈社区
2. Production AI Agent Architecture: Lessons From Building CrewKit
3. GLM-5.2 - probably the most powerful text-only open weights LLM
4. From Hugging Face Hub to robot hardware with Strands Agents
5. Agentic Resource Discovery: Let agents search
6. Vercel Connect: Secure access to external services for agents
7. A near-autonomous AI chemist improves a challenging reaction
8. Quoting Charity Majors (Simon Willison 动态)
```

**处理后产出：**
- 8 篇文章 → AI 筛选 + 评分 + 摘要
- 创建飞书转存文档（含全文 + 原文链接）
- 创建总结文档（排版标准化：📊 采集概览 + 📚 精选内容 + 🧠 智囊团）
- 推送飞书交互卡片到聊天窗口
- 卡片中每篇文章附带「📄 飞书原文」链接，一键直达

### 节省时间估算

| 活动 | 人工耗时（估算） | 系统耗时 |
|------|-----------------|---------|
| 每日扫描 14+ 源找文章 | ~30-45 分钟 | 2.5s（RSS 扫描）+ 3-5min（AI 处理） |
| 阅读 + 筛选 10-20 篇 | ~20-30 分钟 | 自动完成 |
| 写摘要 + 归档 | ~15-20 分钟 | 自动完成 |
| 每日总计节省 | **~60-90 分钟/天** | 无需人工介入 |

---

## 二、实现原理

### 数据源架构

```
┌─────────────────────────────────────────────────────────┐
│                     数据采集层                           │
│                                                         │
│  技术日报 (ai_daily_v3.py):                             │
│    ├─ 14 个 RSS 源（TechCrunch, Simon Willison,         │
│    │   MIT Tech Review, HuggingFace, Ars Technica 等）   │
│    └─ blogwatcher CLI 管理（读/未读状态 + 去重）         │
│                                                         │
│  学习日报 (ai_agent_learning_weekly.py):                 │
│    ├─ 8 个 RSS 源（LangChain, LlamaIndex, Modal,        │
│    │   Latent Space, HuggingFace, Vercel 等）            │
│    ├─ 3 个 Anthropic 补采源（News/Research/Engineering） │
│    ├─ 2 个微信公众号（wechat-article-exporter 本地代理） │
│    └─ 4 组 Exa 语义搜索（mcporter MCP 网关）            │
└─────────────────────┬───────────────────────────────────┘
                      │ stdout 注入
                      ▼
┌─────────────────────────────────────────────────────────┐
│                   AI 处理层 (Hermes Agent)                │
│                                                         │
│  Step 1:  读取注入的文章 JSON                            │
│  Step 1.5: AI 语义过滤（学习类 vs 时讯类）               │
│  Step 2:  评分（1-10）+ 摘要 + 分类 + 洞察              │
│  Step 5.5: 智囊团圆桌会议（2-3 角色分析文章）            │
└─────────────────────┬───────────────────────────────────┘
                      │ feishu_doc_create / write
                      ▼
┌─────────────────────────────────────────────────────────┐
│                 飞书归档层                                │
│                                                         │
│  Wiki 首页                                              │
│  ├── 📊 系统概览（Dashboard 每日更新）                   │
│  ├── 📁 技术时讯                                        │
│  │   ├── 📄 YYYY-MM-DD AI 技术日报（总结文档）           │
│  │   └── 📁 YYYY-MM-DD（转存文档）                       │
│  └── 📁 学习博客                                        │
│      ├── 📄 YYYY-MM-DD AI/Agent 学习日报（总结文档）     │
│      └── 📁 YYYY-MM-DD（转存文档）                       │
│                                                         │
│  每篇转存文档 = 文章全文 + 🔗 阅读原文                   │
│  总结文档 = 精选文章 + 评分摘要 + 🔗 飞书原文 + 智囊团  │
└─────────────────────┬───────────────────────────────────┘
                      │ send_feishu_card.py
                      ▼
┌─────────────────────────────────────────────────────────┐
│                  通知层（飞书交互卡片）                    │
│                                                         │
│  Feishu 聊天窗口 oc_2736b5a... ← 彩色卡片                │
│  ├─ 技术日报 → 绿色主题 header                           │
│  ├─ 学习日报 → 蓝色主题 header                           │
│  ├─ 每篇文章：标题 + 来源 + 评分 + 「📄 飞书原文」按钮    │
│  └─ 底部：统计概况 + 时间戳                              │
└─────────────────────────────────────────────────────────┘
```

### Hermes 编排流程

```
┌──────────────────────────────────────────────────────────────┐
│ cron job 触发 → 执行 script（Python 预采集）                   │
│                                                              │
│  1. ~/.hermes/scripts/ai_daily_collect.py → ai_daily_v3.py   │
│     → blogwatcher scan → RSS 解析 → 去重 → stdout JSON       │
│                                                              │
│  2. stdout 自动注入到 Agent prompt                            │
│     → 若无新文章 → {"wakeAgent": false} → 跳过本轮           │
│                                                              │
│  3. Hermes Agent 启动 → 读取注入数据 → 执行 Workflow          │
│     └─ 所有 feishu_doc_create/write 用原生工具（非 Python）   │
│                                                              │
│  4. AI 处理完成后 → feishu_doc 创建/写入 → 卡片推送            │
│     → [SILENT] 结束                                          │
└──────────────────────────────────────────────────────────────┘
```

### AI 筛选和总结逻辑

**双层过滤机制：**

1. **脚本层（关键词评分）**：`ai_agent_learning_weekly.py` 内置 `NEGATIVE_KEYWORDS`，含 "announces"、"融资"、"收购"、"合作" 等新闻特征词 → 减 4 分；分数 < 3 自动淘汰

2. **Agent 层（语义过滤）**：Hermes Agent 读取每篇文章的 title + description + archive_content，判断「学习类」vs「时讯类」
   - ✅ 学习类：技术教程、最佳实践、架构分析、工程经验、论文解读 → 保留
   - ❌ 时讯类：产品发布、公司公告、融资收购、行业报告 → 跳过

3. **多级摘要**：AI 为每篇保留文章生成 2-3 句精炼中文摘要 + 评分（1-10）+ 分类标签

4. **智囊团圆桌会议**（2026-06-15 新增）：选取 2-3 个 AI 角色（Musk/Jobs/Buffett/Hinton/李飞飞/Karpathy 等），对当日 Top 文章从各自视角发表独特观点，以 quote block 追加到总结文档末尾

### 飞书文档归档和通知逻辑

**文档创建流程：**
1. `feishu_doc_create(title="📁 YYYY-MM-DD", wiki_parent_node_token=技术时讯|学习博客)` → 日期文件夹
2. `feishu_doc_create + feishu_doc_write` → 每篇文章创建独立转存文档（含原文链接 + 全文）
3. `feishu_doc_create(title="YYYY-MM-DD AI 技术日报/学习日报", wiki_parent_node_token=根目录)` → 总结文档
4. `feishu_doc_write` 分批写入：📰 标题 → 统计 → 精选文章 → 其他关注 → 趋势 → 智囊团

**通知流程：**
```bash
cd ~/ai-daily-assistant && python3 send_feishu_card.py \
  "oc_2736b5a8979f47e1cf3d8b3479e4df5e" \
  "<总结文档URL>" \
  "YYYY-MM-DD AI/Agent 学习日报" \
  '<文章JSON>' \
  '<统计JSON>'
```
脚本调用飞书 API 发送交互卡片，自动识别日报类型匹配主题色。

---

## 三、实践难点

### 1. RSS 扫描超时（2026-04 ~ 05）

| 问题 | 解决 |
|------|------|
| 19 个源中 5 个死源（404/301），每次扫描逐一超时 | 从 blogwatcher 数据库删除 5 个死源，19→14 |
| 最差扫描耗时 730s（12 分钟），触发 cron 闲置超时 | 新增 TimeoutGuard（SIGALRM，120s 总超时），超时自动降级 |
| 修复后扫描耗时 2.5s（**295 倍提速**） | 双层保护：根治死源 + 安全网超时 |

### 2. AI 模型流中断（2026-05 ~ 06）

**问题：** deepseek-v4-flash 通过阿里云 API 流式传输持续中断，技术日报连续失败 19 次

| 表现 | 根因 |
|------|------|
| 流停滞 mid tool-call → feishu_doc_write 未执行 | 阿里云 deepseek 代理的流式推理在 ~900-1050s 后卡死 |
| 闲置超时 600s | 学习日报上下文更大（archive_content 完整正文），更易触发 |

**解决方案（2026-06-02 三重措施）：**
1. **切换模型**：两个 cron job 从 deepseek-v4-flash → qwen3.6-plus（阿里云原生模型，稳定）
2. **截断上下文**：技术日报描述截断 300 字符，学习日报 archive 从 9000→5000 字符
3. **改窗口匹配**：严格精确日期匹配 → 2 天窗口匹配，确保重跑有数据

### 3. Feishu 文档权限和写入问题

| 问题 | 解决 |
|------|------|
| tenant_access_token 创建文档成功但写入失败（code 1770001/1770029） | 改用 user_access_token（OAuth 授权），feishu_doc_* 原生工具自动使用 |
| OAuth 授权码单次有效，过期需重新扫码 | 配置 httpbin.org/anything 作为 redirect URI，授权码即时消费 |
| feishu_doc_delete_blocks 不能批量删除（跨度>1 报错 1770001） | 逐 block 调用，或绕道重建文档 |

### 4. AI 输出不稳定点

| 问题 | 调整方式 |
|------|---------|
| Agent 创建文档但标题用文章日期而非执行日期 | prompt 明确约束：`title = f"{execution_date} AI 技术日报"` |
| 排版格式不统一（标题层级、emoji、链接格式） | 在 prompt 中贴入完整排版模板 + 每个区段的格式规范表 |
| 智囊团角色重复（连续两天同角色组合） | prompt 添加「连续两天尽量避免相同角色组合」规则 |
| 时讯类文章混入学习日报 | 新增 Step 1.5 语义过滤 + 脚本层负面关键词评分 |
| 卡片链接走向（原文 URL vs 飞书原文 URL） | 通过 `feishu_doc_url` 字段区分，卡片自动识别显示对应按钮 |
| Agent 有时候输出 prompt 本身而非执行 | 加强 prompt 中「execute, do not explain」指令 |

### 5. 微信公众号集成坑点

| 问题 | 解决 |
|------|------|
| API 路径不匹配（v2.2 vs v2.3.18） | 3 处 API 调用路径全部修改 |
| 响应字段名变更（content→content_noencode） | 适配新的字段名 |
| auth-key 认证 | 从浏览器扫码获取长期有效 key |
| 本地部署 Nuxt 构建问题 | 中文 zip 文件名、npm mirror、Vite 兼容等逐一排查 |

---

## 四、反思与沉淀

### 值得复用的做法

1. **数据采集与 AI 处理分离**：Python 脚本只做不需要 LLM 的事（RSS 扫描、数据解析、Feishu API 调用），AI 推理由 Hermes Agent prompt 完成。脚本 stdout 注入 prompt，无 Agent idle timeout 问题。

2. **双层过滤机制**：脚本关键词评分（粗筛）+ Agent 语义过滤（精筛），兼顾效率和准确率。

3. **原生工具优先**：Hermes 的 `feishu_doc_create/write` 原生工具在 cron 会话中同样可用，比 Python 脚本更稳定（自动 OAuth 管理）。

4. **Graceful 降级**：任何环节故障不阻塞后续运行——超时跳过本轮、无新文章跳过本轮、卡片发送失败文档已保存。

5. **文档即数据**：每篇文章创建独立转存文档 + 总结文档双备份，即使通知失败，知识已经在知识库中。

6. **Prompt 即文档**：cron prompt 中嵌入完整排版模板和注意事项，既是执行指令也是系统文档。

### 别人如何照着搭

**前提条件：**
- Hermes Agent 环境（cron 调度 + 工具集 + 模型接入）
- 飞书企业账号 + 创建一个飞书应用（App ID/Secret）
- 本地 Python 3.9+ 运行环境

**最小可运行步骤：**

1. **克隆代码** `~/ai-daily-assistant/`（核心 Python 采集脚本）
2. **安装 blogwatcher** `go install github.com/.../blogwatcher-cli` + 添加 RSS 源
3. **配置飞书 OAuth**：创建飞书应用 → 配置 redirect URI → 获取 user_access_token
4. **配置环境变量**：`FEISHU_APP_ID`、`FEISHU_APP_SECRET`、Wiki 参数
5. **创建 cron jobs**（使用 `cronjob create`）：
   - 技术日报：`schedule: "0 8 * * *"`, `script: "ai_daily_collect.py"`, `enabled_toolsets: ["feishu_doc", "terminal"]`
   - 学习日报：类似但用 `ai_agent_learning_weekly_collect.py`
6. **飞书 Wiki 建目录**：首页 → 技术时讯 / 学习博客 / 系统概览 三个子页
7. **验证运行**：手动触发 cron 检查文档创建 + 卡片推送

**可选扩展：**
- 微信公众号：本地部署 wechat-article-exporter（需个人订阅号）
- Exa 语义搜索：配置 mcporter MCP 网关
- Dashboard：安装 Playwright + 生成 HTML 截图