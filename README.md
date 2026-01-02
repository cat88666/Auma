
# 🤖 LLM 快速开始

1. 将您喜欢的编程代理（Cursor、Claude Code 等）指向 [Agents.md](https://docs.browser-use.com/llms-full.txt)
2. 开始提示吧！

<br/>

# 👋 用户快速开始

**1. 使用 [uv](https://docs.astral.sh/uv/) 创建环境（Python>=3.11）：**
```bash
uv init
```

**2. 安装 Browser-Use 包：**
```bash
#  我们每天发布 - 使用最新版本！
uv add browser-use
uv sync
```

**3. 从 [Browser Use Cloud](https://cloud.browser-use.com/new-api-key) 获取您的 API 密钥，并将其添加到您的 `.env` 文件中（新注册用户可获得 $10 免费积分）：**
```
# .env
BROWSER_USE_API_KEY=your-key
```

**4. 安装 Chromium 浏览器：**
```bash
uvx browser-use install
```

**5. 运行您的第一个代理：**
```python
from browser_use import Agent, Browser, ChatBrowserUse
import asyncio

async def example():
    browser = Browser(
        # use_cloud=True,  # 取消注释以在 Browser Use Cloud 上使用隐身浏览器
    )

    llm = ChatBrowserUse()

    agent = Agent(
        task="查找 browser-use 仓库的 star 数量",
        llm=llm,
        browser=browser,
    )

    history = await agent.run()
    return history

if __name__ == "__main__":
    history = asyncio.run(example())
```

查看[库文档](https://docs.browser-use.com)和[云文档](https://docs.cloud.browser-use.com)了解更多！

<br/>

# 🔥 在沙箱上部署

我们处理代理、浏览器、持久化、身份验证、Cookie 和 LLM。代理直接在浏览器旁边运行，实现最低延迟。

```python
from browser_use import Browser, sandbox, ChatBrowserUse
from browser_use.agent.service import Agent
import asyncio

@sandbox()
async def my_task(browser: Browser):
    agent = Agent(task="查找 HN 置顶帖子", browser=browser, llm=ChatBrowserUse())
    await agent.run()

# 就像调用任何异步函数一样调用它
asyncio.run(my_task())
```

查看[进入生产环境](https://docs.browser-use.com/production)了解更多详情。

<br/>

# 🚀 模板快速开始

**想要更快地开始？** 生成一个可直接运行的模板：

```bash
uvx browser-use init --template default
```

这将创建一个带有工作示例的 `browser_use_default.py` 文件。可用模板：
- `default` - 最简设置，快速开始
- `advanced` - 所有配置选项，附详细注释
- `tools` - 自定义工具和扩展代理的示例

您也可以指定自定义输出路径：
```bash
uvx browser-use init --template default --output my_agent.py
```

<br/>

# 演示


### 📋 表单填写
#### 任务 = "用我的简历和信息填写这份工作申请。"

![工作申请演示](https://github.com/user-attachments/assets/57865ee6-6004-49d5-b2c2-6dff39ec2ba9)
[示例代码 ↗](https://github.com/browser-use/browser-use/blob/main/examples/use-cases/apply_to_job.py)


### 🍎 购物
#### 任务 = "将这个商品列表添加到我的 instacart 中。"

https://github.com/user-attachments/assets/a6813fa7-4a7c-40a6-b4aa-382bf88b1850

[示例代码 ↗](https://github.com/browser-use/browser-use/blob/main/examples/use-cases/buy_groceries.py)


### 💻 个人助理
#### 任务 = "帮我找到定制 PC 的零件。"

https://github.com/user-attachments/assets/ac34f75c-057a-43ef-ad06-5b2c9d42bf06

[示例代码 ↗](https://github.com/browser-use/browser-use/blob/main/examples/use-cases/pcpartpicker.py)


### 💡查看[更多示例 ↗](https://docs.browser-use.com/examples)并给我们一个 star！

<br/>

## 集成、托管、自定义工具、MCP 等更多内容请查看我们的[文档 ↗](https://docs.browser-use.com)

<br/>

# 常见问题

<details>
<summary><b>最好的模型是什么？</b></summary>

我们专门针对浏览器自动化任务优化了 **ChatBrowserUse()**。平均而言，它以 SOTA 准确度完成任务的速度比其他模型快 3-5 倍。

**定价（每 100 万 tokens）：**
- 输入 tokens：$0.20
- 缓存的输入 tokens：$0.02
- 输出 tokens：$2.00

有关其他 LLM 提供商，请参阅我们的[支持的模型文档](https://docs.browser-use.com/supported-models)。
</details>


<details>
<summary><b>我可以在代理中使用自定义工具吗？</b></summary>

可以！您可以添加自定义工具来扩展代理的功能：

```python
from browser_use import Tools

tools = Tools()

@tools.action(description='描述此工具的功能。')
def custom_tool(param: str) -> str:
    return f"结果: {param}"

agent = Agent(
    task="您的任务",
    llm=llm,
    browser=browser,
    tools=tools,
)
```

</details>

<details>
<summary><b>我可以免费使用吗？</b></summary>

可以！Browser-Use 是开源且免费使用的。您只需要选择一个 LLM 提供商（如 OpenAI、Google、ChatBrowserUse，或使用 Ollama 运行本地模型）。
</details>

<details>
<summary><b>如何处理身份验证？</b></summary>

查看我们的身份验证示例：
- [使用真实浏览器配置文件](https://github.com/browser-use/browser-use/blob/main/examples/browser/real_browser.py) - 重用您现有的 Chrome 配置文件（包含保存的登录信息）
- 如果您想使用带有收件箱的临时账户，请选择 AgentMail
- 要将您的身份验证配置文件同步到远程浏览器，请运行 `curl -fsSL https://browser-use.com/profile.sh | BROWSER_USE_API_KEY=XXXX sh`（将 XXXX 替换为您的 API 密钥）

这些示例展示了如何无缝维护会话和处理身份验证。
</details>

<details>
<summary><b>如何解决验证码（CAPTCHA）？</b></summary>

对于验证码处理，您需要更好的浏览器指纹识别和代理。使用[Browser Use Cloud](https://cloud.browser-use.com)，它提供旨在避免检测和验证码挑战的隐身浏览器。
</details>

<details>
<summary><b>如何进入生产环境？</b></summary>

Chrome 可能消耗大量内存，并行运行多个代理可能难以管理。

对于生产用例，请使用我们的[Browser Use Cloud API](https://cloud.browser-use.com)，它处理：
- 可扩展的浏览器基础设施
- 内存管理
- 代理轮换
- 隐身浏览器指纹识别
- 高性能并行执行
</details>

<br/>

<div align="center">

