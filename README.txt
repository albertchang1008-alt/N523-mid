# N523 期中成績個別輔導問卷系統

這是一套給導師班使用的期中考成績輔導系統，包含學生問卷、家長回饋、教師後台與 Google Apps Script 後端。系統會依學生各科成績產生反思題目，讓學生勾選多個原因與策略，也可自行填寫「其他」內容；第二階段會依學生回答進行追問，遇到抗拒、倦怠、想休學、不想念書等回答時，會改用關心式語氣追問。教師後台可查看學生與家長回饋、AI 建議、雙方差異分析，並一鍵產生可設定字數的輔導紀錄。

## 檔案說明

- `index.html`：學生端問卷。學生輸入學號與姓名後查詢成績，填寫各科反思、作息、讀書方法、學習盲點與承諾，並進入追問流程。
- `Parent.html`：家長端問卷。家長透過專屬 token 連結查看學生自填內容與師生追問紀錄，並填寫家庭觀察與支持方式。
- `Dashboard.html`：教師後台。提供班級總覽、填寫狀況、科目分析、壓力熱點、AI 建議庫、家長連結、家長回饋、學生與家長對照、AI 輔導紀錄。
- `Code.gs`：Google Apps Script 後端。負責讀寫 Google 試算表、驗證學生與教師、產生家長連結、呼叫 Gemini 生成追問與輔導建議。
- `image.png`：前端頁面使用的背景圖。

## 試算表需求

請建立一份 Google 試算表，並準備以下工作表：

- `班級成績總表`
- `問卷回饋表`
- `家長連結`
- `家長回饋表`
- `教師帳號`

成績表預設格式：

- 第 5 列：科目名稱
- 第 6 列開始：學生資料
- B 欄：學號
- C 欄：姓名
- D 欄起：各科成績

教師帳號表格式：

- 第 1 列為標題：`帳號`、`密碼`、`顯示名稱`
- 第 2 列開始放實際帳號資料

## 後端部署

1. 在 Google 試算表中開啟 `擴充功能 > Apps Script`。
2. 將 `Code.gs` 內容貼到 Apps Script 專案中。
3. 設定 Gemini API Key。建議使用 Apps Script 的 Script Properties，不要把金鑰硬寫在程式碼或公開檔案中。
4. 部署為網頁應用程式：
   - 執行身分：我
   - 存取權：知道連結的任何人
5. 複製 Web App URL，填入三個前端 HTML 的 `const API = "...";`。

建議的 Script Properties：

- `GEMINI_API_KEY`：Gemini API 金鑰
- `DASHBOARD_TOKEN`：後台 URL token

若目前 `Code.gs` 仍使用硬編碼設定，建議改為讀取 `PropertiesService.getScriptProperties()`。

## PUBLIC_PARENT_URL 設定方式

`PUBLIC_PARENT_URL` 是後端產生「家長專屬連結」時使用的家長端公開網址。它位於 `Code.gs` 的 `CFG` 設定區：

```javascript
const CFG = {
  PUBLIC_PARENT_URL: "",
};
```

### 什麼時候需要設定

如果 `Parent.html` 是放在 GitHub Pages、校內網站或其他公開網址，就應該設定 `PUBLIC_PARENT_URL`。例如：

```javascript
PUBLIC_PARENT_URL: "https://你的帳號.github.io/N523-mid/Parent.html"
```

設定後，教師後台產生的家長連結會長得像：

```text
https://你的帳號.github.io/N523-mid/Parent.html?parent_token=xxxxxxxx
```

家長開啟這個連結後，`Parent.html` 會讀取網址中的 `parent_token`，再呼叫 Apps Script API 取得學生資料。

### 什麼時候可以留空

如果沒有另外部署 `Parent.html`，而是使用 Apps Script 內建的 HTML 頁面，`PUBLIC_PARENT_URL` 可以留空：

```javascript
PUBLIC_PARENT_URL: ""
```

