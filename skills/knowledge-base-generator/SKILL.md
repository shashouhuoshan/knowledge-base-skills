---
name: knowledge-base-generator
description: Use when generating structured code knowledge bases for projects, when running /kb:init, /kb:generate, or /kb:status commands, or when integrating knowledge base updates into SDD workflows
---

# Knowledge Base Generator

为项目生成三层结构化代码知识库（系统 → 子系统 → 模块），结合 Superpowers SDD 模式，在 finishing 阶段自动增量更新。

## 命令概览

| 命令 | 用途 |
|------|------|
| `/kb:init` | 初始化项目配置、目录结构和默认模板 |
| `/kb:generate [--update]` | 全量生成 / 增量更新知识库 |
| `/kb:status` | 查看知识库覆盖率和过期状态 |

## 前置条件

- 项目根目录可写
- 建议项目为 Git 仓库（非 Git 项目降级为 Hash 模式）

---

## /kb:init — 项目初始化

### 流程

1. 检查是否已初始化（`.knowledge-base/config.yaml` 是否存在）
   - 已存在：提示用户，询问是否覆盖（保留旧文件为 `.bak`）
   - 不存在：继续

2. 创建目录结构：

```bash
mkdir -p .knowledge-base/templates
mkdir -p docs/kb/{system,subsystem,module}
mkdir -p skills/knowledge-base
```

3. 写入默认配置文件 `.knowledge-base/config.yaml`：
   - 内容来自 `config/default-config.yaml`
   - 将内容复制写入项目

4. 复制默认模板到 `.knowledge-base/templates/`：
   - 从 `templates/system.md` → `.knowledge-base/templates/system.md`
   - 从 `templates/subsystem.md` → `.knowledge-base/templates/subsystem.md`
   - 从 `templates/module.md` → `.knowledge-base/templates/module.md`

5. 生成空的索引 Skill 骨架 `skills/knowledge-base/SKILL.md`：

```markdown
---
name: knowledge-base
description: Use when needing to understand project architecture, module interfaces, system dependencies, or codebase structure before making changes
---

# 项目知识库

本知识库由 knowledge-base-generator 管理。运行 `/kb:generate` 生成完整内容。

## 系统

（待生成 — 运行 `/kb:generate`）

## 子系统

（待生成 — 运行 `/kb:generate`）

## 模块

（待生成 — 运行 `/kb:generate`）
```

6. 输出初始化报告：

```
✅ 知识库生成器初始化完成

创建的文件：
  .knowledge-base/config.yaml          — 项目配置
  .knowledge-base/templates/system.md  — 系统级模板
  .knowledge-base/templates/subsystem.md — 子系统级模板
  .knowledge-base/templates/module.md  — 模块级模板
  skills/knowledge-base/SKILL.md       — 索引 Skill（空骨架）
  docs/kb/                             — 知识库产物目录

下一步：运行 /kb:generate 开始生成知识库
```

---

## /kb:generate — 知识库生成

### 模式判断

1. 检查 `.knowledge-base/config.yaml` 是否存在 → 不存在则提示先运行 `/kb:init`
2. 检查 `--update` 标志：
   - 无标志 → **全量生成模式**
   - 有标志 → **增量更新模式**

### 全量生成流程

#### 阶段 1：结构识别

**调度识别 Agent：**

读取 `prompts/recognition-agent.md` 作为 Subagent 系统提示词。

向识别 Agent 提供：
- 项目目录树（`ls -R` 或 `tree`，排除 scan.exclude 中的目录）
- 依赖文件内容（自动发现并读取 package.json / go.mod / Cargo.toml / requirements.txt 等）
- 设计文档（按三级发现策略搜索并读取）
  - Level 1: `docs/superpowers/specs/`, `docs/superpowers/plans/`
  - Level 2: `docs/`, `design/`, `README.md`
  - Level 3: config.yaml 中的 `design_docs` 显式映射
- 配置文件内容

识别 Agent 输出分层建议树后，**呈现给用户确认**：

```
识别结果 — 分层建议树：

系统: my-app
├── 子系统: auth (src/auth/)
│   ├── 模块: jwt-service (src/auth/jwt.ts)
│   └── 模块: user-repository (src/auth/user-repo.ts)
├── 子系统: api-gateway (src/api/)
│   ├── 模块: router (src/api/router.ts)
│   └── 模块: middleware (src/api/middleware/)
└── 子系统: data-layer (src/data/)
    └── 模块: database-connector (src/data/db.ts)

关联设计文档:
  - docs/superpowers/specs/2026-06-15-architecture-design.md → 系统级

置信度: high

请确认或修改（输入调整后的结构，或输入 "ok" 确认）：
```

用户确认后进入阶段 2。

#### 阶段 2：并行生成

**并行调度生成 Agent（无依赖关系，全部并行）：**

1. **系统级 Agent × 1**：
   - 提示词：`prompts/system-agent.md`
   - 输入：分层树系统节点 + 系统级设计文档 + 模板
   - 输出文件：`docs/kb/system/<系统名>.md`

2. **子系统 Agent × N**（并行）：
   - 提示词：`prompts/subsystem-agent.md`
   - 输入：各自的子系统节点 + 代码目录 + 设计文档 + 模板
   - 输出文件：`docs/kb/subsystem/<子系统名>.md`

3. **模块 Agent × M**（并行）：
   - 提示词：`prompts/module-agent.md`
   - 输入：各自的模块节点 + 源代码文件 + 设计文档 + 模板
   - 输出文件：`docs/kb/module/<模块名>.md`

