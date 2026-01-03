# 贡献代理任务
贡献您自己的代理任务，我们会测试代理是否能解决它们，用于 CI 测试！

## 如何添加任务
1. 在此目录（`tests/agent_tasks/`）中创建一个新的 `.yaml` 文件。
2. 使用以下格式：

```yaml
name: 我的任务名称
task: 描述代理要执行的任务
judge_context:
  - 列出成功标准，每行一个
max_steps: 10
```

## 指南
- 在任务和标准中要具体明确。
- `judge_context` 应该列出什么算作成功的结果。
- 代理的输出将由 LLM 使用这些标准进行评判。

## 运行测试
要运行所有代理任务：
```bash
pytest tests/ci/test_agent_real_tasks.py
```
---
