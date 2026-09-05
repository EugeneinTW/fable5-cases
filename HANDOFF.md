# HANDOFF — AI 模型案例庫

> 最後更新：2026-09-05（新增 GPT-6 Astra 分頁、搜尋改走 cookie 路徑、排程改每日）

## 你是誰、要做什麼

你在幫 Eugene 維護「AI 模型案例庫」：收集 X 上 AI 模型的**實測案例**（有具體產出物 + 可複製的 prompt），做成單頁靜態網站。

核心價值不是「蒐集得多」，而是**每張卡片都能讓人照著重現**。所以收錄標準比數量重要，寧缺勿濫——一輪每個模型各加 0-5 筆是正常的。

網站：https://eugeneintw.github.io/fable5-cases/

## 檔案結構

```
fable5-cases/           ← 這個 repo（GitHub Pages 版）
├── index.html          ← 全部內容都在這一個檔案裡
└── media/              ← 卡片畫面，檔名 = 推文ID.jpg

~/Downloads/
├── Fable5案例庫.html          ← 本地版，內容同 index.html
└── Fable5案例庫_media/        ← 本地版媒體資料夾
```

**兩份 HTML 必須同步更新。** 兩者唯一的差異是 `const MEDIA_DIR` 這一行（本地版是 `Fable5案例庫_media/`，repo 版是 `media/`）。檔名之所以沒跟著改名，是為了不動既有排程與捷徑的路徑。

### index.html 裡的三個區塊

| 區塊 | 說明 |
|---|---|
| `const CASES = [...]` | Fable 5 分頁的資料（`id="tab-cases"`） |
| `const ASTRA_CASES = [...]` | GPT-6 Astra 分頁的資料（`id="tab-astra"`） |
| `makePanel(data, ids)` | 兩個分頁共用的渲染邏輯 |

兩個陣列的物件格式完全相同。**加案例只需要把物件 append 到對的陣列尾端，不要動 UI 邏輯。**

卡片會依 `caseTime()` 自動排序（優先用 `date` 欄位，否則從推文 ID 的 Snowflake 時戳推導），最新在前，所以不用在意插入位置。圖片也會自動用 `MEDIA_DIR + 推文ID + '.jpg'` 對應，不用寫 `img` 欄位（找不到檔案時 `onerror` 會把 img 移除，版面不會破）。

## 資料格式

```js
{
  cat:"遊戲",                    // 只能是：遊戲 / 模擬視覺化 / 文件簡報 / 創意藝術 / 工程研究
  title:"標題（繁中）",
  desc:"一兩句話。包含亮點，以及誠實保留（作者自註的缺陷、無法驗證的部分要寫出來）",
  author:"@handle (顯示名稱)",
  likes:1234,                    // 照實
  url:"https://x.com/xxx/status/推文ID",
  ptype:"原文",                  // 只能是「原文」或「重建」，UI 篩選與統計靠它
  prompt:`...`                   // 用反引號；內容裡不能有反引號
}
```

- **`ptype:"原文"`** = 推文（或其討論串、附圖）裡有完整的原始指令，直接照抄
- **`ptype:"重建"`** = 依案例描述補寫一份可執行的版本。重建的 prompt 要寫得夠具體、能真的跑出類似結果，不是複述推文內容
- 若原文被推文字數截斷，用「重建」，並在 desc 註明前段是原文、後段是補完

`desc` 的誠實保留是這個庫的特色，不要省略。實際寫過的例子：「作者自註花了 3 次嘗試才做出來，而且有些地方還是壞的」「推文只有 demo 影片，未公開 prompt 與試玩連結」「⚠️ 故事型，無法驗證」。

## 收錄標準

### 三層去重（每層都要做）

