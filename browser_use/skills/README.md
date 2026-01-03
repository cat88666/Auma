# Skills 模块

Skills 模块提供与 Browser Use API 的集成，用于获取和执行技能。

## 基本用法

```python
import asyncio
from browser_use.skills import SkillService

async def main():
    skill_ids = ['skill-id-1', 'skill-id-2']

    # 初始化服务
    service = SkillService(skill_ids=skill_ids, api_key='your-api-key')

    # 获取所有已加载的技能（首次调用时自动初始化）
    skills = await service.get_all_skills()

    # 执行技能（如需要会自动初始化）
    result = await service.execute_skill(
        skill_id='skill-id-1',
        parameters={'param1': 'value1'}
    )

    if result.success:
        print(f'Success! Result: {result.result}')

    # 清理
    await service.close()

asyncio.run(main())
```
