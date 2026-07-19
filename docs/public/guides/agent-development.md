# 代理开发指南

构建能够生成 A2UI 界面的 AI 代理。本指南涵盖如何从 LLM 生成并流式传输 UI 消息。

## 快速概览

构建一个 A2UI 代理的步骤：

1. **理解用户意图** → 决定展示什么 UI
2. **生成 A2UI JSON** → 使用 LLM 的结构化输出或提示词
3. **校验并流式输出** → 检查 schema，发送给客户端
4. **处理动作** → 响应用户交互

## 从一个简单代理开始

我们将使用 ADK 来构建一个简单代理。先从文本开始，最终再升级到 A2UI。

逐步说明请参见 [ADK quickstart](https://google.github.io/adk-docs/get-started/python/)。

```bash
pip install google-adk
adk create my_agent
```

然后编辑 `my_agent/agent.py`，写一个用于餐厅推荐的非常简单的代理。

```python
import json
from google.adk.agents.llm_agent import Agent
from google.adk.tools.tool_context import ToolContext

def get_restaurants(tool_context: ToolContext) -> str:
    """调用这个 tool 获取餐厅列表。"""
    return json.dumps([
        {
            "name": "Xi'an Famous Foods",
            "detail": "Spicy and savory hand-pulled noodles.",
            "imageUrl": "http://localhost:10002/static/shrimpchowmein.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.xianfoods.com/)",
            "address": "81 St Marks Pl, New York, NY 10003"
        },
        {
            "name": "Han Dynasty",
            "detail": "Authentic Szechuan cuisine.",
            "imageUrl": "http://localhost:10002/static/mapotofu.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.handynasty.net/)",
            "address": "90 3rd Ave, New York, NY 10003"
        },
        {
            "name": "RedFarm",
            "detail": "Modern Chinese with a farm-to-table approach.",
            "imageUrl": "http://localhost:10002/static/beefbroccoli.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.redfarmnyc.com/)",
            "address": "529 Hudson St, New York, NY 10014"
        },
    ])

AGENT_INSTRUCTION="""
You are a helpful restaurant finding assistant. Your goal is to help users find and book restaurants using a rich UI.

To achieve this, you MUST follow this logic:

1.  **For finding restaurants:**
    a. You MUST call the `get_restaurants` tool. Extract the cuisine, location, and a specific number (`count`) of restaurants from the user's query (e.g., for "top 5 chinese places", count is 5).
    b. After receiving the data, you MUST follow the instructions precisely to generate the final a2ui UI JSON, using the appropriate UI example from the `prompt_builder.py` based on the number of restaurants."""

root_agent = Agent(
    model='gemini-2.5-flash',
    name="restaurant_agent",
    description="An agent that finds restaurants and helps book tables.",
    instruction=AGENT_INSTRUCTION,
    tools=[get_restaurants],
)
```

别忘了设置 `GOOGLE_API_KEY` 环境变量来运行这个示例。

```bash
echo 'GOOGLE_API_KEY="YOUR_API_KEY"' > .env
```

你可以通过 ADK web 界面测试这个代理：

```bash
adk web
```

从列表中选择 `my_agent`，然后询问纽约的餐厅。你应该会在 UI 中看到一个以纯文本形式展示的餐厅列表。

## 生成 A2UI 消息

让 LLM 生成 A2UI 消息需要一些提示工程。SDK 提供了 `A2uiSchemaManager`，帮助你生成一段包含 A2UI schema 以及组件 catalog 示例的系统提示词。

首先，确保你已经安装了 `a2ui-agent-sdk`（示例中已经包含它）。

在你的代理文件（例如 `agent.py`）中，导入所需的类：

```python
from a2ui.schema.constants import VERSION_0_8, VERSION_0_9
from a2ui.strategies.schema import A2uiSchemaManager
from a2ui.basic_catalog.provider import BasicCatalog
```

接下来，你可以使用 `A2uiSchemaManager` 来生成系统提示词。这能确保 schema 和示例始终保持格式正确、内容最新。

```python
# 定义代理的角色
ROLE_DESCRIPTION = (
    "You are a helpful restaurant finding assistant. Your final output MUST be a a2ui"
    " UI JSON response."
)

# 定义在什么情况下使用哪个 UI 模板的规则
UI_DESCRIPTION = """
-   If the query is for a list of restaurants, use the restaurant data you have already received from the `get_restaurants` tool to populate the `dataModelUpdate.contents` (v0.8) or `updateDataModel.value` (v0.9+) object (e.g., for the "items" key).
-   If the number of restaurants is 5 or fewer, you MUST use the `SINGLE_COLUMN_LIST_EXAMPLE` template.
-   If the number of restaurants is more than 5, you MUST use the `TWO_COLUMN_LIST_EXAMPLE` template.
-   If the query is to book a restaurant (e.g., "USER_WANTS_TO_BOOK..."), you MUST use the `BOOKING_FORM_EXAMPLE` template.
-   If the query is a booking submission (e.g., "User submitted a booking..."), you MUST use the `CONFIRMATION_EXAMPLE` template.
"""

# 用 Basic Catalog 初始化 schema manager
schema_manager = A2uiSchemaManager(
    version=VERSION_0_8, # 更新的协议请使用 VERSION_0_9
    catalogs=[
        BasicCatalog.get_config(
            version=VERSION_0_8, examples_path="examples/0.8"
        )
    ],
)

# 生成完整的系统提示词
A2UI_AND_AGENT_INSTRUCTION = schema_manager.generate_system_prompt(
    role_description=ROLE_DESCRIPTION,
    ui_description=UI_DESCRIPTION,
    include_schema=True,
    include_examples=True,
    validate_examples=True,
)

root_agent = Agent(
    model='gemini-2.5-flash',
    name="restaurant_agent",
    description="An agent that finds restaurants and helps book tables.",
    instruction=A2UI_AND_AGENT_INSTRUCTION,
    tools=[get_restaurants],
)
```

## 理解输出

你的代理不再只输出文本，而是会输出文本和一份 **A2UI 消息的 JSON 列表**。

我们导入的 `A2UI_SCHEMA` 是一个标准 JSON schema，定义了如下有效操作：

* `render`（展示 UI）
* `update`（修改已有 UI 中的数据）

由于输出是结构化 JSON，你可以在发送给客户端之前先解析并校验它。

```python
# 1. 解析 JSON
# 警告：把输出直接按 JSON 解析只是文档中的示意实现，比较脆弱。
# LLM 经常会给 JSON 输出包上 Markdown fence，也可能犯其他错误。
# 生产环境里应尽量依赖框架帮你完成 JSON 解析。
parsed_json_data = json.loads(json_string_cleaned)

# 2. 使用 A2UI_SCHEMA 进行校验
# 这样可以确保 LLM 生成的是合法的 A2UI 命令
jsonschema.validate(
    instance=parsed_json_data, schema=self.a2ui_schema_object
)
```

通过对输出进行 `A2UI_SCHEMA` 校验，你可以确保客户端永远不会收到格式错误的 UI 指令。

TODO：继续补充如何在不依赖 A2A extension 的情况下，解析、校验并把输出发送到客户端渲染器的示例。
