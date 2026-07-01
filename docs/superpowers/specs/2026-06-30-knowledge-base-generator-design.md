# 代码知识库生成 Plugin 设计规格

> **日期：** 2026-06-30
> **状态：** 已确认
> **目标：** 为 Coding Agent 设计一个 plugin，通过命令生成分层代码知识库，结合 Superpowers SDD 模式

---

## 1. 概述

### 1.1 目标

设计一个 Superpowers Skill Plugin，为项目自动生成结构化、分层级的代码知识库。知识库主要消费者为 AI Agent（Coding Agent），同时也对人类开发者友好。

### 1.2 核心特性

- **三层分层**：系统级 → 子系统级 → 模块级
- **智能识别**：Agent 自动识别项目边界，用户审核确认
- **增量更新**：基于 Git Diff 精确更新受影响的条目
- **可配置模板**：插件提供默认模板，项目可自定义覆盖
- **SDD 深度集成**：知识库作为 SDD 各阶段共享上下文，finishing 阶段自动更新

---

## 2. 架构

### 2.1 整体架构

采用 **Skill 编排 + Subagent 并行** 模式。一个编排 Agent 调度 5 类 Subagent，各自独立、并行工作。

```
/kb-generate 命令 (skill: knowledge-base-generator)
          │
          ▼
    编排 Agent（主控）
    读取配置 & 调度子任务
          │
  ┌───────┼───────┐
  ▼       ▼       ▼
识别    系统级   子系统级
Agent   Agent   Agent(s)
扫描    分析    并行分析
建议    依赖    生成条目
分层    生成
  │       │       │
  └───────┼───────┘
          ▼
    索引 Agent
    汇总 & 生成索引
```

### 2.2 Agent 职责

| Agent | 数量 | 职责 |
|-------|------|------|
| **识别 Agent** | 1 | 扫描目录树、依赖文件、设计文档，输出分层建议树，等待用户确认 |
| **系统级 Agent** | 1 | 分析顶层系统边界、技术栈、外部依赖，生成系统级条目 |
| **子系统 Agent** | N（并行） | 分析子系统内部结构、接口、依赖关系，生成子系统级条目 |
| **模块 Agent** | M（并行） | 分析模块 API、关键文件、设计决策，生成模块级条目 |
| **索引 Agent** | 1 | 汇总所有条目，生成 `.manifest.yaml` 和索引 Skill 文件 |

---

## 3. 命令体系

| 命令 | 用途 |
|------|------|
| `/kb:init` | 初始化项目配置、目录结构和默认模板 |
| `/kb:generate [--update]` | 全量生成 / 增量更新知识库 |
| `/kb:status` | 查看知识库覆盖率和过期状态 |

### 3.1 `/kb:init` 流程

```
/kb:init
  ├─ 创建 .knowledge-base/config.yaml（默认配置）
  ├─ 创建 .knowledge-base/templates/{system,subsystem,module}.md
  ├─ 创建 docs/kb/{system,subsystem,module}/ 目录
  ├─ 生成 skills/knowledge-base/SKILL.md（空索引骨架）
  └─ ✅ 完成 → 提示运行 /kb:generate
```

`/kb:generate` 检测到未初始化时自动引导用户先执行 `/kb:init`。

### 3.2 `/kb:generate` 流程

```
/kb:generate
  ├─ 阶段 0：读取 .knowledge-base/config.yaml
  │
  ├─ 阶段 1：调度识别 Agent
  │   ├─ 输入：目录树、依赖文件、设计文档（三级发现策略）
  │   ├─ 输出：分层建议树
  │   └─ ⏸️ 等待用户确认
  │
  ├─ 阶段 2：用户确认后，并行调度生成 Agent
  │   ├─ 系统级 Agent × 1
  │   ├─ 子系统 Agent × N（并行）
  │   └─ 模块 Agent × M（并行）
  │
  └─ 阶段 3：调度索引 Agent
      ├─ 更新 .manifest.yaml
      ├─ 更新 skills/knowledge-base/SKILL.md
      └─ 输出生成报告
```

