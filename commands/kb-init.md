初始化项目知识库生成器。执行以下步骤：

1. 检查 `.knowledge-base/config.yaml` 是否存在
   - 已存在：询问用户是否覆盖（保留旧文件为 `.bak`）
   - 不存在：继续

2. 创建目录结构：
```bash
mkdir -p .knowledge-base/templates
mkdir -p docs/kb/{system,subsystem,module}
mkdir -p skills/knowledge-base
```

3. 写入默认配置文件 `.knowledge-base/config.yaml`：

```yaml
strategy:
  mode: hybrid
scan:
  exclude:
    - node_modules
    - .git
    - dist
    - build
    - __pycache__
    - .next
    - target
design_docs:
  scan_paths:
    - docs/superpowers/specs/
    - docs/superpowers/plans/
    - docs/
    - README.md
update:
  trigger: finishing
  hook_type: post-merge
  auto_commit: true
templates:
  path: .knowledge-base/templates/
  use: default
output:
  kb_dir: docs/kb/
  skill_dir: skills/knowledge-base/
```

4. 复制默认模板到 `.knowledge-base/templates/`。创建以下三个模板文件：

**.knowledge-base/templates/system.md** — 系统级模板：
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
### 上游依赖
{{UPSTREAM_DEPENDENCIES}}
### 下游消费者
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

**.knowledge-base/templates/subsystem.md** — 子系统级模板：
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
### 上游依赖
{{UPSTREAM_DEPENDENCIES}}
### 下游消费者
{{DOWNSTREAM_CONSUMERS}}

## 接口
### 对外 API
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

**.knowledge-base/templates/module.md** — 模块级模板：
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
### 函数/方法签名
{{FUNCTION_SIGNATURES}}
### 数据结构
{{DATA_STRUCTURES}}

## 依赖关系
### 内部依赖
{{INTERNAL_DEPENDENCIES}}
### 外部依赖
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

5. 生成空的索引 Skill `skills/knowledge-base/SKILL.md`：

```markdown
---
name: knowledge-base
description: Use when needing to understand project architecture, module interfaces, system dependencies, or codebase structure before making changes
---

# 项目知识库

本知识库由 knowledge-base-generator 管理。运行 /kb-generate 生成完整内容。

## 系统
（待生成 — 运行 /kb-generate）

## 子系统
（待生成 — 运行 /kb-generate）

## 模块
（待生成 — 运行 /kb-generate）
```

6. 输出初始化报告，提示用户下一步运行 `/kb-generate`
