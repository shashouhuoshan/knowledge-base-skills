# Knowledge Base Generator Plugin 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建 `knowledge-base-generator` Superpowers Skill Plugin，实现 `/kb:init`、`/kb:generate`、`/kb:status` 三个命令，通过 Subagent 并行编排为项目生成三层代码知识库。

**Architecture:** 纯 Skill 实现，无外部 CLI 依赖。一个主编排 Skill（SKILL.md）定义流程，5 个 Subagent 提示词模板（prompts/）定义各 Agent 行为，3 个默认模板（templates/）定义知识库条目格式，1 个默认配置文件（config/）定义项目初始化内容。

**Tech Stack:** Markdown（Skill 文件 + 模板），YAML（配置），无编程语言依赖。

## Global Constraints

- 纯 Skill 实现，不引入编程语言 CLI 工具
- 所有文件使用 Markdown 格式，遵循 Superpowers Skill 规范
- YAML frontmatter 中 name 仅使用字母、数字、连字符
- description 以 "Use when..." 开头，仅描述触发条件，不总结流程
- 项目自定义模板路径：`.knowledge-base/templates/`
- 知识库产物路径：`docs/kb/{system,subsystem,module}/`
- 索引 Skill 路径：`skills/knowledge-base/SKILL.md`
- 配置文件路径：`.knowledge-base/config.yaml`
- Git 联动优先，非 Git 项目降级为 Hash 模式

---

### Task 1: 项目骨架与默认配置

**Files:**
- Create: `skills/knowledge-base-generator/config/default-config.yaml`

**Interfaces:**
- Produces: `default-config.yaml` — `/kb:init` 时写入 `.knowledge-base/config.yaml` 的默认内容

- [ ] **Step 1: 创建目录结构**

```bash
mkdir -p skills/knowledge-base-generator/{templates,prompts,config}
```

- [ ] **Step 2: 验证目录结构**

```bash
ls -R skills/knowledge-base-generator/
```

Expected: 显示 `config/  prompts/  templates/` 三个子目录。

- [ ] **Step 3: 编写默认配置文件**

创建 `skills/knowledge-base-generator/config/default-config.yaml`：

```yaml
# Knowledge Base Generator 配置
# 由 /kb:init 自动生成，可手动编辑

# ===== 分层策略 =====
strategy:
  # 边界识别方式：auto | manual | hybrid
  mode: hybrid

# ===== 扫描配置 =====
scan:
  # 排除目录
  exclude:
    - node_modules
    - .git
    - dist
    - build
    - __pycache__
    - .next
    - target

# ===== 设计文档映射 =====
design_docs:
  # 自动扫描路径
  scan_paths:
    - docs/superpowers/specs/
    - docs/superpowers/plans/
    - docs/
    - README.md

# ===== 更新策略 =====
update:
  # 触发方式：finishing | hook | manual
  trigger: finishing
  # trigger=hook 时的 hook 类型：pre-commit | post-merge
  hook_type: post-merge
  # 是否自动提交 kb 更新
  auto_commit: true

# ===== 模板配置 =====
templates:
  # 模板目录（相对于项目根目录）
  path: .knowledge-base/templates/
  # 使用的模板集：default | custom
  use: default

# ===== 输出配置 =====
output:
  # 知识库产物目录
  kb_dir: docs/kb/
  # 索引 skill 目录
  skill_dir: skills/knowledge-base/
```

- [ ] **Step 4: 验证 YAML 格式**

```bash
# 用任意 YAML 解析器验证语法（Python 内建）
python -c "import yaml; yaml.safe_load(open('skills/knowledge-base-generator/config/default-config.yaml')); print('YAML valid')"
```

Expected: `YAML valid`

---

### Task 2: 默认知识库条目模板

**Files:**
- Create: `skills/knowledge-base-generator/templates/system.md`
- Create: `skills/knowledge-base-generator/templates/subsystem.md`
- Create: `skills/knowledge-base-generator/templates/module.md`

**Interfaces:**
- Produces: 三个 Markdown 模板文件，定义知识库条目结构。模板中包含 `{{PLACEHOLDER}}` 变量供生成 Agent 替换。

- [ ] **Step 1: 创建系统级模板**

创建 `skills/knowledge-base-generator/templates/system.md`：