### 3.3 `/kb:status` 流程

```
/kb:status
  ├─ 对比 git diff（自上次生成以来）
  ├─ 标记变更影响的层级
  ├─ 输出覆盖率和过期条目列表
  └─ 建议：哪些条目需要更新
```

---

## 4. 分层边界识别

### 4.1 识别策略（混合模式，默认）

1. Agent 扫描项目结构（目录、依赖文件、import 图）
2. 自动推断分层边界
3. 输出分层建议树
4. 用户审核确认（增删改）
5. 确认后正式生成

### 4.2 支持的模式

| 模式 | 配置值 | 行为 |
|------|--------|------|
| 混合（默认） | `hybrid` | Agent 建议 + 用户确认 |
| 自动 | `auto` | Agent 全自动识别和生成 |
| 手动 | `manual` | 用户在 config.yaml 中显式声明分层 |

---

## 5. 设计文档发现

### 5.1 三级发现策略

**Level 1：Superpowers 规范路径（默认）**

```
docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md
docs/superpowers/plans/YYYY-MM-DD-<feature>-plan.md
```

**Level 2：常见文档目录**

```
docs/  docs/design/  docs/architecture/  design/  specs/  README.md
```

**Level 3：配置显式映射**

在 `.knowledge-base/config.yaml` 中声明文档与层级的对应关系。

### 5.2 优先级

配置显式映射 > Superpowers 规范路径 > 常见目录扫描。设计文档不存在时对应章节留空。

---

## 6. 模板系统

### 6.1 默认模板章节

```markdown
# <条目名称>

## 概述
<2-3 句话描述>

## 技术栈
<关键技术和框架>

## 依赖关系
- 上游依赖 / 下游消费者

## 接口
<对外暴露的 API、接口、数据结构>

## 关键文件
| 文件 | 职责 |

## 设计决策
<关键架构决策>

## 注意事项
<陷阱、边界条件、已知限制>
```

### 6.2 三层级差异

| 章节 | 系统级 | 子系统级 | 模块级 |
|------|--------|---------|--------|
| 概述 | 项目全局 | 功能域描述 | 单一职责 |
| 技术栈 | 整体技术选型 | 子系统特定技术 | 模块级依赖 |
| 依赖关系 | 外部系统/服务 | 子系统间依赖 | 模块间调用 |
| 接口 | 系统边界 API | 子系统接口 | 函数/类签名 |
| 关键文件 | 入口文件、配置 | 子系统目录结构 | 具体源文件 |

### 6.3 自定义模板

项目模板存放于 `.knowledge-base/templates/`，`/kb:init` 时自动生成默认副本。生成时优先级：**项目自定义模板 > 插件默认模板**。

---

## 7. 存储结构

```
.knowledge-base/                  # 配置 & 模板
├── config.yaml
└── templates/
    ├── system.md
    ├── subsystem.md
    └── module.md

docs/kb/                          # 知识库产物（提交 git）
├── system/
│   └── <系统名>.md
├── subsystem/
│   └── <子系统名>.md
├── module/
│   └── <模块名>.md
└── .manifest.yaml                # 元数据清单

skills/knowledge-base/            # 索引 Skill
└── SKILL.md
```

### 7.1 `.manifest.yaml` 结构

```yaml
last_commit: <commit_hash>        # 上次生成时的 HEAD commit
entries:
  system/my-app:
    level: system
    file: docs/kb/system/my-app.md
    hash: a1b2c3d4
    updated: 2026-06-30
    sources:
      code: [src/, package.json]
      docs: [docs/superpowers/specs/2026-06-15-architecture-design.md]
  subsystem/auth:
    level: subsystem
    parent: system/my-app
    file: docs/kb/subsystem/auth.md
    hash: e5f6g7h8
    updated: 2026-06-28
    sources:
      code: [src/auth/]
```

