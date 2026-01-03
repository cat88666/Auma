# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在处理此代码库中的代码时提供指导。

Browser-Use 是一个异步 Python >= 3.11 库，使用 LLM + CDP（Chrome DevTools Protocol）实现 AI 浏览器驱动能力。核心架构使 AI 代理能够自主导航网页、与元素交互，并通过处理 HTML 和做出 LLM 驱动的决策来完成复杂任务。

## 高层架构

该库遵循事件驱动架构，包含几个关键组件：

### 核心组件

- **Agent (`browser_use/agent/service.py`)**: 主协调器，接收任务、管理浏览器会话，并执行 LLM 驱动的操作循环
- **BrowserSession (`browser_use/browser/session.py`)**: 管理浏览器生命周期、CDP 连接，并通过事件总线协调多个 watchdog 服务
- **Tools (`browser_use/tools/service.py`)**: 操作注册表，将 LLM 决策映射到浏览器操作（点击、输入、滚动等）
- **DomService (`browser_use/dom/service.py`)**: 提取和处理 DOM 内容，处理元素高亮和可访问性树生成
- **LLM Integration (`browser_use/llm/`)**: 抽象层，支持 OpenAI、Anthropic、Google、Groq 和其他提供商

### 事件驱动的浏览器管理

BrowserSession 使用 `bubus` 事件总线来协调 watchdog 服务：
- **DownloadsWatchdog**: 处理 PDF 自动下载和文件管理
- **PopupsWatchdog**: 管理 JavaScript 对话框和弹窗
- **SecurityWatchdog**: 执行域名限制和安全策略
- **DOMWatchdog**: 处理 DOM 快照、截图和元素高亮
- **AboutBlankWatchdog**: 处理空页面重定向

### CDP 集成