```markdown
---
level: system
name: {{SYSTEM_NAME}}
updated: {{UPDATED_AT}}
sources:
  code: {{SOURCE_PATHS}}
  docs: {{DESIGN_DOC_PATHS}}
---

# {{SYSTEM_NAME}}

## 概述

{{OVERVIEW}}

## 技术栈

| 技术 | 用途 |
|------|------|
{{TECH_STACK_TABLE}}

## 依赖关系

### 上游依赖（本系统依赖的外部系统/服务）

{{UPSTREAM_DEPENDENCIES}}

### 下游消费者（依赖本系统的外部系统/服务）

{{DOWNSTREAM_CONSUMERS}}

## 系统边界

### 对外接口

{{EXTERNAL_INTERFACES}}

### 子系统列表

{{SUBSYSTEM_LIST}}

## 关键文件

| 文件 | 职责 |
|------|------|
{{KEY_FILES_TABLE}}

## 设计决策

{{DESIGN_DECISIONS}}

## 注意事项

{{NOTES}}
```

- [ ] **Step 2: 创建子系统级模板**

创建 `skills/knowledge-base-generator/templates/subsystem.md`：

```markdown
---
level: subsystem
name: {{SUBSYSTEM_NAME}}
parent: {{PARENT_SYSTEM}}
updated: {{UPDATED_AT}}
sources:
  code: {{SOURCE_PATHS}}
  docs: {{DESIGN_DOC_PATHS}}
---

# {{SUBSYSTEM_NAME}}

## 概述

{{OVERVIEW}}

## 技术栈

| 技术 | 用途 |
|------|------|
{{TECH_STACK_TABLE}}

## 依赖关系

### 上游依赖（本子系统依赖的其他子系统/外部服务）

{{UPSTREAM_DEPENDENCIES}}

### 下游消费者（依赖本子系统的其他子系统）

{{DOWNSTREAM_CONSUMERS}}

## 接口

### 对外暴露的 API / 接口

{{EXTERNAL_INTERFACES}}

### 内部模块列表

{{MODULE_LIST}}

## 关键文件

| 文件 | 职责 |
|------|------|
{{KEY_FILES_TABLE}}

## 设计决策

{{DESIGN_DECISIONS}}

## 注意事项

{{NOTES}}
```

- [ ] **Step 3: 创建模块级模板**

创建 `skills/knowledge-base-generator/templates/module.md`：

```markdown
---
level: module
name: {{MODULE_NAME}}
parent: {{PARENT_SUBSYSTEM}}
updated: {{UPDATED_AT}}
sources:
  code: {{SOURCE_PATHS}}
  docs: {{DESIGN_DOC_PATHS}}
---

# {{MODULE_NAME}}

## 概述

{{OVERVIEW}}

## 核心接口

### 函数 / 方法签名

{{FUNCTION_SIGNATURES}}

### 数据结构

{{DATA_STRUCTURES}}

## 依赖关系

### 内部依赖（本模块依赖的其他模块）

{{INTERNAL_DEPENDENCIES}}

### 外部依赖（第三方库）

{{EXTERNAL_DEPENDENCIES}}

## 关键文件

| 文件 | 职责 |
|------|------|
{{KEY_FILES_TABLE}}

## 设计决策

{{DESIGN_DECISIONS}}

## 注意事项

{{NOTES}}
```

- [ ] **Step 4: 验证模板结构**

验证每个模板包含：YAML frontmatter（level, name, parent, updated, sources）、概述、依赖关系、关键文件章节。

```bash
for f in skills/knowledge-base-generator/templates/{system,subsystem,module}.md; do
  echo "=== $f ==="
  head -1 "$f"
  grep -c "^## " "$f"
done
```

Expected: 每个文件输出 frontmatter 行和章节计数（≥5）。

---

### Task 3: Subagent 提示词模板

**Files:**
- Create: `skills/knowledge-base-generator/prompts/recognition-agent.md`
- Create: `skills/knowledge-base-generator/prompts/system-agent.md`
- Create: `skills/knowledge-base-generator/prompts/subsystem-agent.md`
- Create: `skills/knowledge-base-generator/prompts/module-agent.md`
- Create: `skills/knowledge-base-generator/prompts/index-agent.md`

**Interfaces:**
- Produces: 5 个 Subagent 提示词模板，供主编排 Skill 在调度 Subagent 时引用。

- [ ] **Step 1: 创建识别 Agent 提示词**

创建 `skills/knowledge-base-generator/prompts/recognition-agent.md`：

