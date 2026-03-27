# my-claude-code

## GitHub Trending → Obsidian 自动化想法

目标：每天中午 12:00 获取 GitHub 前一天的 Trending 榜单，提炼每个项目的核心要点，并写入 Obsidian 的 `daily/github_trend.md`。

### 收敛后的方案

- **数据源**：第三方 GitHub Trending archive
- **调度方式**：本地定时任务，每天 12:00 运行
- **输出位置**：Obsidian `daily/github_trend.md`
- **写入策略**：按日期区块幂等更新，避免重复追加
- **失败策略**：失败时不覆盖已有成功内容，保留状态信息

### 推荐流程

1. 本地调度器在每天 12:00 触发任务
2. 读取前一天的 GitHub Trending archive 数据
3. 标准化项目信息（仓库名、链接、描述、语言等）
4. 为每个项目生成简明摘要
5. 生成当日“趋势总结”
6. 将结果写入 `daily/github_trend.md`
7. 若当天区块已存在，则替换该日期区块而不是重复追加

### Markdown 输出建议

```md
## 2026-03-26

- 状态：成功
- 生成时间：2026-03-27 12:00
- 数据来源：GitHub Trending Archive
- 榜单日期：2026-03-26
- 项目数：10

### 今日趋势总结
- ...
- ...
- ...

### owner/repo
https://github.com/owner/repo

一句话总结。

- 它是什么
- 为什么上榜
- 值得关注的点
```

### 实现规划（v1 脚本版）

第一版按**独立本地 Node 脚本**实现，不做 VS Code 扩展集成。目标是先把“可运行、可定时、可重跑、失败可见”的主流程跑通。

#### 架构

- 本地 Node 脚本负责拉取 archive、解析数据、生成摘要、更新 Markdown
- 本地系统调度器负责每天 12:00 触发执行
- 不依赖远程调度，不把 Claude Code 当作唯一自动执行层

#### 建议目录

```text
scripts/github_trend/
├── run_daily.mjs
├── config.mjs
├── fetch_archive.mjs
├── parse_archive.mjs
├── summarize.mjs
├── render_markdown.mjs
├── upsert_obsidian.mjs
├── lib/
│   └── date.mjs
└── fixtures/
```

#### 核心流程

1. 调度器运行 `node scripts/github_trend/run_daily.mjs`
2. 默认计算前一天日期，也支持 `--date YYYY-MM-DD`
3. 从第三方 archive 拉取目标日期榜单
4. 解析为统一结构：`owner`、`repo`、`url`、`description`、`language`、`stars`、`rank`
5. 生成简洁的趋势总结和项目摘要
6. 渲染为 Markdown 区块
7. 幂等写入 Obsidian 的 `daily/github_trend.md`

#### 配置

建议先使用环境变量：

- `TRENDING_ARCHIVE_URL_TEMPLATE`
- `OBSIDIAN_VAULT_PATH`
- `GITHUB_TREND_NOTE_PATH`（默认 `daily/github_trend.md`）
- `TOP_N`（默认 10）
- `REQUEST_TIMEOUT_MS`
- `LOG_LEVEL`

#### 幂等写入策略

为避免误伤手工内容，使用托管块标记而不是只靠标题匹配：

```md
<!-- github-trend:2026-03-26:start -->
## 2026-03-26
...
<!-- github-trend:2026-03-26:end -->
```

同一天重跑时：
- 如果块已存在，则替换该块
- 如果内容完全一致，则不写文件
- 如果文件不存在，则新建
- 文件中的其他手工内容保持不变

#### 失败处理

- 抓取失败：不修改现有笔记，直接报错退出
- 解析失败：保留原始响应，提示数据格式异常
- 写入失败：保留中间结果，不覆盖已有成功内容
- 支持 `--dry-run`，便于先验证输出再落盘

#### 验证思路

- 用固定 fixture 验证 archive 解析
- 用固定输入验证 Markdown 渲染结果
- 验证 upsert 场景：创建、追加、替换、相同内容 no-op、保留手工内容
- 在本地手动运行通过后，再接入系统定时任务

#### 后续扩展

- 增加 README 首段抓取，提升摘要质量
- 支持补跑指定日期
- 支持多输出目标（固定文件 / Obsidian Daily Note 区块）
- 后续迁移到自有快照采集，降低对第三方 archive 的依赖

### 后续可扩展方向

- 增加原始榜单缓存，支持重跑
- 增加手动补跑某一天的能力
- 后续升级为“前夜快照 + 次日摘要”方案，降低对第三方 archive 的依赖
