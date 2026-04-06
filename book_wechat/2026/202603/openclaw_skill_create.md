技能（Skill）是一组指令——以一个简单的文件夹形式打包——用于教会 Claude 如何处理特定的任务或工作流程。技能是根据你的具体需求对 Claude 进行定制的最强大方式之一。与其在每一次对话中反复解释你的偏好、流程和领域知识，不如通过技能一次性教会 Claude，并在之后的每次使用中持续受益。

当你拥有可重复的工作流程时，技能尤其强大。例如：根据规格生成前端设计、按照统一的方法进行研究、创建符合团队风格指南的文档，或编排多步骤的流程。技能还能很好地与 Claude 的内置能力协同工作，例如代码执行和文档创建。对于那些正在构建 MCP 集成的开发者来说，技能还增加了一层强大的能力，可以将原始的工具访问能力转化为可靠且经过优化的工作流程。

本指南将介绍构建高效技能所需了解的一切——从规划和结构，到测试与分发。无论你是为自己、为团队，还是为社区开发技能，都可以在这里找到实用的模式和真实世界的示例。

**你将学到：**

- 技能结构的技术要求与最佳实践
- 独立技能（standalone skills）以及结合 MCP 的增强工作流模式
- 在不同使用场景中被证明行之有效的设计模式
- 如何对技能进行测试、迭代和分发

**适合人群：**

- 希望 Claude 能够稳定地按照特定工作流程执行任务的开发者
- 希望 Claude 遵循固定流程完成任务的高级用户（Power Users）
- 希望在整个组织内部标准化 Claude 使用方式的团队

**阅读本指南的两种路径**

如果你是在**构建独立技能（standalone skills）**，可以重点关注 **Fundamentals（基础）、Planning and Design（规划与设计）以及第 1–2 类内容**。
如果你是在**为 MCP 集成增强功能**，那么 **“Skills + MCP” 部分以及第 3 类内容**更适合你。

这两条路径遵循相同的技术要求，但你可以根据自己的使用场景选择最相关的部分进行学习。

**你将从本指南中获得什么：**
在阅读完成后，你将能够在一次完整的实践中构建一个可用的技能。使用 **skill-creator** 创建并测试你的第一个可运行技能，通常只需要 **约 15–30 分钟**。

让我们开始吧。

**什么是技能（Skill）？**

技能是一个文件夹，其中包含：

- **SKILL.md（必需）**：使用 Markdown 编写的说明文件，并带有 YAML frontmatter（前置信息）
- **scripts/（可选）**：可执行代码，例如 Python、Bash 等
- **references/（可选）**：按需加载的参考文档
- **assets/（可选）**：在输出中使用的模板、字体、图标等资源

**核心设计原则**

### 渐进式披露（Progressive Disclosure）

技能采用一个**三级结构系统**：

- **第一层（YAML frontmatter）**：始终加载到 Claude 的系统提示（system prompt）中。
  这一层只提供足够的信息，让 Claude 知道在什么情况下应该使用该技能，而不会把全部内容都加载到上下文中。
- **第二层（SKILL.md 正文）**：当 Claude 认为该技能与当前任务相关时才会加载。
  这里包含完整的指令和使用指导。
- **第三层（关联文件）**：位于技能目录中的其他文件，Claude 只会在需要时主动查看和发现。

这种**渐进式披露**的方式能够在保持专业能力的同时，**最大限度减少 token 的使用**。

### 可组合性（Composability）

Claude 可以**同时加载多个技能**。因此，你编写的技能应当能够与其他技能良好协作，而不是假设它是系统中唯一可用的能力。

### 可移植性（Portability）

技能在 **Claude.ai、Claude Code 以及 API** 中的工作方式是完全一致的。
只要运行环境支持技能所需的依赖，你**只需创建一次技能**，它就可以在所有平台上使用，而无需修改。

------

## 对于 MCP 构建者：Skills + Connectors

💡 如果你只是**构建独立技能而没有使用 MCP**，可以先跳到 **Planning and Design（规划与设计）** 部分——之后随时都可以再回到这里。

