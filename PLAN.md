# 台股每日收盤報表靜態網站

## 1. 專案目標

建立可部署至 GitHub Pages 的部落格式靜態網站，收錄每個交易日的台股收盤報表。網站提供：

- 最新一篇報表的摘要卡
- 依年月分組的全部報表清單
- 單篇報表頁：總結、大盤指標、籌碼、熱門族群、成交排行

報表由 ChatGPT 雲端排程每日產生，動作固定為「新增一頁報表 ＋ 在首頁插入連結」。網站本身不抓資料、不做運算。

## 2. 技術與檔案結構

採純 HTML 與 CSS。**不使用 JavaScript**，也不使用框架、CDN、第三方依賴或建置流程。

```text
stock_report/
├── index.html                          # 首頁（含 LATEST／LIST 兩組區塊標記）
├── css/style.css
├── templates/report.html               # 報表頁模板：零註解、五個佔位符
├── reports/YYYY/MM/YYYY-MM-DD.html     # 每日報表頁
├── prompts/tw-stock-daily-report.md    # 排程流程
├── .agents/skills/tw-stock-daily-report/
│   ├── SKILL.md                        # 資料與內容規格
│   └── agents/openai.yaml
├── .nojekyll
├── PLAN.md
└── README.md
```

- 所有頁面設定 `lang="zh-Hant"`、viewport 與深色 `color-scheme`。
- favicon 使用 📈 emoji 的 inline SVG data URI。
- 頁面完全靜態，可直接以 `file://` 開啟。
- 報表頁的相對路徑 `../../../` 以產出位置 `reports/YYYY/MM/` 計算；模板本身不是可預覽頁面。

## 3. 內容產生流程

| 角色 | 檔案 | 負責 |
|---|---|---|
| 流程 | `prompts/tw-stock-daily-report.md` | 載入規則、驗證、產檔、更新首頁、提交 GitHub、狀態碼 |
| 內容 | `.agents/skills/tw-stock-daily-report/SKILL.md` | Yahoo 擷取欄位、驗證計算、判讀規則、章節與顯示規則 |
| 結構 | `templates/report.html` | 標籤、class、章節順序、相對路徑的唯一依據 |

三者不重複描述同一件事：流程不談版面，內容不談 commit，模板不寫說明文字。

每次產報表固定兩個動作：

1. 複製模板為 `reports/YYYY/MM/YYYY-MM-DD.html`，取代五個佔位符後填入五個章節。
2. 更新 `index.html`：覆寫 `LATEST` 區塊、在 `LIST` 區塊最上方插入一列；新月份先補一個 `section.month-group`。

| 佔位符 | 內容 |
|---|---|
| `{{DATE}}` | 資料日期 `YYYY/MM/DD`，與檔名同一個交易日 |
| `{{WEEKDAY}}` | 該交易日星期，例如 `週五` |
| `{{TREND}}` | 盤勢偏向：偏多／偏空／中性 |
| `{{TREND_CLASS}}` | 徽章樣式：`is-up`／`is-down`／`is-flat` |
| `{{UPDATED}}` | 產生當下的 `Asia/Taipei` 時間 `YYYY/MM/DD HH:MM:SS` |

報表頁與首頁必須提交在同一分支（GitHub Pages 來源，目前為 `main`）。

## 4. 首頁

```text
.page
├── header.page-header      # 站名（漸層字）、說明、資料來源提醒
├── LATEST 區塊             # section.hero：最新一篇
├── section（所有報表）      # h2.section-title ＋ LIST 區塊的月份分組
└── footer.page-footer
```

- 區塊標記 `<!-- LATEST:START/END -->` 與 `<!-- LIST:START/END -->` 必須保留，只能改動標記之間的內容。
- 清單列為 `li.post-item`，三欄：日期（含星期）、盤勢徽章、一句摘要。
- 月份新到舊、日期新到舊。
- 首頁最大寬度 1280px，報表頁 960px。

## 5. 報表頁

頁首 `meta-row` 依序放盤勢徽章、星期、資料日期，最後是靠右的更新時間（`span.meta-updated`）。資料日期是行情所屬交易日，更新時間是這一頁被產生的時間，兩者不可互換。

章節順序與標題固定，詳細規格見 SKILL：

| # | 章節 | 內容與列數 |
|---:|---|---|
| 1 | 總結 | 100～200 字，首句為「盤勢偏向：偏多／偏空／中性。」 |
| 2 | 大盤指標 | 兩欄 9 列，每列說明開頭一個判讀 tag |
| 3 | 籌碼 | 買賣金額 4 列 ＋ 近五個交易日 5 列，說明點出趨勢 |
| 4 | 熱門族群 | 強勢 3 列、弱勢 3 列，說明資金流向 |
| 5 | 成交排行 | 過濾非普通股後 10 列，說明集中度 |

