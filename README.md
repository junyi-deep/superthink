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

思想来源是需求访谈、系统调试和质量审查，融合了用户提出的 `interview-me`、`systematic-debugging`、`code-review-and-quality` 三类方法；正文独立编写，没有复制或引用其文件。

无需 Superpowers、Addy Osmani agent-skills、grill-me、其他 Skill、外部 references、MCP、插件框架或附带脚本/CLI。Claude Code 可使用目标项目已有的测试和命令，但没有本 Skill 指定的外部工具依赖。

这是推理纪律，不是正确性保证。真实效果仍取决于模型、代码库信息和可执行验证；不能把静态格式检查当作实际行为评测。
