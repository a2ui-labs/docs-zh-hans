# 简体中文翻译完成记录 2026-03-21

## 本次已完成

1. 创建 `docs-zh-hans/` 目录，并从最新 `A2UI/docs` 复制文档骨架与资源文件。
2. 完整翻译 `README.md`、`mkdocs.yaml` 以及 `docs/**` 下的全部 Markdown 文档。
3. 保持与上游最新文档结构一致，包括 `concepts`、`guides`、`reference`、`ecosystem`、`specification` 与 `docs/scripts/README.md`。
4. 移除了从上游继承的 `docs/CNAME`，避免后续部署时误用 `a2ui.org` 域名。

## 说明

- 某些页面中保留了上游原文里的 `TODO`、实验性说明与外部链接，这些内容仅做语言翻译，不改变其技术含义。
- `docs/specification/*.md` 仍沿用上游 `--8<--` include 形式；如果后续要独立建站，还需要补齐对应的 include 文件来源。
- 如果后续要单独发布，建议把该目录初始化为独立 Git 仓库，并补充对应的部署流水线。
