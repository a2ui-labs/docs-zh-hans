# 在任何 Agent 框架中使用 A2UI（使用 AG-UI）

A2UI 是一种声明式 UI 格式。[AG-UI](https://ag-ui.com/) 是在 agent 与浏览器之间承载 A2UI 消息的传输层。CopilotKit 的 AG-UI 实现是目前最快把 A2UI 放到用户面前的路径：CopilotKit 支持的任何 agent 框架（ADK、LangGraph、CrewAI、Mastra、自定义 Python/TS 服务等）都可以发出 A2UI，并且无需编写传输胶水代码就能在 React 应用中渲染。

!!! info "事实来源"

    本指南同步了 CopilotKit [ADK + A2UI 文档](https://docs.copilotkit.ai/adk/generative-ui/a2ui)中的关键步骤。
    最新 API surface 请以 CopilotKit 文档为准。

## 1. 设置 CopilotKit

在你选择的框架（ADK、LangGraph、CrewAI、Mastra 等）的 React/Next.js 应用中安装 CopilotKit：

```bash
npx copilotkit@latest init
```

也可以按照 [CopilotKit quickstart](https://docs.copilotkit.ai/quickstart) 将其接入现有项目。这是标准 CopilotKit 设置，不包含 A2UI 专用脚手架。

## 2. 启用 A2UI

### 后端

在 `CopilotRuntime` 中打开 A2UI，并注入 `render_a2ui` tool，让 agent 可以生成 A2UI surface：

```ts title="app/api/copilotkit/route.ts"
import {CopilotRuntime} from '@copilotkit/runtime';

const runtime = new CopilotRuntime({
  agents: {default: myAgent},
  a2ui: {injectA2UITool: true},
});
```

可以用 `a2ui: { injectA2UITool: true, agents: ["my-agent"] }` 将其限制到特定 agent。

### 前端

A2UI renderer 会自动激活。也可以选择传入主题：

{% raw %}

```tsx
import {CopilotKitProvider} from '@copilotkit/react-core/v2';
import '@copilotkit/react-core/v2/styles.css';
import {myCustomTheme} from '@copilotkit/a2ui-renderer';

<CopilotKitProvider runtimeUrl="/api/copilotkit" a2ui={{theme: myCustomTheme}}>
  {children}
</CopilotKitProvider>;
```

{% endraw %}

### 自定义组件（BYOC）

A2UI 内置 catalog（Text、Image、Card 等），可以立即得到可工作的 surface。真正的能力在于用 **你的** React 组件扩展它：你的设计系统、你的数据形状，让 agent 可以从你已经信任的原语中组合界面。一个 catalog 包含三部分：

1. **Definitions**：Zod schema 加自然语言描述。这是 agent 在 system prompt 中看到的内容。
2. **Renderers**：带类型的 React 组件，每个 definition 对应一个。这是用户看到的内容。
3. **Registration**：通过 provider 传入 catalog，让 A2UI renderer 知道如何绘制你的组件。

#### 1. 定义组件 schema

用 Zod 创建与平台无关的定义。`description` 字段会注入 agent 的 prompt，让 LLM 知道何时应该使用每个组件；schema 会校验 agent 发来的 props。

```ts title="lib/a2ui/definitions.ts"
import {z} from 'zod';

export const myDefinitions = {
  StatusBadge: {
    description: 'A colored status badge.',
    props: z.object({
      text: z.string(),
      variant: z.enum(['success', 'warning', 'error']).optional(),
    }),
  },
  Metric: {
    description: 'A key metric with label and value.',
    props: z.object({
      label: z.string(),
      value: z.string(),
      trend: z.enum(['up', 'down']).optional(),
    }),
  },
};

export type MyDefinitions = typeof myDefinitions;
```

#### 2. 创建 React renderer

把每个 definition 映射到一个 React 组件。`createCatalog` 会以 definitions 类型为泛型，因此 renderer 接收到的 props 会按 Zod schema 做类型检查，`props.text` 这样的拼写错误会成为编译错误。

{% raw %}

```tsx title="lib/a2ui/renderers.tsx"
'use client';

import {createCatalog, type CatalogRenderers} from '@copilotkit/a2ui-renderer';
import {myDefinitions, type MyDefinitions} from './definitions';

const myRenderers: CatalogRenderers<MyDefinitions> = {
  StatusBadge: ({props}) => {
    const colors = {
      success: {bg: '#dcfce7', text: '#166534'},
      warning: {bg: '#fef3c7', text: '#92400e'},
      error: {bg: '#fee2e2', text: '#991b1b'},
    };
    const c = colors[props.variant ?? 'success'];
    return (
      <span
        style={{
          padding: '2px 8px',
          borderRadius: 9999,
          fontSize: '0.75rem',
          background: c.bg,
          color: c.text,
        }}
      >
        {props.text}
      </span>
    );
  },

  Metric: ({props}) => (
    <div>
      <div style={{fontSize: '0.75rem', color: '#6b7280'}}>{props.label}</div>
      <div style={{fontSize: '1.5rem', fontWeight: 700}}>
        {props.value} {props.trend === 'up' ? '↑' : props.trend === 'down' ? '↓' : ''}
      </div>
    </div>
  ),
};

export const myCatalog = createCatalog(myDefinitions, myRenderers, {
  catalogId: 'my-app-catalog',
  includeBasicCatalog: true, // 与内置组件合并
});
```

{% endraw %}

`catalogId` 是 agent 用来定位这个 catalog 的稳定句柄；`includeBasicCatalog: true` 会让内置组件与你自己的组件一起可用（省略它则只渲染你的组件）。

#### 3. 将 catalog 传给 CopilotKit

{% raw %}

```tsx title="app/layout.tsx"
'use client';

import {CopilotKitProvider} from '@copilotkit/react-core/v2';
import '@copilotkit/react-core/v2/styles.css';
import {myCatalog} from '@/lib/a2ui/renderers';

export default function Layout({children}: {children: React.ReactNode}) {
  return (
    <CopilotKitProvider runtimeUrl="/api/copilotkit" a2ui={{catalog: myCatalog}}>
      {children}
    </CopilotKitProvider>
  );
}
```

{% endraw %}

Agent 现在会在内置组件之外看到你的自定义组件，并能在它发出的任何 A2UI surface 中使用这些组件。

完整 BYOC 参考（多个 catalog、主题 hook、高级模式）见 CopilotKit 的 [Custom Components (BYOC) section](https://docs.copilotkit.ai/adk/generative-ui/a2ui#custom-components-byoc)。

## 3. 高级用法

完整的 A2UI 集成 surface（自定义 catalog、细粒度控制、高级模式）见 CopilotKit 的 [A2UI docs](https://docs.copilotkit.ai/generative-ui/a2ui)。

## 接下来

- **[A2UI Composer](https://a2ui-composer.ag-ui.com/)**：可视化构建 widget。
- **[概念 › 传输层](../concepts/transports.md)**：A2UI 如何映射到 AG-UI。
- **[v0.9 规范](../specification/v0.9-a2ui.md)**：底层协议。
