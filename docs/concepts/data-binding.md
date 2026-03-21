# 数据绑定

数据绑定通过 JSON Pointer 路径 ([RFC 6901](https://tools.ietf.org/html/rfc6901)) 将 UI 组件连接到应用状态。A2UI 也正因此能够高效地为大规模数据数组定义布局，或者在无需从头重建的情况下展示更新后的内容。

## 结构与状态

A2UI 会将以下两者分离：

1. **UI 结构**（组件）：界面长什么样
2. **应用状态**（数据模型）：界面展示什么数据

这样就能支持：响应式更新、数据驱动 UI、可复用模板，以及双向绑定。

## 数据模型

每个 surface 都有一个保存状态的 JSON 对象：

```json
{
  "user": {"name": "Alice", "email": "alice@example.com"},
  "cart": {
    "items": [{"name": "Widget", "price": 9.99, "quantity": 2}],
    "total": 19.98
  }
}
```

## JSON Pointer 路径

**语法：**

- `/user/name` - 对象属性
- `/cart/items/0` - 数组索引（从 0 开始）
- `/cart/items/0/price` - 嵌套路径

**示例：**

```json
{"user": {"name": "Alice"}, "items": ["Apple", "Banana"]}
```

- `/user/name` → `"Alice"`
- `/items/0` → `"Apple"`

## 字面值与路径值

=== "v0.8"

    **字面值（固定）：**
    ```json
    {
      "id": "title",
      "component": {
        "Text": {
          "text": { "literalString": "Welcome" }
        }
      }
    }
    ```

    **数据绑定（响应式）：**
    ```json
    {
      "id": "username",
      "component": {
        "Text": {
          "text": { "path": "/user/name" }
        }
      }
    }
    ```

=== "v0.9"

    **字面值（固定）：**
    ```json
    {
      "id": "title",
      "component": "Text",
      "text": "Welcome"
    }
    ```

    **数据绑定（响应式）：**
    ```json
    {
      "id": "username",
      "component": "Text",
      "text": { "path": "/user/name" }
    }
    ```

当 `/user/name` 从 `"Alice"` 变成 `"Bob"` 时，文本会 **自动更新** 为 `"Bob"`。

## 响应式更新

绑定到数据路径的组件会在数据变化时自动更新：

```json
{
  "id": "status",
  "component": {
    "Text": {
      "text": { "path": "/order/status" }
    }
  }
}
```

- **初始：** `/order/status` = "Processing..." → 显示 "Processing..."
- **更新：** 发送一个 `status: "Shipped"` 的数据模型更新 → 显示 "Shipped"

无需更新组件，只需要更新数据。

## 动态列表

使用模板来渲染数组：

```json
{
  "id": "product-list",
  "component": {
    "Column": {
      "children": {
        "template": {
          "dataBinding": "/products",
          "componentId": "product-card"
        }
      }
    }
  }
}
```

**Data:**
```json
{
  "products": [
    { "name": "Widget", "price": 9.99 },
    { "name": "Gadget", "price": 19.99 }
  ]
}
```

**结果：** 渲染出两张卡片，每个商品一张。

### 作用域路径

在模板内部，路径会限定到数组项：

```json
{
  "id": "product-name",
  "component": {
    "Text": {
      "text": { "path": "/name" }
    }
  }
}
```

- 对 `/products/0` 而言，`/name` 会解析为 `/products/0/name` → "Widget"
- 对 `/products/1` 而言，`/name` 会解析为 `/products/1/name` → "Gadget"

添加或移除项目时，渲染出的组件会自动更新。

## 输入绑定

交互式组件会双向更新数据模型：

| 组件 | 示例 | 用户动作 | 数据更新 |
|-----------|---------|-------------|-------------|
| **TextField** | `{"text": {"path": "/form/name"}}` | 输入 "Alice" | `/form/name` = "Alice" |
| **CheckBox** | `{"value": {"path": "/form/agreed"}}` | 勾选复选框 | `/form/agreed` = true |
| **MultipleChoice** | `{"selections": {"path": "/form/country"}}` | 选择 "Canada" | `/form/country` = ["ca"] |

## 最佳实践

**1. 使用细粒度更新** - 只更新发生变化的路径：
```json
{
  "dataModelUpdate": {
    "path": "/user",
    "contents": [
      { "key": "name", "valueString": "Alice" }
    ]
  }
}
```

**2. 按领域组织** - 将相关数据分组：
```json
{"user": {...}, "cart": {...}, "ui": {...}}
```

**3. 预先计算展示值** - 代理在发送前先格式化数据（货币、日期）：
```json
{"price": "$19.99"}  // 而不是: {"price": 19.99}
```
