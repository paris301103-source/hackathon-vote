# 追蹤系統 · Email 通知設定（免費、免升級 Firebase）

用一個 Google Apps Script 當寄信小後端（從你的 Gmail 寄出）。約 5 分鐘。

## 步驟一：建立寄信腳本
1. 到 https://script.google.com → 「新專案」。
2. 把預設程式碼整段刪掉，貼上這段：

```javascript
function doPost(e){
  try{
    var d = JSON.parse(e.postData.contents);
    MailApp.sendEmail({ to: d.to, subject: d.subject, body: d.body });
    return ContentService.createTextOutput("ok");
  }catch(err){
    return ContentService.createTextOutput("err:" + err);
  }
}
```

3. 右上「部署 → 新增部署作業」→ 類型選 **網頁應用程式（Web app）**。
   - 執行身分：**我（你的 Gmail）**
   - 誰可以存取：**所有人（Anyone）**
   - 按「部署」→ 第一次會要你**授權**（允許寄信）。
4. 複製它給你的 **網頁應用程式 URL**（長得像 `https://script.google.com/macros/s/AKfyc.../exec`）。

## 步驟二：把 URL 貼進追蹤系統
1. 用文字編輯器打開 `追蹤看板.html`。
2. 最上面找到這一行：

```javascript
const NOTIFY_URL="";
```

3. 把 URL 貼進雙引號裡：

```javascript
const NOTIFY_URL="https://script.google.com/macros/s/AKfyc..../exec";
```

4. 存檔。（若已上線到 Vercel/Netlify，記得重新上傳這個檔）

## 步驟三：填每個人的 Email
- 用「**後台**」登入（名字打「後台」、密碼 0000）→「🔑 帳號 · Email · 密碼」區，把每個人的 email 填進去。
- 有填 email 的人才會收到通知。頁面上會顯示「Email 通知：已啟用 / 未設定」。

## 什麼時候會自動寄信？
- **主管指派任務**給某案 → 通知該案「追蹤對象」。
- **負責人分派協作 / 設定追蹤對象** → 通知被指派的協作者。
- 信件從你的 Gmail 寄出；一般 Gmail 每日約 100 封額度，內部用足夠。

## 常見問題
- 沒收到信？檢查：① NOTIFY_URL 有貼對且已重新部署；② 對方 email 有填；③ 到垃圾信匣看看。
- 想之後改用 LINE？可行但較複雜（要開官方帳號、每人加好友、抓 userId），需要時再幫你接。
