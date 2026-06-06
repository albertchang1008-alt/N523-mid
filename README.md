# 導生班輔導問卷系統

這個資料夾是整理後的導生班輔導問卷系統專案，已和舊的成績入口快照分開。

## 檔案

- `index.html`：學生端問卷，已使用 GitHub 最後一版貼上的程式碼還原。
- `入口.html`：本機入口頁，可開啟教師後台與家長問卷。
- `Dashboard.html`：教師後台管理介面。
- `Parent.html`：家長學習支持問卷。
- `FollowUp.html`：學生期中後追蹤問卷，顯示原成績與期中問卷內容，再詢問目前執行情況。
- `image.png`：問卷與後台背景圖。
- `gas/README.md`：Google Apps Script 後端整理提醒。
- `gas/FollowUpBackend.gs`：學生期中後追蹤問卷的 Apps Script 後端增補。

## 使用方式

1. 開啟 `入口.html`，選擇要進入教師後台或家長問卷。
2. 若要開學生問卷，可開啟 `index.html`。
3. 若要直接開後台，可開啟 `Dashboard.html`。
4. 若要直接開家長端，可開啟 `Parent.html`。
5. 若要測試學生追蹤問卷頁面，可開啟 `FollowUp.html`，正式使用時需要帶 `follow_token` 參數。

## 學生期中後追蹤問卷

`FollowUp.html` 是新增的第二次追蹤問卷。它會先顯示學生期中成績、原本填寫的學習盲點、行動承諾與讀書作息，再讓學生回報目前是否有執行、卡在哪裡、希望老師如何協助。

教師後台 `Dashboard.html` 已新增「發送追蹤問卷」頁面，可為已完成期中問卷的學生產生追蹤問卷連結，並顯示追蹤問卷狀態：未發送、未填寫、已填寫。

「老師想追蹤的重點」不呼叫 AI，而是由後端 `generateTeacherFollowUpFocus_()` 依下列資料自動整理：

- 不及格科目與最低分科目。
- 期中問卷中的不及格原因。
- 期中問卷中的改進策略。
- 學生行動承諾與希望老師協助的內容。
- 原本壓力指數與作息資料。

正式部署時，請將 `gas/FollowUpBackend.gs` 貼到同一個 Google Apps Script 專案，並依 `gas/README.md` 的說明把新的 action 接到原本的 `doPost(e)`。

## 注意

目前工作區中的 `grade-portal-snapshot-2026-05-20/` 是另一套成績入口快照，沒有搬進這個資料夾，避免和導生班輔導問卷系統混用。

若 Apps Script 後端是部署在 Google Apps Script，請以原 Apps Script 專案中的 `Code.gs` 為準，再另外放入 `gas/` 資料夾保存。
