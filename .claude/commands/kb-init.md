初始化项目知识库生成器配置。

执行以下步骤：

1. 检查 `.knowledge-base/config.yaml` 是否存在
   - 已存在：询问用户是否覆盖（保留旧文件为 `.bak`）
   - 不存在：继续

2. 创建目录结构：
```bash
mkdir -p .knowledge-base/templates
mkdir -p docs/kb/{system,subsystem,module}
mkdir -p skills/knowledge-base
```

3. 写入默认配置文件 `.knowledge-base/config.yaml`，内容来自 `skills/knowledge-base-generator/config/default-config.yaml`

4. 复制默认模板到 `.knowledge-base/templates/`：
   - `skills/knowledge-base-generator/templates/system.md` → `.knowledge-base/templates/system.md`
   - `skills/knowledge-base-generator/templates/subsystem.md` → `.knowledge-base/templates/subsystem.md`
   - `skills/knowledge-base-generator/templates/module.md` → `.knowledge-base/templates/module.md`

5. 生成空的索引 Skill 骨架 `skills/knowledge-base/SKILL.md`

6. 输出初始化报告，提示用户下一步运行 `/kb:generate`