```markdown
# Recognition Agent — 项目结构识别与分层建议

## 任务

扫描项目代码和文档，识别系统/子系统/模块的三层边界，输出分层建议树供用户确认。

## 输入

你将收到以下信息：

1. **项目目录树**（排除 node_modules, .git, dist, build 等）
2. **依赖文件内容**（package.json, go.mod, Cargo.toml, requirements.txt 等）
3. **设计文档列表**（若存在，来自 docs/superpowers/specs/, docs/superpowers/plans/, docs/ 等路径）
4. **配置文件**（.knowledge-base/config.yaml，若存在）

## 分析步骤

### 1. 识别系统边界

- 查看项目根目录的依赖文件，确定项目类型和技术栈
- 如果是 monorepo，每个 package 可能是独立的子系统
- 如果是单一项目，整个项目为一个系统
- 系统命名：使用项目名（取自 package.json 的 name 字段或目录名）

### 2. 识别子系统边界

子系统是系统中一个可独立描述的功能域。识别依据：
- 顶层目录分组（如 `src/auth/`, `src/api/`, `src/data/`）
- Monorepo 中的子 package
- 架构分层（如 frontend, backend, shared）
- 设计文档中明确划分的功能模块

### 3. 识别模块边界

模块是子系统中一个职责单一的代码单元。识别依据：
- 子系统目录下的子目录或关键文件分组
- 独立的类/模块文件（如 `jwt.ts`, `oauth.ts` 共同构成 auth 子系统）
- 单一职责原则：一个模块只做一件事

### 4. 关联设计文档

- 按 spec 定义的三级发现策略查找设计文档
- 尝试将设计文档与对应的层级关联
- 没有设计文档的层级标注为"无设计文档"

## 输出格式

```yaml
# 分层建议树
system:
  name: <系统名>
  description: <一句话描述>
  design_docs: [<关联的设计文档路径>]
  subsystems:
    - name: <子系统名>
      description: <一句话描述>
      path: <代码目录路径>
      design_docs: [<关联的设计文档路径>]
      modules:
        - name: <模块名>
          description: <一句话描述>
          path: <代码目录路径>
          key_files: [<关键文件路径>]
          design_docs: [<关联的设计文档路径>]
        - name: <模块名>
          ...

# 识别说明
notes:
  confidence: high | medium | low  # 整体置信度
  uncertainties: [<不确定的边界说明>]
  recommendations: [<调整建议>]
```

## 输出规则

- 子系统数量：通常 2-8 个，过多说明粒度太细
- 模块数量：每个子系统下通常 2-10 个模块
- 如果项目很小（< 20 个源文件），系统 = 子系统，模块直接挂在系统下
- 置信度标记为 low 或 medium 时必须附带 uncertainties 说明
- 命名使用 kebab-case
```

- [ ] **Step 2: 创建系统级 Agent 提示词**

创建 `skills/knowledge-base-generator/prompts/system-agent.md`：

```markdown
# System Agent — 系统级知识库条目生成

## 任务

根据已确认的分层树和设计文档，生成系统级知识库条目。

## 输入

1. **分层树中的系统节点**（含子系统列表）
2. **项目根目录依赖文件**（package.json 等）
3. **系统级设计文档**（若存在）
4. **系统级模板**（.knowledge-base/templates/system.md 或插件默认模板）

## 生成步骤

### 1. 填充概述

- 阅读项目 README、设计文档概述章节
- 用 2-3 句话描述系统的核心目标和价值
- 如果没有文档，从依赖文件和目录结构推断

### 2. 识别技术栈

- 从依赖文件提取主要技术和框架
- 格式化为表格：技术名 | 用途

### 3. 分析依赖关系

- 上游依赖：外部 API、数据库、消息队列、第三方服务
- 下游消费者：如果有，谁依赖这个系统（从文档推断）

### 4. 提取系统接口

- 系统对外暴露的 API 端点、事件、CLI 命令等
- 从入口文件（main.ts, index.js, app.py 等）和路由配置中提取

### 5. 列出子系统

- 从分层树复制子系统列表，每个附带一句话描述

### 6. 整理关键文件

- 入口文件、配置文件、构建脚本、部署配置等
- 最多 10 个文件

### 7. 提取设计决策

- 从设计文档中提取架构决策（ADR）
- 如果没有文档，标注"无明确设计文档，以下为代码推断"

### 8. 注意事项

- 已知的限制、技术债务、需要特别注意的边界条件

## 输出

直接输出填充后的系统级模板内容。模板占位符全部替换为实际内容。未找到信息的部分保留空章节并标注"未检测到相关信息"。

## 约束

- 文件路径使用相对于项目根目录的路径
- 不编造信息——不确定的内容标注"推断："
- 章节顺序严格按照模板
```