大盤指標 9 列依序為：大盤、均線 MA5／MA10、均線 MA20／MA60、均線 MA120／MA240、成交金額、漲跌家數、RSI5／RSI10、K9／D9／J9、BIAS10／BIAS20／B10-B20。數值寫在說明文字裡，不另開數值欄。

判讀 tag 只代表單一指標，不做多數決；BIAS 與其他指標一樣標偏多／偏空／中性。總結的盤勢偏向另依均線、動能、量價、法人、類股五個面向判定，其中動能面向由 KD、RSI、BIAS 取多數方向。

## 6. 視覺與響應式規格

### 6.1 設計方向

- 固定暗色、高資訊密度的單欄版面；系統切換淺色時仍維持暗色。
- 全站等寬數字：`font-variant-numeric: tabular-nums`。
- 字族：`"Noto Sans TC", "PingFang TC", "Microsoft JhengHei", system-ui, sans-serif`。
- 基準字級 17px、行高 1.6；任何文字不得小於 14px。
- 著色文字對比至少 4.5:1；`:focus-visible` 為 `2px solid var(--accent)`，offset 2px。
- 卡片圓角 10px，主要面板 12px，按鈕 8px；頁面內距 `24px 20px 64px`。

### 6.2 色票

```css
:root {
  color-scheme: dark only;
  --bg: #12151a;
  --panel: #1a1f27;
  --panel-2: #212836;
  --border: #2e3644;
  --text: #e9edf3;
  --muted: #b7c0cd;
  --faint: #98a3b3;
  --accent: #5aa9ff;
  --ok: #4ade80;
  --danger: #ff6b6b;
  --warn: #ffc14d;
  --up: #ff6b6b;    /* 台股慣例：紅漲 */
  --down: #4ade80;  /* 綠跌 */
  --flat: #b7c0cd;
  --tone-base: #7cc4ff;    /* 總結 */
  --tone-income: #5ee9a4;  /* 大盤指標 */
  --tone-expense: #ffc14d; /* 籌碼 */
  --tone-general: #ff9ec4; /* 熱門族群 */
  --tone-debt: #c9a6ff;    /* 成交排行 */
}
```

漲跌顏色只能來自 `--up` / `--down`，徽章、表格數字與說明文字必須一致。

### 6.3 元件

| 元件 | 規則 |
|---|---|
| 站名 | 藍→綠→紫漸層文字，並提供不支援 `background-clip` 時的純色退路 |
| 最新報表卡 | 上緣 3px 藍色條；月份標題為左側 3px 綠色條 |
| 章節標題 | `.article h2` 依出現順序輪替五個色調，對應五個章節 |
| 徽章 | 偏多 `is-up` 紅、偏空 `is-down` 綠、中性 `is-flat` 灰 |
| 更新時間 | `span.meta-updated`，以 `margin-left: auto` 推到 `meta-row` 最右側 |
| 表格 | 放在 `.table-wrap`（`overflow-x: auto`）內，表頭 sticky，第一欄靠左其餘靠右 |
| 數字 | 上漲 `up`、下跌 `down`；均線數值改依與收盤比較上色（低於收盤 `down`、高於 `up`） |
| 合計列 | `tr.is-total`，上緣分隔線並加粗 |

### 6.4 響應式

- 清單列寬版為 `150px 74px 1fr`；640px 以下改為兩欄，摘要獨占一行。
- 375px 寬時頁面不得橫向捲動，寬表在自身容器內橫捲，更新時間換行後仍靠右。

## 7. 部署

1. 將本目錄推送至 `main`。
2. Settings → Pages 選擇 `main` 與 `/ (root)`。
3. 網站路徑為 `https://<user>.github.io/<repo>/`。
4. 保留 `.nojekyll`。

## 8. 驗證標準

### 8.1 產出檢查（每次產報表）

- 沒有殘留的 `{{`，也沒有新增 HTML 註解。
- 五個章節齊全且順序正確。
- 列數正確：大盤指標 9、籌碼 4、法人五日 5、強勢 3、弱勢 3、成交排行 10。
- 每列大盤指標都有 tag；均線數值依與收盤比較上色。
- 頁首日期、`資料日期` 與檔名三者一致；更新時間為本次執行的實際時間。
- 首頁 `LATEST` 已更新、`LIST` 已插入新列，兩組標記未被刪除。

### 8.2 網站檢查

- 首頁與報表頁在 375px 與 1280px 皆不橫向捲動，寬表於自身容器橫捲。
- 章節標題五個色調依序套用；`td.up` 為紅、`td.down` 為綠。
- 首頁所有連結指向存在的報表頁；報表頁可回到首頁。
- 系統偏好為淺色時仍維持暗色，文字對比符合 4.5:1。
- 全站無 JavaScript，停用 JS 後功能完全不受影響。
