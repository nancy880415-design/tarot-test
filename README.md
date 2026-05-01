# 每日塔羅占卜系統

一個可以發布到 GitHub Pages 的塔羅牌每日占卜小工具，後台用 Firebase Firestore 同步資料。

## 檔案說明

| 檔案 | 用途 |
|------|------|
| `index.html` | 公開的占卜頁面，給使用者抽牌用 |
| `admin.html` | 後台管理頁面，需密碼登入，用來設定每天的牌陣 |
| `firestore.rules` | Firebase Firestore 安全規則 |
| `README.md` | 你正在看的這份說明 |

## 一次設定（5 分鐘）

### 1. 啟用 Firebase Firestore

1. 打開 https://console.firebase.google.com/project/tarot-test-10961
2. 左側選單 → **Firestore Database** → 點「建立資料庫」
3. 選「在生產模式中啟動」(Production mode)
4. 區域選離你近的（例如 `asia-east1` 台灣／`asia-northeast1` 東京）
5. 點「啟用」

### 2. 設定 Firestore 安全規則

1. Firestore Database → **規則** 分頁
2. 把 `firestore.rules` 整份內容貼進去
3. 點「發布」

### 3. 改掉預設密碼（重要！）

打開 `admin.html`，找到這一行：

```js
const ADMIN_PASSWORD = "tarot-admin-2026";
```

改成你自己的密碼（建議 12 位以上，英數混合）。

> ⚠️ 注意：因為密碼寫在前端 JS 裡，懂 F12 的人會看到。這套適合「不想被路人改、但不擔心駭客」的場景。如果要更安全，請改用 Firebase Auth。

## 發布到 GitHub Pages

```bash
# 1. 在你的 GitHub 建立一個新 repo，例如 tarot
# 2. 在這個資料夾執行：
git init
git add index.html admin.html README.md firestore.rules
git commit -m "init tarot"
git branch -M main
git remote add origin https://github.com/你的帳號/tarot.git
git push -u origin main
```

接著在 GitHub 上：

1. Settings → Pages
2. Source 選 `Deploy from a branch`
3. Branch 選 `main` → `/(root)` → Save
4. 等 1–2 分鐘，會給你一個網址，例如：
   - 公開頁：`https://你的帳號.github.io/tarot/`
   - 後台：`https://你的帳號.github.io/tarot/admin.html`

## 每日使用流程

1. 打開 `admin.html` → 輸入密碼進入
2. 選好今天的日期、張數
3. 為每張牌選牌名、（選填）上傳圖片、填標題與解說
4. 點「儲存到雲端」
5. 公開網址 `index.html` 立刻會看到當天的內容

## 資料結構

Firestore 集合 `tarot_readings`，每份文件用日期當 ID（例如 `2026-05-01`）：

```json
{
  "date": "2026-05-01",
  "brandName": "每日塔羅",
  "brandSub": "今日運勢占卜",
  "contactLabel": "預約占卜",
  "contactLink": "https://lin.ee/xxxxx",
  "contactCta": "如果你想深入了解更多…",
  "cardCount": 3,
  "cards": [
    {
      "name": "戀人",
      "imageData": "data:image/jpeg;base64,...",
      "title": "心意相通",
      "tip": "今天適合主動表達",
      "reading": "段落一…\n\n段落二…"
    }
  ],
  "updatedAt": "2026-05-01T08:30:00.000Z"
}
```

## 圖片限制

- 圖片用 Base64 直接存進 Firestore
- 上傳時會自動壓縮到 600×900 / JPEG quality 0.82
- Firestore 單份文件上限 1MB，所以一天最多大約 4–5 張中等尺寸的圖
- 如果需要放更大／更多的圖，請改用 Firebase Storage

## 常見問題

**Q：部署到 GitHub Pages 後出現「連線失敗」？**
A：檢查 Firestore 規則是否已發布、Firestore 是否已啟用、Firebase 設定的 `authDomain` 是否包含你的 GitHub Pages 網域（通常不需要，但若有問題可到 Firebase Console → Authentication → Settings → 授權的網域 加入）。

**Q：我想換成另一個 Firebase 專案？**
A：把 `index.html` 與 `admin.html` 兩個檔案裡的 `firebaseConfig` 物件都換成你的設定即可。

**Q：忘記管理密碼？**
A：打開 `admin.html` 重新看一次 `ADMIN_PASSWORD` 的值，或直接改成新密碼後重新部署。

**Q：使用者那邊看到的時間不準？**
A：日期是用使用者本機的時區計算。所有人都會看到「他自己當地的今天」。如果你希望大家一律看「台灣時間的今天」，需要改 `index.html` 的 `today()` 函式。
