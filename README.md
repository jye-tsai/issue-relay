# 問題分析交換站

公司電腦 → 加密存到 GitHub → 手機解密 → 貼給 Claude 分析 → 加密回填 → 公司電腦解密 → 貼給 MS Copilot 執行。

## 部署（一次性）

1. 在 GitHub 建立 repo（例如 `issue-relay`），上傳 `index.html` 與 `.nojekyll`。
2. Settings → Pages → Source: *Deploy from a branch*，Branch: `main` / `(root)`。
3. 網址會是 `https://<帳號>.github.io/issue-relay/`。
4. 建立 Fine-grained token：GitHub → Settings → Developer settings → Fine-grained tokens
   - Repository access：**Only select repositories** → 只選這個 repo
   - Permissions：**Contents → Read and write**（其他都不給）
   - 設定到期日（建議 90 天內）
5. 在網頁「解鎖」頁的 ⚙️ GitHub 設定填入 token（公司電腦、手機各填一次）。

第一次解鎖時若沒有 `data/cases.json`，會用你輸入的密語建立新的加密資料庫。

## 使用流程

| 步驟 | 裝置 | 操作 |
|---|---|---|
| 1 | 公司電腦 | ➕ 新增 → 貼上問題描述與程式碼 → 🔐 加密並儲存 |
| 2 | 手機 | 解鎖 → 點案件 → 📋 複製給 Claude → 開 Claude App 貼上 |
| 3 | 手機 | 複製 Claude 回覆 → 📥 貼上 → 🔐 加密存回 |
| 4 | 公司電腦 | 解鎖 → 點案件 → 📋 複製給 MS Copilot → 貼到 Copilot 逐步執行 |

## 安全說明

- AES-256-GCM 加密，金鑰由 PBKDF2-SHA256（600,000 次）從密語推導；密語只存在記憶體，閒置 10 分鐘自動上鎖。
- repo 是 public 時，**任何人都能下載密文**，安全性完全取決於密語強度：請用 16 字以上、不與其他地方重複的密語。
- 刪除案件後，git 歷史仍保留舊密文。
- 解密後的內容會出現在剪貼簿與 Claude / Copilot 對話中，那部分不受這裡的加密保護。
- 資料檔 1MB 以上 GitHub API 無法直接讀取，請定期刪除已完成案件。
