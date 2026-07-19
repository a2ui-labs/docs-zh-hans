# 智能体（服务端）

智能体是运行在服务端的程序，用来根据用户请求生成 A2UI 消息。

真正的组件渲染由 [渲染器](renderers.md) 完成，
消息通过 [传输层](../concepts/transports.md) 发送到客户端之后再被渲染。
智能体只负责生成 A2UI 消息本身。

## 智能体如何工作

```
用户输入 → 智能体逻辑 → LLM → A2UI JSON → 发送到客户端
```

1. **接收** 用户消息
2. **交给 LLM 处理**（Gemini、GPT、Claude 等）
3. **生成** 结构化输出形式的 A2UI JSON 消息
4. **通过传输层发送** 给客户端

来自客户端的用户交互也可以被视为新的用户输入。

## 示例智能体

A2UI 仓库中提供了一些可供学习的示例智能体：

- [Restaurant Finder](https://github.com/a2ui-project/a2ui/tree/main/samples/agent/adk/restaurant_finder)
    - 使用表单完成餐桌预订
    - 使用 ADK 编写
- [Rizzcharts](https://github.com/a2ui-project/a2ui/tree/main/samples/community/agent/adk/rizzcharts/python)
    - A2UI 自定义组件演示
    - 使用 ADK 编写
- [Orchestrator](https://github.com/a2ui-project/a2ui/tree/main/samples/community/agent/adk/orchestrator)
    - 透传来自远端子智能体的 A2UI 消息
    - 使用 ADK 编写

## 在 A2A 场景中你会遇到的几类智能体

### 1. 面向用户的智能体（独立）

面向用户的智能体是指用户会直接与之交互的智能体。

### 2. 作为远端智能体宿主的面向用户智能体

这是一种常见模式：面向用户的智能体作为一个或多个远端智能体的宿主。面向用户的智能体会调用远端智能体，而远端智能体负责生成 A2UI 消息。这在 [A2A](https://a2a-protocol.org) 中非常常见，即客户端智能体调用服务端智能体。

- 面向用户的智能体可以直接“透传” A2UI 消息而不做修改
- 也可以在发送给客户端之前，对 A2UI 消息进行加工

### 3. 远端智能体

远端智能体并不直接构成用户可见 UI 的一部分。它通常作为远端智能体被注册，然后由面向用户的智能体调用。这同样是 [A2A](https://a2a-protocol.org) 中常见的模式，即客户端智能体调用服务端智能体。
