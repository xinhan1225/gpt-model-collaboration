# GPT 多模型协作

一个用于 Codex 的子代理路由技能：根据任务复杂度，决定由当前聊天主模型独立完成，还是把适合拆分的工作交给 Sol、Luna 与 Astra，并由主模型整合、验证和交付。

它不能更换当前聊天的主模型，也不修改模型配置。安装技能不代表授权调用所有模型，更不保证每次任务都自动触发；可用模型、思考强度和并发数取决于实际工具与账户权限。

## 模型分工

| 任务 | 建议安排 |
| --- | --- |
| 简单问答、润色、小改动、依赖紧密的短任务 | 主模型独立完成 |
| 多步骤执行、代码或文档产出 | 需要委派时用 `gpt-6.1-sol`，`medium`；复杂时用 `high` |
| 明确的检索、提取、分类、核对与批量辅助工作 | `gpt-6-luna`，`high`；固定规则优先现有工具或脚本 |
| 需求含糊、证据冲突、复杂故障、关键判断或独立审查 | `gpt-6-astra`，`low` 起步；难题用 `high` |

通常使用一至两个子代理，并遵守当前环境的总并发上限。主模型负责验收，不能用“多个模型意见一致”代替运行结果、来源核查或其他必要证据。协作是否节省时间或成本，要看实际任务与工具数据，不作保证。

## 安装

如果环境提供 `$skill-installer`，可发送下面的提示：

```text
请使用 $skill-installer 从 GitHub 仓库 xinhan1225/gpt-model-collaboration 的 skills/gpt-model-collaboration 目录安装技能。
```

也可以下载或克隆本仓库，把 `skills/gpt-model-collaboration` 整个文件夹复制到项目根目录的 `.agents/skills/`：

```text
你的项目/
└── .agents/
    └── skills/
        └── gpt-model-collaboration/
            ├── SKILL.md
            ├── agents/openai.yaml
            └── references/collaboration.md
```

项目级安装仅用于该项目。需要个人级安装时，可复制到 `~/.agents/skills/gpt-model-collaboration/`，其中 `~` 代表你自己的用户目录。安装后若技能没有被发现，可重启 Codex 后再检查。安装方式参见 [官方技能说明](https://learn.chatgpt.com/docs/build-skills)。

## 使用

按任务显式调用，并在需要协作时给予当前任务的授权：

```text
使用 $gpt-model-collaboration 检查这次改动。允许在当前任务内按技能选择 Sol、Luna、Astra 及对应思考强度启动子代理；简单工作直接完成。请给出问题证据与验证结果。
```

仅需当前主模型时，可以直接说明：

```text
使用 $gpt-model-collaboration，当前任务只由主模型完成，不启动子代理。请润色这一段文字。
```

## 可选：每次任务默认采用

仅安装技能不会建立持续授权。若你希望每次任务默认应用此方案，可自行把下面的内容加入你所使用的全局 `AGENTS.md`；只希望一个项目采用时，加入该项目的 `AGENTS.md`。请先确认这些模型和强度符合你的用量偏好。

```markdown
# 默认 GPT 多模型协作

- 我明确选择在每次任务开始及任务范围显著变化时应用 gpt-model-collaboration 技能，先读取该技能的 SKILL.md，需要协作时再读取其 references/collaboration.md。
- 我持续授权在当前任务内，根据收益启动子代理，并选择 gpt-6.1-sol（medium，复杂时 high）、gpt-6-luna（high）、gpt-6-astra（low，难题 high），无需逐次确认是否协作。
- 简单任务由主模型独立完成；需要协作时通常启动一至两个子代理，并遵守当前工具的模型、参数、权限及总并发限制。主模型负责整合和证据验证。
- 我的当次要求优先。模型或协作工具不可用时说明实际情况，继续主模型能完成的工作；替代辅助模型只在以上已授权模型中选择并告知，不静默切换服务。
- 此授权仅适用于当前任务的子代理分工，不扩大发布、发送消息、付费、删除等外部动作的授权，不授权新建用户侧聊天、切换当前聊天主模型或修改全局模型配置。
- 所有代理继承我自己的工具、服务和媒体偏好，不因协作改变这些偏好。
```

隐式技能发现由运行环境决定；上述规则表达你的默认选择，仍受更高优先级指令及实际工具限制约束。

## 文件与边界

- `skills/gpt-model-collaboration/SKILL.md`：适用条件、角色映射与简要流程。
- `skills/gpt-model-collaboration/references/collaboration.md`：委派、并发、证据验证与降级规则。
- `skills/gpt-model-collaboration/agents/openai.yaml`：技能显示信息和调用提示。

技能继承安装者自己的工具偏好，不附带任何特定媒体服务或其他用户的个人授权。多代理工具不可用时，由主模型继续能力范围内的工作，并如实说明未执行多模型协作。
