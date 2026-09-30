# 問題分析交換站

公司電腦 → 加密存到 GitHub → 手機解密 → 貼給 Claude 分析 → 加密回填 → 公司電腦解密 → 貼給 MS Copilot 執行。

## 部署（一次性）

1. 在 GitHub 建立 repo（例如 `issue-relay`），上傳 `index.html`、`manifest.webmanifest`、`icon-*.png` 與 `.nojekyll`。
2. Settings → Pages → Source: *Deploy from a branch*，Branch: `main` / `(root)`。
3. 網址會是 `https://<帳號>.github.io/issue-relay/`。
4. 建立 Fine-grained token：GitHub → Settings → Developer settings → Fine-grained tokens
   - Repository access：**Only select repositories** → 只選這個 repo
   - Permissions：**Contents → Read and write**（其他都不給）
   - 設定到期日（建議 90 天內）
5. 在網頁「解鎖」頁的 ⚙️ GitHub 設定填入 token（公司電腦、手機各填一次）。

第一次解鎖時若沒有資料，會用你輸入的密語建立新的加密資料庫。存檔需要 token；沒有 token 的裝置只能讀取。

## 資料結構

```
data/index.json          案件清單（標題加密；id、日期、狀態、紀錄筆數為明文）
data/cases/<id>.json     單一案件：原始問題、處理紀錄、結案摘要（全部加密）
```

每次存檔都是一個 commit，同時更新 index 與案件檔。若另一台裝置剛好先存，網頁會自動重新載入最新版本再存一次，不會互相覆蓋。
舊版的單一 `data/cases.json` 會在第一次解鎖時自動升級（需 token）。

## 使用流程

| 步驟 | 裝置 | 操作 |
|---|---|---|
| 1 | 公司電腦 | ➕ 新增 → 貼上問題描述與程式碼 → 🔐 加密並儲存 |
| 2 | 手機 | 解鎖 → 點案件 → 📤 分享給 Claude（或 📋 複製）→ 在 Claude App 送出 |
| 3 | 手機 | 複製 Claude 回覆 → 新增紀錄選「Claude 回覆」→ 🔐 加密存入 |
| 4 | 公司電腦 | 解鎖 → 點案件 → 📋 複製最新計劃給 MS Copilot → 逐步執行 |
| 5 | 公司電腦 | 執行出錯時：新增紀錄選「Copilot 執行結果」貼上錯誤 → 回到步驟 2 |
| 6 | 任一 | 解決後：📝 請 Claude 產生結案摘要 → 回覆存入「📌 結案摘要」→ 自動標記完成 |

每個案件保存完整處理紀錄；第二輪起送給 Claude 的內容會自動附上原始問題與全部紀錄，可直接開新對話貼上。

**結案摘要**固定格式：一句話摘要、問題現象、根本原因、解決方式、關鍵修改、預防與注意事項、分類、關鍵字，供日後建立查詢網站使用。

**手機加到主畫面**：iPhone 用 Safari 分享 →「加入主畫面」；Android 用 Chrome 選單 →「安裝應用程式 / 加到主畫面」。

**更換密語**：解鎖後在「解鎖」頁展開 🔁 更換密語，會用新密語重新加密全部資料並提交；其他裝置之後要改用新密語。

## 安全說明

- AES-256-GCM 加密，金鑰由 PBKDF2-SHA256（600,000 次）從密語推導；密語只存在記憶體，閒置 10 分鐘自動上鎖。
- repo 是 public 時，**任何人都能下載密文**，安全性完全取決於密語強度：請用 16 字以上、不與其他地方重複的密語。
- 刪除案件或更換密語後，git 歷史仍保留舊密文（舊版本可用舊密語解開）。
- 解密後的內容會出現在剪貼簿與 Claude / Copilot 對話中，那部分不受這裡的加密保護。
- 案件全部保留；單一案件檔超過 1MB 才會無法讀取（一般案件約數十 KB）。
