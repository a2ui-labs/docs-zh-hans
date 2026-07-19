# 2026-07-19 翻译同步记录

将 `docs-zh-hans` 与上游 A2UI 仓库（`google/A2UI` → `a2ui-project/a2ui`）同步至提交 `305336f1` → `3708c069`（223 个提交）。

## 1. 结构迁移：`docs/` → `docs/public/`

上游把 `mkdocs.yaml` 的 `docs_dir` 从 `docs` 改为 `docs/public`。`docs-zh-hans` 做了同样的调整：

- 使用 `git mv` 将 `docs/` 下除 `docs/scripts/**` 之外的所有文件迁移到 `docs/public/`，保留历史记录。
- `docs/glossary.md` 迁移到 `docs/public/concepts/glossary.md`（与上游一致，并入 `concepts/` 子目录，而不是简单的路径前缀迁移）。
- `docs/scripts/` 保持原位（`convert_docs.py`、`test_convert_docs.py` 是开发工具脚本，不翻译，已用上游最新版本原样替换；`docs/scripts/README.md` 上游未改动，保持不变）。
- 迁移后的最终目录树已与上游 `git ls-tree` 输出逐项核对，结构完全一致（上游独有的 `docs/public/CNAME` 按要求未添加）。

新增的二进制资源（`assets/favicon.svg` 等）已从上游原样复制。反过来，`assets/agent-and-renderer.png` 在上游已被删除（`glossary.md` 中的静态图片被替换为 mermaid 时序图），因此这个不再被引用的文件也一并删除。

## 2. `mkdocs.yaml`

- 新增 `docs_dir: docs/public`。
- 核心概念导航新增「术语表：concepts/glossary.md」。
- 指南导航：「在任意 Agent 框架中使用 A2UI (AG-UI)」改为「在任意 Agent 框架与 Harness 中使用 A2UI」；把 3 个 MCP 相关指南（`a2ui_over_mcp.md`、`mcp-apps-in-a2ui.md`、`a2ui-in-mcp-apps.md`）归并为「A2UI + MCP」子分组。
- 规范导航扩展为四级：v1.0（候选版）、v0.9.1（当前版）、v0.9（前一稳定版）、v0.8（旧版），各版本下补齐 A2UI 协议 / A2A 扩展 / 演进指南 / Basic Catalog 指南等条目（v0.8、v0.9 沿用上游只提供部分条目的做法）。
- `exclude_docs` 新增 `specification/v*/**/*.md`。
- `repo_name` / `repo_url` / `edit_uri` 由 `google/A2UI` 改为 `a2ui-project/a2ui`（以及 `raw/main/docs/public/`）。
- `favicon` 由 `assets/A2UI_dark.svg` 改为 `assets/favicon.svg`。
- 检查确认没有残留的、注释掉的 `llmstxt` 插件配置块。

## 3. 内容翻译

按上游改动更新了以下文件；未变化的既有译文段落原样保留，仅新增/变更的内容重新翻译：

- **根目录**：`README.md`（v0.9.1 当前发布状态说明、Corepack/Yarn 安装流程、`a2ui-project/a2ui` 仓库地址、"AG-UI CLI" 流程、`npx create-ag-ui-app@latest`）。
- **顶层文档**：`index.md`、`quickstart.md`、`roadmap.md`。
- **核心概念（concepts/）**：`actions.md`、`catalogs.md`、`components.md`、`data-binding.md`、`data-flow.md`（仅修正 AG-UI 拼写）、`overview.md`（术语表链接、消息类型标签页重排为 v0.9/v1.0/v0.8 三段式）、`transports.md`、`glossary.md`（新位置、图片替换为 mermaid 图、相对链接层级修正）。
- **生态（ecosystem/）**：`a2ui-in-the-world.md`、`community.md`。`renderers.md` 上游整体重写（结构与内容都变了），因此不是逐段打补丁，而是参考旧译文的术语与风格重新翻译。
- **指南（guides/）**：`a2ui-in-mcp-apps.md`、`a2ui_over_mcp.md`、`agent-development.md`、`authoring-components.md`、`client-setup.md`、`defining-your-own-catalog.md`、`mcp-apps-in-a2ui.md`、`theming.md`。
  - `renderer-development.md`、`a2ui-with-any-agent-framework.md` 在上游被整体重写（分别新增 186 行、511 行），沿用同样的处理方式：旧译文仅作为术语与文风参考，内容重新翻译。