**模板选择逻辑：**
- 检查 `.knowledge-base/templates/<level>.md` 是否存在
- 存在 → 使用项目自定义模板
- 不存在 → 使用插件默认模板（`templates/<level>.md`）

#### 阶段 3：索引汇总

**调度索引 Agent：**

- 提示词：`prompts/index-agent.md`
- 输入：分层树 + 所有已生成的条目文件路径 + Git 信息
- 输出：
  - `docs/kb/.manifest.yaml`
  - `skills/knowledge-base/SKILL.md`
  - 生成报告

生成报告格式：

```
✅ 知识库生成完成

| 层级 | 条目数 | 有设计文档 | 无设计文档 |
|------|--------|-----------|-----------|
| 系统 | 1 | 1 | 0 |
| 子系统 | 3 | 2 | 1 |
| 模块 | 5 | 3 | 2 |

总计：9 个条目

⚠️ 无设计文档的条目：
  - subsystem/data-layer
  - module/database-connector
  - module/middleware
```

### 增量更新流程（`/kb:generate --update`）

#### 1. 检测变更

**Git 模式（优先）：**
- 读取 `docs/kb/.manifest.yaml` 中的 `last_commit`
- 执行 `git diff --name-only <last_commit> HEAD`
- 获取变更文件列表

**Hash 模式（降级，非 Git 项目）：**
- 扫描 `.manifest.yaml` 中所有条目的 sources.code 文件
- 计算当前 hash，对比存储的 hash
- 标记 hash 不一致的条目为 stale

#### 2. 映射影响范围

| 变更类型 | 影响范围 | 重生成策略 |
|---------|---------|-----------|
| 单模块源文件变更 | 该模块 | 重生成该模块条目 |
| 子系统内多文件变更 | 该子系统 + 所有子模块 | 重生成子系统条目 + 所有子模块条目 |
| 依赖文件变更 (package.json) | 系统级 | 重生成系统条目 |
| 新增文件/目录 | 对应的新条目 | 先更新分层树，再生成新条目 |
| 删除文件/目录 | 对应的条目 | 标记为删除，从索引移除 |
| 设计文档新增/更新 | 关联的条目 | 重生成关联条目 |

#### 3. 重新生成

仅调度受影响的 Agent，并行重新生成变更条目。

#### 4. 更新索引

调度索引 Agent 更新 `.manifest.yaml` 和索引 Skill。

---

## /kb:status — 状态检查

### 流程

1. 读取 `docs/kb/.manifest.yaml`
   - 不存在 → 报告"知识库未生成，请运行 /kb:generate"

2. 检测变更：
   - Git 模式：`git diff --name-only <last_commit> HEAD`
   - Hash 模式：对比 hash

3. 输出状态报告：

```
知识库状态报告
生成时间：2026-06-30 14:30
最后提交：a1b2c3d (匹配 / 不匹配)

| 层级 | 总计 | 最新 | 过期 | 缺失 |
|------|------|------|------|------|
| 系统 | 1 | 1 | 0 | 0 |
| 子系统 | 3 | 2 | 1 | 0 |
| 模块 | 5 | 3 | 2 | 0 |

过期条目：
  - subsystem/auth — 源文件已变更（src/auth/ 下 3 个文件修改）
  - module/jwt-service — 源文件已变更（src/auth/jwt.ts）
  - module/user-repository — 父级子系统过期

建议：运行 /kb:generate --update 更新过期条目
```

---

## 模板系统

### 模板查找优先级

生成知识库条目时：
1. 检查 `.knowledge-base/templates/<level>.md` → 项目自定义模板
2. 回退到插件默认模板 `templates/<level>.md`

### 默认模板

插件提供三套默认模板，位于 `templates/` 目录：
- `system.md` — 系统级模板（概述、技术栈、依赖、接口、子系统列表、关键文件、设计决策、注意事项）
- `subsystem.md` — 子系统级模板（概述、技术栈、依赖、接口、模块列表、关键文件、设计决策、注意事项）
- `module.md` — 模块级模板（概述、核心接口、依赖、关键文件、设计决策、注意事项）

---

## 配置参考

完整配置 Schema 见 `config/default-config.yaml`。

关键配置项：

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| strategy.mode | hybrid | 分层识别模式 |
| update.trigger | finishing | 更新触发方式 |
| output.kb_dir | docs/kb/ | 知识库产物目录 |
| output.skill_dir | skills/knowledge-base/ | 索引 Skill 目录 |

---

## 与 SDD 集成

### brainstorming 阶段
Agent 自动发现并加载 `skills/knowledge-base/SKILL.md`，了解现有项目架构。

### writing-plans 阶段
Agent 主动查询相关知识库条目，获取接口签名和依赖关系。

### subagent-driven-development 阶段
编排 Agent 在 Dispatch 实现子 Agent 时，附带相关模块的知识库条目内容。

### finishing 阶段
自动执行 `/kb:generate --update`：
1. `git diff main..HEAD` 确定变更范围
2. 更新受影响的条目
3. `git add docs/kb/ skills/knowledge-base/`
4. 随分支合并提交

---

## Red Flags

- 不要跳过识别 Agent 的用户确认步骤（hybrid 模式下）
- 不要在生成 Agent 失败时静默跳过——报告失败并重试
- 不要在没有变更时仍运行全量生成
- 不要在非 Git 项目上尝试 Git 命令——降级到 Hash 模式
- 不要修改用户自定义的模板文件