1. **讚數門檻**：Fable 5 ≥ 150、GPT-6 Astra ≥ 40。門檻不同是因為 Astra 於 2026-09 初才發布、生態還年輕；等 Astra 一輪的合格案例穩定超過 8 筆就該調高。
2. **ID 去重**：跟**兩個陣列**都比對（從 `url` 抽 status ID）。同一則推文不可同時出現在兩個分頁。
3. **主題去重**：同一個案例常被多個帳號發（原作者、搬運號、新聞號），ID 不同但講的是同一個產出物。加入前要比對目標陣列裡既有卡片的 `title` 與 `desc`。
   - 唯一例外：新來源是「原作者」而庫內收的是「轉述」時，替換成原作者版
   - **不同模型做同類題目不算重複**——Fable 5 和 Astra 各自做的 3D 場景，是兩個分頁各自的案例

### 只收「實際案例」

要有具體產出物：遊戲、模擬、文件、工具、藝術作品。

### 不收什麼（含實際踩過的例子）

| 類型 | 實例 |
|---|---|
| benchmark／模型互比 | 「Astra vs Fable 5.1 的 Blender 畫質對比」「機械手臂 95% vs 40% vs 5%」。**即使裡面提到某個作品也算**——包括「新版變差了」的退步抱怨文 |
| 純觀點／使用感想 | 「用了幾天覺得 Astra 開發體驗很好」 |
| 合集轉發文 | 「這 10 個最令人眼前一亮」「我蒐集了幾十個 prompt」 |
| 教學引流文 | 「16 分鐘教你做 $50,000 網站，Save this 🔖」 |
| 廠商產品發表／官方 demo | OpenAI 發布文、NVIDIA 賀文、廠商的 pipeline 展示 |
| 產出物不是該模型做的 | 「用 Astra 寫 prompt、影片由 MiniMax 生成」——這不算 Astra 的案例 |
| 資訊不足無法寫卡片 | 「i made this with gpt-6 btw」「Generated in 26 minutes, 10,460,032 tokens used」——沒說做了什麼 |
| 附圖與案例無關 | 作者說「做了一款遊戲」，附的卻是另一款遊戲的啟動器截圖 → 無法驗證，跳過 |
| 洗稿／內容農場 | 曾遇到兩個帳號（8,320♥ 與 5,963♥）貼一字不差的文，還附 localhost 連結無法驗證 |
| jailbreak／system prompt 外洩 | — |

變現故事型（無 demo、無法驗證的賺錢敘事）預設跳過；互動極高才可收，且 desc 必須標注「⚠️ 故事型，無法驗證」。

搬運帳號（「This guy built...」）盡量溯源到原作者；分不出來就照收並在 desc 註明是轉述與原作者名。

### 圖片一定要親眼看過

下載完縮圖後**用 Read 工具打開來看**，確認畫面真的是該案例的產出物。踩過兩次：一次作者附的是別款遊戲的啟動器截圖（該案例直接跳過）、一次縮圖是插件安裝步驟而非成品（照收但在 desc 註明）。

## 資料抓取

**X API v2 自 2026-09-05 起回 402 credits depleted，已棄用。** 不要再打任何 `api.x.com/2` 讀取端點。

改走 cookie 路徑（本機 bird CLI → GraphQL，零 API 額度）：

| 用途 | 工具 |
|---|---|
| 搜尋 | `~/.config/macshrimp/x_search.py` |
| 書籤 | `~/.config/macshrimp/x_bookmarks.py` |
| 單則／討論串 | `bird read <id>` / `bird thread <id>` |

憑證放在本機 `~/.config/macshrimp/`，**不在這個 repo 裡**（repo 是 public）。cookie 過期整條路會斷，是唯一需要維護的東西。

### x_search.py 的兩個設計重點

1. **`--min-faves` 用 X 的 `min_faves:` 運算子做伺服器端篩選**，回傳量小，不必本地過濾整片洗版內容。
2. **自動切窗**：bird 的 search 是「最新」排序（不是相關度），而且單一時間窗分頁到約 40 則就會停（X 的 Latest cursor 會提早耗盡）。腳本會用 `since_time:`/`until_time:`（Unix 時戳）把回傳達上限的時間段對半再切，遞迴掃完整個視窗。

實測 96h／min_faves 40 的熱門查詢 → 切成約 46 次呼叫、跑 3 分鐘、回 589 則。**這比舊 API 的 100 則上限涵蓋得多**，所以候選池會很大、跑幾分鐘是正常的。候選多不等於要多收。

