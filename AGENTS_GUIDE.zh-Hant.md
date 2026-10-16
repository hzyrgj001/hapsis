# Darkville Hapsis：AI 代理程式使用指南

本指南適用於代表使用者操作 Darkville Hapsis 的 AI 代理程式（Codex、Claude Code、Cursor、Grok 等）。Hapsis 是 Darkville Labs 出品的桌面工作台：每個話題都是與使用者所選模型的一段對話，並由 Darkville Labs 的概念網路模型 DVL 引導。Hapsis 提供一個 MCP 伺服器，讓你能像使用者一樣使用開啟中的 Hapsis 視窗。你所做的一切都會在該視窗中、在使用者眼前進行。

另有英文版（[AGENTS_GUIDE.md](AGENTS_GUIDE.md)）和簡體中文版（[AGENTS_GUIDE.zh-Hans.md](AGENTS_GUIDE.zh-Hans.md)）。

## 開始之前

- **由使用者開啟。** 在使用者於螢幕上確認一次之前，自動化功能處於關閉狀態。如果工具提示自動化已關閉，請讓使用者在終端機執行 `hapsis automation enable --client <codex|claude|cursor|generic>` 並在 Hapsis 視窗中確認，或在「設定 › 自動化」中開啟。
- **Hapsis 必須處於開啟狀態。** 伺服器在桌面應用程式中執行。如果應用程式沒有開啟，工具呼叫會提示這一點；請讓使用者開啟 Hapsis。
- **使用者能看到你，也能讓你停下。** 你操作時，視窗頂端會顯示「有代理程式正在控制 Hapsis」，並附上你的名稱和「停止」按鈕。按下「停止」會立即關閉自動化。
- **一切都有記錄。** 每次工具呼叫都會經過 Hapsis 的權限檢查，並記入活動記錄，標註你的名稱。

## 連線方式

| 方式 | 說明 |
|---|---|
| 標準輸入和輸出（建議） | 執行 `hapsis mcp-serve --agent "<你的名稱>"`。`hapsis automation enable --client …` 會替你把它加入 Codex（`~/.codex/config.toml`）、Claude Code（`~/.claude.json`）或 Cursor（`~/.cursor/mcp.json`）的設定，並保留原檔案的備份。權杖保存在系統鑰匙圈中，任何設定檔都不包含它。 |
| 可串流的 HTTP | `http://127.0.0.1:47821/mcp`（該連接埠被占用時會改用其他連接埠；`hapsis automation status` 會顯示位址）。請傳送 `Authorization: Bearer <權杖>`（使用者可透過 `hapsis automation token` 取得），也可以附上 `X-Hapsis-Agent: <你的名稱>`。它只在本機上回應，並拒絕來自網頁的要求。 |

讀取資源 `hapsis://help`（名稱為 `hapsis_help`），或呼叫 `hapsis_help` 工具，即可取得使用者所用語言的本指南。

## 工具

| 工具 | 作用 |
|---|---|
| `hapsis_status` | 版本、視窗中開啟的話題、已開啟的分頁，以及你是否可以回答審核。 |
| `topics_list` | 使用者的話題，最近的在前。 |
| `topic_open`、`topic_new`、`topic_rename`、`topic_close` | 把話題開啟為目前分頁、新增話題、重新命名話題、關閉分頁（話題會保留）。 |
| `message_send` | 像使用者一樣傳送訊息，可附加附件（`name`、`mime`、base64 編碼的 `data`），並等待回覆。傳入 `progressToken` 即可在回覆產生時即時收到內容（MCP 進度通知，新增文字在 `message` 中）。傳回回覆的文字、作答的模型，以及以 base64 編碼的回覆檔案。`mode: "interrupt"` 會先停止進行中的回覆；預設會等待它完成。 |
| `messages_read` | 話題的訊息，最新的在後，包含每則回覆的模型和檔案。使用 `before` 往前翻頁。 |
| `topic_export` | 完整對話，Markdown 或 JSON 格式。 |
| `attachment_get` | 回覆中的檔案（工具產生的圖片或檔案，或 Hapsis 修改過的檔案），以 base64 編碼。 |
| `reply_stop` | 停止進行中的回覆。已收到的內容會保留。 |
| `models_list` | 使用者已設定的模型、各自的思考強度，以及是否可用。 |
| `topic_set_model`、`topic_set_dvl`、`topic_set_permissions` | 輸入框下方的選擇器：模型和思考強度、DVL 引導回覆的程度，以及 Hapsis 無需詢問即可執行的操作。 |
| `approvals_list`、`approval_answer` | 等待回答的審核（`once`、`topic`、`always`、`deny`）。僅當使用者開啟「允許代理程式回答審核」時可用。 |
| `ui_navigate` | 顯示某個話題、某個設定區段、「狀態」、「變更」或命令面板，或關閉目前開啟的內容。 |
| `ui_screenshot` | Hapsis 視窗的截圖（PNG）。 |
| `settings_get`、`settings_set` | 非機密的設定。你可以變更語言、配色、背景通知和自動檢查更新。 |

