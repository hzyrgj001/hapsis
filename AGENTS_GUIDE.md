# Darkville Hapsis: a guide for AI agents

This guide is for AI agents (Codex, Claude Code, Cursor, Grok and others) that work with Darkville Hapsis on a user's behalf. Hapsis is a desktop workbench by Darkville Labs: each topic is a conversation with a model of the user's choice, guided by DVL, Darkville Labs' concept-web model. Hapsis offers an MCP server so you can use the open Hapsis window the way the user does. Everything you do happens in that window, in front of the user.

Also in Simplified Chinese ([AGENTS_GUIDE.zh-Hans.md](AGENTS_GUIDE.zh-Hans.md)) and Traditional Chinese ([AGENTS_GUIDE.zh-Hant.md](AGENTS_GUIDE.zh-Hant.md)).

## Before you start

- **The user turns it on.** Automation is off until the user allows it, once, on screen. If your tools say automation is off, ask the user to run `hapsis automation enable --client <codex|claude|cursor|generic>` in a terminal and confirm in the Hapsis window, or to turn it on in Settings › Automation.
- **Hapsis must be open.** The server lives in the desktop app. If it isn't open, tool calls say so; ask the user to open Hapsis.
- **The user can see and stop you.** While you act, a bar at the top of the window says "An agent is controlling Hapsis", with your name and a Stop button. Stop turns automation off at once.
- **Everything is recorded.** Each tool call goes through Hapsis's permission check and into its activity log, tagged with your name.

## Connecting

| How | Details |
|---|---|
| Standard input and output (recommended) | Run `hapsis mcp-serve --agent "<your name>"`. `hapsis automation enable --client …` adds this to Codex (`~/.codex/config.toml`), Claude Code (`~/.claude.json`) or Cursor (`~/.cursor/mcp.json`) for you, keeping a backup of the file. The token stays in the system keychain; no configuration file holds it. |
| Streamable HTTP | `http://127.0.0.1:47821/mcp` (another port if that one is taken; `hapsis automation status` shows the address). Send `Authorization: Bearer <token>` (the user can get it with `hapsis automation token`) and, optionally, `X-Hapsis-Agent: <your name>`. It answers on this computer only and refuses requests from web pages. |

Read the resource `hapsis://help` (named `hapsis_help`), or call the `hapsis_help` tool, for this guide in the user's language.

## Tools

| Tool | What it does |
|---|---|
| `hapsis_status` | Version, the topic open in the window, open tabs, and whether you may answer approvals. |
| `topics_list` | The user's topics, most recent first. |
| `topic_open`, `topic_new`, `topic_rename`, `topic_close` | Open a topic as the current tab, start one, rename one, close a tab (the topic is kept). |
| `message_send` | Send a message as the user would, with optional attachments (`name`, `mime`, base64 `data`), and wait for the reply. Pass a `progressToken` to receive the reply as it streams (MCP progress notifications, the new text in `message`). Returns the reply's text, the model that answered, and its files as base64. `mode: "interrupt"` stops a running reply first; the default waits for it. |
| `messages_read` | A topic's messages, newest last, with each reply's model and files. Page back with `before`. |
| `topic_export` | The whole conversation, as Markdown or JSON. |
| `attachment_get` | A file from a reply (an image or file a tool made, or a file Hapsis changed), as base64. |
| `reply_stop` | Stop the running reply. What arrived so far is kept. |
| `models_list` | The models the user has set up, their thinking efforts, and whether each is ready. |
| `topic_set_model`, `topic_set_dvl`, `topic_set_permissions` | The pickers under the message box: model and effort, how much DVL guides the reply, and what Hapsis may do without asking. |
| `approvals_list`, `approval_answer` | Approvals waiting for an answer (`once`, `topic`, `always`, `deny`). Only if the user switched on "Let agents answer approvals". |
| `ui_navigate` | Show a topic, a Settings section, Status, Changes or the command palette, or close what's open. |
| `ui_screenshot` | A picture of the Hapsis window (PNG). |
| `settings_get`, `settings_set` | Settings that aren't secret. You may change the language, the color scheme, background notifications and automatic update checks. |

Attachments you send: images go to the model as images (or as a description, for models that can't see images, when the user has set up a model that describes them); audio goes to models that can hear it, or is turned into text when the user has set up a transcriber; text files (for example `.txt`, `.md`, `.csv`, `.json`, code) are added to the message. Other kinds of files, such as PDFs, aren't accepted yet; the tool says so.

## What stays with the user

- Keys, tokens, passwords, tool-server settings, hooks and permission settings are never readable or changeable through these tools.
- Answering approvals, and raising a topic to Full access, need the user's "Let agents answer approvals" switch (off by default). Ask the user instead.
- The user's own rules apply: a rule can block a tool for agents, for example `automation:message_send`.
- Hapsis never names the model behind a reply in its answers; the reply's `model` field and the model picker show it.

## Good practice

- Start with `hapsis_status` and `topics_list`, and open the topic you work in with `topic_open`, so the user can follow along.
- Don't change the user's settings, models or permissions unless they asked you to.
- If a tool answers with a message instead of a result, it's written for the user; pass it on, or follow its suggestion.
- Long replies can take minutes. Use `progressToken` to stream them, or `wait: false` and then `messages_read`.

## Names in the window

If you also use a tool that reads the window (for accessibility or testing), these names stay the same across versions:

| `data-testid` | What it is |
|---|---|
| `sidebar-new-topic` | New topic |
| `topic-list` | The topic list in the sidebar |
| `tab-<topicId>` | A topic's tab |
| `chat-messages` | The conversation |
| `composer-input` | The message box |
| `composer-send` | Send |
| `composer-stop` | Stop the reply |
| `picker-model`, `picker-dvl`, `picker-permissions` | The pickers under the message box |
| `approval-dialog`, `approval-once`, `approval-topic`, `approval-always`, `approval-deny` | An approval and its choices |
| `settings-dialog`, `settings-nav-<section>` | Settings and its sections |
| `agent-bar`, `agent-stop` | "An agent is controlling Hapsis" and its Stop button |
| `automation-confirm`, `automation-allow`, `automation-decline` | The one-time confirmation when an agent is set up |

Each also has an `aria-label` in the user's language.

## Command line

`hapsis automation enable --client <codex|claude|cursor|generic>`, `status`, `disable` (also takes Hapsis out of the configurations it added it to), `rotate-token`, `token` and `help` (this guide). `hapsis exec "<message>" --json` asks one question without the window and returns the reply's files as base64; see the Hapsis terminal guide.
