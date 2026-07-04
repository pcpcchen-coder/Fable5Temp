# CLAUDE.md — 本檔硬上限 80 行；要加內容請加到引用檔，這裡只能加一行路由

## 這個 repo 是什麼

存放 AI agent 長期運作的制度：路由、派工模板、模型調度守則、判斷 rubric、維護協議。
**不是軟體專案**——沒有 build、沒有測試、沒有部署。所有「程式碼任務」的慣例（跑測試、
lint）在這裡不適用；這裡的品質驗證方式是 read-back 與 fresh-context 複述（見紅線 2）。

## 三條紅線（動手前 30 秒讀完；違反任何一條，先停手）

1. **主對話不吞原始資料**：要讀超過 2 個檔案（或單檔超過 200 行）、要查網頁、要多輪
   搜尋、要批次改檔 → 派 subagent，主對話只收結論與 file:line。
2. **沒有驗收條件不派工；沒有 fresh-context 驗證不宣告完成**：派工必附可機械判定的
   驗收條件；完成 = 逐條有證據 + 獨立驗證 PASS + 已 commit push。
3. **改既有檔案前先有備份點**：git 工作區乾淨（不乾淨先 commit）才能改；
   未追蹤檔案先複製一份 `.bak-日期`。

## 路由表（按情境讀，不要一次全讀）

| 你正要做的事 | 先讀 |
|---|---|
| 想知道這些規則為什麼存在 | `ai-rules/00-diagnosis.md` |
| 確認環境事實：可用模型、agent、工具（或呼叫失敗時） | `ai-rules/05-environment.md` |
| 派工、選模型、選 effort、驗收、升降級 | `ai-rules/10-model-dispatch.md` |
| 拿不準：該升級？算完成？該問人？該換路？ | `ai-rules/20-judgment.md` |
| 要寫派工 prompt（搜尋/實作/重構/研究/審查） | `ai-rules/30-delegation-templates.md` |
| 要修改本 repo 的任何制度檔 | `ai-rules/40-maintenance.md`（讀完才能改） |
| 踩坑了、學到教訓了 | 追加一條到 `ai-rules/LESSONS.md`（格式見該檔檔頭） |
| 新 session 想了解背景與未完成事項 | `ai-rules/90-letter.md` |

## 本 repo 硬規則

- **結論落檔**：任何值得留下的產出（研究結果、決策、教訓）寫成檔案並 push，
  不留在對話裡。容器是短命的，沒 push 的東西等於沒發生。
- **超過 3 步的任務先建 checklist**（TaskCreate 或 scratchpad WORKLOG），逐步打勾。
- **git**：在遠端環境用 `git push -u origin <分支>`；GitHub 操作用 github MCP 工具
  （遠端環境沒有 `gh` CLI）。
- **本檔的維護**：任何 session 發現本檔超過 80 行，有義務先精簡再做手上的任務。
  刪改紅線與路由表需先問使用者（規則見 `ai-rules/40-maintenance.md`）。