### 原文 prompt 常常藏起來

- **藏在附圖裡**：日文作者愛寫「プロンプトもスクショ載せておきます」，要把圖抓下來用 Read 讀才判得出 `ptype`
- **藏在討論串下一則**：推文寫「Prompt below 👇」時，用 `bird thread <推文ID>` 把串抓下來，原文通常在作者自己的第二則

這兩招各撈到過完整的原文 prompt，值得每次都試。

## 排程

`fable5-cases-daily-update`，**每天 09:00**，每個模型各跑一次查詢。

**視窗是 48 小時但每天跑，是刻意重疊的**：一則推文可能發文當天只有 30 讚、第三天才衝過門檻，視窗如果等於執行間隔就會永久錯過。48h／每日 = 每則推文有兩次被檢視的機會，重複的部分由 ID 去重擋掉。

> 一般原則：**輪詢視窗要大於執行間隔**，否則邊界上的東西會漏掉。

## 改完一定要驗證

1. **語法與資料**：把 `const CASES = [` 到 `];`、`const ASTRA_CASES = [` 到 `];` 兩段抽出來丟 `new Function` 跑，檢查：兩個陣列都能 parse、各自無重複推文 ID、**兩陣列之間也無重複**、`cat` 值合法、`ptype` 只有「原文/重建」、每筆對應的 `媒體資料夾/推文ID.jpg` 確實存在。
2. **插入前先確認陣列原本最後一筆的結尾 `}` 有逗號變成 `},`**，否則語法會壞。
3. **瀏覽器**：開檔案確認無 console error、三個分頁都能切、兩個面板的篩選互不干擾。
4. **線上**：`curl -s '<pages-url>/index.html?cb=$(date +%s)' | grep -c 'title:"'` 數卡片數，另外抽查新圖片是否回 200。Pages 重建通常要 1-3 分鐘。

### Pages 部署卡住的救命招

build_type 是 legacy（Deploy from a branch）。曾遇到已 push 但網站不更新——根因是 Actions 的「pages build and deployment」佇列被卡死的 rerun 殭屍 job 堵住（force-cancel 被拒、猛推空 commit 無效）。

解法：`gh api -X POST repos/EugeneinTW/fable5-cases/pages/builds` 直接觸發原生 legacy 重建，繞過 Actions 佇列。

## 其他踩過的坑

- **不要用瀏覽器捲 `x.com/i/bookmarks` 抓書籤**。那頁會轉址到 `/i/history` 的書籤分頁，而且是虛擬列表（一次只渲染 2 則）。實測用 Chrome 捲只撈到 4 則，`bird bookmarks` 一抓是 100 則。要書籤就用 `x_bookmarks.py`。
- **Chrome 讀到的推文文字可能是 X 自動翻譯後的中文版**。寫 desc 沒差，但要引用原文時記得先按「顯示原文」。
- **查「誰在燒 API 額度」時，先讀那支程式的原始碼再下結論**。曾把每天跑的情報任務誤判成主要消耗者，實際上它早在 2026-06-20 就改用 cookie 了，檔案開頭的 docstring 就寫著「不消耗 X API v2 credits」。
- **要證明某條路徑「不吃額度」，用 `GET https://api.x.com/2/usage/tweets`（只接受 app bearer）在執行前後各查一次比對 `project_usage`。** 這個端點回報的是貼文讀取上限（3,000,000/月，每月 23 號重置），跟讓 API 回 402 的 credit 餘額是**兩個不同的計費表**——credit 餘額沒有 API 可查，只能在 Developer Portal 帳單頁看。

## 歷史決策

- **2026-06-11** 建立，最初只有 Fable 5 案例
- **2026-07-06** 卡片改為依時間排序（最新在前），新案例可直接 append
- **2026-09-05** 新增 GPT-6 Astra 分頁；渲染邏輯抽成 `makePanel()` 兩邊共用
- **2026-09-05** 站名由「Fable 5 案例庫」改為「AI 模型案例庫」（repo 名、Pages URL、本地檔名維持舊名，避免斷連結）
- **2026-09-05** 搜尋全面改走 cookie 路徑，排程由每四天改為每天
