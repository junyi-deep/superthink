# superthink

为 Claude Code 增加一层轻量工程判断：先弄清问题，再依据证据定位根因，最后对照需求验证实现。按任务风险选择最低必要深度，不增加固定工作流。

## 目录

```text
.
├── README.md
└── superthink/
    └── SKILL.md
```

安装包只有 `superthink/`，其中的 `SKILL.md` 完全自包含。README 是仓库说明，不是 Skill 的运行依赖。

## 安装

下载此仓库，或使用 Git 获取：

```sh
git clone https://github.com/junyi-deep/superthink.git
cd superthink
```

在仓库根目录执行以下命令，安装到个人 Claude Code，适用于所有本地项目：

```sh
mkdir -p "$HOME/.claude/skills" && cp -R superthink "$HOME/.claude/skills/"
```

目标文件为 `~/.claude/skills/superthink/SKILL.md`；已有同名 Skill 时，先检查并备份其内容再复制。也可手动复制，无需 Git 或安装脚本。

仅供某项目使用时，把同一个 `superthink/` 文件夹复制到该项目的 `.claude/skills/`。安装后打开一个新的 Claude Code 会话，输入 `/superthink` 检查是否可用。

目录位置、YAML frontmatter 与自动发现方式遵循 [Claude Code 官方 Skills 文档](https://code.claude.com/docs/en/skills)。Skill 仅声明 `name` 和 `description`，保留默认自动调用行为。

## 使用与自动路由

像平时一样描述任务，不需要选择模式。Claude 根据 description 判断是否加载，再根据上下文选择内部认知状态：

| 当前需要 | 自动选择 |
| --- | --- |
| 新需求存在理解风险、重要取舍或较大影响范围 | UNDERSTAND：先读代码，仅澄清会改变方案的问题 |
| 当前行为异常、失败或回归 | DEBUG：证据 → 单个假设 → 最小实验 → 根因 → 修复 |
| 检查已有实现，或实现/修复准备交付 | VERIFY：需求 → 实现 → 验证证据 |
| 普通解释、机械修改、明确且低风险的任务 | 正常处理，无需完整协议 |

模式自动衔接，例如 `UNDERSTAND → 实现 → VERIFY` 或 `DEBUG → 修复 → VERIFY`；执行中出现失败时转入 DEBUG。只要求计划就交付计划，只要求 review 就报告问题。不会要求用户学习三个子命令，也不会反复宣布进入了哪个模式。

需要显式使用时，可以输入 `/superthink 检查当前实现是否遗漏错误路径`。自动触发由 Claude 根据描述和上下文决定，不是确定性的关键词匹配；这里的示例是设计预期，并非触发率保证。

## 5 个应触发的示例

| 请求 | 预期行为 |
| --- | --- |
| 给订单接口加 Redis 缓存。 | UNDERSTAND：先读调用链与性能证据，区分数据库负载和第三方报价延迟；仅询问影响设计的一致性等关键约束。 |
| 用户偶尔支付成功，但订单还是 pending，帮我修。 | DEBUG：追踪 payment event、webhook、幂等、事务与状态更新，用证据检验事件顺序；不能直接加 retry。修复后 VERIFY。 |
| 升级依赖之后 CI 的集成测试失败了，帮我定位。 | DEBUG：读取失败断言、日志与升级 diff，通过最小实验区分依赖行为变化和环境因素。 |
| 功能实现好了，检查一下有没有遗漏。 | VERIFY：重读需求和 diff，检查关键错误路径；例如证实 timeout 未回滚 optimistic state 时报告 IMPORTANT。 |
| 计划把同步导出改成异步任务，需要兼容现有客户端。 | UNDERSTAND：读取现有 API contract，澄清交付行为和兼容边界，给出具体计划；不擅自开始实现。 |

## 5 个不应自动触发的示例

以下均假定没有额外异常、隐含风险或审查要求：

1. “解释一下这个函数做什么。”——普通代码说明。
2. “把按钮文案从‘提交’改成‘保存’。”——明确的低风险文案修改。
3. “修正文档里 `recieve` 的拼写。”——机械 typo 修正。
4. “把这段 JSON 缩进为两个空格，不改变字段和值。”——纯格式处理。
5. “列出项目里有哪些测试文件。”——文件检索，不涉及正确性评审。

## 设计边界

- 先问代码库，再问用户；只有错误假设成本显著高于提问成本才澄清。
- 低风险直接做；有不确定性时快速检查；复杂或高风险时逐级加深。
- DEBUG 不用猜测式补丁；诊断实验与临时缓解不能冒充根因修复。
- VERIFY 对照原始需求，优先报告实质问题，区分 Verified 与 Not verified。
- 信息充分就停止；证据缺失时说明阻塞与下一步，不无限分析。
- 尊重“直接做”和既有授权，不增设例行审批；review 不自动变成修改。
- 不重建 todo、规划界面、git、shell、测试或 subagent 框架。

Skill 的说明、规则、表格和 description 均使用中文，仅保留标题、模式名、严重性、验证状态及必要技术标识。

## 原文比对与融合记录

本次直接阅读以下上游原文，并固定比对版本；链接仅用于追溯设计来源，运行 Skill 时无需访问或安装它们。这里的 `interview-me` 指 Addy Osmani 的版本，不是其他同名仓库。

| 原始 Skill 与版本 | 补充到 superthink 的内容 |
| --- | --- |
| [interview-me](https://github.com/addyosmani/agent-skills/blob/6ca0cd7db39b41b1c37e26d335c507ee92382c6d/skills/interview-me/SKILL.md) | 明确受益者、使用场景与为什么现在做；将抽象目标转为可检验标准；必要时附当前理解帮助纠正；识别矛盾并重述意图；非交互环境和访谈停滞的处理。 |
| [systematic-debugging](https://github.com/obra/superpowers/blob/b36e0829c6d0140e93cfef2ca599b1b07d4a7797/skills/systematic-debugging/SKILL.md) | 独立的正常/异常差异分析；配置、环境、依赖跨组件传播；修复失败后回到证据；连续失败触发架构复查；外部故障处理与临时缓解的区分。 |
| [code-review-and-quality](https://github.com/addyosmani/agent-skills/blob/6ca0cd7db39b41b1c37e26d335c507ee92382c6d/skills/code-review-and-quality/SKILL.md) | 可读性与简洁性；实际减少复杂度；类型边界及功能归属；具体结构改进建议；无用代码判断；依赖和锁文件审查；检查验证记录是否覆盖最终代码。 |

同时阅读 systematic-debugging 的 [条件等待](https://github.com/obra/superpowers/blob/b36e0829c6d0140e93cfef2ca599b1b07d4a7797/skills/systematic-debugging/condition-based-waiting.md) 与 [分层校验](https://github.com/obra/superpowers/blob/b36e0829c6d0140e93cfef2ca599b1b07d4a7797/skills/systematic-debugging/defense-in-depth.md) 说明，将适用原则直接融合进正文：等待真实状态并限制超时，仅在具有独立职责的边界补必要校验。

为保持轻量与独立，以下原版机制没有照搬：

- 不采用 95% 信心门槛、不反复要求明确确认、不拒绝用户委托自行判断。
- 不将连续三次失败视为架构错误的证明，而作为停止堆叠补丁、升级调查的信号。
- 不要求每层重复校验，不强制为每次修复编写测试脚本或安装其他 Skill。
- 不强制多模型审查、固定行数限制、所有任务全套检查或例行删除确认。
- 不引入外部工作流、参考文件或工具依赖；正文以中文重新组织，保留统一路由、风险分级和停止条件。

无需 Superpowers、Addy Osmani agent-skills、grill-me、其他 Skill、外部 references、MCP、插件框架或附带脚本/CLI。Claude Code 可使用目标项目已有的测试和命令，但没有本 Skill 指定的外部工具依赖。

这是推理纪律，不是正确性保证。真实效果仍取决于模型、代码库信息和可执行验证；不能把静态格式检查当作实际行为评测。
