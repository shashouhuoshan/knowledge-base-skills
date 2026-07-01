生成或更新项目代码知识库（系统 → 子系统 → 模块三层结构）。`--update` 为增量模式。

## 全量生成

### 阶段 1：结构识别

派一个 Subagent 扫描项目并输出分层建议树。

向该 Subagent 提供：
- 项目目录树（排除 node_modules, .git, dist, build 等）
- 依赖文件（package.json, go.mod, Cargo.toml 等）
- 设计文档（按以下顺序搜索：docs/superpowers/specs/ → docs/superpowers/plans/ → docs/ → README.md → config 显式映射）
- `.knowledge-base/config.yaml`（若存在）

Subagent 输出分层建议树后，呈现给用户确认。分层规则：
- 系统：整个项目，名称取自 package.json name 或目录名
- 子系统：顶层目录分组或 monorepo 子 package，通常 2-8 个
- 模块：子系统内职责单一的代码单元，每个子系统 2-10 个
- 命名使用 kebab-case

### 阶段 2：并行生成

用户确认后，并行派 Subagent 生成各层级条目。

**系统级 Agent**：分析系统边界、技术栈、外部依赖、子系统列表。输出到 `docs/kb/system/<name>.md`。

**子系统 Agent（并行 N 个）**：分析子系统内部结构、接口、依赖关系、模块列表。输出到 `docs/kb/subsystem/<name>.md`。

**模块 Agent（并行 M 个）**：从源码提取函数签名、数据结构、依赖。这是最细粒度——签名必须从实际代码逐字复制，不编造。输出到 `docs/kb/module/<name>.md`。

模板优先级：`.knowledge-base/templates/<level>.md` > 内置默认模板（参考 /kb-init 中的模板定义）。

各层级知识库条目章节：

系统级：概述、技术栈、依赖关系、系统边界（对外接口+子系统列表）、关键文件、设计决策、注意事项

子系统级：概述、技术栈、依赖关系、接口（对外 API+模块列表）、关键文件、设计决策、注意事项

模块级：概述、核心接口（函数签名+数据结构）、依赖关系、关键文件、设计决策、注意事项

### 阶段 3：索引汇总

派一个 Subagent：
- 生成 `docs/kb/.manifest.yaml`（记录 last_commit、每个条目的来源文件和 hash）
- 更新 `skills/knowledge-base/SKILL.md`（三层索引，含相对路径链接）
- 输出生成报告（各层级条目数、有/无设计文档统计）

## 增量更新（`--update`）

1. Git 模式：`git diff --name-only <manifest.last_commit> HEAD` 检测变更
2. 映射：单文件变更→对应模块，多文件→子系统+子模块，package.json→系统级
3. 仅重生成受影响条目
4. 更新 manifest 和索引

非 Git 项目降级为文件 Hash 对比。
