# 專案追蹤系統 · 內網部署說明（交給 IT）

## 這份部署是什麼

把 `index.html`（單一檔案）放在**公司內部網站伺服器**，只有連公司網路的人打得開。
**資料仍存在 Firebase 雲端**（Firestore），這是刻意保留的架構，安全性由 Firebase App Check + reCAPTCHA 把關。

> 如果公司法遵要求「資料完全不得離開公司」，這份方案**不適用**，需要另外把後端與資料庫都自建在內部（工程較大）。本說明是「網頁內網、資料雲端」版本。

## 先決條件（很重要）

1. **內網電腦要能連外網**：使用者的瀏覽器需要能連到
   - `*.googleapis.com`、`*.firebaseio.com`、`*.gstatic.com`（Firebase）
   - `www.google.com/recaptcha`（App Check 驗證）
   - `script.google.com`（寄送 Email 通知）
   若公司防火牆有白名單，請放行以上網域。
2. **用「網域名稱」對外，不要用純 IP**：例如 `http://tracker.pershing.local`，**不要**用 `http://192.168.1.50`。
   原因：App Check 的 reCAPTCHA v3 只認網域名稱，純 IP 會驗證失敗 → 一旦 App Check 強制執行，全部人會被擋。

---

## 部署步驟

### 步驟 1：放上內部網站伺服器

擇一即可：

**A. IIS（Windows Server，最常見）**
1. 開「伺服器管理員 → 新增角色 → Web 伺服器 (IIS)」。
2. 把 `index.html` 複製到網站根目錄（預設 `C:\inetpub\wwwroot\`）。
3. 在「站台繫結」設定一個內部主機名稱（見步驟 2）。

**B. nginx / Apache（Linux）**
1. 把 `index.html` 放到網站根目錄（nginx 預設 `/usr/share/nginx/html/`）。
2. 設定 server_name 為內部主機名稱。

**C. 臨時 / 測試用（快速起一台）**
在一台內網主機上放 `index.html` 後執行：
```
# Python
python -m http.server 80
# 或 Node
npx http-server -p 80
```
（正式用建議還是走 IIS/nginx。）

### 步驟 2：給一個內部網域名稱

請 IT 在內部 DNS 加一筆紀錄，例如：
```
tracker.pershing.local  →  該伺服器的內網 IP
```
之後大家用 `http://tracker.pershing.local` 開啟。
（有內部憑證可用 https 更好，但 http 在內網也可運作。）

### 步驟 3：把內部網址加進 Firebase / reCAPTCHA（關鍵）

否則 App Check 會判定為「未驗證」，強制執行後會把內網使用者擋在外面。

1. **reCAPTCHA 後台**（https://www.google.com/recaptcha/admin）
   - 進入「北祥追蹤系統」金鑰 → 設定 → **網域** → 新增 `tracker.pershing.local`（你實際的內部網域）→ 儲存。
2. **（若之前有設 API 金鑰網域限制）** Google Cloud Console → API 和服務 → 憑證 → 該 API key → HTTP 參照網址 → 新增 `tracker.pershing.local/*`。

### 步驟 4：更新系統內的網址（讓通知信連到內網）

用 ADMIN 後台不需要改；但通知信裡的「登入連結」預設是 Vercel 網址。若要改成內網網址：
- 開 `index.html`，找到這一行（檔案上方）：
  ```
  const SITE_URL="https://hackathon-vote-6ner.vercel.app/";
  ```
  改成你的內網網址，例如：
  ```
  const SITE_URL="http://tracker.pershing.local/";
  ```
- 存檔後重新放上伺服器（覆蓋）。

### 步驟 5：測試

1. 在內網某台電腦用 `http://tracker.pershing.local` 開啟 → 能登入、看到資料。
2. Firebase → App Check → API → Cloud Firestore → 看指標，內網流量應顯示「**已驗證**」（可能延遲 5–15 分鐘）。
3. 全部「已驗證」後，App Check 維持強制執行即可。

---

## 常見問題

- **打不開 / 一直轉圈**：多半是內網不能連外網（Firebase 連不到）。請放行先決條件的網域。
- **登入後看不到資料、或被擋**：內部網址沒加進 reCAPTCHA 網域（步驟 3），或用了純 IP。改用網域名稱並加入 reCAPTCHA。
- **收不到通知信**：寄信走 `script.google.com`，確認內網可連該網域，且收件者在系統裡有填 Email。

---

## 兩種部署對照（給決策參考）

| 項目 | 本方案（網頁內網 + 資料雲端） | 完全自建（資料也在內部） |
|---|---|---|
| 工程量 | 小（放個網頁 + 改幾個網域設定） | 大（要自架伺服器 + 資料庫，換掉 Firebase） |
| 資料位置 | Google 雲端（境外） | 公司內部 |
| 需連外網 | 需要 | 不需要 |
| 適合 | 內部信任、非高敏感資料 | 法遵要求資料不得出境 |

若後續需要「完全自建」版本，再告訴我，我會另外規劃。
