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
- [子系统名](../docs/kb/subsystem/<子系统名>.md) — 一句话描述（属于 [父系统]）
```

模块列表格式：
```markdown
- [模块名](../docs/kb/module/<模块名>.md) — 一句话描述（属于 [父系统/父子系统]）
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
