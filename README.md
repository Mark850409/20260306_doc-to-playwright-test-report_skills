# doc-to-playwright-test-report（Agent Skill）

本目錄為 **Cursor / Claude Code 用 Agent Skill**：依專案規格文檔產出測試案例、Excel 清單、透過 Playwright MCP 執行 E2E，並產出含截圖的 Word 測試報告。

## 用途

適用於下列需求：

- 依 `docs/` 內規格（前端開發規格、API 規格、測試說明等）產出可執行測試案例
- 將測試案例寫入 **Excel**（`.xlsx`）供追蹤或匯入
- 透過 **Playwright MCP**（`user-playwright`）對本機前端進行自動化 E2E
- 產出 **Word**（`.docx`）測試報告，內嵌截圖與通過／失敗結果

## 目錄結構

| 檔案 | 說明 |
|------|------|
| `SKILL.md` | Skill 主體：六階段工作流程、檢查清單、與 xlsx／docx／Playwright MCP 的整合規範 |
| `reference.md` | 參考手冊：Excel／DOCX 檔名與欄位、報告章節結構、本專案 URL、Playwright MCP 工具參數速查 |
| `README.md` | 本說明（專案導覽） |

## 工作流程概要

1. **讀取文檔**：鎖定 `docs/` 內相關規格與測試說明。
2. **產出測試案例**：含案例 ID、前置條件、步驟、預期結果，並涵蓋功能與邊界情境。
3. **產出 Excel**：依 **xlsx skill** 規範產出 `YYYYMMDD_XXX測試案例.xlsx`（建議置於 `docs/`）。
4. **執行 E2E**：使用 **user-playwright** MCP；每案例結束須截圖。
5. **回寫 Excel**：將實際結果、通過／失敗、截圖檔名寫回同一測試案例檔。
6. **產出 DOCX**：依 **docx skill** 規範產出 `YYYYMMDD_XXX測試報告.docx`，內文嵌入截圖。

完整步驟、禁止事項與檢查清單請見 [`SKILL.md`](SKILL.md)；檔名範例、URL 與 MCP 參數細節請見 [`reference.md`](reference.md)。

## 相依技能與工具

- **xlsx skill**：建立與更新測試案例 Excel（不得僅依任意腳本繞過 skill 規範）。
- **docx skill**：產出 Word 測試報告（含圖片嵌入）。
- **Playwright MCP**（`user-playwright`）：導頁、快照、點擊、填表、截圖等。

## 安裝／使用方式（Cursor）

將本資料夾複製到專案的 `.cursor/skills/`（或依你環境的 skills 路徑），並在 Agent 設定中讓此 skill 可被讀取。觸發語句可包含：從文檔產生測試、執行 Playwright E2E、產出測試案例表或測試報告等（詳見 `SKILL.md` 開頭 `description` 與「適用時機」）。

## 本機測試前提

執行 E2E 前需啟動前端服務；預設受測 URL 範例（見 `reference.md`）：

- 首頁：`http://localhost:8080/ebg/`
- 表單文件下載：`http://localhost:8080/ebg/form-document-download`

實際路徑請以目標專案為準。

## 授權與維護

內容為內部工作流程與規範文件；若與目標專案之 `docs/`、腳本或 MCP 設定不一致，以該專案為準並更新本 skill 內對應段落。
