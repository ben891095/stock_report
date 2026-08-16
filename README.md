# 台股每日收盤報表

每個交易日收盤後產生一頁台股收盤報表，以部落格形式收錄的靜態網站。純 HTML 與 CSS，**沒有 JavaScript**、沒有依賴、沒有建置流程。

報表內容：總結、大盤指標、三大法人籌碼、熱門族群、成交排行。資料來源為 Yahoo 台灣股市。

## 目錄

```text
index.html                        # 首頁：最新一篇 ＋ 依年月分組的全部報表
css/style.css
templates/report.html             # 報表頁模板
reports/YYYY/MM/YYYY-MM-DD.html   # 每日報表頁
prompts/tw-stock-daily-report.md  # 排程流程（Prompt）
.agents/skills/tw-stock-daily-report/SKILL.md   # 資料與內容規格（Skill）
```

## 本機預覽

全站靜態，直接用瀏覽器開啟 `index.html` 即可。想模擬正式路徑也可以起一個靜態伺服器：

```bash
python -m http.server 8000
```

接著開啟 <http://localhost:8000>。

## 每日報表怎麼來的

由 ChatGPT 雲端排程執行，三份文件各司其職：

| 檔案 | 負責 |
|---|---|
| `prompts/tw-stock-daily-report.md` | 流程：讀規則 → 取資料 → 產頁 → 更新首頁 → 提交 GitHub |
| `.agents/skills/tw-stock-daily-report/SKILL.md` | 內容：Yahoo 擷取欄位、驗證計算、指標判讀、章節規格 |
| `templates/report.html` | 結構：標籤、class、章節順序的唯一依據 |

每次執行會**新增一頁報表並修改 `index.html` 加上連結**，兩者提交在同一分支（`main`）。遇到休市、尚未收盤或核心資料不完整時，流程會回報 `SKIPPED` 且不寫入任何檔案。

## 手動新增一篇報表

1. 複製 `templates/report.html` 為 `reports/YYYY/MM/YYYY-MM-DD.html`。
2. 取代五個佔位符：

   | 佔位符 | 範例 |
   |---|---|
   | `{{DATE}}` | `2026/08/14` |
   | `{{WEEKDAY}}` | `週五` |
   | `{{TREND}}` | `中性`（偏多／偏空／中性） |
   | `{{TREND_CLASS}}` | `is-flat`（偏多 `is-up`、偏空 `is-down`） |
   | `{{UPDATED}}` | `2026/08/16 13:48:03`（產生當下時間） |

3. 依 Skill 填入五個章節。
4. 編輯 `index.html`：
   - `<!-- LATEST:START -->`～`<!-- LATEST:END -->`：整段換成新報表。
   - `<!-- LIST:START -->`～`<!-- LIST:END -->`：最上方插入一列 `li.post-item`；新月份先補一個 `section.month-group`。
   - 兩組標記本身不要刪。

模板裡的 `../../../` 是以 `reports/YYYY/MM/` 計算的，模板檔案本身不是可預覽的頁面。

## 部署到 GitHub Pages

1. 將本目錄推送至 `main`。
2. Settings → Pages 選擇 `main` 與 `/ (root)`。
3. 網站路徑為 `https://<user>.github.io/<repo>/`。
4. 保留 `.nojekyll`。

## 閱讀報表的幾個約定

- **紅漲綠跌**：沿用台股慣例，與歐美市場相反。
- **判讀 tag**：大盤指標每列標偏多／偏空／中性，只代表該項指標，不是結論。總結的「盤勢偏向」另依均線、動能、量價、法人、類股五個面向判定，其中動能面向由 KD、RSI、BIAS 取多數方向。
- **均線數值顏色**：低於當日收盤為綠、高於為紅。
- **資料日期 vs 更新時間**：資料日期是行情所屬的交易日，更新時間是這一頁被產生的時間。
- **成交排行**：已排除 ETF、槓桿及反向商品等非普通股，名次為過濾後重新編排；上榜只代表交易熱度。

設計與版面規格見 [PLAN.md](PLAN.md)。

## 免責聲明

本站為市場資料整理與技術分析，不構成投資建議。數據來源為 Yahoo 台灣股市，可能有延遲或誤差。