- **参考（reference/）**：`agents.md`、`components.md`、`messages.md`、`renderers.md`（新增 v1.0 列，更新各渲染器状态）。
- **简介（introduction/）**：`agent-ui-ecosystem.md`（AG-UI 拼写修正）、`how-to-use.md`（AG-UI 拼写修正）、`what-is-a2ui.md`、`who-is-it-for.md`。
- **规范（specification/）**：
  - 既有文件小幅修改：`v0.8-a2ui.md`、`v0.8-a2a-extension.md`、`v0.9-a2ui.md`、`v0.9-evolution-guide.md`（版本状态标签与相互引用链接，随 v1.0、v0.9.1 的加入一并更新）。
  - 新增 8 个文件并完整翻译：`v0.9.1-a2ui.md`、`v0.9.1-a2ui-extension-specification.md`、`v0.9.1-basic-catalog-implementation-guide.md`、`v0.9.1-evolution-guide.md`、`v1.0-a2ui.md`、`v1.0-a2ui-extension-specification.md`、`v1.0-basic-catalog-implementation-guide.md`、`v1.0-evolution-guide.md`。均沿用 `v0.8-a2ui.md`/`v0.9-a2ui.md` 已确立的 docs-zh-hans 模板（"活文档" admonition + `--8<--` 片段引用 + 点号版本路径）。

全文统一将 "AG UI"（无连字符）改为 "AG-UI"（带连字符），并将所有 `google/A2UI` 仓库链接改为 `a2ui-project/a2ui`。

## 4. 顺带修复的历史遗留问题

在逐文件比对过程中发现，docs-zh-hans 的部分内容其实早于本次同步基线（`305336f1`）就已经与上游脱节——不是这次上游改动引入的差异，而是更早遗留下来的翻译缺口。考虑到这是面向真实读者的交付物，本次一并做了修复，而不是只处理严格意义上的 diff：

- `concepts/catalogs.md`：中文译文在"实现渲染器"一节中途戛然而止，缺少后面近一半内容（A2UI Catalog Negotiation、Catalog 命名与版本管理、Schema 验证与回退、内联 Catalog 等整节）——已补全翻译。
- `guides/a2ui_over_mcp.md`：中文结构与当前英文版本明显不同，缺少 Prerequisites、Quick Start、How It Works、Resources vs. Tools 等大段内容——已按新版英文结构重新翻译。
- `roadmap.md`：中文的协议/渲染器/传输/框架状态表与里程碑时间线均已过时，且存在一个上游从未有过的"社区需求"小节——已整体核对英文原文重写。
- `ecosystem/a2ui-in-the-world.md`：缺少 "AG2" 完整小节——已补充翻译。
- `guides/agent-development.md`："生成 A2UI 消息"一节使用的是早已废弃的 `contact_lookup` 示例流程，与当前 `A2uiSchemaManager` 方案不符——已更新为当前方案。
- `guides/client-setup.md`：多处包名错误（`@a2ui/web-lib` 应为 `@a2ui/web_core`）、过时的"尚未发布到 NPM"提示、React 渲染器 v0.9 支持状态标注错误，以及多处 "TODO：补充示例" 占位符（实际英文原文早已有真实内容）——均已修正。
- `guides/theming.md`：约三分之一内容是占位性质的"TODO"描述，与当前英文的 `theme` 属性、Catalog 主题等实际内容不符，且存在一个英文中从未出现过的"响应式设计"小节——已参照英文结构重新翻译。
- `quickstart.md`：安装流程使用的是已废弃的 `demo:all`/`npm run demo:contact` 命令与联系人检索示例，与当前 `yarn demo:restaurant` 流程不符，且缺少"手动运行"备选步骤——已更新为当前流程。
- `index.md`："5 分钟快速上手"卡片区缺少 AG-UI 卡片、A2UI Composer 卡片、A2UI Theater 卡片——已补充。

## 5. 交叉引用检查

`glossary.md` 从顶层 `docs/` 移动到 `concepts/` 子目录后，检查了引用它的链接：`concepts/overview.md` 中的链接由 `../glossary.md` 改为同目录下的 `glossary.md`。检索全树后确认这是唯一引用 `glossary.md` 的位置，且与上游新版本的链接文本完全一致。

`glossary.md` 内部指向 `concepts/data-flow.md`、`concepts/data-binding.md`、`concepts/actions.md` 的链接，也相应改为同目录下的相对路径 `data-flow.md`、`data-binding.md`、`actions.md`；指向仓库根目录 `renderers/` 的链接则因为多迁移了一层目录，从 `../renderers/...` 改为 `../../../renderers/...`。

## 6. 范围之外（未处理）

- 顶层 `specification/`（原始 JSON 规范等）与 `eval/` 目录 —— 按要求完全排除在外。
- `docs/scripts/convert_docs.py`、`docs/scripts/test_convert_docs.py` —— 不翻译，直接使用上游版本。
- `docs/scripts/README.md` —— 上游本次也未改动，保持不变。
- `docs/public/CNAME` —— 按要求未添加。
- `composer.md` —— 上游本次仅涉及路径重命名，内容 100% 未变，无需重新翻译。

## 7. 已知可关注点

- `guides/mcp-apps-in-a2ui.md`（MCP Apps 在 A2UI 中的集成）与 `guides/theming.md` 存在与 `concepts/catalogs.md` 类似的历史遗留缺口，本次已一并核对英文原文全面更新。
- 建议今后同步时，除了看上游 diff，也定期抽查中文译文是否已经"悄悄"落后于当前上游正文（而不仅仅是落后于本次同步的起点提交），避免类似缺口再次累积。
