# Knowledge Base Generator

为 Coding Agent 设计的项目代码知识库自动生成插件。通过三层分层结构（系统 → 子系统 → 模块），结合 Superpowers SDD 模式，在开发流程中自动维护结构化的项目知识库。

## 特性

- **三层分层**：系统级、子系统级、模块级，逐层细化
- **智能识别**：Agent 自动扫描项目结构，建议分层边界，用户审核确认
- **并行生成**：Subagent 并行分析各层级代码，高效产出
- **增量更新**：基于 Git Diff 精确检测变更，仅更新受影响的条目
- **可配置模板**：插件提供默认模板，项目可自定义覆盖
- **SDD 深度集成**：知识库作为 brainstorming → writing-plans → subagent-driven-dev → finishing 全流程的共享上下文
- **零依赖**：纯 Markdown + YAML，无需编程语言运行时

## 安装

### 通过 Plugin Marketplace 安装（推荐）

```bash
# 1. 添加插件市场
/plugin marketplace add shashouhuoshan/knowledge-base-skills

# 2. 安装插件
/plugin install knowledge-base-generator@knowledge-base-skills
```

安装后运行 `/reload-plugins`，输入 `/kb-` 即可看到三个命令。

### 通过 settings.json 安装

在 `~/.claude/settings.json` 中配置：

```json
{
  "extraKnownMarketplaces": {
    "knowledge-base-skills": {
      "source": {
        "source": "github",
        "repo": "shashouhuoshan/knowledge-base-skills"
      }
    }
  },
  "enabledPlugins": {
    "knowledge-base-generator@knowledge-base-skills": true
  }
}
```

配置后运行 `/reload-plugins` 即可。

### 离线安装

将仓库的 `.claude/commands/` 和 `skills/` 复制到 `~/.claude/` 下：

```bash
cp -r <repo>/.claude/commands/kb-*.md ~/.claude/commands/
cp -r <repo>/skills/knowledge-base-generator ~/.claude/skills/
```

运行 `/reload-plugins` 即可。

### 从源码安装（开发模式）

```bash
git clone https://github.com/shashouhuoshan/knowledge-base-skills.git
cd knowledge-base-skills
# 项目本身即插件，在 Claude Code 中打开即可使用
```

## 命令

| 命令 | 用途 |
|------|------|
| `/kb-init` | 初始化项目配置、目录结构和默认模板 |
| `/kb-generate [--update]` | 全量生成知识库 / 增量更新 |
| `/kb-status` | 查看知识库覆盖率和过期状态 |

### 典型工作流

```bash
/kb-init                    # 1. 初始化项目
/kb-generate                # 2. 全量生成知识库
# ... 正常开发 ...
/kb-status                  # 3. 查看哪些条目过期
/kb-generate --update       # 4. 增量更新过期条目
```

## 架构

```
/kb-generate 命令
      │
      ▼
  编排 Agent（主控）
  读取配置 & 调度子任务
      │
 ┌────┼────┐
 ▼    ▼    ▼
识别  系统级 子系统级  模块级   索引
Agent Agent Agent×N  Agent×M  Agent
```

### Agent 职责

| Agent | 数量 | 职责 |
|-------|------|------|
| 识别 Agent | 1 | 扫描项目结构，输出分层建议树，等待用户确认 |
| 系统级 Agent | 1 | 分析系统边界、技术栈、外部依赖，生成系统级条目 |
| 子系统 Agent | N（并行） | 分析子系统内部结构、接口、依赖关系 |
| 模块 Agent | M（并行） | 提取模块 API 签名、数据结构、关键文件 |
| 索引 Agent | 1 | 汇总所有条目，生成 `.manifest.yaml` 和索引 Skill |

## 生成产物

