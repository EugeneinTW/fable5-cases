# AI 模型案例庫

AI 模型的社群實測案例合集——看別人做了什麼，複製 prompt 自己重現。目前有三個分頁：**TypeSafe Jev**、**GPT-6 Astra**、**Claude Fable 5**（repo 名沿用最早的 Fable 5 案例庫，沒有改）。

🔗 **線上瀏覽**: https://eugeneintw.github.io/fable5-cases/

## 特色

- 三個模型分頁、共 210 個案例（Jev 35、Astra 84、Fable 5 91），五大分類：遊戲、模擬視覺化、文件簡報、創意藝術、工程研究
- 每張卡片附案例畫面、原推連結、可一鍵複製的 prompt
- Prompt 標籤誠實標注：**原文**（推文直接引用）vs **重建**（依描述補寫的可執行版）
- Jev 分頁的 prompt 多為重建版的「給 coding agent 的建置指令」，含 Choice／Score／Noul 題目設計與信心值門檻
- 純靜態單頁，無任何依賴

## 結構

- `index.html` — 案例庫本體（資料在檔案開頭的三個陣列：`JEV_CASES`、`ASTRA_CASES`、`CASES`，格式相同）
- `media/` — 案例畫面（推文截圖/影片縮圖，檔名為推文 ID，卡片自動對應）
- `HANDOFF.md` — 維護手冊：收錄標準、三層去重、卡片格式、驗證與部署步驟、踩過的坑

## 新增案例

在 `index.html` 對應模型的陣列照格式加一筆，畫面圖丟進 `media/`（檔名取推文 ID 即自動對應）。細節與收錄標準見 `HANDOFF.md`。

## 工作流

每天由 Claude Code 的排程任務跑一輪：用 [bird](https://github.com/steipete/bird) CLI 走 X 的 GraphQL 搜尋（cookie 登入、不用 X API），伺服器端用讚數門檻篩，再做 ID／主題三層去重，寫卡片、抓縮圖、node 驗證後推上來。判斷標準與每一步的細節都寫在 `HANDOFF.md`。搜尋腳本與排程提示詞目前在維護者本機，尚未放進 repo。

> 資料來源為 X 公開推文，畫面縮圖版權屬原作者；本庫僅作學習索引用途。
