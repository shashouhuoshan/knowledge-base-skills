检查项目知识库覆盖率和过期状态。

1. 读取 `docs/kb/.manifest.yaml`
   - 不存在 → 报告"知识库未生成，请运行 /kb:generate"

2. 检测变更：
   - Git 模式：`git diff --name-only <last_commit> HEAD`
   - Hash 模式（非 Git 项目）：对比源文件 hash

3. 输出状态报告，包含：
   - 生成时间和最后提交
   - 各层级（系统/子系统/模块）的 总计/最新/过期/缺失 统计
   - 过期条目列表及变更详情
   - 建议操作（如运行 `/kb:generate --update`）
