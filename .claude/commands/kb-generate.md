生成或更新项目代码知识库（系统 → 子系统 → 模块三层结构）。

参数：`--update` 表示增量更新模式，无参数为全量生成。

## 全量生成流程

### 阶段 1：结构识别
调度识别 Agent（提示词：`skills/knowledge-base-generator/prompts/recognition-agent.md`）：
- 扫描项目目录树（排除 node_modules, .git, dist, build 等）
- 读取依赖文件（package.json, go.mod, Cargo.toml 等）
- 按三级策略发现设计文档
- 输出分层建议树 → **呈现给用户确认**

### 阶段 2：并行生成
用户确认后，并行调度生成 Agent：
- 系统级 Agent × 1（提示词：`skills/knowledge-base-generator/prompts/system-agent.md`）
- 子系统 Agent × N（提示词：`skills/knowledge-base-generator/prompts/subsystem-agent.md`）
- 模块 Agent × M（提示词：`skills/knowledge-base-generator/prompts/module-agent.md`）

模板优先级：`.knowledge-base/templates/<level>.md` > `skills/knowledge-base-generator/templates/<level>.md`

输出路径：`docs/kb/{system,subsystem,module}/<name>.md`

### 阶段 3：索引汇总
调度索引 Agent（提示词：`skills/knowledge-base-generator/prompts/index-agent.md`）：
- 生成 `docs/kb/.manifest.yaml`
- 更新 `skills/knowledge-base/SKILL.md`
- 输出生成报告

## 增量更新流程（`--update`）

1. Git 模式：`git diff --name-only <last_commit> HEAD` 检测变更
2. 映射变更文件到受影响的知识库条目
3. 仅重新生成受影响的条目
4. 更新 `.manifest.yaml` 和索引 Skill
