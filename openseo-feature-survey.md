# OpenSEO 功能调研

> 调研对象：github.com/every-app/open-seo，main @ 1523750b（v0.1.22）
> 依据：源码、设计 spec、用户文档、发布说明
> 日期：2026-11-02

OpenSEO 是一款开源的 SEO 工具，定位为 Semrush、Ahrefs 的平价替代品。它覆盖关键词、排名、竞品、外链、站点审计和 AI 可见度等核心工作流，并通过 MCP 服务和 Agent Skills 把这些数据交给 Claude、ChatGPT、Codex、Cursor 等 AI 助手直接使用。

## 目录

1. [产品概览](#01-产品概览)
2. [功能地图](#02-功能地图)
3. [调研类功能](#03-调研类功能)
4. [自有站点监控](#04-自有站点监控)
5. [站点审计](#05-站点审计)
6. [AI 可见度（GEO / AEO）](#06-ai-可见度geo--aeo)
7. [Agent 能力：MCP、Skills 与 SAM](#07-agent-能力mcpskills-与-sam)
8. [报告与项目记忆](#08-报告与项目记忆)
9. [账户、团队与计费](#09-账户团队与计费)
10. [部署形态](#10-部署形态)
11. [技术架构](#11-技术架构)
12. [调研结论](#12-调研结论)

---

## 01 产品概览

**定位：**“给你和你的 AI Agent 用的一站式 SEO 工具”。卖点有四个：MCP 与 Skills 做得好；界面简洁，只做聚焦的工作流；没有强制订阅，自带 DataForSEO Key 就能按量付费；代码开源，可以 fork 后自行改造。

**目标用户：**独立站长和中小企业（Base 套餐“为你自己的网站”）、自由职业者和成长中的团队（Pro）、多客户代理商（Agency）。另外还有自托管的技术型用户。

**数据来源：**付费 SEO 数据几乎全部来自 DataForSEO（Labs、Keywords Data、SERP、Backlinks、Business Data、LLM 响应、Lighthouse）。Google Search Console 和 GA4 通过 OAuth 直连，免费。部分 AI 生成功能走 OpenRouter，Local SEO 地图依赖 Google Maps API。

> **商业模式：**托管版按积分计费，1,000 积分 = 1 美元。平台调用 DataForSEO 时在实际成本上加收 28%。订阅套餐每月赠送积分，另外可以单独购买永不过期的充值积分。自托管用户直接向 DataForSEO 付费，不经过 OpenSEO。

## 02 功能地图

以项目（Project）为单位组织功能。左侧导航分为五组，下表按导航顺序列出每个模块。

| 分组 | 模块 | 路由 | 一句话说明 |
| --- | --- | --- | --- |
| Overview | Dashboard | `/p/:id` | 自有网站的监控首页：概览、关键词、外链三个标签页，附带新手引导清单 |
| Research | Keyword Research | `/keywords` | 种子词扩展、搜索量、难度、意图、SERP 分析 |
| Research | Domain Overview | `/domain` | 任意域名的自然流量、排名关键词和热门页面 |
| Research | Backlinks | `/backlinks` | 外链概况、引荐域名、热门页面、新增与丢失链接 |
| AI Visibility | Prompt Research | `/ai-visibility/research` | 找出某个关键词下用户会问 AI 的问题，以及 ChatGPT 的答案和引用来源 |
| AI Visibility | Prompt Explorer | `/prompt-explorer` | 同一个提问在 ChatGPT、Claude、Gemini、Perplexity 之间对比 |
| AI Visibility | Prompt Tracking | `/ai-visibility` | 定期跟踪品牌在 AI 回答里被提及和被引用的情况 |
| My Site | Saved Keywords | `/saved` | 收藏的关键词，支持标签、筛选和导出 |
| My Site | Rank Tracking | `/rank-tracking` | 全国或城市级的 Google 排名跟踪，支持定时检查 |
| My Site | Local SEO（Beta） | `/local-seo` | Google 地图网格排名 |
| My Site | GSC Insights | `/search-performance` | Search Console 的查询、页面、国家、设备维度分析 |
| My Site | Site Audit | `/audit` | 爬虫审计、问题清单、Lighthouse 性能 |
| AI Tools | Reports | `/reports` | 由 Agent 写成的 HTML 报告，支持模板和公开分享 |
| AI Tools | Context | `/context` | 项目记忆，SAM、MCP 和 Skills 共用 |
| 全局 | Agent setup / Settings / Billing | `/ai` 等 | 接入 Agent、组织与团队、集成、计费 |

另有一个未在导航中出现的 `/brand-lookup` 页面（品牌在 AI 搜索中的声量查询），以及应用内 AI 助手 SAM（侧边栏聊天）。

## 03 调研类功能

这一组工具可以指向任意关键词或域名，用于研究市场和竞品，与项目自己的网站无关。

### 关键词研究（Keyword Research）

- 输入种子词后返回扩展关键词，字段包括搜索量、关键词难度（KD）、CPC、竞争度、搜索意图和趋势。默认每个种子词返回 150 行。
- **数据源按国家自动路由：**DataForSEO Labs 覆盖 94 个国家，默认使用；冰岛等 Labs 不覆盖的国家改走 Google Ads 接口（可达 217 个国家），但这类数据没有 KD 和意图。用户不需要手动选择数据源。
- 可选开启“点击流数据”修正搜索量，费用翻倍，默认关闭。
- 内置 SERP 分析卡片，可以查看某个关键词当前的搜索结果页。
- 同一搜索量的近似词会合并展示；结果可以一键收藏到 Saved Keywords。

### 域名概览（Domain Overview）

- 查看任意域名（或子目录）的自然流量估算、排名关键词列表和热门页面（Keywords 与 Pages 两个标签）。
- 支持筛选、排序、分页，并会记住用户的筛选偏好。
- MCP 侧还提供域名关键词建议和 SERP 竞品发现（`find_serp_competitors`）。

### 外链分析（Backlinks）

- 概况面板与图表、外链表、引荐域名表、热门被链页面。
- 可按链接类型、域名评级、垃圾分数、文本筛选，支持按子目录分析和导出。
- Dashboard 的外链标签会展示新增链接、丢失链接和最新的存活链接。

## 04 自有站点监控

### Dashboard

- **Overview：**新手引导清单（添加竞品、做关键词研究、审计、连接 Agent、邀请队友等），步骤在整个工作区共享完成状态，可以“稍后再做”；另有站点健康卡片、新出现的搜索词和近期外链动态。
- **Keywords：**基于 GSC 数据，分为新关键词、Top 关键词（按点击排序）和 Striking distance（平均排名 5–20、按曝光排序，即“差一点就上首页”的词）。
- **Backlinks：**新增、丢失和最新的链接，每个视图最多 25 个引荐域名。
- 付费数据 24 小时刷新一次，只在用户访问时触发；刷新失败时保留旧数据。保存网站时会自动启动首次 50 页审计（开启 JS 渲染）。

### 关键词收藏（Saved Keywords）

收藏的关键词列表，支持标签管理、批量打标签、按标签和指标筛选，以及导出。Agent 也能通过 `save_keywords`、`list_saved_keywords` 读写收藏。

### 排名跟踪（Rank Tracking）

- 以“域名 + 国家（+ 城市）”为一个 tracker，添加关键词后检查 Google 排名，记录历史。
- 支持城市级跟踪，例如 `Enid,Oklahoma,United States`。城市级搜索量来自 Google Ads 的本地数据，KD 和意图仍用全国数据。设计文档举了一个例子：“rv storage near me”全国月搜索 135K，在匹兹堡只有 70。
- 支持按日、周、月定时检查，也可以手动检查，检查前会弹窗显示费用。
- 提供趋势图、历史矩阵、排名变化提醒、关键词置顶、筛选，以及积分不足时暂停的提示。
- 检查任务由 Cloudflare Workflow（`RankCheckWorkflow`）执行，带重试机制和卡死检测（12 小时）。

### 本地 SEO（Local SEO，Beta）

- 以商家位置为中心生成 N×N 网格，每个网格点做一次地图搜索，得到“在不同地点搜索时商家排第几”的热力地图。
- 支持多个关键词，显示每个点的平均排名、进入前三的点数，以及最常排第一的竞争对手。
- MCP 侧补充了商家资料（GBP）、评论、动态、问答、类目查询和本地 SERP，配合 `local-seo` Skill 生成本地 SEO 诊断报告。
- 地图渲染需要 `GOOGLE_MAPS_API_KEY`。

### Search Console 洞察（GSC Insights，免费）

- 通过独立的 Google OAuth 授权，只读取 Search Console 数据。一个项目绑定一个属性，组织内成员共享。
- 按查询、页面、国家、设备、日期分维度查看点击、曝光、CTR 和排名。数据最多回溯 16 个月，约有 3 天延迟。
- Agent 可以查询表现数据，也可以做 URL Inspection（单次 1–10 个 URL）。

### Google Analytics 4（免费）

同样通过 OAuth 连接 GA4 属性，没有独立页面，主要供 Dashboard 卡片和 Agent 使用。MCP 提供 10 个只读报表：自然流量概览、自然流量落地页、页面表现、流量获取、关键事件、电商、站内搜索、受众细分、埋点健康检查，以及搜索机会（GSC × GA4 交叉分析）。

### IndexNow

在 Settings → Integrations 中为项目生成 IndexNow key，并检查站点根目录（含 www 和裸域）是否正确托管了 key 文件。之后可以批量提交 URL（单次最多 1 万个），通知 Bing、Yandex、Seznam、Naver、Yep 等引擎重新抓取。Google 不支持 IndexNow。提交免费，Agent 可以调用 `submit_indexnow_urls`。

## 05 站点审计

- 输入起始 URL 和页数上限：默认 50 页，免费版最多 50 页，付费版最多 10,000 页。
- 可选开启 JavaScript 渲染（无头浏览器，额外计费，单次爬取最长 22 小时）。Lighthouse 性能检查默认自动执行。
- 结果分为 Issues、Pages、Performance 三个标签，问题详情可以下钻到具体页面。
- 爬虫 UA 为 `OpenSEO-Audit/1.0`，托管版请求带 Web Bot Auth 签名。爬虫遵守 robots.txt，速率约每秒 1 个请求，遇到 429 会退避，连续被限流时提前终止。
- 支持为受保护站点配置爬虫凭证（Crawler Access），对 Shopify 站点有专门处理。
- 架构上采用 Cloudflare Workflow 加 Durable Object 草稿区（AuditScratchpad），避开 Worker 内存上限和单步输出 1 MiB 的限制。

目前定义了 32 类问题：

- 爬虫被拦截 / 疑似需 JS 渲染
- 单页被限流（429）
- 爬取因限流、时间、费用上限提前终止
- 服务器错误（5xx）/ 页面 4xx
- 内部死链
- 缺少 title / 标题重复 / 标题过长或过短
- 缺少 meta description / 描述重复 / 过长或过短
- 页面内容重复 / 内容单薄
- 缺少 H1 / 多个 H1 / 标题层级跳级
- 重定向链 / 重定向循环
- canonical 信号冲突 / 被 canonical 到其他 URL
- 图片缺 alt
- 孤立页面 / 无出链页面 / 层级过深
- 服务器响应慢
- 页面 noindex

## 06 AI 可见度（GEO / AEO）

这是 OpenSEO 相对传统 SEO 工具差异最大的一块，目标是回答“AI 回答里有没有提到我的品牌、有没有引用我的网站”。

| 功能 | 说明 |
| --- | --- |
| Prompt Research（付费套餐） | 输入关键词，找到用户相关的提问（数据主要来自 Google“People also ask”），附 ChatGPT 的回答和引用来源。目前只支持美国英语，每个关键词约 0.25 美元。 |
| Prompt Explorer（付费套餐） | 单个提问在 ChatGPT、Claude、Gemini、Perplexity 之间横向对比答案和引用。走模型 API，可能与消费端网页的回答不同。 |
| Prompt Tracking | 把问题按主题保存，选择引擎和市场，手动检查或按日、周、月调度。默认引擎为 ChatGPT 和 Gemini，可加 Google AI Overviews、Claude、Perplexity。改写问题会生成新问题，以保留历史回答对应的原始措辞。 |
| Competitors / Citations | 汇总品牌提及、竞品出现次数和被引用页面，提供趋势图（每天取最后一次完成的检查）和导出。查看历史结果不消耗积分。 |

- **Brand Lookup：**品牌或域名在 AI 搜索中的声量（Share of Voice），最多对比 5 个竞品。
- **初始化研究：**根据网站自动补全项目上下文并生成追踪关键词，不覆盖已有内容。
- **计费：**ChatGPT 和 Gemini 从消费端网页采集；Claude 和 Perplexity 只能走 API，单价更高。“立即检查”比调度检查更贵。

## 07 Agent 能力：MCP、Skills 与 SAM

### MCP 服务器

托管地址 `/mcp`，支持 OAuth（带授权同意页）和 API Key 两种认证。每个凭证绑定一个组织，计费也记到这个组织。支持 ChatGPT（含 Apps 挑战验证和交互小组件）、Claude、Codex、Cursor 等客户端。当前共注册了 83 个工具，按领域分类如下：

| 领域 | 数量 | 代表工具 |
| --- | ---: | --- |
| 项目与上下文 | 8 | `create_project` `get_project_context` `research_project_website` `whoami` |
| 关键词与 SERP | 12 | `research_keywords` `get_keyword_metrics` `get_serp_results` `save_keywords` |
| 域名与外链 | 6 | `get_domain_overview` `get_ranked_keywords` `get_backlinks_profile` |
| 排名跟踪 | 8 | `create_rank_tracker` `estimate_rank_tracker_cost` `run_rank_tracker` |
| 本地 SEO | 9 | `get_local_rank_grid` `get_business_profile` `get_business_reviews` |
| 站点审计 | 6 | `run_site_audit` `get_audit_issues` `get_audit_pages` |
| AI 可见度 | 15 | `explore_prompt` `run_ai_visibility_check` `get_ai_visibility_trend` |
| GSC / GA4 | 15 | `get_search_console_performance` `inspect_urls` `get_search_opportunities` |
| 报告 | 8 | `save_report` `save_report_template` `set_report_sharing` |
| 其他 | 1 | `submit_indexnow_urls` |

分类数量为调研时人工归类的结果，可能有 ±1 的误差。付费工具在托管模式下先冻结积分再扣费，GSC、GA4、报告、IndexNow 类工具免费。

### Agent Skills（12 个）

以插件形式分发到 Claude Code、Codex 和 Cursor，作用是用工作流指令引导 Agent 组合调用 MCP 工具：

| Skill | 用途 |
| --- | --- |
| `seo-project-setup` | 填写项目上下文：站点、目标、定位、竞品、关键页面，检查 MCP 和 GSC 连接 |
| `seo-audit` | 审计网站，结合真实搜索机会输出“最该改的几件事” |
| `keyword-research` | 发现、评估、收藏并标记关键词 |
| `keyword-clustering` | 按意图聚类关键词，并映射到现有或新建页面 |
| `competitor-analysis` | 单个竞品的自然流量、排名词、内容主题、外链和差距 |
| `competitive-landscape` | 整个市场的头部玩家、内容主题、覆盖度和战略空白 |
| `link-prospecting` | 找外链目标和联系方式，起草外联邮件 |
| `local-seo` | GBP 审计、本地竞品对比、地图网格可见度 |
| `ai-prompt-research` | 研究用户会问 AI 什么问题（只研究，不开启跟踪） |
| `ai-visibility-audit` | 诊断品牌在 AI 回答中的表现，给出改进建议 |
| `seo-report` | 把研究结果写成单页 HTML 报告并保存到项目 |
| `seo-coach` | 教练模式，讲解工作流并推荐下一步 |

### SAM：应用内 Agent

- 侧边栏聊天助手，基于 Cloudflare `agents` 框架，复用 MCP 的同一套工具，并自动作用于当前项目。
- 额外有 `map_links`、`read_pages`（免费读取网站内容）和 `get_product_info`（回答产品本身的问题）三个工具。
- 新项目进入“intake 模式”：只问网址，自己读取最多 10 个页面，推断业务、定位和竞品，再请用户确认，并写入项目记忆。
- 规则约束：不编造指标；30 天内研究过的问题优先复用研究日志里的结论；批量付费查询前先征求确认；不在聊天里推销套餐。

## 08 报告与项目记忆

### Reports

- 由 Agent 生成的自包含 HTML 报告（最大 500 KB），在沙箱 iframe 中渲染，支持导出（打印）、全屏、新标签打开和删除。
- 记录创建者，例如“Claude Code”。用同一 ID 保存会覆盖原报告，不保留版本历史。
- **模板：**每个项目最多 10 个，描述受众、章节、语气、落款和主题色。Agent 在用户点名时套用。
- **公开分享**（仅托管版）：`/s/<token>` 无需登录即可查看，页面 noindex，带 OG 图。关闭后链接失效，重新分享会生成新链接。

### Project Context（项目记忆）

每个项目一份结构化记忆：业务概述、当前目标、定位、写作偏好、竞品清单、关键页面、自定义段落和研究日志。用户可以在 UI 中查看和编辑，SAM、MCP Agent 和 Skills 读写同一份数据，解决了以前“SAM 的记忆用户看不到、Skills 的记忆留在本地”的割裂问题。

## 09 账户、团队与计费

- **认证：**Better Auth 邮箱密码登录，带 Turnstile 人机验证和一次性邮箱拦截，支持忘记密码和邮箱验证。
- **组织：**托管版支持多成员组织，角色为 owner 和 admin。只有 owner 能管理计费；创建或删除项目、管理集成由 owner 和 admin 负责。所有被邀请者都是 admin。支持转移所有权和工作区合并。
- **新手引导：**注册后进入 onboarding，并有邮件培育流程（Loops）。

| 套餐 | 年付 | 面向 | 相对上一档增加 |
| --- | ---: | --- | --- |
| Free | $0 | 试用 | 审计上限 50 页；Prompt Research 和 Explorer 不可用 |
| Base | $100 | 自己的网站 | 全部功能；README 写的月付价格为 $10/月 |
| Pro | $300 | 自由职业者、团队 | 多送 20% 用量，邮件优先支持 |
| Agency | $2,000 | 代理商 | 再多送 20%，Slack Connect 专属频道，AI 落地陪跑 |

- **积分体系**（Autumn 加 Stripe）：先消耗套餐赠送的积分，再消耗充值积分。每次调用先原子冻结积分，再按 DataForSEO 实际返回的成本扣费，防止并发超支。
- **自动充值：**充值积分低于 $5 时自动扣卡，可设置每次充值金额和月度预算上限。扣款失败时会在页面横幅和报错信息中说明原因。
- Billing 页面展示用量图表、按功能的消费明细和发票。

## 10 部署形态

| 形态 | 说明 | 认证 |
| --- | --- | --- |
| 托管版 openseo.so | 官方 SaaS，生产环境使用 Postgres | 账号、组织、计费完整 |
| Cloudflare 自托管 | 一条命令部署到用户自己的 Cloudflare 账户，免费套餐即可运行 | 邮箱密码，或 Cloudflare Access |
| Docker 自托管 | 预构建镜像，内含 celld 运行时（在 SQLite 和 S3 上模拟 D1、DO、KV、R2、Workflows），配合 RustFS、Nginx 和爬虫出口网关；适用于 Coolify、Portainer、Dokploy 等平台 | `password` 或 `local_noauth` |
| Every App | 运行在 Every App Gateway 之后，由网关负责登录 | 网关 SDK |

自托管需要自备 `DATAFORSEO_API_KEY`。可选 `OPENROUTER_API_KEY`（AI 研究类功能）、`GOOGLE_MAPS_API_KEY`（Local SEO 地图）和 Google OAuth 凭证（GSC、GA4）。分享链接仅托管版可用。

## 11 技术架构

| 层 | 选型 |
| --- | --- |
| 框架 | TanStack Start（React 19、TanStack Router、Query、Form），Vite 8，Tailwind 4，shadcn（Base UI） |
| 运行时 | Cloudflare Workers；审计使用独立的 Worker（`audit-worker`） |
| 数据 | Drizzle ORM，同时兼容 D1/SQLite 和 Postgres；KV 缓存、R2 存储、Durable Objects |
| 异步任务 | Cloudflare Workflows：站点审计、排名检查、本地排名、AI 可见度；cron 调度 |
| AI | Vercel AI SDK v6，Cloudflare `agents`（SAM），OpenRouter |
| MCP | `@modelcontextprotocol/sdk`，OAuth Provider，ChatGPT Apps 小组件 |
| 认证与计费 | Better Auth，Autumn 加 Stripe，Svix webhook，Loops 邮件 |
| 后端分层约定 | server function → service → repository；凡是项目级接口都必须显式传入 `projectId`，由中间件做组织级鉴权 |

## 12 调研结论

### 亮点

- **以 Agent 为第一入口：**83 个 MCP 工具、12 个 Skills、应用内 SAM 共用同一套工具和项目记忆，这在同类开源工具中少见。
- **AI 可见度功能完整：**研究、对比、跟踪、引用分析形成闭环，覆盖 5 个 AI 引擎。
- **成本透明：**按 DataForSEO 实际成本加价，付费操作前预估费用，自托管可以直接零加价使用。
- **部署灵活：**同一套代码支持 SaaS、Cloudflare、Docker 和 Every App，并兼容 SQLite 与 Postgres 两种数据库。
- **免费数据优先：**GSC 和 GA4 的第一方数据不消耗积分，Dashboard 的 Keywords 视图完全依赖 GSC。

### 局限与待观察

- 付费 SEO 数据只依赖 DataForSEO 一家，数据质量和覆盖范围受其限制。例如 Domain Overview 在 Labs 不覆盖的国家不可用；Prompt Research 只支持美国英语。
- Local SEO 仍是 Beta，且依赖 Google Maps Key。
- 报告没有版本历史；分享链接仅托管版可用。
- 团队权限较粗：所有被邀请者都是 admin，`member` 角色已定义但尚未开放。
- 没有内容写作或编辑器、没有站点变更自动监控（审计按需触发，IndexNow 也不会自动提交），这些工作交给外部 Agent 完成。

> 本文依据仓库在 2026-11-02 的 main 分支整理。导航中的功能以 `src/client/navigation/items.ts` 为准，未上线或隐藏的入口（如 Brand Lookup）已单独注明。价格和限额可能随发布调整，以 openseo.so/pricing 为准。

---

资料来源：README、docs/maintainers/specs 0001–0018、web/content/docs、src/server/mcp/tools、src/shared/audit-issues.ts、发布说明 v0.1.20–v0.1.22
