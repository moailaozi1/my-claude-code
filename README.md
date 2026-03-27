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

### 后续可扩展方向

- 增加原始榜单缓存，支持重跑
- 增加手动补跑某一天的能力
- 后续升级为“前夜快照 + 次日摘要”方案，降低对第三方 archive 的依赖
