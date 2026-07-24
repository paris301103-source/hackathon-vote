# 修正「儲存失敗：Missing or insufficient permissions」

## 這是什麼問題
畫面能載入（讀取 OK），但按儲存被擋 → 是 **Firebase 的 Firestore 安全規則**拒絕了寫入。
**這不是網頁程式的問題，改程式沒有用**，要去 Firebase 後台改「規則（Rules）」。

---

## 修正步驟（約 2 分鐘）

1. 打開 https://console.firebase.google.com/ → 選你的專案
2. 左側選單 **建構 → Firestore Database**
3. 上方分頁點 **規則（Rules）**
4. 把整段內容**全部刪掉**，貼上下面這段，然後按 **發布（Publish）**

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // 追蹤系統用到的四個資料表：允許 App 讀寫
    match /track/{docId}     { allow read, write: if true; }
    match /treports/{docId}  { allow read, write: if true; }
    match /dreports/{docId}  { allow read, write: if true; }
    match /control/{docId}   { allow read, write: if true; }

    // 其他一律禁止
    match /{document=**}     { allow read, write: if false; }
  }
}
```

5. 發布後回系統重試儲存，應該就成功了（規則生效約 10–30 秒）。

---

## 這樣安全嗎？

安全性由 **App Check + reCAPTCHA** 這一層把關 —— 只有從你這個網站（已註冊網域）發出的請求才過得了 App Check，外部程式打不進來。所以規則寫 `if true` 是把「誰能進來」交給 App Check 管，這是這套架構刻意的設計。

**前提：App Check 必須維持「強制執行（Enforced）」**，這你之前已經設好了。
可到 Firebase → App Check → API → Cloud Firestore 確認狀態是「已強制執行」。

---

## 如果還要更嚴（可選）

若想「連讀寫都要求先通過 App Check」寫進規則本身，把 `if true` 改成 `if request.app != null`：

```
match /track/{docId}     { allow read, write: if request.app != null; }
match /treports/{docId}  { allow read, write: if request.app != null; }
match /dreports/{docId}  { allow read, write: if request.app != null; }
match /control/{docId}   { allow read, write: if request.app != null; }
match /{document=**}     { allow read, write: if false; }
```

⚠️ 這個版本**只有在 App Check 已強制執行時才可用**，否則會把所有人（包含你自己）擋在外面。不確定就先用上面 `if true` 的版本，能運作最重要。
```
```