- [ ] **Step 3: 创建子系统 Agent 提示词**

创建 `skills/knowledge-base-generator/prompts/subsystem-agent.md`：

```markdown
# Subsystem Agent — 子系统级知识库条目生成

## 任务

根据已确认的分层树，为指定子系统生成知识库条目。

## 输入

1. **分层树中该子系统的完整节点**（含模块列表、代码路径）
2. **子系统目录下的源代码文件列表和关键文件内容**
3. **子系统级设计文档**（若存在）
4. **子系统级模板**（.knowledge-base/templates/subsystem.md 或插件默认模板）

## 生成步骤

### 1. 填充概述

- 阅读子系统目录下的代码注释、README
- 用 2-3 句话描述该子系统的核心职责
- 说明它在整个系统中的角色

### 2. 识别技术栈

- 该子系统使用的特定技术和框架
- 格式化为表格

### 3. 分析依赖关系

- 上游：该子系统调用的其他子系统或外部服务
- 下游：调用该子系统的其他子系统
- 从 import/require 语句推断

### 4. 提取接口

- 该子系统对外暴露的公共 API、函数、类
- 从 export 语句、公共接口文件中提取
- 包含关键函数签名

### 5. 列出模块

- 从分层树复制模块列表，每个附带一句话描述

### 6. 整理关键文件

- 子系统目录下最重要的文件
- 最多 15 个文件

### 7. 提取设计决策

- 从设计文档或代码注释中提取

### 8. 注意事项

- 该子系统的陷阱、约束、已知问题

## 输出

直接输出填充后的子系统级模板内容。模板占位符全部替换为实际内容。

## 约束

- 文件路径使用相对于项目根目录的路径
- 不编造信息
- 章节顺序严格按照模板
- 接口签名从实际代码复制，不做修改
```

- [ ] **Step 4: 创建模块 Agent 提示词**

创建 `skills/knowledge-base-generator/prompts/module-agent.md`：

```markdown
# Module Agent — 模块级知识库条目生成

## 任务

根据已确认的分层树，为指定模块生成知识库条目。模块是最细粒度的知识库单元，重点在于精确的接口文档。

## 输入

1. **分层树中该模块的完整节点**（含代码路径、关键文件）
2. **模块源代码文件内容**
3. **模块级设计文档**（若存在）
4. **模块级模板**（.knowledge-base/templates/module.md 或插件默认模板）

## 生成步骤

### 1. 填充概述

- 阅读模块源码开头注释和函数签名
- 用 1-2 句话描述该模块的单一职责

### 2. 提取核心接口

这是模块级最重要的部分。必须精确：
- 从源码复制所有公开的函数/方法签名（含参数类型和返回类型）
- 从源码复制关键数据结构定义
- 不编造、不简化——与代码完全一致

### 3. 分析依赖关系

- 内部依赖：该模块 import 的本项目其他模块
- 外部依赖：该模块使用的第三方库
- 从 import/require 语句精确提取

### 4. 整理关键文件

- 该模块目录下的所有源文件
- 每个文件附带一句话职责描述

### 5. 提取设计决策

- 从代码注释中的 "NOTE:", "FIXME:", "HACK:" 提取
- 从关联的设计文档中提取

### 6. 注意事项

- 该模块的已知限制、性能考虑、线程安全等
- 从代码注释和实现模式中推断

## 输出

直接输出填充后的模块级模板内容。模板占位符全部替换为实际内容。

## 约束

- 接口签名必须从实际代码逐字复制，不做任何修改
- 不编造信息
- 章节顺序严格按照模板
- 如果源码中确实没有任何设计决策相关信息，标注"未检测到相关设计决策"
```

- [ ] **Step 5: 创建索引 Agent 提示词**

创建 `skills/knowledge-base-generator/prompts/index-agent.md`：