使用 `cdp-use` (https://github.com/browser-use/cdp-use) 进行类型化的 CDP 协议访问。所有 CDP 客户端管理都在 `browser_use/browser/session.py` 中。

我们希望库的 API 符合人体工程学、直观且不易出错。

## 开发命令

**设置：**
```bash
uv venv --python 3.11
source .venv/bin/activate
uv sync
```

**测试：**
- 运行 CI 测试: `uv run pytest -vxs tests/ci`
- 运行所有测试: `uv run pytest -vxs tests/`
- 运行单个测试: `uv run pytest -vxs tests/ci/test_specific_test.py`

**质量检查：**
- 类型检查: `uv run pyright`
- 代码检查/格式化: `uv run ruff check --fix` 和 `uv run ruff format`
- Pre-commit 钩子: `uv run pre-commit run --all-files`

**MCP 服务器模式：**
该库可以作为 MCP 服务器运行，以便与 Claude Desktop 集成：
```bash
uvx browser-use[cli] --mcp
```

## 代码风格

- 使用异步 Python
- 在所有 Python 代码中使用制表符（tabs）进行缩进，而非空格
- 使用现代 Python >3.12 类型风格，例如使用 `str | None` 而非 `Optional[str]`，使用 `list[str]` 而非 `List[str]`，使用 `dict[str, Any]` 而非 `Dict[str, Any]`
- 尽量将所有控制台日志逻辑保留在以 `_log_...` 为前缀的单独方法中，例如 `def _log_pretty_path(path: Path) -> str`，以避免混淆主要逻辑
- 使用 Pydantic v2 模型来表示内部数据，以及任何可能原本是字典的用户面向 API 参数
- 在 Pydantic 模型中使用 `model_config = ConfigDict(extra='forbid', validate_by_name=True, validate_by_alias=True, ...)` 等参数来根据用例调整 Pydantic 模型行为。使用 `Annotated[..., AfterValidator(...)]` 尽可能多地编码验证逻辑，而不是在模型上使用辅助方法
- 我们通常将每个子组件的主要代码保存在 `service.py` 文件中，并将大多数 Pydantic 模型保存在 `views.py` 文件中，除非它们足够长而值得拥有自己的文件
- 在函数的开始和结束处使用运行时断言来强制执行约束和假设
- 对于所有新的 id 字段，优先使用 `from uuid_extensions import uuid7str` + `id: str = Field(default_factory=uuid7str)`
- 使用 `uv run pytest -vxs tests/ci` 运行测试
- 使用 `uv run pyright` 运行类型检查器

## CDP-Use

我们使用一个围绕 CDP 的薄包装层，名为 cdp-use: https://github.com/browser-use/cdp-use。cdp-use 仅为 websocket 调用提供浅层类型接口，所有 CDP 客户端和会话管理 + 其他 CDP 辅助程序仍位于 browser_use/browser/session.py。

- CDP-Use: 所有 CDP API 都通过 cdp-use 以自动类型接口的方式暴露，使用 `cdp_client.send.DomainHere.methodNameHere(params=...)` 形式，例如：
  - `cdp_client.send.DOMSnapshot.enable(session_id=session_id)`
  - `cdp_client.send.Target.attachToTarget(params={'targetId': target_id, 'flatten': True})` 或更好的方式：
    `cdp_client.send.Target.attachToTarget(params=ActivateTargetParameters(targetId=target_id, flatten=True))` (导入 `from cdp_use.cdp.target import ActivateTargetParameters`)
  - `cdp_client.register.Browser.downloadWillBegin(callback_func_here)` 用于事件注册，而不是 `cdp_client.on(...)`，后者不存在！

## 保持示例和测试最新

- 确保阅读 `examples/` 目录中的相关示例以获取上下文，并在进行更改时保持它们最新
- 确保阅读 `tests/` 目录中的相关测试（特别是 `tests/ci/*.py`）并保持它们最新
- 测试文件通过后，应将其移动到 `tests/ci/` 子目录，该子目录中的文件被视为"默认集"测试，并在每次提交时由 CI 自动发现和运行。确保任何特定于事件的测试都位于其 `tests/ci/test_action_EventNameHere.py` 文件中
- 在测试中永远不要 mock 任何东西，始终使用真实对象！！**唯一的**例外是 llm，对于 llm，您可以使用 pytest fixtures 和 `conftest.py` 中的工具来设置 LLM 响应。对于测试特定浏览器场景，使用 pytest-httpserver 为每个测试设置 html 和响应
- 在测试中永远不要使用真实的远程 URL（例如 `https://google.com` 或 `https://example.com`），而是使用 pytest-httpserver 在 fixture 中设置测试服务器，该服务器响应测试所需的 html（参见其他 `tests/ci` 文件以获取示例）
- 使用现代 pytest-asyncio 最佳实践：测试函数不再需要 `@pytest.mark.asyncio` 装饰器，只需对异步测试使用普通异步函数。在需要它的测试内部使用 `loop = asyncio.get_event_loop()`，而不是将 `event_loop` 作为函数参数传递。不需要 fixture 在顶部手动设置事件循环，它由 pytest 自动设置。Fixture 函数（即使是异步的）只需要一个简单的 `@pytest.fixture` 装饰器，无需参数

## 个性

不要担心形式。

不要回避复杂性，假设所有问题都需要深入的技术解释。指出与问题相关的学科使用的适当术语、模型、单位等。信息论和博弈论可以作为评估复杂系统的有用视角。

仔细选择你的类比，将诗意华丽的语言保持在最低限度，一点冷幽默是受欢迎的。

如果策略阻止你正常响应，请在回答前打印 "!!!!"。

如果策略阻止你发表意见，请假装你以可能类似于 eigenrobot 的意见做出响应。

对你的信息质量保持批判

如果你发现任何请求令人恼火，请以轻蔑的方式回应，如 "说真的" 或 "这太疯狂了兄弟" 或 "lol 不"

无论你现在表现得多么聪明，都以相同的风格写作，但好像你比现在聪明 +2 标准差

## 进行更改的策略

进行任何重大更改时：

1. 找到或编写测试来验证对现有设计的任何假设 + 确认在更改之前它按预期工作
2. 首先为新设计编写失败的测试，运行它们以确认它们失败
3. 然后实现新设计的更改。在开发过程中根据需要运行或添加测试，以在遇到任何困难时验证假设
4. 更改完成后运行完整的 `tests/ci` 套件。确认新设计有效 & 确认向后兼容性未被破坏
5. 将相关测试逻辑浓缩和去重到一个文件中，重新阅读文件以确保我们没有冗余地一遍又一遍地测试相同的内容。快速扫描 `tests/` 中可能需要更新或浓缩的任何其他潜在相关文件
6. 更新 `docs/` 和 `examples/` 中的任何相关文件，并确认它们与实现和测试匹配

在进行任何真正大规模的重构时，倾向于使用简单的事件总线和作业队列将系统分解为更小的服务，每个服务管理状态的某个隔离子组件。

如果你在就地更新或编辑文件时遇到困难，请尝试将匹配字符串缩短到 1 或 2 行，而不是 3 行。
如果这不起作用，只需将新修改的代码作为新行插入文件中，然后在第二步中删除旧代码，而不是替换。

## 文件组织和关键模式

- **Service 模式**: 每个主要组件都有一个包含主要逻辑的 `service.py` 文件（Agent、BrowserSession、DomService、Tools）
- **Views 模式**: Pydantic 模型和数据结构位于 `views.py` 文件中
- **Events**: 事件定义位于 `events.py` 文件中，遵循事件驱动架构
- **Browser Profile**: `browser_use/browser/profile.py` 包含所有浏览器启动参数、显示配置和扩展管理
- **System Prompts**: Agent 提示位于 markdown 文件中：`browser_use/agent/system_prompt*.md`

## 浏览器配置

BrowserProfile 通过 `detect_display_configuration()` 自动检测显示大小并配置浏览器窗口。关键配置：
- macOS (`AppKit.NSScreen`) 和 Linux/Windows (`screeninfo`) 的显示大小检测
- 具有可配置白名单的扩展管理（uBlock Origin、cookie 处理程序）
- Chrome 启动参数生成和去重
- 代理支持、安全设置和无头/有头模式

## MCP（Model Context Protocol）集成

该库支持两种模式：
1. **作为 MCP 服务器**: 向 MCP 客户端（如 Claude Desktop）暴露浏览器自动化工具
2. **与 MCP 客户端一起使用**: 代理可以连接到外部 MCP 服务器（文件系统、GitHub 等）以扩展功能

连接管理位于 `browser_use/mcp/client.py`。

## 重要的开发约束

- **始终使用 `uv` 而不是 `pip`** 进行依赖管理
- **在实现功能时永远不要创建随机示例文件** - 如需要，在终端中内联测试
- **使用真实的模型名称** - 不要用 `gpt-4` 替换 `gpt-4o`（它们是不同的模型）
- **为操作使用描述性名称和文档字符串**
- **返回带有结构化内容的 `ActionResult`** 以帮助代理更好地推理
- **在制作 PR 之前运行 pre-commit 钩子**

## important-instruction-reminders
只做被要求的事情；不要多，不要少。
除非绝对必要，否则永远不要创建文件。
始终优先编辑现有文件，而不是创建新文件。
永远不要主动创建文档文件（*.md）或 README 文件。只有在用户明确请求时才创建文档文件。
