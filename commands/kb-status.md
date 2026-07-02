检查项目知识库覆盖率和过期状态。

1. 读取 `docs/kb/.manifest.yaml`，不存在则报告"未生成，请运行 /kb-generate"

2. 检测变更：
   - Git 项目：`git diff --name-only <manifest.last_commit> HEAD`
   - 非 Git 项目：对比源文件 hash

3. 输出状态报告：

```
知识库状态报告
生成时间：YYYY-MM-DD HH:MM
最后提交：<hash> (匹配/不匹配)

| 层级 | 总计 | 最新 | 过期 | 缺失 |
|------|------|------|------|------|
| 系统 | N | N | N | N |
| 子系统 | N | N | N | N |
| 模块 | N | N | N | N |

过期条目：
  - <条目名> — 变更详情

建议：运行 /kb-generate --update 更新过期条目
```