### 7.2 索引 Skill 结构

```markdown
---
name: knowledge-base
description: 项目代码知识库索引，包含系统/子系统/模块三层结构化参考
---

# 项目知识库

## 系统
- [My App](docs/kb/system/my-app.md) — 整体架构与技术栈

## 子系统
- [Auth](docs/kb/subsystem/auth.md) — 认证与授权

## 模块
- [JWT Service](docs/kb/module/jwt-service.md) — Token 签发与验证
```

---

## 8. 增量更新

### 8.1 触发方式

| 方式 | 配置值 | 行为 |
|------|--------|------|
| SDD finishing（默认） | `finishing` | 分支完成时自动触发 |
| Git Hook | `hook` | pre-commit 或 post-merge 触发 |
| 手动 | `manual` | 用户运行 `/kb:generate --update` |

### 8.2 Git 联动机制（优先）

```
/kb:generate --update
  ├─ 读取 .manifest.yaml 中的 last_commit
  ├─ git diff --name-only <last_commit> HEAD
  ├─ 变更文件 → 映射到受影响的知识库条目
  │   ├─ 单文件变更 → 对应模块
  │   ├─ 多文件变更 → 对应子系统及其所有子模块
  │   └─ 依赖变更 (package.json) → 系统级
  └─ 仅重新生成受影响的条目
```

非 Git 项目降级为文件 Hash 对比模式。

### 8.3 SDD Finishing 集成

```
finishing-a-development-branch
  ├─ 代码审查通过
  ├─ 测试通过
  ├─ 🆕 自动执行 /kb:generate --update
  │     ├─ git diff main..HEAD → 变更文件列表
  │     ├─ 映射受影响的条目
  │     ├─ 调度对应 Agent 重新生成
  │     └─ git add docs/kb/ skills/knowledge-base/
  ├─ 合并 / 创建 PR
  └─ ✅ 知识库始终与代码同步
```

---

## 9. 配置系统

### 9.1 `.knowledge-base/config.yaml` Schema

```yaml
strategy:
  mode: hybrid                     # auto | manual | hybrid

scan:
  exclude:                         # 排除目录
    - node_modules
    - .git
    - dist
    - build
    - __pycache__
    - .next
    - target
  extensions: []                   # 关注扩展名（空=自动推断）
  max_depth: 10

design_docs:
  scan_paths:                      # 自动扫描路径
    - docs/superpowers/specs/
    - docs/superpowers/plans/
    - docs/
    - README.md

update:
  trigger: finishing               # finishing | hook | manual
  hook_type: post-merge            # trigger=hook 时生效
  auto_commit: true

templates:
  path: .knowledge-base/templates/
  use: default                     # default | custom

output:
  kb_dir: docs/kb/
  skill_dir: skills/knowledge-base/
```

### 9.2 默认配置（`/kb:init` 生成）

```yaml
strategy:
  mode: hybrid

scan:
  exclude: [node_modules, .git, dist, build, __pycache__, .next, target]

design_docs:
  scan_paths:
    - docs/superpowers/specs/
    - docs/superpowers/plans/
    - docs/
    - README.md

update:
  trigger: finishing
  auto_commit: true

templates:
  path: .knowledge-base/templates/
  use: default

output:
  kb_dir: docs/kb/
  skill_dir: skills/knowledge-base/
```

---

## 10. SDD 集成

### 10.1 各阶段集成点

| SDD 阶段 | 知识库角色 | 触发方式 |
|----------|-----------|---------|
| brainstorming | 加载索引 Skill，了解现有架构，辅助需求分析和边界判断 | 自动（Skill 被发现加载） |
| writing-plans | 读取相关模块接口定义和依赖关系，产出精确实现计划 | Agent 主动查询 |
| subagent-driven-dev | 实现子 Agent 获取模块接口、依赖、注意事项 | 编排 Agent 在 Dispatch 时附带 |
| code-review | Reviewer 对照设计决策，验证代码架构合规 | Reviewer Agent 参考 |
| finishing | 自动执行增量更新，更新受影响条目并提交 | **自动触发** |