```
docs/kb/
├── system/<系统名>.md          # 系统级：技术栈、子系统列表、设计决策
├── subsystem/<子系统名>.md     # 子系统级：接口、依赖、模块列表
├── module/<模块名>.md          # 模块级：函数签名、数据结构、注意事项
└── .manifest.yaml              # 元数据：来源追踪、hash、更新时间

skills/knowledge-base/
└── SKILL.md                    # 索引：Agent 发现入口
```

## 配置

项目初始化后在 `.knowledge-base/config.yaml` 中可配置：

```yaml
strategy:
  mode: hybrid          # auto | manual | hybrid

scan:
  exclude:              # 排除目录
    - node_modules
    - .git
    - dist

design_docs:
  scan_paths:           # 设计文档搜索路径
    - docs/superpowers/specs/
    - docs/

update:
  trigger: finishing    # finishing | hook | manual
  auto_commit: true

output:
  kb_dir: docs/kb/
  skill_dir: skills/knowledge-base/
```

## 自定义模板

项目可在 `.knowledge-base/templates/` 下放置自定义模板，覆盖插件默认：

```
.knowledge-base/templates/
├── system.md      # 自定义系统级模板
├── subsystem.md   # 自定义子系统级模板
└── module.md      # 自定义模块级模板
```

## SDD 集成

| SDD 阶段 | 知识库角色 |
|----------|-----------|
| brainstorming | 加载索引 Skill，了解现有架构 |
| writing-plans | 参考接口定义和依赖关系，产出精确计划 |
| subagent-driven-dev | 实现 Agent 获取模块接口和注意事项 |
| code-review | 对照设计决策验证架构合规 |
| finishing | **自动触发增量更新**，知识库随代码持续保鲜 |

## 文件结构

```
.claude-plugin/
├── plugin.json                     # 插件清单（注册 commands/skills/agents）
└── marketplace.json               # Marketplace 清单（source: "./"）

commands/                          # 斜杠命令（根目录，符合插件规范）
├── kb-init.md                     # /kb-init 命令
├── kb-generate.md                 # /kb-generate 命令
└── kb-status.md                   # /kb-status 命令

agents/                            # 注册的子 agent（按需调用）
├── recognition-agent.md           # 结构识别 → 分层建议树
├── system-agent.md                # 系统级条目生成
├── subsystem-agent.md             # 子系统级条目生成
├── module-agent.md                # 模块级条目生成
└── index-agent.md                 # 索引与清单汇总

skills/knowledge-base-generator/
├── SKILL.md                       # 主编排 Skill（流程定义）
├── config/
│   └── default-config.yaml        # 默认配置模板
└── templates/                     # 默认知识库条目模板
    ├── system.md
    ├── subsystem.md
    └── module.md
```

### Subagent 架构

`/kb-generate` 通过 Task 工具按需调用注册的 5 个子 agent：

| Agent | 触发时机 | 工具权限 |
|-------|---------|---------|
| `kb-recognition-agent` | 阶段 1：扫描项目结构 | Read, Grep, Glob, Bash |
| `kb-system-agent` | 阶段 2：生成系统级条目（×1） | Read, Grep, Glob, Write |
| `kb-subsystem-agent` | 阶段 2：生成子系统条目（并行 ×N） | Read, Grep, Glob, Write |
| `kb-module-agent` | 阶段 2：生成模块条目（并行 ×M） | Read, Grep, Glob, Write |
| `kb-index-agent` | 阶段 3：汇总索引与清单 | Read, Grep, Glob, Write, Bash |

## 质量评估

生成的知识库通过以下维度评估：

| 维度 | 标准 |
|------|------|
| 结构准确性 | 覆盖率 ≥ 90%，误分类率 ≤ 5% |
| 内容完整性 | 关键章节填充率 100% |
| 接口准确性 | 函数签名与实际代码一致率 ≥ 95% |
| Agent 可用性 | 以知识库为唯一上下文完成修改任务 |
| 保鲜度 | 变更条目被准确标记，无关条目不受影响 |

## 许可证

MIT
