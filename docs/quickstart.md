# 快速开始：5 分钟跑通 A2UI

通过运行餐厅查找示例，亲手体验 A2UI。本指南会让你在 5 分钟内看到由智能体生成的 UI。

## 你将构建什么

完成本快速开始后，你将获得：

- ✅ 一个正在运行、使用 A2UI Lit renderer 的 Web 应用
- ✅ 一个由 Gemini 驱动、可以生成动态 UI 的智能体
- ✅ 一个支持表单生成、时间选择和确认流程的交互式餐厅查找示例
- ✅ 对 A2UI 消息如何从智能体流向 UI 的基本理解

## 前置条件

开始之前，请确认你已经具备：

- **Node.js**（v18 或更高版本）— [下载地址](https://nodejs.org/)
- **uv**（Python 包管理器）— [安装指南](https://docs.astral.sh/uv/getting-started/installation/)（用于运行 Python 智能体后端）
- **Gemini API Key** — [可在 Google AI Studio 免费获取](https://aistudio.google.com/apikey)

> ⚠️ **安全提醒**
>
>
> 这个示例会运行一个使用 Gemini 生成 A2UI 响应的 A2A 智能体。该智能体能够访问你的 API Key，并向 Google 的 Gemini API 发起请求。在生产环境中运行前，请务必先审查智能体代码。

## 第 1 步：克隆仓库

```bash
git clone https://github.com/google/a2ui.git
cd a2ui
```

## 第 2 步：设置 API Key

将你的 Gemini API Key 导出为环境变量：

```bash
export GEMINI_API_KEY="your_gemini_api_key_here"
```

## 第 3 步：进入 Lit 客户端目录

```bash
cd samples/client/lit
```

## 第 4 步：安装并运行

运行一键启动示例的命令：

```bash
npm install
npm run demo:all
```

该命令会：

1. 安装全部依赖
2. 构建 A2UI renderer
3. 启动 A2A 餐厅查找智能体（Python 后端）
4. 启动开发服务器
5. 在浏览器中打开 `http://localhost:5173`

> ✅ **示例已运行**
>
> 如果一切正常，你会在浏览器中看到 Web 应用，智能体现在已经可以开始生成 UI。

## 第 5 步：开始体验

在 Web 应用中，你可以尝试这些提示词：

1. **"Book a table for 2"** - 观察智能体生成预约表单
2. **"Find Italian restaurants near me"** - 查看动态搜索结果
3. **"What are your hours?"** - 体验针对不同意图生成的不同 UI 布局

### 背后发生了什么

```
┌─────────────┐         ┌──────────────┐         ┌────────────────┐
│   You Type  │────────>│ A2A Agent    │────────>│  Gemini API    │
│  a Message  │         │  (Python)    │         │  (LLM)         │
└─────────────┘         └──────────────┘         └────────────────┘
                               │                         │
                               │ Generates A2UI JSON     │
                               │<────────────────────────┘
                               │
                               │ Streams JSONL messages
                               v
                        ┌──────────────┐
                        │   Web App    │
                        │ (A2UI Lit    │
                        │  Renderer)   │
                        └──────────────┘
                               │
                               │ Renders native components
                               v
                        ┌──────────────┐
                        │   Your UI    │
                        └──────────────┘
```

1. **你通过 Web UI 发送消息**
2. **A2A 智能体** 接收消息并把对话发送给 Gemini
3. **Gemini 生成** 描述 UI 的 A2UI JSON 消息
4. **A2A 智能体将这些消息流式返回** 给 Web 应用
5. **A2UI renderer** 将它们转换为原生 Web 组件
6. **你在浏览器中看到 UI** 被渲染出来

## A2UI 消息的结构

下面来看一下智能体发送的内容。这里给出一个简化版 JSON 消息示例：

=== "v0.8 (Stable)"

    **定义 UI：**

    ```json
    {"surfaceUpdate": {"surfaceId": "main", "components": [
      {"id": "header", "component": {"Text": {"text": {"literalString": "Book Your Table"}, "usageHint": "h1"}}},
      {"id": "date-picker", "component": {"DateTimeInput": {"label": {"literalString": "Select Date"}, "value": {"path": "/reservation/date"}, "enableDate": true}}},
      {"id": "submit-text", "component": {"Text": {"text": {"literalString": "Confirm Reservation"}}}},
      {"id": "submit-btn", "component": {"Button": {"child": "submit-text", "action": {"name": "confirm_booking"}}}}
    ]}}
    ```

    **填充数据：**

    ```json
    {"dataModelUpdate": {"surfaceId": "main", "contents": [
      {"key": "reservation", "valueMap": [
        {"key": "date", "valueString": "2025-12-15"},
        {"key": "time", "valueString": "19:00"},
        {"key": "guests", "valueInt": 2}
      ]}
    ]}}
    ```

    **发出渲染信号：**

    ```json
    {"beginRendering": {"surfaceId": "main", "root": "header"}}
    ```

=== "v0.9 (Draft)"

    **创建 surface：**

    ```json
    {"version": "v0.9", "createSurface": {"surfaceId": "main", "catalogId": "https://a2ui.org/specification/v0_9/basic_catalog.json"}}
    ```

    **定义 UI：**

    ```json
    {"version": "v0.9", "updateComponents": {"surfaceId": "main", "components": [
      {"id": "header", "component": "Text", "text": "# Book Your Table", "variant": "h1"},
      {"id": "date-picker", "component": "DateTimeInput", "label": "Select Date", "value": {"path": "/reservation/date"}, "enableDate": true},
      {"id": "submit-text", "component": "Text", "text": "Confirm Reservation"},
      {"id": "submit-btn", "component": "Button", "child": "submit-text", "variant": "primary", "action": {"event": {"name": "confirm_booking"}}}
    ]}}
    ```

    **填充数据：**

    ```json
    {"version": "v0.9", "updateDataModel": {"surfaceId": "main", "path": "/reservation", "value": {"date": "2025-12-15", "time": "19:00", "guests": 2}}}
    ```

    注意：在 v0.9 中，`createSurface` 取代了 `beginRendering`；组件格式更扁平；数据模型也改为普通 JSON 值，而不是带类型的 adjacency list。

> 💡 **它本质上就是 JSON**
>
> 你会发现它既清晰又结构化。LLM 很容易生成这种格式，而且传输和渲染都很安全，不需要执行代码。

## 体验其他示例

仓库中还包含其他几个示例：

### 组件画廊（无需智能体）

查看所有可用的 A2UI 组件：

```bash
npm start -- gallery
```

这会启动一个纯客户端示例，展示所有标准组件（Card、Button、TextField、Timeline 等）的实时示例和代码片段。

### 联系人检索示例

体验另一个智能体应用场景：

```bash
npm run demo:contact
```

该示例展示了一个联系人检索智能体，它会生成搜索表单和结果列表。

## 接下来做什么

现在你已经看到了 A2UI 的实际运行效果，接下来可以继续：

- **[学习核心概念](concepts/overview.md)**：理解 surface、component 与 data binding
- **[搭建自己的客户端](guides/client-setup.md)**：将 A2UI 集成进你的应用
- **[构建智能体](guides/agent-development.md)**：创建能够生成 A2UI 响应的智能体
- **[阅读协议细节](reference/messages.md)**：深入了解技术规范

## 故障排查

### 端口已被占用

如果 5173 端口已被占用，开发服务器会自动尝试下一个可用端口。请查看终端输出中的实际访问地址。

### API Key 问题

如果你看到缺少 API Key 的错误：

1. 确认该变量已导出：`echo $GEMINI_API_KEY`
2. 确认它是从 [Google AI Studio](https://aistudio.google.com/apikey) 获取的有效 Gemini API Key
3. 尝试重新导出：`export GEMINI_API_KEY="your_key"`

### 启动时连接错误

如果浏览器打开时看到 `ERR_CONNECTION_REFUSED`，**不用担心**。这是一个已知的竞态问题（[#587](https://github.com/google/A2UI/issues/587)）。Web 应用启动速度比 Python 智能体后端更快。等待几秒后刷新页面即可。

### Python / uv 问题

这些示例智能体需要 [uv](https://docs.astral.sh/uv/) 才能运行。如果你看到 `uv: command not found`：

```bash
# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 验证
uv --version
```

如果你遇到其他 Python 错误：

```bash
# 确认 Python 3.10+ 可用
python3 --version

# 尝试手动运行智能体
cd samples/agent/adk/restaurant_finder
uv run .
```

### 还有问题？

- 查看 [GitHub Issues](https://github.com/google/a2ui/issues)
- 阅读 [samples/client/lit/README.md](https://github.com/google/a2ui/tree/main/samples/client/lit)
- 参与社区讨论

## 理解示例代码

如果你想看看它是如何实现的，可以查看：

- **Agent 代码**：`samples/agent/adk/restaurant_finder/` — Python A2A 智能体
- **客户端代码**：`samples/client/lit/` — 集成了 A2UI renderer 的 Lit Web 客户端
- **A2UI Renderers**：`renderers/lit/`（Lit）与 `renderers/web_core/`（框架无关核心）

每个目录下都带有自己的 README，提供更详细的说明。

---

**恭喜！** 你已经成功运行了第一个 A2UI 应用。你已经看到，AI 智能体如何通过安全、声明式的 JSON 消息，在 Web 应用中生成丰富且可交互的原生 UI。
