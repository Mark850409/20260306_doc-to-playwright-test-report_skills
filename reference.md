# doc-to-playwright-test-report — 參考說明

## 一、測試案例 Excel 產出（Phase 2b）

- **格式**：Excel 活頁簿（**.xlsx**）。
- **檔名**：`YYYYMMDD_XXX測試案例.xlsx`
  - `YYYYMMDD` = 執行當天日期（例：20260306）
  - `XXX` = 報告類型（例：E2E、表單文件下載、首頁）
  - 範例：`20260306_表單文件下載測試案例.xlsx`、`20260306_首頁測試案例.xlsx`
- **輸出路徑**：建議 `docs/`，例如 `docs/20260306_表單文件下載測試案例.xlsx`。
- **產出方式**：必須使用 **xlsx 技能**（讀取並遵循 xlsx skill），依其規範建立活頁簿（如 openpyxl / pandas），字型與格式符合 xlsx skill 要求。
- **建議欄位**（第一列表頭，自第二列起為資料）：

| 欄位名稱   | 說明 |
|------------|------|
| 案例 ID    | 如 TC-01 |
| 場景/標題  | 一句話描述 |
| 前置條件   | 登入狀態、環境、資料前提 |
| 步驟       | 編號步驟說明 |
| 預期結果   | 畫面/訊息/API 行為 |
| 備註       | 可選 |
| 實際結果   | **Phase 3 完成後必填**：該案例實際執行結果說明 |
| 通過/失敗  | **Phase 3 完成後必填**：通過 或 失敗 |
| 截圖檔名   | **Phase 3 完成後必填**：該案例對應之截圖檔名（如 e2e-fdd-tc01-page-load.png） |

**結果回寫（Phase 3b）**：Playwright 執行完成後，必須使用 xlsx 技能（openpyxl 讀取既有 Excel）依案例 ID 對應列，將「實際結果」、「通過/失敗」、「截圖檔名」寫入並儲存，不得留白。專案提供 `scripts/update_test_results_xlsx.py`，可於 Phase 3 完成後執行並傳入測試案例 Excel 路徑，即可將當次執行結果回寫；若為其他報告類型可擴充該腳本內結果清單或自行以 openpyxl 更新。

---

## 二、測試報告 DOCX 產出格式與檔名

- **格式**：Word 文件（**.docx**），不得僅產出 Markdown。
- **檔名**：`YYYYMMDD_XXX測試報告.docx`
  - `YYYYMMDD` = 執行當天日期（例：20260306）
  - `XXX` = 報告類型（例：E2E、表單文件下載）
  - 範例：`20260306_E2E測試報告.docx`
- **輸出路徑**：建議 `docs/`，例如 `docs/20260306_E2E測試報告.docx`。

## 三、報告結構（DOCX 內容）

產出 .docx 時請依下列結構撰寫，並**在文件內嵌入截圖**（見下方「截圖與圖片嵌入」）。

1. **標題**：如「E2E 自動化測試報告」
2. **摘要**：以表格呈現  
   - 執行時間、測試環境、總案例數、通過、失敗、跳過
3. **測試案例結果**（每個案例一個區塊）  
   - 案例 ID 與標題（如 TC-01：關鍵字檢核）  
   - 場景、步驟、預期結果、實際結果（通過/失敗與說明）  
   - **嵌入該案例至少一張截圖**（勿只寫路徑）  
   - 備註（可選）
4. **失敗分析與建議**（若有失敗）
5. **附錄**（可選）：參考規格、案例清單

**截圖與圖片嵌入**：使用 docx skill 的 ImageRun（或 unpack → 加入 media → pack）將 Phase 3 存下的截圖檔（如 `docs/e2e-tc01-keyword-validation.png`）嵌入到對應案例段落中，使報告一打開即可看到畫面與結果。

---

## 四、本專案連線網址

執行 Playwright E2E 時，請使用以下 URL：

| 頁面 | URL |
|------|-----|
| 首頁 | `http://localhost:8080/ebg/` |
| 表單文件下載 | `http://localhost:8080/ebg/form-document-download` |

導頁範例：`browser_navigate` 的 `url` 參數填寫 `http://localhost:8080/ebg/form-document-download` 即可測試表單文件下載頁。

---

## 五、Playwright MCP 工具參數速查

呼叫時使用 `call_mcp_tool`，`server` 固定為 `"user-playwright"`。

### browser_navigate
- **toolName**: `browser_navigate`
- **arguments**: `{ "url": "https://..." }`（必填 `url`）

### browser_snapshot
- **toolName**: `browser_snapshot`
- **arguments**: `{}` 或 `{ "filename": "snapshot.md" }`（可選，存成檔案）

### browser_click
- **toolName**: `browser_click`
- **arguments**: `{ "ref": "<來自 snapshot 的 ref>" }`；可選 `element`（人類可讀描述）、`doubleClick`、`button`、`modifiers`

### browser_fill_form
- **toolName**: `browser_fill_form`
- **arguments**: `{ "fields": [ { "name": "...", "type": "textbox"|"checkbox"|"radio"|"combobox"|"slider", "ref": "<來自 snapshot>", "value": "..." } ] }`

### browser_take_screenshot
- **toolName**: `browser_take_screenshot`
- **arguments**: `{ "type": "png" }`（必填）；可選 `filename`、`fullPage`、`element`+`ref`

### browser_console_messages
- **toolName**: `browser_console_messages`
- **arguments**: `{ "level": "info"|"warning"|"error"|"debug" }`（必填）；可選 `filename`

### browser_network_requests
- 依 MCP 描述檔內 schema 傳入參數（執行前請讀取該工具之 JSON descriptor）。

---

## 六、本專案文檔與測試對應

| 文件 | 可產出測試類型 |
|------|----------------|
| `docs/表單文件下載_前端開發規格.md` | 表單文件下載頁：搜尋、選單連動、關鍵字檢核、列表、下載、分頁 |
| `docs/EOSP_表單文件下載_API_開發規格.md` | API 契約、錯誤碼、下載行為 |
| `docs/EOSP_測試說明.md` | Token 驗證、跑馬燈/輪播 API、環境與 curl 步驟（可轉為 E2E 或手動檢查清單） |

撰寫測試案例時：

- **功能全覆蓋**：每個功能至少一則正向案例；優先從上述文件的「檢查清單」、「錯誤與邊界處理」、「測試步驟」擷取場景與預期結果。
- **邊界測試**：必含空值、長度邊界、無效格式、錯誤路徑、無資料/極端分頁等案例。

## 七、截圖與 DOCX 報告規範

**執行階段截圖**：

- 每個測試案例執行結束時**必須**呼叫 `browser_take_screenshot`，不論通過或失敗。
- 檔名建議：`e2e-tc{N}-{簡短描述}.png`（例如 `e2e-tc01-keyword-validation.png`），存放於 `docs/` 或 `docs/screenshots/`。

**DOCX 報告中的截圖**：

- 每個案例區塊須**嵌入**至少一張對應截圖（使用 docx skill 的圖片嵌入方式），不得僅以文字路徑代替。
- 產出檔名：`YYYYMMDD_XXX測試報告.docx`（當天日期 + 底線 + 報告類型 + 測試報告.docx）。