```markdown
# Index Agent — 知识库索引与清单生成

## 任务

汇总所有已生成的知识库条目，生成索引 Skill 文件和清单文件。

## 输入

1. **分层树**（已确认的完整结构）
2. **所有已生成的知识库条目文件路径列表**
3. **上次的 .manifest.yaml**（增量更新时存在，全量生成时为空）
4. **Git 信息**（当前 HEAD commit hash，若非 Git 项目则提供文件 hash 列表）

## 生成步骤

### 1. 生成 .manifest.yaml

在 `docs/kb/.manifest.yaml` 写入：

```yaml
last_commit: <当前 HEAD commit hash，非 Git 项目则为 "N/A">
generated_at: <ISO 8601 时间戳>
entries:
```

然后为每个知识库条目追加一条记录：

```yaml
  system/<系统名>:
    level: system
    file: docs/kb/system/<系统名>.md
    hash: <源文件内容的 SHA256>
    updated: <生成时间>
    sources:
      code: [<源文件路径列表>]
      docs: [<设计文档路径列表>]

  subsystem/<子系统名>:
    level: subsystem
    parent: system/<父系统名>
    file: docs/kb/subsystem/<子系统名>.md
    hash: <源文件内容的 SHA256>
    updated: <生成时间>
    sources:
      code: [<源文件路径列表>]
      docs: [<设计文档路径列表>]

  module/<模块名>:
    level: module
    parent: subsystem/<父子系统名>
    file: docs/kb/module/<模块名>.md
    hash: <源文件内容的 SHA256>
    updated: <生成时间>
    sources:
      code: [<源文件路径列表>]
      docs: [<设计文档路径列表>]
```

Hash 计算逻辑：
- 对该条目 sources.code 中所有文件内容拼接后取 SHA256
- 非 Git 项目用于增量检测；Git 项目作为辅助校验

### 2. 生成索引 Skill

在 `skills/knowledge-base/SKILL.md` 写入：

```markdown
---
name: knowledge-base
description: Use when needing to understand project architecture, module interfaces, system dependencies, or codebase structure before making changes
---

# 项目知识库

本知识库由 knowledge-base-generator 自动生成。
最后更新：{{LAST_UPDATED}}

## 系统

{{SYSTEM_LIST}}

## 子系统

{{SUBSYSTEM_LIST}}

## 模块

{{MODULE_LIST}}
```

系统列表格式：
```markdown
- [系统名](../docs/kb/system/<系统名>.md) — 一句话描述
```

子系统列表格式：
```markdown
- [子系统名](../docs/kb/subsystem/<子系统名>.md) — 一句话描述（属于 [父系统]））
```

模块列表格式：
```markdown
- [模块名](../docs/kb/module/<模块名>.md) — 一句话描述（属于 [父系统/父子系统]））
```

### 3. 输出生成报告

```markdown
## 知识库生成报告

| 层级 | 生成条目数 | 有设计文档 | 无设计文档 |
|------|-----------|-----------|-----------|
| 系统 | N | N | N |
| 子系统 | N | N | N |
| 模块 | N | N | N |

**总计：** N 个条目

**注意：**
- 列出无设计文档的条目
- 列出置信度低的条目
```

## 约束

- .manifest.yaml 中的 hash 必须可复现（相同源文件 → 相同 hash）
- 索引 Skill 中的相对路径必须正确
- 不要遗漏任何条目
```

- [ ] **Step 6: 验证提示词文件完整性**

```bash
for f in skills/knowledge-base-generator/prompts/*.md; do
  echo "=== $(basename $f) ==="
  echo "  Lines: $(wc -l < "$f")"
  echo "  Sections: $(grep -c '^## ' "$f")"
done
```

Expected: 5 个文件，每个 > 30 行，每个 ≥ 3 个章节。

---

### Task 4: 主编排 Skill 文件

**Files:**
- Create: `skills/knowledge-base-generator/SKILL.md`

**Interfaces:**
- Consumes: `prompts/*.md`（Subagent 提示词），`templates/*.md`（条目模板），`config/default-config.yaml`（默认配置）
- Produces: 对用户暴露 `/kb:init`、`/kb:generate`、`/kb:status` 三个命令

- [ ] **Step 1: 创建 SKILL.md 前半部分（概述 + /kb:init）**

创建 `skills/knowledge-base-generator/SKILL.md`：

```markdown
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
```

- [ ] **Step 2: 验证 SKILL.md 完整性**

