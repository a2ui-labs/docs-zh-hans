# 文档转换脚本

此目录包含用于为 **MkDocs** 构建流程准备文档的工具脚本。

## 用途

为了同时保证 GitHub 与托管站点上的阅读体验，项目使用 **GitHub-flavored Markdown** 作为主要事实来源。该脚本会在构建流水线中把 GitHub 原生语法转换为 **MkDocs 兼容语法**（特别是面向 `pymdown-extensions`）。

## 支持的转换（单向）

该脚本执行单向转换：**GitHub Markdown → MkDocs Syntax**。

### Alert/Admonition 转换

脚本会处理以下转换：

- GitHub 使用基于 blockquote 的语法来表示 alert。
- MkDocs 需要 `!!!` 或 `???` 语法来渲染彩色提示框。

## 运行转换

转换会作为构建流水线的一部分运行，不需要额外步骤。如果需要手动运行转换，可以在仓库根目录运行 `convert_docs.py` 脚本。

```bash
python docs/scripts/convert_docs.py
```

### 示例

- **源文件（GitHub-flavored Markdown）：**

    ```markdown
    > ⚠️ **Attention**
    >
    > This is an alert.
    ```

- **目标（MkDocs Syntax）：**

    ```markdown
    !!! warning "Attention"
    This is an alert.
    ```
