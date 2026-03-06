---
name: doc-to-playwright-test-report
description: Generates test cases from project specification docs (e.g. docs/*.md), outputs test cases to an Excel file via xlsx skill, runs E2E tests via Playwright MCP (user-playwright), and produces a DOCX test report (YYYYMMDD_XXX測試報告.docx) with embedded screenshots and results. Use when the user asks to generate tests from docs, run automated E2E tests with Playwright, produce a test case spreadsheet, or produce a test report from specification documents.
---

# 依專案文檔產生測試案例、Excel 清單、Playwright 執行與 DOCX 報告

## 適用時機

- 需依 `docs/` 內規格（前端開發規格、API 規格、測試說明等）產出測試案例
- 需將**測試案例產出為 Excel 檔案**（.xlsx），供追蹤或匯入使用
- 需透過 Playwright MCP 對本專案前端進行自動化 E2E 測試
- 需產出結構化測試報告（.docx 格式，內嵌截圖與測試結果，檔名：今日日期_XXX測試報告.docx）

---

## 工作流程（六階段）

### Phase 1：讀取專案說明文檔

1. 鎖定 `docs/` 下相關文件，例如：
   - `*_前端開發規格.md`、`*_API_開發規格.md`、`EOSP_測試說明.md`、`表單文件下載.md` 等
2. 擷取：功能概述、畫面規格、欄位/按鈕、API 對應、錯誤處理、檢查清單、測試步驟
3. 若有「測試說明」類文件，直接沿用其中的測試步驟與預期結果

### Phase 2：產出測試案例

依文檔整理成可執行案例，**須涵蓋所有功能與邊界測試**，每案包含：

| 欄位 | 說明 |
|------|------|
| 案例 ID | 簡短代碼（如 TC-01） |
| 場景/標題 | 一句話描述 |
| 前置條件 | 登入狀態、環境、資料前提 |
| 步驟 | 編號步驟（導向 URL、填表、點擊、等待） |
| 預期結果 | 畫面/訊息/API 行為 |
| 備註 | 可選（如需 mock、特定環境） |

**案例產出原則**：

1. **功能全覆蓋**：依規格書逐項對應，確保每個功能（按鈕、表單、列表、篩選、下載、分頁等）至少有一個正向案例。
2. **邊界測試必含**：
   - 空值/空白：必填欄位留空、關鍵字空白送出
   - 長度邊界：欄位最大長度、超長輸入
   - 格式/型別：無效日期、非法字元、錯誤格式
   - 錯誤路徑：API 失敗、權限不足、無資料狀態
   - 極端值：分頁首/末頁、無搜尋結果、單筆/零筆列表

優先從規格書的「檢查清單」、「錯誤與邊界處理」、「測試步驟」轉成案例；API 規格可用於驗證請求/回應與錯誤碼。

### Phase 2b：產出測試案例 Excel（必做）

**測試案例產出後**，必須使用 **`xlsx` 技能**將案例清單寫入 Excel 檔案，供後續追蹤或匯入。

**⚠️ 產出方式（必守）**：

- **必須使用 `xlsx` 技能**產出 Excel，不得自行撰寫產生 .xlsx 的程式碼而不依 xlsx skill 規範。
- 執行前請先**讀取並遵循 `xlsx` skill**（例如 `C:\Users\zanehsu\.claude\skills\xlsx\SKILL.md` 或專案內對應的 xlsx skill），依其規範建立活頁簿（如 openpyxl 或 pandas 寫入 .xlsx）。
- 字型與格式：使用一致、專業字型（如 Arial）；欄位對齊、欄寬適中，表頭可加粗或底色以便辨識。

**Excel 內容規範**：

- **檔名**：`YYYYMMDD_XXX測試案例.xlsx`，其中 `YYYYMMDD` 為**執行當天日期**，`XXX` 為報告類型（如 E2E、表單文件下載、首頁）。範例：`20260306_表單文件下載測試案例.xlsx`
- **輸出路徑**：建議存於 `docs/`，例如 `docs/20260306_表單文件下載測試案例.xlsx`
- **欄位（至少）**：案例 ID、場景/標題、前置條件、步驟、預期結果、備註。若已執行測試，可另欄補上：實際結果、通過/失敗、截圖檔名或路徑。

**實作建議**：以 openpyxl 或 pandas 建立一工作表，第一列為表頭，自第二列起每列一筆測試案例；欄寬可依內容調整，避免公式錯誤（本表以資料為主，若無合計/公式可不使用公式）。

### Phase 3：使用 Playwright MCP 執行測試

- **本專案連線網址**（執行 E2E 時請使用下列 URL）：
  - **首頁**：`http://localhost:8080/ebg/`
  - **表單文件下載**：`http://localhost:8080/ebg/form-document-download`
- **MCP 伺服器**：`user-playwright`
- **呼叫方式**：使用 `call_mcp_tool`，`server` 為 `"user-playwright"`，`toolName` 為工具名稱，`arguments` 依各工具 schema 傳入

常用工具與用途：

| 工具 | 用途 | 關鍵參數 |
|------|------|----------|
| `browser_navigate` | 開啟受測頁 | `url`（必填） |
| `browser_snapshot` | 取得頁面可及性快照（用於取得 ref） | 可選 `filename` 存成 md |
| `browser_click` | 點擊元素 | `ref`（必填，來自 snapshot） |
| `browser_fill_form` | 填寫表單 | `fields`：每項含 `name`, `type`, `ref`, `value` |
| `browser_take_screenshot` | 截圖 | `type`（必填）, 可選 `filename`, `fullPage` |
| `browser_console_messages` | 取主控台訊息 | `level`（必填：error/warning/info/debug） |
| `browser_network_requests` | 檢視網路請求 | 依 schema 傳入 |

**執行要點**：

1. 執行前先以 `browser_snapshot` 取得目前頁面結構，從快照中取得欲操作元素的 `ref`
2. 點擊、填表一律使用 snapshot 提供的 `ref`，勿臆測
3. **每個測試案例執行結束時必須截圖**：不論通過或失敗，皆須呼叫 `browser_take_screenshot`，並以可辨識的檔名（如 `e2e-tc01-keyword-validation.png`、`e2e-tc02-empty-search.png`）存檔，供報告引用；若案例含多個關鍵檢查點，可於各檢查點後各截一張圖
4. 若規格要求驗證 API 或錯誤訊息，可搭配 `browser_network_requests` 或 `browser_console_messages` 比對
5. 測試前確認前端服務已啟動；導頁時使用本專案約定 URL，詳見 [reference.md](reference.md)

### Phase 3b：測試結果回寫 Excel（必做）

**Phase 3 執行完成後**，必須將每個案例的執行結果**回寫**至 Phase 2b 產出的測試案例 Excel，補上原本空白的欄位。

**⚠️ 回寫內容（必填）**：

- **實際結果**：該案例執行後的實際畫面或行為說明（與預期比對後的簡述）。
- **通過/失敗**：填寫「通過」或「失敗」。
- **截圖檔名**：該案例對應的截圖檔名（如 `e2e-fdd-tc01-page-load.png`），與 Phase 3 存檔名稱一致。

**產出方式**：使用 **xlsx 技能**（openpyxl 的 `load_workbook` 讀取既有的 `YYYYMMDD_XXX測試案例.xlsx`），依「案例 ID」對應列，寫入上述三欄（實際結果、通過/失敗、截圖檔名），儲存後覆蓋原檔。可依專案提供的回寫腳本（如 `scripts/update_test_results_xlsx.py`）或自行依 xlsx skill 規範撰寫更新邏輯。字型與儲存格格式須維持與原表一致（如 Arial、自動換列）。

**順序**：先完成 Phase 3（含所有案例截圖）→ 再執行 Phase 3b 回寫 Excel → 最後 Phase 4 產出 DOCX。

### Phase 4：產出測試報告（DOCX）

最終產出**必須為 Word 文件（.docx）**，依 [reference.md](reference.md) 的報告結構撰寫，並**在文件內嵌入截圖與測試結果**。

**⚠️ 產出方式（必守）**：

- **必須使用 `docx` 技能**產出報告，不得自行撰寫產生 .docx 的程式碼。
- 執行 Phase 4 前請先**讀取並遵循 `docx` skill**，依其規範建立文件（例如 docx-js 的 Document/Packer、或 unpack → 編輯 XML → pack）。
- 嵌入圖片、表格、標題、頁碼等格式須依 docx skill 的說明實作。

**報告內容須包含**：

- 測試摘要（總數、通過、失敗、跳過）— 以表格呈現
- 環境與執行時間
- **各案例結果**：每個案例皆須寫入 ID、標題、步驟、預期 vs 實際、**通過/失敗**，並**在該案例段落中嵌入對應截圖**（至少一張）
- 失敗原因與建議（若有失敗）
- 附錄（可選）：案例清單、參考規格文件、可註明測試案例 Excel 檔名與路徑

**產出規範**：

1. **格式**：使用 `.docx`，**一律透過 docx 技能**產生，禁止手寫產生 DOCX 的腳本或程式碼。
2. **截圖**：將 Phase 3 存下的截圖檔**嵌入**報告內文，不要只寫路徑；每個測試案例區塊須含至少一張對應截圖。
3. **檔名**：`YYYYMMDD_XXX測試報告.docx`，其中 `YYYYMMDD` 為**執行當天日期**，`XXX` 為報告類型。範例：`20260306_E2E測試報告.docx`
4. **輸出路徑**：建議存於 `docs/`，例如 `docs/20260306_E2E測試報告.docx`

---

## 檢查清單（執行前）

- [ ] 已讀取相關規格/測試說明文檔
- [ ] 測試案例已涵蓋**所有功能**與**邊界測試**（空值、長度、格式、錯誤路徑、極端值）
- [ ] 測試案例已列出步驟與預期結果
- [ ] **已使用 xlsx 技能產出測試案例 Excel**（檔名 `YYYYMMDD_XXX測試案例.xlsx`，存於 `docs/`）
- [ ] **Phase 3 完成後已將執行結果回寫至該 Excel**（實際結果、通過/失敗、截圖檔名 三欄已補上）
- [ ] 前端服務可連（URL 正確）
- [ ] 呼叫 MCP 前已確認各工具 schema（參數必填/選填）
- [ ] 已約定每個案例執行後必截圖，且報告中每個案例皆會嵌入截圖
- [ ] 最終產出為 DOCX，檔名為 `YYYYMMDD_XXX測試報告.docx`，報告內含嵌入截圖與測試結果
- [ ] 產出 DOCX 時**使用 docx 技能**（讀取並遵循 docx skill），不自行撰寫產生 .docx 的程式碼

---

## 參考

- 報告結構、DOCX/Excel 產出與 MCP 工具參數：[reference.md](reference.md)
- **Excel 測試案例產出**：必須使用 **`xlsx` skill**（讀取並遵循其 SKILL.md），依其規範建立 .xlsx（欄位、字型、格式），不得僅手寫腳本而不依 xlsx skill 要求
- **Word 報告產出**：必須使用 **`docx` skill**（讀取並遵循其 SKILL.md），依其規範建立 .docx（嵌入圖片、表格、標題、頁碼等），不得自行撰寫產生文檔的程式碼
- 專案開發與規範：`.cursor/rules/` 與 `e-operation-development` skill