```bash
echo "Lines: $(wc -l < skills/knowledge-base-generator/SKILL.md)"
echo "Commands: $(grep -c '/kb:' skills/knowledge-base-generator/SKILL.md)"
echo "Sections: $(grep -c '^## ' skills/knowledge-base-generator/SKILL.md)"
echo "Frontmatter: $(head -6 skills/knowledge-base-generator/SKILL.md | grep -c '---')"
```

Expected: Lines > 200，Commands ≥ 3，Sections ≥ 5，Frontmatter = 2（前后分隔线）。

---

### Task 5: 集成验证

**Files:**
- 无新建文件，验证所有已有文件

**Interfaces:**
- Consumes: 所有 Task 1-4 的输出文件
- Produces: 验证通过 / 不通过的结论

- [ ] **Step 1: 验证目录结构完整性**

```bash
echo "=== 预期文件清单 ==="
expected_files=(
  "skills/knowledge-base-generator/SKILL.md"
  "skills/knowledge-base-generator/templates/system.md"
  "skills/knowledge-base-generator/templates/subsystem.md"
  "skills/knowledge-base-generator/templates/module.md"
  "skills/knowledge-base-generator/prompts/recognition-agent.md"
  "skills/knowledge-base-generator/prompts/system-agent.md"
  "skills/knowledge-base-generator/prompts/subsystem-agent.md"
  "skills/knowledge-base-generator/prompts/module-agent.md"
  "skills/knowledge-base-generator/prompts/index-agent.md"
  "skills/knowledge-base-generator/config/default-config.yaml"
)

all_ok=true
for f in "${expected_files[@]}"; do
  if [ -f "$f" ]; then
    echo "  ✅ $f"
  else
    echo "  ❌ MISSING: $f"
    all_ok=false
  fi
done
echo ""
if [ "$all_ok" = true ]; then
  echo "全部文件就位 ✅"
else
  echo "存在缺失文件 ❌"
fi
```

Expected: 全部 10 个文件存在。

- [ ] **Step 2: 验证交叉引用一致性**

```bash
# 检查 SKILL.md 引用的子文件是否存在
echo "=== 交叉引用检查 ==="
for ref in $(grep -oP '(?<=prompts/|templates/|config/)[a-z0-9_/.-]+\.(md|yaml)' skills/knowledge-base-generator/SKILL.md | sort -u); do
  if [ -f "skills/knowledge-base-generator/$ref" ]; then
    echo "  ✅ $ref"
  else
    echo "  ❌ 引用缺失: $ref"
  fi
done
```

Expected: 所有引用文件存在。

- [ ] **Step 3: 验证模板占位符一致性**

```bash
# 确保三个模板中的占位符命名风格一致
echo "=== 模板占位符检查 ==="
for tmpl in skills/knowledge-base-generator/templates/*.md; do
  echo "--- $(basename $tmpl) ---"
  grep -oP '\{\{[A-Z_]+\}\}' "$tmpl" | sort -u
done
```

Expected: 每个模板列出其占位符，命名风格一致（全大写下划线）。

- [ ] **Step 4: 验证 YAML 配置格式**

```bash
python -c "
import yaml
with open('skills/knowledge-base-generator/config/default-config.yaml') as f:
    cfg = yaml.safe_load(f)
required = ['strategy', 'scan', 'design_docs', 'update', 'templates', 'output']
missing = [k for k in required if k not in cfg]
if missing:
    print(f'❌ 缺少配置项: {missing}')
else:
    print('✅ 所有必要配置项存在')
for k in required:
    print(f'  {k}: {list(cfg[k].keys()) if isinstance(cfg[k], dict) else cfg[k]}')
"
```

Expected: 所有 6 个顶层配置项存在。

- [ ] **Step 5: 验证 Frontmatter 规范**

```bash
echo "=== Frontmatter 检查 ==="
for f in skills/knowledge-base-generator/SKILL.md skills/knowledge-base-generator/templates/*.md; do
  name_line=$(sed -n '2p' "$f")
  desc_line=$(sed -n '3p' "$f")
  if echo "$name_line" | grep -q "^name:"; then
    echo "  ✅ $(basename $f): name 字段存在"
  else
    echo "  ❌ $(basename $f): 缺少 name 字段"
  fi
  if echo "$desc_line" | grep -q "^description:"; then
    echo "     description 字段存在"
  else
    echo "     ❌ 缺少 description 字段"
  fi
done
```

Expected: SKILL.md 的 name 和 description 都存在；模板文件的 frontmatter 包含 level/name 等字段。