如果你已经拥有一个**可运行的 MCP 服务器**，那么最困难的部分其实已经完成了。
**技能（Skills）是在其之上的知识层**——它用于记录你已经掌握的工作流程和最佳实践，让 Claude 能够稳定、持续地应用这些经验。

------

## 厨房类比

- **MCP** 提供的是一个**专业厨房**：可以访问各种工具、食材和设备。
- **Skills** 提供的是**食谱**：一步一步说明如何创造出有价值的成果。

两者结合在一起，使用户能够完成复杂任务，而无需自己弄清每一个步骤。

**它们是如何协同工作的：**

| **MCP（连接能力）**                                  | **Skills（知识能力）**           |
| ---------------------------------------------------- | -------------------------------- |
| 将 Claude 连接到你的服务（Notion、Asana、Linear 等） | 教会 Claude 如何高效使用你的服务 |
| 提供实时数据访问和工具调用                           | 捕获工作流程和最佳实践           |
| **Claude 可以做什么**                                | **Claude 应该如何去做**          |

### 为什么这对你的 MCP 用户很重要

**没有技能时：**

- 用户连接了你的 MCP，但不知道下一步该做什么
- 支持工单经常问：“如何使用你的集成完成 X？”
- 每次对话都要从头开始
- 由于用户每次的提示不同，结果不一致
- 当问题出在工作流程指导上时，用户却会责怪你的连接器

**有技能时：**

- 预构建的工作流程会在需要时自动激活
- 工具使用一致且可靠
- 每次交互中都嵌入最佳实践
- 降低集成的学习成本

### 规划与设计（Planning and Design）

#### 从使用场景开始

在编写任何代码之前，先确定 **2–3 个具体的技能使用场景（use cases）**，你的技能应当能够实现这些场景。

------

#### 优秀的使用场景定义示例

**Use Case（使用场景）：** 项目冲刺计划（Project Sprint Planning）
**Trigger（触发条件）：** 用户说 “help me plan this sprint” 或 “create sprint tasks”

**步骤（Steps）：**

1. 从 Linear（通过 MCP）获取当前项目状态
2. 分析团队的工作速度和容量
3. 提出任务优先级建议
4. 在 Linear 中创建带有正确标签和预估时间的任务

**结果（Result）：** 完整规划的冲刺，所有任务已创建

------

#### 自问自答：

- 用户希望完成什么目标？
- 这个目标需要哪些多步骤的工作流程？
- 需要用到哪些工具（内置工具还是 MCP）？
- 应该嵌入哪些领域知识或最佳实践？

### 常见技能使用场景类别

在 **Anthropic**，我们观察到三类常见的使用场景：

------

**类别 1：文档与资源创建（Document & Asset Creation）**

**用途（Used for）：**
用于创建一致且高质量的输出，包括文档、演示文稿、应用程序、设计、代码等。

**真实示例（Real example）：**
`frontend-design` 技能（同时还有 docx、pptx、xlsx 和 ppt 相关技能）

> “创建具有高设计质量的独特生产级前端界面。在构建网页组件、页面、产物、海报或应用程序时使用。”

**关键技术（Key techniques）：**

- 嵌入风格指南和品牌标准
- 使用模板结构确保输出一致
- 最终确认前的质量检查清单
- 不需要外部工具——使用 Claude 的内置能力

### 类别 2：工作流自动化（Workflow Automation）

**用途（Used for）：**
适用于受益于一致方法的多步骤流程，包括跨多个 MCP 服务器的协调。

**真实示例（Real example）：**
`skill-creator` 技能

> “用于创建新技能的交互式指南。引导用户完成使用场景定义、frontmatter 生成、指令编写以及验证。”

**关键技术（Key techniques）：**

- 带有验证关卡的逐步工作流程
- 常用结构的模板
- 内置的审查与改进建议
- 迭代优化循环

------

### 类别 3：MCP 增强（MCP Enhancement）

**用途（Used for）：**
提供工作流程指导，以增强 MCP 服务器提供的工具访问能力。

**真实示例（Real example）：**
`sentry-code-review` 技能（来自 Sentry）

> “使用 Sentry 的错误监控数据，通过 MCP 服务器自动分析并修复 GitHub Pull Request 中检测到的错误。”

