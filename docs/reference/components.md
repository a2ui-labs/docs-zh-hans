# 组件画廊

本页展示所有 A2UI 组件的示例和用法模式。

!!! abstract "规范文件"

    === "v0.8"

        [:material-code-json: 标准目录定义（JSON Schema）](https://a2ui.org/specification/v0_8/standard_catalog_definition.json)

    === "v0.9"

        [:material-code-json: 基础目录定义（JSON Schema）](https://a2ui.org/specification/v0_9/basic_catalog.json)

---

## 布局组件

### Row

横向布局容器。子项从左到右排列。

=== "v0.8"

    **属性：** `children`（`explicitList` 或 `template`）、`distribution`、`alignment`

    ```json
    {
      "id": "toolbar",
      "component": {
        "Row": {
          "children": { "explicitList": ["btn1", "btn2", "btn3"] },
          "distribution": "spaceBetween",
          "alignment": "center"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `children`（数组或模板）、`justify`、`align`

    ```json
    {
      "id": "toolbar",
      "component": "Row",
      "children": ["btn1", "btn2", "btn3"],
      "justify": "spaceBetween",
      "align": "center"
    }
    ```

### Column

纵向布局容器。子项从上到下排列。

=== "v0.8"

    **属性：** `children`（`explicitList` 或 `template`）、`distribution`、`alignment`

    ```json
    {
      "id": "content",
      "component": {
        "Column": {
          "children": { "explicitList": ["header", "body", "footer"] },
          "distribution": "start",
          "alignment": "stretch"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `children`（数组或模板）、`justify`、`align`

    ```json
    {
      "id": "content",
      "component": "Column",
      "children": ["header", "body", "footer"],
      "justify": "start",
      "align": "stretch"
    }
    ```

### List

可滚动的列表。支持静态子项与动态模板。

=== "v0.8"

    **属性：** `children`（`explicitList` 或 `template`）、`direction`、`alignment`

    ```json
    {
      "id": "message-list",
      "component": {
        "List": {
          "children": {
            "template": {
              "dataBinding": "/messages",
              "componentId": "message-item"
            }
          },
          "direction": "vertical"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `children`（数组或模板）、`direction`、`align`

    ```json
    {
      "id": "message-list",
      "component": "List",
      "children": {
        "componentId": "message-item",
        "path": "/messages"
      },
      "direction": "vertical"
    }
    ```

---

## 显示组件

### Text

显示带样式提示的文本内容。

=== "v0.8"

    **属性：** `text`（BoundValue）、`usageHint`

    `usageHint` 取值：`h1`、`h2`、`h3`、`h4`、`h5`、`caption`、`body`

    ```json
    {
      "id": "title",
      "component": {
        "Text": {
          "text": { "literalString": "Welcome to A2UI" },
          "usageHint": "h1"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `text`（string 或 DataBinding）、`variant`

    `variant` 取值：`h1`、`h2`、`h3`、`h4`、`h5`、`caption`、`body`

    ```json
    {
      "id": "title",
      "component": "Text",
      "text": "Welcome to A2UI",
      "variant": "h1"
    }
    ```

### Image

显示来自 URL 的图像。

=== "v0.8"

    **属性：** `url`（BoundValue）、`fit`、`usageHint`

    ```json
    {
      "id": "hero",
      "component": {
        "Image": {
          "url": { "literalString": "https://example.com/hero.png" },
          "fit": "cover",
          "usageHint": "hero"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `url`（string 或 DataBinding）、`fit`、`variant`

    ```json
    {
      "id": "hero",
      "component": "Image",
      "url": "https://example.com/hero.png",
      "fit": "cover",
      "variant": "hero"
    }
    ```

### Icon

显示 catalog 中定义的标准图标集里的图标。

=== "v0.8"

    **属性：** `name`（BoundValue）

    ```json
    {
      "id": "check-icon",
      "component": {
        "Icon": {
          "name": { "literalString": "check" }
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `name`（string 或 DataBinding）

    ```json
    {
      "id": "check-icon",
      "component": "Icon",
      "name": "check"
    }
    ```

### Divider

视觉分隔线。

=== "v0.8"

    **属性：** `axis`

    ```json
    {
      "id": "separator",
      "component": {
        "Divider": {
          "axis": "horizontal"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `axis`

    ```json
    {
      "id": "separator",
      "component": "Divider",
      "axis": "horizontal"
    }
    ```

---

## 交互组件

### Button

可点击按钮，用于触发动作。

=== "v0.8"

    **属性：** `child`（组件 ID）、`primary`（布尔值）、`action`

    ```json
    {
      "id": "submit-btn",
      "component": {
        "Button": {
          "child": "submit-text",
          "primary": true,
          "action": {
            "name": "submit_form"
          }
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `child`（组件 ID）、`variant`、`action`

    ```json
    {
      "id": "submit-btn",
      "component": "Button",
      "child": "submit-text",
      "variant": "primary",
      "action": {
        "event": {
          "name": "submit_form"
        }
      }
    }
    ```

### TextField

带可选验证的文本输入框。

=== "v0.8"

    **属性：** `label`（BoundValue）、`text`（BoundValue）、`textFieldType`、`validationRegexp`

    `textFieldType` 取值：`shortText`、`longText`、`number`、`obscured`、`date`

    ```json
    {
      "id": "email-input",
      "component": {
        "TextField": {
          "label": { "literalString": "Email Address" },
          "text": { "path": "/user/email" },
          "textFieldType": "shortText"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `label`（string）、`value`（string 或 DataBinding）、`textFieldType`、`validationRegexp`

    `textFieldType` 取值：`shortText`、`longText`、`number`、`obscured`、`date`

    ```json
    {
      "id": "email-input",
      "component": "TextField",
      "label": "Email Address",
      "value": { "path": "/user/email" },
      "textFieldType": "shortText"
    }
    ```

### CheckBox

布尔开关。

=== "v0.8"

    **属性：** `label`（BoundValue）、`value`（BoundValue, boolean）

    ```json
    {
      "id": "terms-checkbox",
      "component": {
        "CheckBox": {
          "label": { "literalString": "I agree to the terms" },
          "value": { "path": "/form/agreedToTerms" }
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `label`（string）、`value`（DataBinding, boolean）

    ```json
    {
      "id": "terms-checkbox",
      "component": "CheckBox",
      "label": "I agree to the terms",
      "value": { "path": "/form/agreedToTerms" }
    }
    ```

### Slider

数值范围输入。

=== "v0.8"

    **属性：** `value`（BoundValue）、`minValue`、`maxValue`

    ```json
    {
      "id": "volume",
      "component": {
        "Slider": {
          "value": { "path": "/settings/volume" },
          "minValue": 0,
          "maxValue": 100
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `value`（DataBinding）、`minValue`、`maxValue`

    ```json
    {
      "id": "volume",
      "component": "Slider",
      "value": { "path": "/settings/volume" },
      "minValue": 0,
      "maxValue": 100
    }
    ```

### DateTimeInput

日期和/或时间选择器。

=== "v0.8"

    **属性：** `value`（BoundValue）、`enableDate`、`enableTime`

    ```json
    {
      "id": "date-picker",
      "component": {
        "DateTimeInput": {
          "value": { "path": "/booking/date" },
          "enableDate": true,
          "enableTime": false
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `value`（DataBinding）、`enableDate`、`enableTime`

    ```json
    {
      "id": "date-picker",
      "component": "DateTimeInput",
      "value": { "path": "/booking/date" },
      "enableDate": true,
      "enableTime": false
    }
    ```

### MultipleChoice (v0.8) / ChoicePicker (v0.9)

从列表中选择一个或多个选项。

=== "v0.8"

    **属性：** `options`（array）、`selections`（BoundValue）、`maxAllowedSelections`

    ```json
    {
      "id": "country-select",
      "component": {
        "MultipleChoice": {
          "options": [
            { "label": { "literalString": "USA" }, "value": "us" },
            { "label": { "literalString": "Canada" }, "value": "ca" }
          ],
          "selections": { "path": "/form/country" },
          "maxAllowedSelections": 1
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `options`（array）、`selections`（DataBinding）、`maxAllowedSelections`

    ```json
    {
      "id": "country-select",
      "component": "ChoicePicker",
      "options": [
        { "label": "USA", "value": "us" },
        { "label": "Canada", "value": "ca" }
      ],
      "selections": { "path": "/form/country" },
      "maxAllowedSelections": 1
    }
    ```

---

## 容器组件

### Card

带有层级感、边框和内边距的容器。

=== "v0.8"

    **属性：** `child`（组件 ID）

    ```json
    {
      "id": "info-card",
      "component": {
        "Card": {
          "child": "card-content"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `child`（组件 ID）

    ```json
    {
      "id": "info-card",
      "component": "Card",
      "child": "card-content"
    }
    ```

### Modal

由入口组件触发的遮罩式对话框。

=== "v0.8"

    **属性：** `entryPointChild`（组件 ID）、`contentChild`（组件 ID）

    ```json
    {
      "id": "confirmation-modal",
      "component": {
        "Modal": {
          "entryPointChild": "open-modal-btn",
          "contentChild": "modal-content"
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `entryPointChild`（组件 ID）、`contentChild`（组件 ID）

    ```json
    {
      "id": "confirmation-modal",
      "component": "Modal",
      "entryPointChild": "open-modal-btn",
      "contentChild": "modal-content"
    }
    ```

### Tabs

用于将内容组织为可切换面板的标签界面。

=== "v0.8"

    **属性：** `tabItems`（包含 `{ title, child }` 的数组）

    ```json
    {
      "id": "settings-tabs",
      "component": {
        "Tabs": {
          "tabItems": [
            { "title": { "literalString": "General" }, "child": "general-tab" },
            { "title": { "literalString": "Privacy" }, "child": "privacy-tab" }
          ]
        }
      }
    }
    ```

=== "v0.9"

    **属性：** `tabItems`（包含 `{ title, child }` 的数组）

    ```json
    {
      "id": "settings-tabs",
      "component": "Tabs",
      "tabItems": [
        { "title": "General", "child": "general-tab" },
        { "title": "Privacy", "child": "privacy-tab" }
      ]
    }
    ```

---

## 通用属性

所有组件共享以下属性：

- `id`（必需）：在同一个 surface 内唯一。
- `accessibility`：可访问性属性（label、role）。
- `weight`：在 Row 或 Column 中的 flex-grow 值。

## 版本差异总结

组件名称和属性在两个版本之间大体一致。结构差异如下：

| 方面 | v0.8 | v0.9 |
|--------|------|------|
| 组件包装器 | `"component": { "Text": { ... } }` | `"component": "Text", ...props` |
| 字符串值 | `{ "literalString": "Hello" }` | `"Hello"` |
| 子项 | `{ "explicitList": ["a", "b"] }` | `["a", "b"]` |
| 数据绑定 | `{ "path": "/data" }` | `{ "path": "/data" }`（相同） |
| 文本/图片样式 | `usageHint` | `variant` |
| 按钮样式 | `primary: true` | `variant: "primary"` |
| 动作格式 | `{ "name": "..." }` | `{ "event": { "name": "..." } }` |
| 选择组件 | `MultipleChoice` | `ChoicePicker` |
| 布局对齐 | `distribution`、`alignment` | `justify`、`align` |
| TextField 值 | `text` | `value` |

## 在线示例

要查看所有组件的实际效果：

```bash
cd samples/client/angular
npm start -- gallery
```

## 进一步阅读

!!! abstract "规范文件"

    === "v0.8"

        [:material-code-json: 标准目录定义（JSON Schema）](https://a2ui.org/specification/v0_8/standard_catalog_definition.json)

    === "v0.9"

        [:material-code-json: 基础目录定义（JSON Schema）](https://a2ui.org/specification/v0_9/basic_catalog.json)

- **[自定义组件指南](../guides/custom-components.md)**：构建你自己的组件
- **[主题指南](../guides/theming.md)**：让组件样式与你的品牌一致
