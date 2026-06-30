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
