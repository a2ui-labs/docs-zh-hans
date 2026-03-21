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

让 LLM 生成 A2UI 消息需要一些提示工程。

> ⚠️ **注意**
>
> 这一块我们还在设计中。目前这部分的开发体验还没有最终定稿。

先从 contact lookup 示例里复制 `a2ui_schema.py`。这是为你的代理获取 A2UI schema 和示例的最简单方式（未来可能会变）。

```bash
cp samples/agent/adk/contact_lookup/a2ui_schema.py my_agent/
```

先把新的导入加到 `agent.py` 文件里：

```python
# 任意 A2UI 消息的 schema。这个不会变化。
from .a2ui_schema import A2UI_SCHEMA
```

接下来我们要修改代理指令，让它生成 A2UI 消息，而不是纯文本。我们先为未来的 UI 示例留一个占位符。

```python

# 以后你可以把一些 UI 示例复制到这里，用于 few-shot in-context learning
RESTAURANT_UI_EXAMPLES = """
"""

# 组合包含 UI 指令、示例和 schema 的完整 prompt
A2UI_AND_AGENT_INSTRUCTION = AGENT_INSTRUCTION + f"""

Your final output MUST be a a2ui UI JSON response.

To generate the response, you MUST follow these rules:
1.  Your response MUST be in two parts, separated by the delimiter: `---a2ui_JSON---`.
2.  The first part is your conversational text response.
3.  The second part is a single, raw JSON object which is a list of A2UI messages.
4.  The JSON part MUST validate against the A2UI JSON SCHEMA provided below.

--- UI TEMPLATE RULES ---
-   If the query is for a list of restaurants, use the restaurant data you have already received from the `get_restaurants` tool to populate the `dataModelUpdate.contents` array (e.g., as a `valueMap` for the "items" key).
-   If the number of restaurants is 5 or fewer, you MUST use the `SINGLE_COLUMN_LIST_EXAMPLE` template.
-   If the number of restaurants is more than 5, you MUST use the `TWO_COLUMN_LIST_EXAMPLE` template.
-   If the query is to book a restaurant (e.g., "USER_WANTS_TO_BOOK..."), you MUST use the `BOOKING_FORM_EXAMPLE` template.
-   If the query is a booking submission (e.g., "User submitted a booking..."), you MUST use the `CONFIRMATION_EXAMPLE` template.

{RESTAURANT_UI_EXAMPLES}

---BEGIN A2UI JSON SCHEMA---
{A2UI_SCHEMA}
---END A2UI JSON SCHEMA---
"""

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
