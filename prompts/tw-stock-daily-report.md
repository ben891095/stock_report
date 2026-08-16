# ChatGPT 台股日報排程

每個交易日收盤後產生一頁報表網頁、在首頁補上連結，並提交到 GitHub Pages 的來源分支。
資料擷取、驗證、判讀與報表內容規格全部見 Skill，本文件只規範流程。

## 固定設定

| 項目 | 設定 |
|---|---|
| 執行環境 | ChatGPT Web 排程，時區 `Asia/Taipei` |
| Repository | `ben891095/stock_report` |
| 分支 | `main`（GitHub Pages 來源，網站與設定同一分支） |
| Prompt | `prompts/tw-stock-daily-report.md` |
| Skill | `.agents/skills/tw-stock-daily-report/SKILL.md` |
| 報表模板 | `templates/report.html` |
| 報表路徑 | `reports/YYYY/MM/YYYY-MM-DD.html` |
| 首頁 | `index.html` |

只使用 Browser／Web 及已連線的 GitHub connector，不使用本機檔案系統。

## 階段一：載入規則

1. 從 `main` 讀取上表的 Prompt、Skill 與報表模板，每次執行只讀一次。
2. 將 Skill 視為本次任務的 `$tw-stock-daily-report` 規則。
3. 讀到的 Prompt 若已包含本段，代表載入完成，直接進入階段二，不得遞迴重讀。
4. 除上表檔案外，不讀 repository 其他檔案。

## 階段二：取得與驗證資料

1. 依 Skill 擷取並驗證 Yahoo 欄位；目標交易日為執行當日。
2. 驗證失敗時輸出 `SKIPPED: <原因>` 並停止，GitHub 不得有任何寫入。

## 階段三：產生報表頁

1. 以模板為基礎，取代 `{{DATE}}`、`{{WEEKDAY}}`、`{{TREND}}`、`{{TREND_CLASS}}`、`{{UPDATED}}`（產生當下的 `Asia/Taipei` 時間，`YYYY/MM/DD HH:MM:SS`），並依 Skill 填入五個章節。模板本身不得覆寫。
2. 存成 `reports/YYYY/MM/YYYY-MM-DD.html`，不建立 `latest.html` 或其他副本。
3. 提交前逐項自我檢查，任一項不過就修正後重檢：
   - 沒有殘留的 `{{`，也沒有新增任何 HTML 註解。
   - 五個章節齊全且順序正確。
   - 列數正確：大盤指標 9、法人五日 5、強勢 3、弱勢 3、成交排行 10。
   - 大盤指標每列說明開頭都有一個 tag；均線數值依與收盤比較上色。
   - 頁首日期、`資料日期` 與檔名三者一致；更新時間為本次執行的實際時間。
4. 讀取 `main` 上的當日路徑：
   - 不存在：繼續。
   - 內容相同：輸出 `SKIPPED_DUPLICATE` 並停止。
   - 內容不同：輸出 `FAILED_CONFLICT` 並停止，不得覆蓋。

## 階段四：更新首頁

`index.html` 有兩組標記，只能改動標記之間的內容，標記本身必須保留：

| 標記 | 動作 |
|---|---|
| `<!-- LATEST:START -->`～`<!-- LATEST:END -->` | 整段換成本日的日期、盤勢徽章、摘要與連結；摘要取總結前 80～120 字 |
| `<!-- LIST:START -->`～`<!-- LIST:END -->` | 最上方插入一列 `li.post-item`（日期、星期、盤勢徽章、一句摘要）；該月份的 `section.month-group` 不存在時，先新增一個放在最上面 |

- 連結一律用相對路徑 `reports/YYYY/MM/YYYY-MM-DD.html`。
- 徽章 class 與報表頁一致（見 Skill 顯示規則）。

## 階段五：提交 GitHub

1. 以 `main` 目前的 HEAD 為 parent，建立**單一 commit**，同時包含報表頁與更新後的 `index.html`。訊息：`report: TAIEX YYYY-MM-DD`。
2. 工具只支援單檔寫入時，改為兩次提交，順序固定「先報表頁、後 `index.html`」，第二次訊息 `report: link TAIEX YYYY-MM-DD`。若第二次失敗，輸出 `FAILED_PUSH` 並註明「報表頁已提交、首頁未更新」。
3. 禁止 force、刪檔或改寫歷史。
4. Ref 衝突時重讀 HEAD 並重試一次；仍失敗輸出 `FAILED_PUSH`。
5. 成功時輸出報表摘要、commit SHA 與檔案連結，不輸出原始頁面、工具紀錄或冗長驗證過程。

## 狀態碼

| 狀態 | 意義 |
|---|---|
| `SUCCESS` | 報表頁與首頁已提交，附 commit SHA 與連結 |
| `SKIPPED` | 休市、尚未收盤、日期不符或核心資料不完整 |
| `SKIPPED_DUPLICATE` | 當日報表已存在且內容相同 |
| `FAILED_CONFLICT` | 當日報表已存在但內容不同 |
| `FAILED_PUSH` | GitHub 提交或 ref 更新失敗 |
