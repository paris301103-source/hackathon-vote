> **給 AI 的指令**：請全程使用「繁體中文（台灣）」回覆與生成，不要用日文或簡體中文。以下是要你協助開發的產品規格，請依此產出程式碼與設計。
>
> **範圍要求**：本規格所列 **A 現有功能 + B 建議強化功能全部都要實作**，做出一套**功能完整**的產品，不要只做局部或 MVP。介面要**非常親切、好上手**，讓非工程師也能無痛使用（詳見 G 節）。

# 本機 Agent 控制台（代號 ACP）· 開發規格

## 一句話定位
一個**純本機的桌面 App**，把散落在不同 AI Agent CLI 工具與各專案目錄裡的 **skills、設定、專案綁定、使用紀錄**，收斂成一個可檢視、編輯、同步、追蹤的控制台。使用者在單一 canonical 主檔編輯一次，App 再依各 Agent 的原生格式自動散佈（fan-out）到對應位置。

## 核心原則（務必遵守）
1. **Local-only 純本機**：只讀寫本機檔案，**沒有伺服器、沒有 telemetry、資料不外送**。
2. **尊重原生格式**：每個 Agent 的輸出仍寫回它自己的原生目錄與格式，不強迫使用者改變既有工作流。
3. **Canonical first**：使用者編輯的主檔集中在一個 canonical store（例如 `~/.acp/skills/`），各 Agent 原生目錄是 fan-out 的輸出端。
4. **Explicit sync 顯性同步**：同步、覆蓋、刪除、衝突處理都要讓使用者看得見，不靠背景隱性魔法。
5. **能力泛化架構**：先把「Skills」做完整，但架構不可鎖死成只能編 skill；要能延伸到 hooks、subagents、workflows、MCP tools、prompt templates、policy packs 等其他 Agent 能力。

## 支援對象
聚焦三大 Agent CLI 的 multi-agent 能力管理：
- Anthropic Claude Code（原生目錄 `.claude/skills/`）
- OpenAI Codex CLI（原生目錄 `.agents/skills/` 與相關設定）
- Google Gemini / Antigravity CLI（原生目錄 `.gemini/skills/` 或相容目錄）

## 適合誰
- 已在用上述任一 Agent CLI 的開發者。
- 需要在多個 project 之間管理 agent skills 的團隊或個人。
- 不想手動維護多份 `SKILL.md`、多個 agent-native 目錄與同步狀態的人。
- 想用本機工具檢視 token / session 用量、又不願把資料送到遠端服務的人。

## 要解決的痛點
Agent CLI 的設定與擴充檔通常分散在不同位置、不同 project 各自有不同版本與同步狀態，還會出現同名 skill 衝突。核心解法：**在 canonical store 編輯一次 → App 依各 agent 原生格式 fan-out 到對應位置**。

---

## A. 現有功能（第一版必須做到）

### A1. Skills（第一個落地的能力類型）
- 以一個 canonical skill 主目錄（如 `~/.acp/skills/`）作為單一真相來源。
- 可**編輯、修復、刪除、同步** canonical skills。
- 可從各 agent 原生 skill 目錄**匯入**既有 skills。
- 可把同一份 canonical skill **fan-out** 到 Claude / Codex / Gemini 等多個目標。
- 依 target 管理 **global / project scope**、**enabled 狀態**、**tracked / detached 模式**。
- 顯示 **coverage matrix（覆蓋矩陣）**：一眼看出每個 skill 已同步到哪些 agent／哪些 project。

### A2. Projects
- 管理「已知 project」清單。
- 檢視每個 project 內的 agent skill 存在狀態。
- 從 project 視角分辨：哪些 skill 已被納管、哪些仍只是 agent-native 檔案。
- 對不存在的 project 路徑要顯示可見的錯誤狀態，不可靜默失效。

### A3. Tokens 與 History（用量分析）
- 掃描本機 agent CLI 的 session / token 使用資料。
- Dashboard 呈現每日、每模型、每 project、每 session 的用量。
- 從 token summary 可連到 History 檢視對應 session。
- 資料全部留在本機，不需要 server 或 telemetry。

### A4. Settings、Memory、Templates
- 提供各 agent 相關設定與 memory 檔案的視覺化管理介面。
- 提供可重用 templates，降低建立新設定或 skill 的成本。
- 全部維持 local-file-first，不依賴遠端帳號。

---

## B. 建議強化功能（依投資報酬率排序，第二版起）

### B1. 衝突解決與版本控管（最高優先）
多人協作最痛、也是本產品最核心的護城河。
- Skill 的 **diff 檢視**、**版本歷史**、**一鍵回滾**。
- 同名 skill 衝突時提供**三方合併（3-way merge）**。
- 把「顯性同步」再往前推：每次覆蓋前顯示將變更的內容清單。

### B2. 擴充能力類型：MCP tools ＋ hooks
現況只做完 Skills。大家都在接 MCP，把「管理 MCP server 設定」與「hooks」納入後，適用面立刻放大。
- 集中管理各 agent 的 MCP server 連線設定，並可 fan-out。
- 管理 pre/post hooks，套用同樣的 canonical → fan-out 模型。

### B3. 企業治理層（最有商業價值）
面向公司內部採用的殺手級功能。
- **政策（policy）**：規定哪些 skill／MCP 允許在哪些 project 使用。
- 敏感能力需**審批**流程。
- 完整**稽核紀錄（audit log）**：誰在何時改了什麼、同步到哪裡。

