# OpenClaw 运行期 AGENTS.md 关系分析报告

## 1. 核心概念

在 OpenClaw 运行期，系统区分了两种主要的 Agent 环境：**主 Agent (Main Agent)** 和 **专业/子 Agent (Specialized/Sub-Agent)**。虽然它们都使用名为 `AGENTS.md` 的文件，但其加载逻辑、注入范围和预期用途完全不同。

## 2. 职责与加载模式对比

OpenClaw 根据 **Session Key（会话键）** 的类型来决定一个 Agent 到底能看到多少引导文件。这里需要严格区分“作为独立人格运行的专业 Agent”与“作为任务执行者运行的子 Agent”。

| 特性                 | 配置态专业 Agent (Scoped Agent)                        | 运行时子智能体 (Sub-Agent)                  |
| :------------------- | :----------------------------------------------------- | :------------------------------------------ |
| **触发方式**         | 在 `agents.list` 中定义，作为频道/私信的主回复者。     | 由主 Agent 调用 `sessions_spawn` 动态生成。 |
| **Session Key 特征** | 通常为 `agent:<agentId>:<mainKey>`                     | 包含 `:subagent:<uuid>` 标记                |
| **加载模式**         | **Full Mode** (完整模式)                               | **Minimal Mode** (极简模式)                 |
| **注入文件**         | `AGENTS.md`, `SOUL.md`, `USER.md`, `TOOLS.md` 等全套。 | **仅** `AGENTS.md` 和 `TOOLS.md`。          |
| **适用文档**         | 符合 `docs/concepts/multi-agent.md` 的描述。           | 符合 `src/agents/system-prompt.ts` 的逻辑。 |

## 3. 运行期注入机制

### 3.1 基于 Session 类型的动态过滤 (Dynamic Filtering)

系统通过检测当前会话的性质来决定上下文边界。核心逻辑位于 `src/agents/workspace.ts` 的 `filterBootstrapFilesForSession` 函数中。

- **逻辑判定**：

  ```typescript
  // src/agents/workspace.ts
  const MINIMAL_BOOTSTRAP_ALLOWLIST = new Set(["AGENTS.md", "TOOLS.md"]);

  export function filterBootstrapFilesForSession(files, sessionKey) {
    // 判定是否为 Sub-Agent 或 Cron 任务
    if (!sessionKey || (!isSubagentSessionKey(sessionKey) && !isCronSessionKey(sessionKey))) {
      return files; // 非子智能体 Session：加载所有文件
    }
    // 子智能体 Session：仅保留白名单内的 AGENTS.md 和 TOOLS.md
    return files.filter((file) => MINIMAL_BOOTSTRAP_ALLOWLIST.has(file.name));
  }
  ```

### 3.2 为什么存在这种区分？

1.  **人格完整性**：当你将一个 WhatsApp 号码绑定到名为 `work` 的专业 Agent 时，你希望它是一个有“灵魂”的助手。因此它必须读取 `SOUL.md` 和 `USER.md` 来理解你是谁以及如何交流。
2.  **任务抽象与隐私**：当 `main` Agent 派出一个子 Agent 去分析一段代码时，这个子 Agent 只需要知道“任务书”（`AGENTS.md`）和“工具书”（`TOOLS.md`）。它不需要知道主人的私人偏好，这不仅节省了 Token，也提供了隐私屏障。

## 4. 协作流程总结

1.  **身份识别**：系统根据绑定路由（Bindings）或任务派发指令确定当前运行的 `agentId` 和 `sessionKey`。
2.  **上下文加载**：
    - 若是**独立角色**，从其专属工作区搜集全部灵魂文件。
    - 若是**受托专家 (Sub-Agent)**，仅提取该工作区的功能性指令。
3.  **Prompt 组装**：通过 `buildAgentSystemPrompt` 将解析后的 Markdown 内容注入到 System Prompt 的 `# Project Context` 区域。

## 5. 结论

官方文档中关于专业 Agent 拥有独立 `SOUL.md` 和 `USER.md` 的描述是针对**配置态的独立角色**而言的。而对于**运行中产生的职能型子 Agent**，OpenClaw 采取了更为精简和隔离的加载策略。这种层次化的注入机制实现了“人格隔离”与“任务专注”的平衡。
