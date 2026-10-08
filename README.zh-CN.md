# Plan, Build, Review

[English](README.md) | [简体中文](README.zh-CN.md)

一个基于角色分工的工作流，通过独立审查、明确批准和任务级检查点，可靠地
交付边界明确的工作（从 feature 到 milestone）—— 同时支持 [Codex](https://developers.openai.com/codex/) 和
[Claude Code](https://code.claude.com/)。

## 为什么使用它

- 规划者基于证据制定最小而决策完整的计划；独立审查者必须同时通过前提和规格审查。
- 未获得用户明确批准前，不会开始实现。
- 工作按照构建、验证、审查和检查点提交的顺序逐项推进。
- 每轮审查都使用新的审查者和按风险选择的 charter，而不是通用检查清单。
- 验证方式取决于产物的真实消费者，本地与远程 gate 必须承担不同目的。
- 仅当存在跨任务风险时，独立集成审查才检查整体行为和跨任务契约。
- 发布必须获得明确批准，并且工作流永远不会自动合并。

```text
预检 → 规划 → 审查 → 批准 → 构建任务 → 审查任务 → 创建检查点
     → 重复 → 按需集成审查 → 发布批准
```

每个角色只接收简短的决策摘要，以及完成当前任务所需的计划片段和证据。工作流不会在每次交接时默认传递完整计划。

## 它避免什么，以及何时最有价值

这个工作流不会把“看起来合理的计划”当作可以立即实现的充分理由。它会暴露未经挑战的产品前提、范围膨胀、没有依据的兼容性或安全工作、不完整的合并单元，以及意外依赖开发环境的测试。

当一个 feature 或 milestone 跨越多个组件、需要在实现前获得 owner 决策，或必须留下可信的审查和发布线索时，它最有价值。它的价值不在于更多 agent 或更多仪式，而在于让每个角色承担不同的 challenge、获得小而权威的上下文，并有清晰的升级路径，同时由主 agent 维护整体结果。

## 角色

| 角色 | 职责 | 权限 | Codex 模型 · 推理强度 | Claude 模型 · 推理强度 |
| --- | --- | --- | --- | --- |
| `pbr-explorer` | 范围明确的调查 | 只读 | Terra · medium | Haiku |
| `pbr-planner` | 制定最小而决策完整的计划 | 只读 | Sol · xhigh | Opus · xhigh |
| `pbr-builder` | 实现一项已批准的任务 | 工作区写入 | Sol · medium | Sonnet · high |
| `pbr-reviewer` | 审查前提、计划、任务和按风险进行的集成 | 只读 | Sol · high | Opus · high |

角色定义已随仓库提供，可以根据任一运行环境进行调整。Haiku 不支持可配置的
推理强度等级，因此 `pbr-explorer` 对应的 Claude 一栏留空。

## 审查模型

计划审查包含两个必须独立通过的判断。前提审查会挑战项目目的、目标用户、架构、部署模式、兼容性立场、安全立场和相对 roadmap 的变化。规格审查检查选定计划是否完整且可执行。只有两者均通过，计划才可通过。计划在计算 hash 前先通过机械检查；可证明的纯字节修正只需聚焦复审和重新批准的新 hash，而语义修改返回完整审查。

对于实现工作，主 agent 选择最小且足够的审查 charter，例如实现行为、最终公共表面或跨任务集成。涉及异步、并发、轮询或其他时间敏感型协调行为的工作，必须说明时序不变量和消费者可观察行为；实现和审查再据此使用有界的场景矩阵。一个任务必须是可独立合并和验证的语义完整单元。工作流不按文件类型或代码层机械拆分任务。每个 verdict 还会说明采用的 charter、实际审查范围和覆盖限制。
当不存在独立的集成风险时，一次完整的任务审查也可以同时作为最终整体审查。一个 feature 通常在同一个 PR 中包含实现、测试、当前公开描述和 release-ready 文档；发布本身不需要 closeout PR。

## 验证模型

工作流会先判断产物由谁消费，再选择检查方式。面向人的产物根据 owner 意图
和本次变更的具体主张进行审查，不受通用结构或语义 policy 约束。仅使用与
当前变更相关的机械检查。机器消费的产物则使用与风险相称的结构和行为检查。

本地、PR、main、release 和 scheduled gate 分别服务于不同目的。计划必须
解释实质相同的覆盖为什么必要，而不是默认重复昂贵检查。候选内容和相关输入
未变化时，可以继续复用已有证据；审查者的独立性不要求重复 builder 的完整
测试套件。纯源码 release 验证 tag、release metadata 和 archive；只有存在
tag-specific 行为或产物时才增加 tag CI。

## 安装

### Codex

使用技能内部名称，将仓库克隆到 Codex 技能目录：

```sh
git clone https://github.com/jameswei/plan-build-review.git \
  ~/.codex/skills/plan-build-review
```

安装四个自定义代理定义：

```sh
mkdir -p ~/.codex/agents
cp ~/.codex/skills/plan-build-review/assets/agents/*.toml ~/.codex/agents/
```

如果已经存在同名代理，请在复制前检查现有文件。`pbr-` 前缀可避免 Codex
将代理安装到扁平全局目录时发生冲突。安装完成后启动新的 Codex 会话，让 Codex
发现这个技能和它的代理。（Codex 也支持项目级的
`.agents/skills` 目录，作为上面用户级 `~/.codex/skills/` 路径之外的
另一种安装位置。）

### Claude Code

需要 Claude Code 2.1.287 或更高版本，这样每个角色才会按其代理定义中设置的推理强度运行。

将此仓库添加为插件市场，然后安装插件：

```text
/plugin marketplace add jameswei/plan-build-review
/plugin install plan-build-review@plan-build-review
/reload-plugins
```

这一步会同时安装技能和全部四个子代理定义，无需手动复制任何文件。安装后，
技能和角色会带上插件名作为命名空间前缀，例如 `pbr-planner` 会变成
`plan-build-review:pbr-planner`。

## 使用

在 Codex 中，明确调用这个技能：

```text
使用 $plan-build-review 为此仓库交付 export-history feature。
```

在 Claude Code 中：

```text
使用 /plan-build-review:plan-build-review 为此仓库交付 export-history feature。
```

工作流会在计划通过审查后暂停，等待你的批准，然后才会创建分支或修改实现
文件。推送分支或创建草稿拉取请求前，它还会再次请求批准。

它适合重视范围、正确性和审查可追溯性的 feature 或 milestone 工作。普通编码或文档任务
不会隐式触发这个技能。

## 兼容性

这个软件包同时支持 Codex 和 Claude Code，不面向其他运行环境。但两边的
安装体验并不完全对等：Claude Code 的插件市场可以一步安装技能和全部四个
子代理，无需手动复制任何文件；Codex 目前没有将自定义代理与技能捆绑安装
的机制，因此它的自定义代理仍需要上文所述的一次性手动复制。

只读权限的实现方式也不对等。Codex 的 `sandbox_mode: read-only` 会在
操作系统层面阻止写入文件系统，即使是在 shell 命令内部也是如此。Claude
Code 没有对应的执行沙箱，因此 `pbr-explorer`、`pbr-planner` 和 `pbr-reviewer` 是通过在
`tools` 允许列表中完全不包含 Bash 和
MCP 工具来实现只读限制的——它们并非被沙箱隔离，而是根本没有被授予可能
写入的工具。

## 许可证

[MIT](LICENSE)