**关键技术（Key techniques）：**

- 按顺序协调多个 MCP 调用
- 嵌入领域专业知识
- 提供用户本应手动指定的上下文
- 对常见 MCP 问题进行错误处理

### 定义成功标准（Define Success Criteria）

**如何判断你的技能是否有效？**

这些标准是**理想目标**——大致的参考指标，而非精确阈值。目标应当严格，但也要接受评估中会有一定的**主观感受（vibes-based assessment）**。我们正在积极开发更完善的测量指南和工具。

------

#### 定量指标（Quantitative Metrics）

- **技能在 90% 的相关查询中触发**
  - **测量方法**：运行 10–20 条应触发技能的测试查询，记录技能自动加载的次数与需要手动调用的次数。
- **在 X 次工具调用内完成工作流**
  - **测量方法**：对比同一任务在启用技能和未启用技能下的表现。统计工具调用次数及总消耗的 token 数量。
- **每个工作流中 API 调用失败次数为 0**
  - **测量方法**：在测试运行期间监控 MCP 服务器日志，记录重试次数和错误码。

------

#### 定性指标（Qualitative Metrics）

- **用户无需提示 Claude 下一步操作**
  - **评估方法**：测试时记录需要重定向或澄清的频率，并向 Beta 用户收集反馈。
- **工作流无需用户修正即可完成**
  - **评估方法**：运行相同请求 3–5 次，比较输出的结构一致性和质量。
- **跨会话结果一致**
  - **评估方法**：新用户是否能够在首次尝试时，依靠最少指导就完成任务？

### 技术要求（Technical Requirements）

#### 文件结构（File Structure）

```
your-skill-name/
├── SKILL.md         # 必需 - 技能主文件
├── scripts/         # 可选 - 可执行代码
│   ├── process_data.py   # 示例
│   └── validate.sh       # 示例
├── references/      # 可选 - 文档资料
│   ├── api-guide.md      # 示例
│   └── examples/         # 示例
└── assets/          # 可选 - 模板等资源
    └── report-template.md # 示例
```

------

#### 关键规则（Critical Rules）

**SKILL.md 文件命名：**

- 必须完全命名为 `SKILL.md`（区分大小写）
- 不允许任何变体（如 `SKILL.MD`、`skill.md` 等）

**技能文件夹命名：**

- 使用 **kebab-case**（短横线分隔）：`notion-project-setup` ✅
- 不允许空格：`Notion Project Setup` ❌
- 不允许下划线：`notion_project_setup` ❌
- 不允许大写字母组合：`NotionProjectSetup` ❌

**禁止包含 README.md：**

- 不要在技能文件夹中包含 `README.md`
- 所有文档应放在 `SKILL.md` 或 `references/` 中
- 注意：如果通过 GitHub 发布，仍然需要在仓库根目录放置一个 README 给人类用户查看 —— 详见“分发与共享（Distribution and Sharing）”。

### YAML frontmatter：最重要的部分

YAML frontmatter 决定了 Claude 是否会加载你的技能，因此务必正确配置。

------

#### 最小必需格式（Minimal Required Format）

```yaml
---
name: your-skill-name
description: What it does. Use when user asks to [specific phrases].
---
```

这就是启动技能所需的全部内容。

------

#### 字段要求（Field Requirements）

**name（必需）**

- 仅使用 **kebab-case**
- 不允许空格或大写字母
- 应与技能文件夹名称一致

**description（必需）**

- 必须同时包含：
  - 技能的功能
  - 使用场景（触发条件）
- 不超过 1024 个字符
- 不允许 XML 标签（`<` 或 `>`）
- 包含用户可能使用的具体任务描述
- 如相关，提及文件类型

**license（可选）**

- 用于技能开源时指定许可证
- 常见值：MIT、Apache-2.0

**compatibility（可选）**

- 1–500 个字符
- 描述环境要求，例如目标产品、所需系统包、网络访问需求等

**metadata（可选）**

- 可包含任意自定义键值对
- 建议包含：author（作者）、version（版本）、mcp-server
- 示例：

```yaml
metadata:
  author: ProjectHub
  version: 1.0.0
  mcp-server: projecthub
```