### 10.2 首次使用完整流程

```
/kb:init                    ← 初始化项目
/kb:generate                ← 全量生成知识库
    ↓
识别 Agent → 用户确认分层
    ↓
并行生成 → 索引汇总
    ↓
✅ 知识库就绪
    ↓
SDD 各阶段自动使用           ← 知识库作为共享上下文
    ↓
finishing 自动触发更新       ← 知识库持续保鲜
```

---

## 11. 实现范围

### 11.1 一期交付物

1. **`knowledge-base-generator` Skill** — 核心 Skill 文件（`SKILL.md`），包含完整流程定义
2. **注册子 Agent** — `agents/` 下 5 个 agent（recognition/system/subsystem/module/index），通过 `plugin.json` 的 `agents` 数组注册，由 Task 工具按需调用
3. **斜杠命令** — `.claude/commands/` 下 3 个命令（kb-init/kb-generate/kb-status）
4. **默认模板** — 系统/子系统/模块三级 Markdown 模板
5. **索引 Skill 生成器** — 自动生成 `skills/knowledge-base/SKILL.md`
6. **配置系统** — `.knowledge-base/config.yaml` 默认配置和 Schema
7. **Plugin 分发清单** — `.claude-plugin/plugin.json` + `marketplace.json`，支持通过 Claude Plugin Marketplace 安装

### 11.2 二期（后续迭代）

- Git Hook 自动化脚本
- 知识库可视化（依赖图、覆盖率仪表盘）
- 跨项目知识库聚合
- 知识库搜索 CLI 工具

### 11.3 不包含

- 独立的 CLI 工具（纯 Skill 实现）
- 知识库的 Web UI
- 第三方知识库格式导出

---

## 12. 质量评估

### 12.1 评估维度

**结构准确性**

- 分层是否合理：系统/子系统/模块边界是否与实际代码结构一致
- 遗漏检测：是否存在未被覆盖的重要模块
- 误分类：是否有模块被错误归入不相关的子系统

**内容完整性**

- 每个条目是否覆盖了模板要求的关键章节（接口、依赖、关键文件等）
- 设计文档中的关键决策是否被正确提取到知识库条目中
- 接口签名是否与实际代码一致

**可用性（Agent 视角）**

- 索引 Skill 是否能被后续 Agent 通过 SDO 机制正确发现
- 条目中的接口信息是否足够精确，使实现 Agent 无需回查源码即可正确调用
- 依赖关系是否完整，Agent 能否据此判断修改影响范围

**保鲜度**

- `/kb:status` 是否能准确检测过期条目
- 增量更新是否仅影响真正变更的条目（无漏更、无多余更新）

### 12.2 评估标准

| 维度 | 评估方式 | 通过标准 |
|------|---------|---------|
| 结构准确性 | 人工抽查 + 与目录结构交叉验证 | 覆盖率 ≥ 90%，误分类率 ≤ 5% |
| 内容完整性 | Agent 自检 + 模板章节必填校验 | 关键章节填充率 100% |
| 接口准确性 | 抽样对比源码中的函数/类签名 | 准确率 ≥ 95% |
| Agent 可用性 | 用知识库条目作为唯一上下文让 Agent 完成一个简单修改任务 | 任务成功完成，Agent 未报"缺少信息" |
| 保鲜度 | 修改代码后运行 `/kb:status`，对比预期 stale 条目与实际输出 | 变更条目被准确标记，无关条目不受影响 |

### 12.3 功能成功标准

1. `/kb:init` + `/kb:generate` 能在任意项目上完成全量生成
2. 生成的知识库通过索引 Skill 被后续 Agent 正确发现和加载
3. SDD finishing 阶段自动触发增量更新且仅更新受影响的条目
4. 项目自定义模板能正确覆盖默认模板
5. 非 Git 项目能降级到 Hash 模式正常工作
