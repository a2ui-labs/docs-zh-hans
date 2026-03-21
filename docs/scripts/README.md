# 文档转换脚本

该目录包含一些辅助脚本，用于在 **MkDocs** 构建流程中预处理文档。

## 目的

为了同时保证 GitHub 与托管站点上的阅读体验，我们将 **GitHub-flavored Markdown** 作为主要事实来源。这些脚本会在构建流水线中，把 GitHub 原生语法转换为 **MkDocs 兼容语法**（主要针对 `pymdown-extensions`）。

## 支持的转换（单向）

脚本执行的是单向转换：**GitHub Markdown → MkDocs 语法**。

### Alert / Admonition 转换

- GitHub 使用基于 blockquote 的 alert 语法。
- MkDocs 则需要使用 `!!!` 或 `???` 语法来渲染带颜色的提示框。

## 运行转换

转换会作为构建流水线的一部分自动执行，不需要额外步骤。如果你需要手动运行，可以在仓库根目录执行 `convert_docs.py`。

```bash
python docs/scripts/convert_docs.py
```

### 示例

- **源格式（GitHub-flavored Markdown）：**
  ```markdown
  > ⚠️ **Attention**
  >
  > This is an alert.
  ```

- **目标格式（MkDocs 语法）：**
  ```markdown
  !!! warning "Attention"
      This is an alert.
  ```
