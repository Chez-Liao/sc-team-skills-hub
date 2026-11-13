---
name: sc-aha
description: >-
  Turn any complex technical knowledge, architecture, protocol, or firmware mechanism
  into a vivid, highly legible, interactive engineering explainer. Optimized for human
  engineers and non-specialists. Produces clean, professional, white/light-themed
  engineering documents (Stripe/Cloudflare/Apple tech-spec aesthetic) with zero-overlap
  SVG diagrams, comfortable typography, hover inspection, step controls, and live interactive
  sandboxes. Strict anti-occlusion and offline-first standards.
---

# Aha — 把專業工程知識轉化為清晰易讀的互動說明

一套專門為**非該領域專家或初階工程師**打造的技術視覺化與概念解析方法論。
核心目標：**「不靠死記名詞，透過直覺互動與清晰排版，5 分鐘內看懂底層真相。」**

---

## 1. 核心視覺與排版規範 (Design & Typography Overhaul)

### 1.1 摒棄「黑底 AI 玩具感」，轉向「頂級白底專業工程手冊風格」
- ❌ **嚴禁預設純黑/深藍發光底色（如 `#0B0F19`）**：深色霓虹主題容易產生廉價 AI 生成感，且在長篇文字閱讀時容易產生視覺疲勞。
- ✅ **全面採用專業工程白底/淺灰風格（Stripe / Cloudflare / Apple Developer 規格）**：
  - **頁面底色**：乾淨純白 `#FFFFFF`，搭配極淺工程灰底 `#F8FAFC` 作為區塊與畫布襯底。
  - **邊框與格線**：高質感工程淺灰線條 `#E2E8F0` 或 `#CBD5E1`，線寬 `1px` 到 `1.5px`。
  - **文字主色**：深灰墨黑 `#0F172A`（Slate-900，提供極致銳利的閱讀對比度，嚴禁使用刺眼純黑 `#000000`）。
  - **說明輔助色**：沉穩灰 `#475569`（Slate-600）與 `#64748B`。

### 1.2 防遮擋與排版安全鐵律 (Strict Anti-Occlusion & Overflow Standards)
- **SVG 原生文字不具備自動換行能力（根因預警）**：
  - SVG 的 `<text>` 標籤若無特定處理，遇到長文字絕對會沿著 X 軸無限延伸，穿透卡片邊界被右側截斷！
  - **核心解法 1（多行陣列模式）**：卡片渲染函數必須原生支援 `lines: ["第一行...", "第二行..."]`，每行獨立計算 y 軸增量（字級 12.5px 時建議行距 `idx * 20px ~ 22px`）。
  - **核心解法 2（自動估算與語意拆行）**：若傳入單一長字串 `sub`，必須依據卡片可用寬度（`width - 32px padding`）計算字元容量。中文每字約佔 13px，超過容量必須自動斷句為多行，或改用 `foreignObject` 包裹 HTML 容器進行 `word-break: break-word`。
- **元件間距與通道走線**：
  - SVG 方塊與節點內部必須預留充足 Padding（水平至少 16px，垂直至少 14px）。
  - 子卡片（Nested Box）與父容器邊界的間距不得小於 14px，高度必須為多行文字留出動態擴充空間（每增加一行文字，高度 `h` 至少增加 24px）。
  - 箭頭連接線（Connectors）必須沿著元件間隔的「空白通道」走，嚴禁橫穿任何文字、卡片或標籤中心。
- **字體層次與極致易讀性**：
  - **章節大標題**：`22px ~ 26px`，`font-weight: 800`，深色墨黑 `#0F172A`。
  - **正文說明文字**：**嚴禁小於 14.5px**，建議主內文設定為 **`15.5px ~ 16px`**，行高（`line-height`）維持舒適的 `1.8 ~ 1.85`。
  - **圖表節點標題**：`14px ~ 15px`，`font-weight: 700`；節點次要說明與內文不低於 `12.5px`。
  - **二進制 / Hex / Offset 標記**：統一使用清晰等寬字型（`ui-monospace, Consolas, Menlo`），字級 `12px ~ 13px`，搭配淺黃/淺灰膠囊底襯。

### 1.3 專業工程語意配色表 (Light Theme Semantics)

| 語意角色 | 推薦色彩 | 淺底背景色 (10%~15% Opacity) | 適用對象 |
|---|---|---|---|
| **明文 / 輸入 (Input)** | `#0284C7` (Sky-600) | `#E0F2FE` (Sky-100) | 原始程式碼、明文 Device Tree、輸入資料 |
| **金鑰 / KEK (Secret)** | `#D97706` (Amber-600) | `#FEF3C7` (Amber-100) | a2k3、AES-256 密鑰、DEK、加密參數 |
| **密文 / 輸出 (Cipher)** | `#059669` (Emerald-600) | `#D1FAE5` (Emerald-100) | 密文映像檔、rootfs.enc、FITE 密文塊 |
| **簽章 / 信任根 (Sign)** | `#7C3AED` (Purple-600) | `#EDE9FE` (Purple-100) | r4k1、RSA 數位簽章、Certificate、ROT |
| **硬體根 / 警告 (Root/Fail)**| `#E11D48` (Rose-600) | `#FFE4E6` (Rose-100) | CPU OTP、HUK、驗證失敗、Kernel Panic |
| **結構容器 / 分區** | `#64748B` (Slate-500) | `#F1F5F9` (Slate-100) | eMMC 分割區、RPMB SKS 庫、SoC 邊界 |