留空時，後端會自動使用 Apps Script Web App 網址並加上 `?page=parent`，產生類似：

```text
https://script.google.com/macros/s/部署ID/exec?page=parent&parent_token=xxxxxxxx
```

這種方式的前提是 Apps Script 專案內也有 `Parent.html` 檔案，且 Web App 已重新部署。

### 建議使用方式

目前本專案有三個本機前端檔案：`index.html`、`Dashboard.html`、`Parent.html`。若你主要用本機或 GitHub Pages 開前端，建議把 `Parent.html` 也部署到同一個公開位置，然後把 `PUBLIC_PARENT_URL` 設成該公開網址。

請注意：

- `PUBLIC_PARENT_URL` 必須指向 `Parent.html`，不是 `index.html` 或 `Dashboard.html`。
- 網址最後可以是 `Parent.html`，不需要自己加 `?parent_token=...`，系統會自動加。
- 修改 `Code.gs` 的 `PUBLIC_PARENT_URL` 後，必須重新部署 Apps Script Web App，後台產生的新家長連結才會套用。
- 舊連結不會自動改網址，若改過 `PUBLIC_PARENT_URL`，建議重新產生家長連結。

## 前端部署

可將 `index.html`、`Parent.html`、`Dashboard.html`、`image.png` 放在同一個公開網站目錄，例如 GitHub Pages 或校內網站空間。

部署後請確認：

- 三個 HTML 內的 `API` 都指向最新 Apps Script Web App URL。
- `Code.gs` 的 `PUBLIC_PARENT_URL` 設為家長端公開網址，或確保產生的家長連結能正確開啟 `Parent.html`。
- 修改 Apps Script 後，必須重新部署新版本。
- 修改 HTML 後，測試時請強制重新整理瀏覽器，避免載入舊快取。

## 使用流程

學生端：

1. 學生輸入學號與姓名。
2. 系統顯示期中考各科成績。
3. 學生針對每科勾選一個或多個反思原因與改進策略，也可勾選「其他」自行填寫。
4. 學生填寫學習自評、讀書方法、盲點、承諾、作息與壓力。
5. 系統進入第二階段追問，包含各科追問、課業整體追問與生活狀態追問。
6. 模糊回答會排入延伸追問；抗拒或倦怠回答會優先以關心語氣追問。

家長端：

1. 教師後台為學生產生家長專屬連結。
2. 家長開啟連結後查看學生自填狀況與師生追問紀錄。
3. 家長填寫對成績反應、壓力、作息與讀書方法的觀察、家庭支持方式與建議。

教師後台：

1. 教師登入後台。
2. 查看班級填寫率、不及格科目、原因分布、高壓學生、AI 建議。
3. 在學生與家長對照頁查看雙方資料。
4. 可產生雙方差異分析。
5. 可設定字數並一鍵產生 AI 輔導紀錄。

## 目前重要維護建議

- 不要把 Gemini API Key 放進公開檔案。
- 學生重複填寫時，後台統計應只取最新一次提交，避免同一學生重複計算。
- 家長端應讀取學生最新一次提交內容，不應讀到舊回覆。
- 家長 token 建議加入建立時間與失效機制，避免舊連結永久有效。
- 教師登入密碼目前為試算表明文比對，正式使用時建議至少改為雜湊或加強後台 token。
- AI 生成內容應作為輔助，不應直接當作最終診斷或處置依據。

## 建議測試清單

- 學生可成功查詢成績。
- 科目原因與策略可複選。
- 勾選「其他」時必填文字。
- 模糊回答會產生延伸追問。
- 抗拒、倦怠、想休學類回答會使用關心式追問。
- 學生送出後，後台能看到完整各科與延伸追問紀錄。
- 家長連結可開啟，且只能提交一次。
- 後台可以產生家長連結、差異分析與輔導紀錄。
- Apps Script 重新部署後，前端 API 呼叫正常。