------

#### 安全限制（Security Restrictions）

**frontmatter 中禁止内容：**

- XML 尖括号（`<` 或 `>`）
- 名称中包含 `"claude"` 或 `"anthropic"` 的技能（保留关键字）

**原因：**
frontmatter 会出现在 Claude 的系统提示（system prompt）中，恶意内容可能注入指令。

### 编写高效技能（Writing Effective Skills）

------

#### 描述字段（The Description Field）

根据 **Anthropic** 的工程博客：

> “这些元数据（metadata）……提供了足够的信息，让 Claude 知道何时使用每个技能，而无需将全部内容加载到上下文中。”

这就是**渐进式披露的第一层**。

**结构：**
`[技能功能] + [使用时机] + [关键能力]`

**优秀描述示例（Good Descriptions）：**

- **具体且可操作**

```yaml
description: Analyzes Figma design files and generates developer handoff documentation. 
Use when user uploads .fig files, asks for "design specs", "component documentation", or "design-to-code handoff".
```

- **包含触发词**

```yaml
description: Manages Linear project workflows including sprint planning, task creation, and status tracking. 
Use when user mentions "sprint", "Linear tasks", "project planning", or asks to "create tickets".
```

- **清晰价值主张**

```yaml
description: End-to-end customer onboarding workflow for PayFlow. Handles account creation, payment setup, and subscription management. 
Use when user says "onboard new customer", "set up subscription", or "create PayFlow account".
```

**不良描述示例（Bad Descriptions）：**

- **过于模糊**

```yaml
description: Helps with projects.
```

- **缺少触发条件**

```yaml
description: Creates sophisticated multi-page documentation systems.
```

- **过于技术化，没有用户触发词**

```yaml
description: Implements the Project entity model with hierarchical relationships.
```

------

#### 编写主要指令（Writing the Main Instructions）

在 frontmatter 后，用 **Markdown** 编写实际操作指令。

**推荐结构（Recommended Structure）：**

适配此模板到你的技能，将方括号部分替换为具体内容：

~~~yaml
---
name: your-skill
description: [...]
---
# Your Skill Name
# Instructions

### Step 1: [First Major Step]
清晰说明该步骤会发生什么。  

示例：
```bash
python scripts/fetch_data.py --project-id PROJECT_ID
~~~

**期望输出：** [描述成功结果]

(根据需要添加更多步骤)

### Examples（示例）

**Example 1: [常见场景]**
用户说: "Set up a new marketing campaign"
操作：

1. 通过 MCP 获取已有活动
2. 使用提供的参数创建新活动
   结果：活动创建完成，并提供确认链接

(根据需要添加更多示例)

### Troubleshooting（排错）

**Error:** [常见错误信息]
**Cause:** [发生原因]
**Solution:** [解决方法]

(根据需要添加更多错误案例)

```
---

#### 指令编写最佳实践（Best Practices for Instructions）  

1. **具体且可操作（Be Specific and Actionable）**  
✅ **好例子**：  
```bash
Run `python scripts/validate.py --input {filename}` to check data format.
```

常见问题：

- 缺少必填字段（请补充到 CSV 中）
- 日期格式无效（请使用 YYYY-MM-DD）

❌ **坏例子**：

```text
Validate the data before proceeding.
```

1. **包含错误处理（Include Error Handling）**
   **常见问题**

- **MCP Connection Failed**
  如果看到 "Connection refused":
  1. 检查 MCP 服务器是否运行：Settings > Extensions
  2. 确认 API Key 是否有效
  3. 尝试重新连接：Settings > Extensions > [Your Service] > Reconnect

1. **清晰引用打包资源（Reference Bundled Resources Clearly）**
   在编写查询前，可参考 `references/api-patterns.md`：

- 限流指南（Rate limiting guidance）
- 分页模式（Pagination patterns）
- 错误码及处理（Error codes and handling）

1. **使用渐进式披露（Use Progressive Disclosure）**
   保持 `SKILL.md` 聚焦核心指令，将详细文档移至 `references/` 并链接引用。
   （参见核心设计原则了解三级系统如何工作）