---

## 2. 知識傳遞原則：The "Aha Moment"（頓悟瞬間）

每個技術主題必須鎖定**一個最核心的頓悟點**，寫碼前先定位：
- **為什麼需要兩層加密？** ➔ 看見「根金鑰 a2k3 永遠不出門，每版隨機拋棄 DEK」的隔離保護。
- **為什麼要簽兩次？** ➔ 看到「第一次防明文邏輯被改，第二次讓開機在不解密前提下直接先篩掉惡意密文」。
- **為什麼換 CPU 會開機失敗？** ➔ 看到「CPU HUK 變更導致 eMMC RPMB 認證 HMAC 不符，金鑰庫無法開啟」。

---

## 3. 版面架構：雙欄專業技術手冊佈局

```
┌────────────────────────────────────────────────────────────────────────┐
│ HEADER: 專案標題 · 專業徽章 · 顏色語意圖例 (Legend)                      │
├────────────────────────────────────────────────────────────────────────┤
│ NAV: 模組切換標籤 (Tab Navigation, 淺灰底 + 圓角選中態)                  │
├──────────────────────────────┬─────────────────────────────────────────┤
│ 左欄：教材與原理解析 (35% ~ 38%)│ 右欄：高精度 SVG 互動實驗室 (62% ~ 65%)    │
│ • 當前階段編號與標題         │ • 大畫布 (960x560 以上)，元素間距寬鬆    │
│ • Aha 頓悟核心提示框         │ • 容器 ➔ 節點 ➔ 連線 錯落有致，嚴禁穿線 │
│ • 易讀正文 (15px/1.8行高)    │ • 互動式元件：可點擊、可切換狀態         │
│ • 關鍵術語點擊彈出說明       │ • 懸停高亮：主動突顯關聯路徑             │
│ • 底部控制：上一步/下一步    ├─────────────────────────────────────────┤
│                              │ 底部 HUD：浮動節點檢測儀 (Byte Offset)   │
└──────────────────────────────┴─────────────────────────────────────────┘
```

---

## 4. 動態與互動鐵律 (Interactive & Motion Standards)

1. **拒絕廉價的連續跑馬燈與循環動畫**：
   - 嚴禁無限循環的閃爍文字、隨機數字洗牌、常駐旋轉光圈。
   - 動畫僅作為**「因果傳遞」**或**「狀態推進」**的短暫引導（時長 `400ms ~ 800ms`，單次播放即定格）。
2. **節點懸停透視 (Node Inspection HUD)**：
   - 右側任何核心方塊（如標頭欄位、金鑰、暫存器、分區），滑鼠懸停時底部的 HUD 即時顯示真實的底層資訊：
     - **實體位移 (Byte Range)**：如 `0x00..0x03`。
     - **真實值 / 格式**：如 `Magic: 0x46495445 ("FITE")`。
     - **工程意義 (Engineering Context)**：該欄位在開機驅動中對應哪一行代碼或哪支函式。
3. **術語飛出抽屜 (Non-Disruptive Flyout)**：
   - 點擊內文術語，於畫面側邊彈出專業名詞卡片，包含：
     - 白話直覺解釋。
     - 具體實作程式碼路徑（例如：`sercomm-bl.patch` / `board_secure_fit.c`）。
     - 💡 **核心價值 (Why it matters)**。

---

## 5. 離線自含交付規範 (100% Offline-First)

- 交付必須是單一 `.html` 檔案或標準多檔純靜態資源。
- 零外部 CDN（無 Tailwind/Bootstrap/Google Fonts 遠端依賴）。
- `file://` 本地直接雙擊秒開，字型依賴作業系統原生高品質無襯線體（`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Microsoft JhengHei"`）。

---

## 6. 品質自檢清單 (Pre-Delivery Checklist)

- [ ] **視覺風格**：是否為專業白底/淺灰工程手冊風格？（非黑底霓虹玩具風）
- [ ] **字級與行高**：左側說明內文是否不小於 14.5px（推薦 15.5-16px）？行高是否達到 1.8？
- [ ] **SVG 文字截斷排查（必查）**：
  - 檢查所有卡片內次要說明與處置方案是否超出卡片右邊界？
  - 超過 15 個中文字或 30 個英文字元的字串，是否已使用 `lines: [...]` 拆為多行？
  - 卡片高度 `h` 是否隨多行文字相應擴大？（防文字被下底線切掉）
- [ ] **連線遮擋檢查**：SVG 連接線是否僅穿行於通道留白處？無任何線條貫穿文字卡片？
- [ ] **二進制與 Hex 排版**：位元組分佈區塊寬度是否足以容納文字，無重疊或擠壓？
- [ ] **互動穩定性**：左右鍵切換、點選 Tab、點選術語、懸停 HUD 是否全部運作正常？
- [ ] **真實工程代碼對齊**：所有金鑰名稱、標頭 Offset、腳本名稱是否 100% 吻合真實 codebase？
- [ ] **零外部依賴**：中斷網路後，本地雙擊檔案能否 100% 完美呈現？
