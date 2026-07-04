# 05 · 環境事實（已查證的，含重驗方法）

> 最後更新：2026-07-04｜原因：初版（事實由官方文件查證，來源附於各節）；同日多代理複審修訂
> 本檔的價值不在清單本身，而在「重驗方法」。清單一定會過期。
> **任何依照本檔呼叫卻失敗（模型 ID 不存在、agent type 不存在、欄位無效），
> 第一動作是照本檔的重驗方法更新本檔，再回頭做原任務。**

## 1. 可用模型（API model ID）

| 用途層級 | 模型 | model ID | Agent 工具 model 參數值 |
|---|---|---|---|
| 便宜、機械性任務 | Haiku 4.5 | `claude-haiku-4-5-20251001` | `haiku` |
| 預設工作馬 | Sonnet 5 | `claude-sonnet-5` | `sonnet` |
| 高難度、升級目標 | Opus 4.8 | `claude-opus-4-8` | `opus` |

查證來源：https://platform.claude.com/docs/en/about-claude/models/overview（2026-07-04）
重驗方法：派 claude-code-guide agent（不在清單則 general-purpose）查上述 URL，
只收更新後的表格列與查證日期；主對話不自行 WebFetch（紅線 1）。不要憑記憶寫 model ID。
注意：ID 格式不一致（Haiku 帶日期尾碼、Sonnet/Opus 不帶）是官方現狀，照表複製即可，
不要自行補上或刪掉日期尾碼。

## 2. Reasoning effort

- 合法值：`low` / `medium` / `high` / `xhigh` / `max`（Sonnet 5、Opus 4.8 支援全部五級；
  舊款 Opus 4.6 / Sonnet 4.6 沒有 `xhigh`）。
- 可設定位置：
  - session 層級：`/effort` 指令、`--effort` 旗標、環境變數 `CLAUDE_CODE_EFFORT_LEVEL`、
    settings.json 的 `effortLevel` 欄位
  - subagent 定義：`.claude/agents/*.md` frontmatter 的 `effort` 欄位
- 查證來源：https://code.claude.com/docs/en/model-config.md（2026-07-04）。重驗方法同第 1 節。

## 3. 自訂 subagent（.claude/agents/*.md）

frontmatter 合法欄位（完整清單）：`name`（必填）、`description`（必填）、`tools`、
`disallowedTools`、`model`（sonnet/opus/haiku/fable/完整 ID/inherit，預設 inherit）、`effort`、
`permissionMode`、`maxTurns`、`skills`、`mcpServers`、`hooks`、`memory`、`background`、
`isolation`、`color`、`initialPrompt`。
查證來源：https://code.claude.com/docs/en/sub-agents.md（2026-07-04）

本 repo 已定義的 agent（在 `.claude/agents/`，session 開始時會自動載入為可用 agent type）：
- `scout`：haiku，唯讀搜尋定位。 - `verifier`：sonnet，fresh-context 驗收。
- `implementer`：sonnet，範圍內實作。用法見 `10-model-dispatch.md` 第 8 節。

## 4. 內建 agent type（本 session 觀測，每個 session 開頭的系統提示會列出當下實際清單）

`Explore`（唯讀搜尋，指定 breadth）、`general-purpose`（多步驟通用）、`Plan`（規劃）、
`claude-code-guide`（查 Claude Code / API 官方文件——查環境事實優先用它）、`claude`（通用）。
重驗方法：讀 session 開頭系統提示的「Available agent types」清單；不在清單上的不要派。

## 5. Skills 與 MCP（波動最大，僅列穩定有用的）

- Skills（本 session 觀測）：`code-review`、`verify`、`simplify`、`deep-research`、
  `update-config`、`fewer-permission-prompts`。重驗：系統提示的 skills 清單。
- MCP servers（本 session 觀測）：`github`（遠端環境沒有 `gh` CLI，GitHub 操作一律用它）、
  `Claude_Code_Remote`（排程、send_later、觸發器）、Gmail / Google Calendar / Google Drive /
  Notion。重驗：用 ToolSearch 搜關鍵字，搜不到才算不可用。
- **主對話與 subagent 的工具清單不同**（已實測：主對話有的任務清單、提問工具，
  subagent 內查不到）。工具是否存在，以「當下這個 context 實際看到的工具清單」為準，
  不以本檔、其他 session 的觀測、或訓練記憶為準。

## 6. 遠端執行環境的硬約束

- 容器短命：閒置就回收。**沒 commit + push 的東西等於沒發生。**
- push 用 `git push -u origin <分支>`；網路錯誤重試最多 4 次（2s/4s/8s/16s 退避）。
- push 被拒（non-fast-forward）＝遠端有別的 session 的工作，不是網路錯誤、不計入重試
  次數：先 `git pull --rebase` 再 push。rebase 衝突時，兩邊都要保留的內容（如 LESSONS.md
  的追加條目）手動合併；拿不準怎麼合 → 停下照 `20-judgment.md` 第 3 節必問處理。
  `git push --force` 與 `--force-with-lease` 一律必問，沒有例外——它會抹掉別的 session
  已 push 的工作，且容器短命，被抹掉的 commit 無處可救。
- 對外 HTTPS 走 agent proxy；TLS 或 403/407 錯誤看 `/root/.ccr/README.md`，
  不要關 TLS 驗證、不要 unset HTTPS_PROXY。
- 暫存檔用系統提示指定的 scratchpad 目錄，不用 `/tmp`。
