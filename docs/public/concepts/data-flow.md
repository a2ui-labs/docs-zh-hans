# 数据流

消息如何从代理流向 UI。

## 架构

```
Agent (LLM) → A2UI Generator → Transport (SSE/WS/A2A)
                                      ↓
Client (Stream Reader) → Message Parser → Renderer → Native UI
```

![end-to-end-data-flow](../assets/end-to-end-data-flow.png)

## 消息格式

A2UI 定义了一组用于描述 UI 的 JSON 消息。以流式方式传输时，这些消息通常会采用 **JSON Lines（JSONL）** 格式，也就是每一行都是一个完整的 JSON 对象。

=== "v0.8"

    ```jsonl
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [...]
      }
    }
    {
      "dataModelUpdate": {
        "surfaceId": "main",
        "contents": [
          {
            "key": "user",
            "valueMap": [
              { "key": "name", "valueString": "Alice" }
            ]
          }
        ]
      }
    }
    {
      "beginRendering": {
        "surfaceId": "main",
        "root": "root-component"
      }
    }
    ```

=== "v0.9"

    ```jsonl
    {
      "version": "v0.9",
      "createSurface": {
        "surfaceId": "main",
        "catalogId": "https://a2ui.org/specification/v0_9/basic_catalog.json"
      }
    }
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [...]
      }
    }
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "main",
        "path": "/user",
        "value": { "name": "Alice" }
      }
    }
    ```

**为什么使用这种格式？**

一组自包含的 JSON 对象更适合流式传输，也更容易让 LLM 增量生成，并且对错误更有韧性。

## 生命周期示例：餐厅预订

**用户：** "明天晚上 7 点帮我订一张 2 人桌"

=== "v0.8"

    **1. 代理定义 UI 结构：**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "booking",
        "components": [
          {
            "id": "root",
            "component": {
              "Column": {
                "children": {
                  "explicitList": ["header", "guests-field", "submit-btn"]
                }
              }
            }
          },
          {
            "id": "header",
            "component": {
              "Text": {
                "text": { "literalString": "Confirm Reservation" },
                "usageHint": "h1"
              }
            }
          },
          {
            "id": "guests-field",
            "component": {
              "TextField": {
                "label": { "literalString": "Guests" },
                "text": { "path": "/reservation/guests" }
              }
            }
          },
          {
            "id": "submit-btn",
            "component": {
              "Button": {
                "child": "submit-text",
                "action": {
                  "name": "confirm",
                  "context": [
                    { "key": "details", "value": { "path": "/reservation" } }
                  ]
                }
              }
            }
          }
        ]
      }
    }
    ```

    **2. 代理填充数据：**

    ```json
    {
      "dataModelUpdate": {
        "surfaceId": "booking",
        "path": "/reservation",
        "contents": [
          { "key": "datetime", "valueString": "2025-12-16T19:00:00Z" },
          { "key": "guests", "valueString": "2" }
        ]
      }
    }
    ```

    **3. 代理发出渲染信号：**

    ```json
    {
      "beginRendering": {
        "surfaceId": "booking",
        "root": "root"
      }
    }
    ```

    **4. 用户把客人数改成 "3"** → 客户端会自动更新 `/reservation/guests`

    **5. 用户点击 "Confirm"** → 客户端发送动作：

    ```json
    {
      "userAction": {
        "name": "confirm",
        "surfaceId": "booking",
        "context": {
          "details": {
            "datetime": "2025-12-16T19:00:00Z",
            "guests": "3"
          }
        }
      }
    }
    ```

    **6. 代理响应** → 更新 UI 或发送：

    ```json
    { "deleteSurface": { "surfaceId": "booking" } }
    ```

=== "v0.9"

    **1. 代理创建 surface：**

    ```json
    {
      "version": "v0.9",
      "createSurface": {
        "surfaceId": "booking",
        "catalogId": "https://a2ui.org/specification/v0_9/basic_catalog.json"
      }
    }
    ```

    **2. 代理定义 UI 结构：**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "booking",
        "components": [
          {
            "id": "root",
            "component": "Column",
            "children": ["header", "guests-field", "submit-btn"]
          },
          {
            "id": "header",
            "component": "Text",
            "text": "Confirm Reservation",
            "variant": "h1"
          },
          {
            "id": "guests-field",
            "component": "TextField",
            "label": "Guests",
            "value": { "path": "/reservation/guests" }
          },
          {
            "id": "submit-btn",
            "component": "Button",
            "child": "submit-text",
            "variant": "primary",
            "action": {
              "event": {
                "name": "confirm",
                "context": {
                  "details": { "path": "/reservation" }
                }
              }
            }
          }
        ]
      }
    }
    ```

    **3. 代理填充数据：**

    ```json
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "booking",
        "path": "/reservation",
        "value": {
          "datetime": "2025-12-16T19:00:00Z",
          "guests": "2"
        }
      }
    }
    ```

    **4. 用户把客人数改成 "3"** → 客户端会自动更新 `/reservation/guests`

    **5. 用户点击 "Confirm"** → 客户端发送动作：

    ```json
    {
      "version": "v0.9",
      "action": {
        "name": "confirm",
        "surfaceId": "booking",
        "context": {
          "details": {
            "datetime": "2025-12-16T19:00:00Z",
            "guests": "3"
          }
        }
      }
    }
    ```

    **6. 代理响应** → 更新 UI 或发送：

    ```json
    {
      "version": "v0.9",
      "deleteSurface": { "surfaceId": "booking" }
    }
    ```

## 传输选项

A2UI 不依赖特定传输层。任何能够传递 JSON 消息的机制都可以：

- **[A2A Protocol](https://a2a-protocol.org/)**：标准化的 agent-to-agent 通信，也可用于 agent-to-UI 传输
- **[AG-UI](https://ag-ui.com/)**：双向、实时的 agent-UI 协议
- **REST / HTTP**：简单的请求-响应，或者使用 Server-Sent Events（SSE）实现单向流式传输
- **WebSocket**：持久化双向连接，非常适合实时更新和用户动作
- **其他任意传输**：gRPC、消息队列、自定义协议，只要能携带 JSON，就能用

实现细节请参见 [transports](transports.md)。

## 渐进式渲染

不必等整个响应生成完毕再向用户展示，响应可以在生成过程中按块流式发送到客户端，并逐步渲染。

用户看到的是 UI 实时搭建的过程，而不是一直盯着加载转圈。

## 错误处理

- **消息格式错误：** 跳过并继续，或者把错误发回代理要求修正
- **网络中断：** 展示错误状态、重新连接，代理重新发送或继续恢复

## 性能

- **批处理：** 将更新缓冲 16ms，再一起渲染
- **差异比较：** 对比新旧组件，只更新发生变化的属性
- **细粒度更新：** 更新 `/user/name`，而不是整个 `/` 模型
