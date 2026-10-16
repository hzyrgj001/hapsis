# Darkville Hapsis：AI 智能体使用指南

本指南面向代表用户使用 Darkville Hapsis 的 AI 智能体（Codex、Claude Code、Cursor、Grok 等）。Hapsis 是 Darkville Labs 出品的桌面工作台：每个话题都是与用户所选模型的一段对话，并由 Darkville Labs 的概念网络模型 DVL 引导。Hapsis 提供一个 MCP 服务器，让你能像用户一样使用打开的 Hapsis 窗口。你所做的一切都会在该窗口中、在用户眼前进行。

另有英文版（[AGENTS_GUIDE.md](AGENTS_GUIDE.md)）和繁体中文版（[AGENTS_GUIDE.zh-Hant.md](AGENTS_GUIDE.zh-Hant.md)）。

## 开始之前

- **由用户开启。** 在用户于屏幕上确认一次之前，自动化功能处于关闭状态。如果工具提示自动化已关闭，请让用户在终端运行 `hapsis automation enable --client <codex|claude|cursor|generic>` 并在 Hapsis 窗口中确认，或在“设置 › 自动化”中开启。
- **Hapsis 必须处于打开状态。** 服务器运行在桌面应用中。如果应用没有打开，工具调用会提示这一点；请让用户打开 Hapsis。
- **用户能看到你，也能让你停下。** 你操作时，窗口顶部会显示“有智能体正在控制 Hapsis”，并附上你的名称和“停止”按钮。点击“停止”会立即关闭自动化。
- **一切都有记录。** 每次工具调用都会经过 Hapsis 的权限检查，并记入活动日志，标注你的名称。

## 连接方式

| 方式 | 说明 |
|---|---|
| 标准输入和输出（推荐） | 运行 `hapsis mcp-serve --agent "<你的名称>"`。`hapsis automation enable --client …` 会替你把它加入 Codex（`~/.codex/config.toml`）、Claude Code（`~/.claude.json`）或 Cursor（`~/.cursor/mcp.json`）的配置，并保留原文件的备份。令牌保存在系统钥匙串中，任何配置文件都不包含它。 |
| 可流式传输的 HTTP | `http://127.0.0.1:47821/mcp`（该端口被占用时会改用其他端口；`hapsis automation status` 会显示地址）。请发送 `Authorization: Bearer <令牌>`（用户可通过 `hapsis automation token` 获取），也可以附上 `X-Hapsis-Agent: <你的名称>`。它只在本机上响应，并拒绝来自网页的请求。 |

读取资源 `hapsis://help`（名称为 `hapsis_help`），或调用 `hapsis_help` 工具，即可获得用户所用语言的本指南。

## 工具

| 工具 | 作用 |
|---|---|
| `hapsis_status` | 版本、窗口中打开的话题、已打开的标签页，以及你是否可以回答审批。 |
| `topics_list` | 用户的话题，最近的在前。 |
| `topic_open`、`topic_new`、`topic_rename`、`topic_close` | 把话题打开为当前标签页、新建话题、重命名话题、关闭标签页（话题会保留）。 |
| `message_send` | 像用户一样发送消息，可附带附件（`name`、`mime`、base64 编码的 `data`），并等待回复。传入 `progressToken` 即可在回复生成时实时收到内容（MCP 进度通知，新增文字在 `message` 中）。返回回复的文字、作答的模型，以及以 base64 编码的回复文件。`mode: "interrupt"` 会先停止正在进行的回复；默认会等待它完成。 |
| `messages_read` | 话题的消息，最新的在后，包含每条回复的模型和文件。使用 `before` 向前翻页。 |
| `topic_export` | 完整对话，Markdown 或 JSON 格式。 |
| `attachment_get` | 回复中的文件（工具生成的图片或文件，或 Hapsis 修改过的文件），以 base64 编码。 |
| `reply_stop` | 停止正在进行的回复。已收到的内容会保留。 |
| `models_list` | 用户已设置的模型、各自的思考强度，以及是否可用。 |
| `topic_set_model`、`topic_set_dvl`、`topic_set_permissions` | 输入框下方的选择器：模型和思考强度、DVL 引导回复的程度，以及 Hapsis 无需询问即可执行的操作。 |
| `approvals_list`、`approval_answer` | 等待回答的审批（`once`、`topic`、`always`、`deny`）。仅当用户开启“允许智能体回答审批”时可用。 |
| `ui_navigate` | 显示某个话题、某个设置分区、“状态”、“更改”或命令面板，或关闭当前打开的内容。 |
| `ui_screenshot` | Hapsis 窗口的截图（PNG）。 |
| `settings_get`、`settings_set` | 非机密的设置。你可以更改语言、配色方案、后台通知和自动检查更新。 |

你发送的附件：图片会以图片形式交给模型（对于看不到图片的模型，如果用户已设置图像描述模型，则交给它描述）；音频会交给能听音频的模型，或在用户设置了转写模型时转成文字；文本文件（例如 `.txt`、`.md`、`.csv`、`.json`、代码）会附加到消息中。其他类型的文件（如 PDF）暂不支持，工具会说明这一点。

## 由用户掌控的部分

- 密钥、令牌、密码、工具服务器设置、钩子和权限设置，都无法通过这些工具读取或更改。
- 回答审批、把话题提升为“完全访问”，都需要用户开启“允许智能体回答审批”（默认关闭）。遇到这类情况，请询问用户。
- 用户自己的规则同样适用：规则可以禁止智能体使用某个工具，例如 `automation:message_send`。
- Hapsis 在回答中从不说出回复背后的模型；回复的 `model` 字段和模型选择器会显示它。

## 建议做法

- 先调用 `hapsis_status` 和 `topics_list`，并用 `topic_open` 打开你要操作的话题，方便用户跟进。
- 除非用户要求，否则不要更改用户的设置、模型或权限。
- 如果工具返回的是一段说明而不是结果，那是写给用户看的；请转告用户，或按其中的建议操作。
- 较长的回复可能需要几分钟。可以用 `progressToken` 实时接收，或设置 `wait: false`，之后再用 `messages_read` 读取。

## 窗口中的名称

如果你同时使用能读取窗口的工具（用于无障碍或测试），以下名称在各版本中保持不变：

| `data-testid` | 含义 |
|---|---|
| `sidebar-new-topic` | 新建话题 |
| `topic-list` | 侧边栏中的话题列表 |
| `tab-<topicId>` | 话题的标签页 |
| `chat-messages` | 对话内容 |
| `composer-input` | 输入框 |
| `composer-send` | 发送 |
| `composer-stop` | 停止回复 |
| `picker-model`、`picker-dvl`、`picker-permissions` | 输入框下方的选择器 |
| `approval-dialog`、`approval-once`、`approval-topic`、`approval-always`、`approval-deny` | 审批及其选项 |
| `settings-dialog`、`settings-nav-<section>` | 设置及其分区 |
| `agent-bar`、`agent-stop` | “有智能体正在控制 Hapsis”及其“停止”按钮 |
| `automation-confirm`、`automation-allow`、`automation-decline` | 设置智能体时的一次性确认 |

每个元素也都有用户所用语言的 `aria-label`。

## 命令行

`hapsis automation enable --client <codex|claude|cursor|generic>`、`status`、`disable`（同时把 Hapsis 从它添加过的配置中移除）、`rotate-token`、`token` 和 `help`（本指南）。`hapsis exec "<消息>" --json` 无需窗口即可提问一次，并以 base64 返回回复的文件；详见 Hapsis 终端版说明。