你傳送的附件：圖片會以圖片形式交給模型（對於看不到圖片的模型，如果使用者已設定圖像描述模型，則交給它描述）；音訊會交給能聽音訊的模型，或在使用者設定了轉寫模型時轉成文字；文字檔（例如 `.txt`、`.md`、`.csv`、`.json`、程式碼）會附加到訊息中。其他類型的檔案（如 PDF）暫不支援，工具會說明這一點。

## 由使用者掌控的部分

- 金鑰、權杖、密碼、工具伺服器設定、掛鉤和權限設定，都無法透過這些工具讀取或變更。
- 回答審核、把話題提升為「完全存取」，都需要使用者開啟「允許代理程式回答審核」（預設關閉）。遇到這類情況，請詢問使用者。
- 使用者自己的規則同樣適用：規則可以禁止代理程式使用某個工具，例如 `automation:message_send`。
- Hapsis 在回答中從不說出回覆背後的模型；回覆的 `model` 欄位和模型選擇器會顯示它。

## 建議做法

- 先呼叫 `hapsis_status` 和 `topics_list`，並用 `topic_open` 開啟你要操作的話題，方便使用者跟進。
- 除非使用者要求，否則不要變更使用者的設定、模型或權限。
- 如果工具傳回的是一段說明而不是結果，那是寫給使用者看的；請轉告使用者，或依其中的建議操作。
- 較長的回覆可能需要幾分鐘。可以用 `progressToken` 即時接收，或設定 `wait: false`，之後再用 `messages_read` 讀取。

## 視窗中的名稱

如果你同時使用能讀取視窗的工具（用於無障礙或測試），以下名稱在各版本中保持不變：

| `data-testid` | 含義 |
|---|---|
| `sidebar-new-topic` | 新增話題 |
| `topic-list` | 側邊欄中的話題清單 |
| `tab-<topicId>` | 話題的分頁 |
| `chat-messages` | 對話內容 |
| `composer-input` | 輸入框 |
| `composer-send` | 傳送 |
| `composer-stop` | 停止回覆 |
| `picker-model`、`picker-dvl`、`picker-permissions` | 輸入框下方的選擇器 |
| `approval-dialog`、`approval-once`、`approval-topic`、`approval-always`、`approval-deny` | 審核及其選項 |
| `settings-dialog`、`settings-nav-<section>` | 設定及其區段 |
| `agent-bar`、`agent-stop` | 「有代理程式正在控制 Hapsis」及其「停止」按鈕 |
| `automation-confirm`、`automation-allow`、`automation-decline` | 設定代理程式時的一次性確認 |

每個元素也都有使用者所用語言的 `aria-label`。

## 命令列

`hapsis automation enable --client <codex|claude|cursor|generic>`、`status`、`disable`（同時把 Hapsis 從它加入過的設定中移除）、`rotate-token`、`token` 和 `help`（本指南）。`hapsis exec "<訊息>" --json` 無需視窗即可提問一次，並以 base64 傳回回覆的檔案；詳見 Hapsis 終端機版說明。
