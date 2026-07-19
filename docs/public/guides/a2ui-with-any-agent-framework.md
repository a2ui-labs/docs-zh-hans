# 在任意 Agent 框架与 Harness 中使用 A2UI

A2UI 是一种声明式 UI 格式。[AG-UI](https://ag-ui.com/) 是在 agent 与应用之间承载 A2UI 消息的传输层。本指南将介绍如何为运行在 ADK、LangGraph、Mastra、Strands、CrewAI、Google Chat、Slack 或任何其他支持 AG-UI 的 agent 框架或服务之上的 AG-UI 应用或 harness（即承载 agent 运行的宿主应用）添加 A2UI。

<style>
  .agui-demo-video {
    border-radius: 8px;
    display: block;
    margin: 24px auto;
    max-width: 100%;
    width: 75%;
  }

  @media (max-width: 700px) {
    .agui-demo-video {
      width: 100%;
    }
  }
</style>

<video class="agui-demo-video" controls playsinline preload="metadata">
  <source src="https://cdn.copilotkit.ai/docs/a2ui/ag-ui-a2ui-demo.mp4" type="video/mp4">
  你的浏览器不支持 video 标签。
</video>

下面的示例使用兼容 AG-UI 的运行时工具，让你可以专注于 A2UI surface 本身：启用 renderer、为你的 agent 配置 catalog，以及把 UI 更新流式传回给用户。关于协议层面的设置与概念，请参见 [AG-UI 文档](https://docs.ag-ui.com/)。

## Agent 技能

如果你正在用编程 agent 来完成这部分接入工作，请在它修改你的应用之前，先加载 [AG-UI 的 `ag-ui-a2ui-integration` skill](https://github.com/ag-ui-protocol/ag-ui/tree/main/skills/ag-ui-a2ui-integration)。它涵盖了 AG-UI 的框架适配器、受支持的 `create-ag-ui-app` 参数、传输层设置、A2UI runtime 与 renderer 的接入方式，以及针对 AG-UI + A2UI 应用的端到端验证。

如果你的应用使用 CopilotKit 来渲染 A2UI，还应加载 [CopilotKit 的 `a2ui-renderer` skill](https://github.com/CopilotKit/CopilotKit/blob/main/skills/a2ui-renderer/SKILL.md)，了解 CopilotKit v2 的 runtime、provider、主题和 catalog 相关约定。

## 1. 设置 AG-UI

从你已经在使用的 agent 框架出发，在 agent 与你的应用之间加上一条 AG-UI runtime 连接。这个 runtime 会把 agent 事件（包括 A2UI 消息）流式传输到客户端 surface。

使用 AG-UI CLI，搭配你想要的客户端与 agent 框架，来搭建一个 AG-UI 应用的脚手架：

```bash
npx create-ag-ui-app@latest
```

你也可以直接从受支持的框架模板开始：

```bash
npx create-ag-ui-app@latest --adk
npx create-ag-ui-app@latest --langgraph-py
npx create-ag-ui-app@latest --langgraph-js
```

Strands 目前还没有对应的脚手架参数——请封装一个已有的 Strands agent（参见下方的 Strands 面板）。

这里真正重要的是传输契约：你的应用接收 AG-UI 事件，并把 A2UI payload 路由给 A2UI renderer。部分脚手架路径在底层使用了 [CopilotKit 的 A2UI runtime](https://docs.copilotkit.ai/generative-ui/a2ui) 搭配 Next.js，但整体设置界面仍然以 AG-UI 为核心。

## 2. 设置你的 agent 或 harness

无论使用哪种框架，A2UI 的接入步骤都是一样的：把你的 agent 连接到 AG-UI、启用 A2UI payload，并在应用中渲染这些 payload。从你已经在使用的框架或 harness 开始。下面的代码片段来自对应的 AG-UI 集成，展示了 AG-UI 所封装的、各框架原生的 agent 形态。

=== "ADK"

    如果你的 agent 已经运行在 Google 的 Agent Development Kit 之上，就可以使用 ADK。AG-UI 的 ADK middleware 会把该 agent 暴露为一个 AG-UI 事件流：

    ```python
    from fastapi import FastAPI
    from ag_ui_adk import ADKAgent, AGUIToolset, add_adk_fastapi_endpoint
    from google.adk.agents import Agent

    my_agent = Agent(
        name="assistant",
        instruction="You are a helpful assistant.",
        tools=[
            AGUIToolset(),  # 添加由 AG-UI 客户端提供的 tools。
        ],
    )

    agent = ADKAgent(
        adk_agent=my_agent,
        app_name="my_app",
        user_id="user123",
    )

    app = FastAPI()
    add_adk_fastapi_endpoint(app, agent, path="/chat")
    ```

    参见 [AG-UI ADK middleware](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/adk-middleware/python)。

=== "LangGraph (Python)"

    如果你的 agent 工作流是一张由多个有状态节点组成的图，就可以使用 LangGraph。从你平常使用的 LangGraph agent 出发即可——A2UI 不需要在这张图上额外接线（tool wiring）：

    ```python
    from copilotkit import CopilotKitMiddleware
    from langchain.agents import create_agent
    from langchain_google_genai import ChatGoogleGenerativeAI

    gemini = ChatGoogleGenerativeAI(
        model="gemini-2.5-pro",
        thinking_budget=1024,
    )

    # 一个普通的 LangGraph agent —— 这张图上不需要额外的 A2UI tool 接线。CopilotKit
    # runtime 会转发你的前端 catalog 并注入 `generate_a2ui` tool；
    # 加入 CopilotKitMiddleware 即可获得 A2UI 能力。
    graph = create_agent(
        model=gemini,
        tools=[],
        middleware=[CopilotKitMiddleware()],
        system_prompt="You are a helpful assistant.",
    )
    ```

    LangGraph 的 A2UI tool 运行在 CopilotKit middleware 这一层，所以需要加入 `CopilotKitMiddleware` 才能获得 A2UI 能力。CopilotKit 的 runtime 会自动转发你的 catalog 并注入 `generate_a2ui`。示例中通过 LangChain 的 Google GenAI 集成使用了 Gemini。

    参见 [AG-UI LangGraph integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/python) 以及 [ChatGoogleGenerativeAI integration](https://docs.langchain.com/oss/python/integrations/chat/google_generative_ai)。

=== "LangGraph (FastAPI)"

    如果你需要把同一张 LangGraph 图跑在 FastAPI 应用之后，就使用 FastAPI 版本。agent 的形态完全相同——导出同一个 `graph`，并通过 AG-UI 的 LangGraph 端点来提供服务：

    ```python
    from copilotkit import CopilotKitMiddleware
    from langchain.agents import create_agent
    from langchain_google_genai import ChatGoogleGenerativeAI

    gemini = ChatGoogleGenerativeAI(
        model="gemini-2.5-pro",
        thinking_budget=1024,
    )

    graph = create_agent(
        model=gemini,
        tools=[],
        middleware=[CopilotKitMiddleware()],
        system_prompt="You are a helpful assistant.",
    )
    ```

    参见 [AG-UI LangGraph integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/python)。

=== "LangGraph (TypeScript)"

    如果你的 LangGraph agent 是用 TypeScript 编写的，就使用 TypeScript 版本。它的结构与 Python 版本的 agent 一致——一张普通的图加上 CopilotKit middleware：

    ```ts
    import { createAgent } from "langchain";
    import { ChatOpenAI } from "@langchain/openai";
    import { copilotkitMiddleware } from "@copilotkit/sdk-js/langgraph";

    export const graph = createAgent({
      model: new ChatOpenAI({ model: "gpt-4o" }),
      tools: [],
      middleware: [copilotkitMiddleware],
      systemPrompt: "You are a helpful assistant.",
    });
    ```

    参见 [AG-UI LangGraph TypeScript integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/typescript)。

=== "Strands (Python)"

    如果你的 agent 编排构建在 AWS Strands 之上，就使用 Strands。用 AG-UI 的 Strands 适配器封装一个普通的 Strands agent：

    ```python
    from strands import Agent
    from ag_ui_strands import StrandsAgent

    strands_agent = Agent(
        system_prompt="You are a helpful assistant.",
    )

    agent = StrandsAgent(
        agent=strands_agent,
        name="my-agent",
        description="A Strands agent exposed via AG-UI",
    )
    ```

    参见 [AG-UI AWS Strands integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/aws-strands/python)。

=== "Strands (TypeScript)"

    如果你的 Strands agent 是用 TypeScript 编写的，就使用 TypeScript 版本。AG-UI 的 Strands 适配器会把这个 Strands agent 封装成可供 AG-UI 客户端使用的形式：

    ```ts
    import { Agent } from "@strands-agents/sdk";
    import { StrandsAgent } from "@ag-ui/aws-strands";
    import { createStrandsApp } from "@ag-ui/aws-strands/server";

    const strandsAgent = new Agent({
      systemPrompt: "You are a helpful assistant.",
      tools: [],
    });

    const aguiAgent = new StrandsAgent({
      agent: strandsAgent,
      name: "MyAgent",
      description: "A Strands agent exposed via AG-UI",
    });

    const app = await createStrandsApp(aguiAgent, { path: "/invocations" });
    app.listen(8000);
    ```

    参见 [AG-UI AWS Strands integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/aws-strands/typescript)。

=== "Slack"

    如果用户体验承载在一个 Slack 应用中，就使用 Slack。把 Slack thread 路由到同一个 AG-UI agent 端点。同一个 AG-UI 事件流既可以提供给 Slack harness，也可以通过该 surface 的客户端 bridge 来渲染 A2UI。

    <video class="agui-demo-video" controls playsinline preload="metadata">
      <source src="https://cdn.copilotkit.ai/docs/a2ui/ag-ui-slack-demo.mp4" type="video/mp4">
      你的浏览器不支持 video 标签。
    </video>

    CopilotKit 的 Slack 适配器已经实现了这种模式：

    ```ts
    import { createBot } from "@copilotkit/bot";
    import {
      slack,
      SanitizingHttpAgent,
      defaultSlackTools,
      defaultSlackContext,
    } from "@copilotkit/bot-slack";

    const bot = createBot({
      adapters: [
        slack({
          botToken: process.env.SLACK_BOT_TOKEN!,
          appToken: process.env.SLACK_APP_TOKEN!,
        }),
      ],
      agent: (threadId) => {
        const agent = new SanitizingHttpAgent({
          url: process.env.AGENT_URL!,
        });
        agent.threadId = threadId;
        return agent;
      },
      tools: [...defaultSlackTools],
      context: [...defaultSlackContext],
    });

    bot.onMention(async ({ thread }) => {
      await thread.runAgent();
    });

    await bot.start();
    ```

以上这些代码片段建立起 AG-UI 服务端连接。Slack 通过它自己的 harness 和客户端 bridge，使用同样的 AG-UI/A2UI 契约。接下来的章节将在应用 surface 内部开启 A2UI 渲染、catalog 与组件定义。

## 3. 启用 A2UI

从你想要的开发者体验出发：定义 agent 可以看到的 catalog definitions，把每个 definition 映射到一个 renderer，创建 catalog，再把这个 catalog 传给 CopilotKit。前端的 catalog 配置就是启用 A2UI 的目标 surface。

{% raw %}

```tsx
import {CopilotKit, CopilotChat} from '@copilotkit/react-core/v2';
import {
  createCatalog,
  type CatalogDefinitions,
  type CatalogRenderers,
} from '@copilotkit/a2ui-renderer';
import {z} from 'zod';

// catalog definitions —— 向 agent 描述这些构成 UI 的基础组件
export const catalogDefinitions = {
  Card: {
    description: 'A titled card container.',
    props: z.object({title: z.string(), subtitle: z.string().optional()}),
  },
  PrimaryButton: {
    description: 'A styled primary button.',
    props: z.object({label: z.string(), action: z.any().optional()}),
  },
} satisfies CatalogDefinitions;

// catalog renderers —— 每个基础组件在 DOM 中如何渲染（本例中使用 React）
export const catalogRenderers = {
  Card: MyCard,
  PrimaryButton: MyPrimaryButton,
} satisfies CatalogRenderers<typeof catalogDefinitions>;

// definitions 加 renderers 共同构成一个 catalog 声明
const catalog = createCatalog(catalogDefinitions, catalogRenderers, {
  catalogId: 'my-catalog',
  includeBasicCatalog: true,
});

<CopilotKit runtimeUrl="/api/copilotkit" a2ui={{catalog}}>
  <CopilotChat />
</CopilotKit>;
```

{% endraw %}

把 catalog 传给 provider 会自动启用 A2UI 并注入 `generate_a2ui` tool，这样你的 agent 无需额外的 runtime 配置就能生成 surface（需要 CopilotKit ≥ 1.61.2）。你也可以选择关闭这个行为，或者在不使用 catalog 的情况下手动开启，只需直接配置 runtime：

```ts title="app/api/copilotkit/route.ts"
import {CopilotRuntime} from '@copilotkit/runtime';

const runtime = new CopilotRuntime({
  agents: {default: myAgent},
  a2ui: {injectA2UITool: true},
});
```

可以用 `a2ui: { injectA2UITool: true, agents: ["my-agent"] }` 将其限制到特定 agent。对于 agent 已经直接返回 `a2ui_operations` 的固定 schema 流程，写 `a2ui: true` 或 `a2ui: {}` 就足够了。

### 自定义组件（BYOC）

A2UI 内置了一个 catalog（Text、Image、Card 等），可以让你立刻得到一个可以工作的 surface。下面展开的 BYOC 流程展示了在真实应用中，如何把同样的 catalog 模式拆分到多个文件中：

1. **Definitions**：Zod schema 加上一段自然语言描述。这是 agent 在 system prompt 中会看到的内容。需要注意的是，对于 client-side function，客户端会在运行时读取当前 catalog definition 中的配置，来判断该 function 的执行边界（例如是否为 clientOnly）。
2. **Renderers**：带类型的 React 组件，每个 definition 对应一个。这是用户会看到的内容。
3. **Registration**：把 catalog 通过 provider 传进去，让 A2UI renderer 知道该如何绘制你的组件。

#### 1. 定义组件 schema

用 Zod 创建与平台无关的 definitions。`description` 字段会被注入 agent 的 prompt，让 LLM 知道什么时候该用到每个组件；schema 则用于校验 agent 发送的 props。

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

把每个 definition 映射到一个 React 组件。`createCatalog` 会以 definitions 的类型作为泛型参数，因此 renderer 收到的 props 会依据 Zod schema 做类型检查，`props.text` 这样的拼写错误会在编译期就被发现。

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

`catalogId` 是 agent 用来定位这个 catalog 的稳定句柄；`includeBasicCatalog: true` 会让内置组件与你自己的组件一起保持可用（省略它则只会渲染你自己的组件）。

#### 3. 把 catalog 传给 CopilotKit

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

现在，agent 会在内置组件之外看到你的自定义组件，并可以在它发出的任何 A2UI surface 中使用它们。

完整的 BYOC 参考（多个 catalog、主题 hook、进阶模式）请参见 CopilotKit 的 [Custom Components (BYOC) 章节](https://docs.copilotkit.ai/generative-ui/a2ui)。

## 4. 进阶用法

完整的 A2UI 集成 surface（自定义 catalog、细粒度控制、进阶模式）请参见 CopilotKit 的 [A2UI 文档](https://docs.copilotkit.ai/generative-ui/a2ui)。

## 接下来

- **[A2UI Composer](https://a2ui-composer.ag-ui.com/)**：可视化构建 widget。
- **[概念 › 传输层](../concepts/transports.md)**：A2UI 如何映射到 AG-UI。
- **[v0.9 规范](../specification/v0.9-a2ui.md)**：底层协议。