### B4. 內部 Skill Marketplace
- 團隊內部的 skill 商店：成員可發佈、瀏覽、一鍵安裝共享 skill。
- 版本標記、相依關係、安裝來源可追溯。
- 產生網路效應，讓平台從「個人工具」變「團隊資產」。
- （可用一個輕量本機／內網 market server 支援，不破壞 local-first 原則。）

### B5. 用量分析升級為成本治理
在既有 token 儀表板上延伸：
- 每人／每 project 的**花費估算**。
- **預算告警**（超過門檻提醒）。
- 月報**匯出**（CSV／PDF）供主管檢視。

### B6. 降低上手門檻
- 提供**簽章安裝檔**與**自動更新**，不要求使用者自行 build。
- **開場精靈（onboarding wizard）**：一鍵掃描並匯入現有 `.claude` / `.codex` / `.gemini` 目錄。
- 讓非工程師也能安裝即用。

### B7.（可選）團隊同步後端
- 讓 canonical store 可用一個 **Git repo 當同步來源**，多人共用同一份主檔又不破壞 local-first。
- 同步、衝突、覆蓋一律走 B1 的顯性流程。

---

## C. 建議技術棧
| 層級 | 技術 |
| --- | --- |
| 桌面外殼 | Tauri v2 |
| 前端 | React 19、TypeScript（strict）、React Router |
| 狀態管理 | Zustand |
| 樣式 | Tailwind CSS v4 |
| 後端 | Rust（Tauri commands） |
| 套件管理 | npm |
| 用量擷取 | 本機 token 用量擷取 CLI（以 sidecar 內建，或 PATH／npx fallback） |

## D. 架構準則
- 前端程式放 `src/`；後端 command 放 `src-tauri/src/commands/`。
- 前端呼叫後端一律透過 `src/lib/tauri/commands.ts` 的 typed wrapper。
- 後端 command 必須註冊在 `src-tauri/src/commands/mod.rs` 與 `src-tauri/src/lib.rs`。
- 路徑比對／project 路徑正規化要用共用 helper，禁止臨時字串比較。
- 建議採 spec-driven 開發：規格放 `specs/`、變更提案放 `changes/`、完成後歸檔。

## E. 開發環境
- Node.js 18+、npm、Rust toolchain、對應平台的 Tauri v2 前置需求。
- 啟動完整桌面 App：`npm install` 後 `npm run tauri dev`（只跑 `npm run dev` 只有 Vite，呼叫後端的頁面無法完整運作）。

## F. 交付優先順序建議
1. 先完成 A（Skills + Projects + Tokens + Settings）＝可用第一版。
2. 接著做 **B1（版本／衝突）＋ B3（治理）＋ B4（內部市集）**，這三項最能把產品從「得獎 demo」推進成「公司會長期使用的內部平台」。
3. 其餘 B 項與 G 節的完整介面一併補齊，達成「功能完整 + 介面親切」的最終目標。

---

## G. 介面與體驗設計原則（重點：非常親切、好上手）

目標：**工程師覺得專業、非工程師也能無痛用**。介面要友善、清楚、少學習成本。

### G1. 整體風格
- 乾淨、現代、留白充足；避免密密麻麻的表格與術語轟炸。
- 提供**深色 / 淺色主題**切換，預設跟隨系統。
- 一致的圖示語言與色彩：同步狀態用固定顏色（已同步＝綠、未同步＝灰、衝突＝紅、進行中＝藍）。
- 主要操作用大而明確的按鈕；破壞性操作（刪除、覆蓋）用紅色並二次確認。

### G2. 導覽
- 左側固定側邊欄分頁：**Skills、Projects、用量、設定**，圖示＋文字，一眼看懂。
- 麵包屑或返回鍵，隨時回得去，不會迷路。
- 全域搜尋框（搜 skill、project、設定），支援鍵盤快捷鍵。

### G3. 新手引導
- 首次開啟有**開場精靈**：三步完成——① 掃描本機現有 agent 目錄 → ② 一鍵匯入既有 skills → ③ 看懂覆蓋矩陣。
- 每個空狀態（還沒有 skill／project）都給**友善插圖＋一句話說明＋一顆行動按鈕**，不要只留空白。
- 重要名詞（canonical、fan-out、tracked/detached）滑鼠移上去有**淺白說明 tooltip**，用大白話解釋。

### G4. 操作回饋
- 每個動作都有明確回饋：成功用 toast、失敗顯示可讀的錯誤訊息（不要丟原始堆疊）。
- 長時間操作（同步、匯入、reindex）顯示進度與可取消。
- 破壞性或同步覆蓋前，先給**預覽 diff**，讓使用者看清楚會改什麼再確認。

### G5. 覆蓋矩陣（核心畫面要特別好懂）
- 用清楚的表格／格狀圖：橫軸是 agent／project、縱軸是 skill，格子用顏色與圖示直接表達「已同步／未同步／衝突」。
- 點任一格可展開細節與快速操作（同步、比對、移除）。

### G6. 親和力細節
- 全繁體中文介面（可保留英文原生名詞但加中文說明）。
- 響應式版面，視窗縮放不跑版。
- 鍵盤可操作、有基本無障礙（focus 樣式、對比度足夠）。
- 適度的微互動動畫（狀態切換、載入），但不浮誇、不拖慢。

### G7. 交付對介面的要求
- 提供每個主要頁面的**實作**（不是佔位頁）：Skills 清單與編輯、覆蓋矩陣、Projects、用量儀表板、設定、開場精靈、內部市集、治理／稽核頁。
- 每頁都要有：正常狀態、空狀態、載入中、錯誤狀態四種畫面。
