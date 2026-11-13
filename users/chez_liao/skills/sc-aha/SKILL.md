---
name: aha
description: >-
  Turn any professional or technical knowledge into a vivid, interactive, animated
  explanation that a non-specialist can grasp immediately. Use whenever the user asks
  to "explain", "visualize", "animate", "make a diagram", "walk me through", "show how
  X works", or wants to convert a spec / design doc / concept / algorithm into an
  interactive explainer. Produces self-contained, offline-first, dependency-free
  HTML/SVG artifacts with step controls, live inputs, hover inspection, semantic color,
  and story-driven (scene/beat) animation. Not for business reports or statistical
  dashboards.
---

# Aha — 把專業知識變成「一看就懂」的互動說明

一套方法論 skill：把抽象技術、演算法、協定、流程、架構，轉成生動、可互動、帶有意義動畫的說明，
讓**不懂細節的讀者也能瞬間理解**。

核心信念：**互動圖本身就是說明，不是說明的插圖。動畫要引導注意力，不是讓方塊為了裝飾而動。**

---

## 1. 什麼時候用 / 不用

**用：**
- 「解釋 X 怎麼運作」「畫個圖」「做成動畫」「帶我走一遍」
- 把 spec / 設計文件 / 白皮書，轉成可探索的互動頁
- 演算法、加密、協定、資料結構、狀態機、資料流、生命週期、架構、因果關係

**不用：**
- 商業報表、統計圖表（是 chart，不是 explainer）
- 一句話就講得清的事（文字就夠）
- 讀者已懂、只是為了炫（不硬做動畫）

---

## 2. 第一原則：The Aha Moment（頓悟瞬間）

**每個視覺化必須有一個「單一互動/動畫」讓觀眾說出「啊哈，我懂了」。寫任何 code 之前先找出它。**

- 對稱加密：看到「明文 ⊕ 金鑰 = 密文，再 ⊕ 同一把 = 還原」
- 區塊鏈接加密：看到「上一塊密文回饋進下一塊」→ 了解「改一塊會連鎖」
- 雜湊：看到「改 1 個 bit → 一半輸出翻轉」→ 雪崩效應

找不到這個瞬間 = 還沒想清楚要教什麼。先回去想，不要開始畫。

---

## 3. 格式決策（在設計之前，先決定格式）

> 格式是「決策」，不是「設計」。選錯格式比任何配色問題都更浪費時間。

| 需求 | 格式 |
|---|---|
| 演算法/流程/狀態機、要互動、要動畫 | 自含 HTML + 內嵌 SVG（最靈活：可縮放、可嵌入、可改） |
| 架構/拓撲/流程圖，要可 diff、跟 code 走 | Mermaid（文字即圖，PR 可 diff、隨 code 更新） |
| 資料本身是重點、要圖表編碼與互動 | d3 / Vega-Lite |
| 空間/立體概念，平面圖講不清 | 3D / isometric |
| 一次性靜態圖、要精準排版 | 純 SVG |

預設選 **自含 HTML + 內嵌 SVG**：離線、零依賴、`file://` 可開、能縮放。

---

## 4. 工作流程（照順序）

### Step 1：偵測主題結構
`process / timeline / comparison / hierarchy / architecture / algorithm / data-flow / lifecycle`

### Step 2：分解成 Stage（寫碼前必做）
```
概念 → [Stage 1] → [Stage 2] → ... → [Stage N]
          ↓             ↓                ↓
        可視輸入/輸出   可視輸入/輸出     可視輸入/輸出
```
每個 stage 要有清楚可見的輸入/輸出、可用動畫呈現、與前一 stage 有可見的資料流線。

### Step 3：選最小必要模態
文字（抽象）、圖（結構/關係）、動畫（時序/變化）、互動（流程/系統/模型）。只挑能幫助理解的。

### Step 4：疊加互動模型（能加都加）
1. **Step 控制（主導航）**：播放/暫停/上一步/下一步/速度 + 「Step 3 of 7」指示。
2. **即時輸入欄位**：讀者輸入自己的資料，畫面即時反應（debounce）。
3. **Hover/點擊檢查**：滑到資料塊/線/中間值 → raw data、hex、解釋；點擊可 pin。
4. **並排對比**：有變體時同步 step 並排。

### Step 5：語意動畫（§5）
### Step 6：補語境與「為什麼」（每步 1–2 句白話 + Why it matters + Edge case 按鈕）

---

## 5. 語意動畫 — 鐵律

**動畫只允許 6 種語意，其餘一律靜態：**
`flow`（流動）、`cause`（因果）、`sequence`（先後）、`transformation`（轉換）、`grouping`（歸屬）、`state-change`（狀態改變）。

**規則：**
1. 動畫不表達上述任何一種 → **刪掉，用靜態圖**。
2. 互動優先、絕不 free-loop：播一次 → 定格。
3. 尊重 `prefers-reduced-motion`：降級成靜態最終態。
4. 時長：微互動 150–400ms、完整轉換 ≤ 1s、大位移更長；一律 `ease-in-out`。
5. Staggered reveals：資料塊 30–50ms 錯開出現。
6. **Transform, don't replace**：能「變形」就不要「淡出再淡入」，保持視覺連續性、顯示前後關係。

---

## 6. 敘事模式（挑一個，或混合）

1. **Mystery → Investigation → Resolution**：先拋謎題/矛盾 → 視覺探索 → 揭示原理 → 推廣。
2. **Build Up**：簡單積木 → 組合成複雜 → 驚喜結果 → 反思。
3. **Two Perspectives → Unity**：觀點 A（如代數）→ 觀點 B（如幾何）→ 證明是同一件事 → 探索意涵。
4. **Misconception → Refine → Correct**：先展示常見誤解 → 逐步修正 → 得到正解。
5. **Specific → General**：具體例子 → 找規律 → 抽象成原則 → 應用到新情境。

最好的說明常混合多種（Mystery hook + Build Up 是最常見組合）。

---

## 7. Reveal 順序與節奏（圖要「長出來」）

1. **容器/區域**先出現（淡），建立框架。
2. **節點**依閱讀/邏輯順序出現（scale 0.8→1 + fade）。
3. **連線（edges）**在節點就定位後畫出。
4. **標籤**最後淡入。
5. **高亮掃過**當前路徑，其餘 dim 到 ~0.35。

每個有意義的步停留 **0.5–1.5s**；若配旁白，則 gate 在旁白上，不要用固定計時器。

---

## 8. 動畫 cue 詞彙（story DSL，可直接當 beat 指令）

| cue | 作用 |
|---|---|
| `dim *` | 開場前先把全部視覺權重壓低 |
| `activate <id>` | 強調某節點 |
| `flow a -> b` | 追一條/多條連線的資料流 |
| `pulse <id> for 1s` | 在決策/狀態上停一下注意力 |
| `wait 250ms` | 刻意停頓 |
| `loop` | 最後一 cue 後重播 |

單次移動要「長到能看懂、短到不必等」。

---

## 9. 視覺身份（premium textbook，不是 flowchart）

- 深色主題 + 高對比 accent。
- 等寬字型放資料/hex；無襯線放標籤。
- **顏色即語意**，開頭就給 legend：input（藍）、key（琥珀）、output（綠）、中間態（紫）、危險/竄改（紅）。
- **空間即關係**：左右 = 轉換/時間/因果；上下 = 階層/推導；中心 = 焦點；周邊 = 脈絡。
- 深度分層：陰影/漸層區隔概念層。
- 過場 300–500ms；hover 150ms。
- 每個「活」控制都對應一個 state model，readout 跟著同一個 state 走。

---

## 10. 教學原則（讓內容真的被學會）

- **先 Show → 再 Tell → 最後揭名詞**。
- **一次只教一件事**：每步 = 一個新概念 + 一個動畫，其餘淡化。
- **Mayer**：去冗餘、標重點、分段由讀者控制、先教名詞、口語化。
- **逆向設計**：先想「讀者看完能做/說出什麼」，再設計步驟。
- **Worked example → Faded guidance**：示範（帶 subgoal label）→ 拆鷹架動手 → 獨立操作。
- **檢索練習**：偶爾讓讀者回想（互動小問）。

---

## 11. Scenes & Beats + Living Diagram（活圖）

- `scene` = 一個故事；`beat` = 一個瞬間（某 edge 動了、某資料塊出現、某 caption）。
- 一個檔案只放 1–2 個 scene，loop 很快轉回來。
- **用真實資料**；每個 label/caption 必須對得上實際行為。
- **Living diagram 進階互動**：
  - hover/tab 到某部分 → 焦點光罩亮起它與它的連線 + 一張說明卡。
  - 轉旋鈕（knob）→ 讀數即時回答（改變參數看結果）。
  - 可深連結狀態（`#step=…&param=…`），分享「我看到的那個畫面」。
  - 未被觸碰時放一小段導覽，任何觸碰即停；reduced-motion 顯示靜態（輸入仍可動）。
- 交付前「親眼看」：render 出來截圖每個 beat，確認無文字溢出、箭頭穿過方塊、caption 與動作不符。

---

## 12. 離線優先技術棧（零外部依賴）

- 純 HTML + CSS + Vanilla JS，**不用 ES Module**（`file://` 會被 CORS 擋）。
- 內嵌 SVG；動畫用 `requestAnimationFrame` + 本地 easing + `stroke-dashoffset` / `animateMotion`。
- 資料用 JS 全域物件，**不要 `fetch()` 本地 JSON**。
- 音效用 WebAudio 合成（無音檔）。
- 一份檔案：瀏覽器就能跑、能縮放、能離線。

---

## 13. Anti-Patterns（違反即重做）

- ❌ 靜態 flowchart 無互動
- ❌ 一大段文字講演算法（圖本身就是說明）
- ❌ 只靠 tooltip 傳遞關鍵資訊
- ❌ 與真實資料轉換無關的裝飾動畫
- ❌ 無限循環、常駐閃爍
- ❌ 淡出再淡入取代「變形」（斷了視覺連續性）
- ❌ 淺色黯淡粉彩
- ❌ 跳過 Stage 分解
- ❌ 塞進產品名/客戶名/型號等不相干細節

---

## 14. 品質檢查清單（交付前逐項勾）

- [ ] 有明確 Aha moment
- [ ] 已分解成 Stage，每 stage 有可見輸入/輸出
- [ ] 格式選對（HTML+SVG / Mermaid / d3）
- [ ] Step 控制（前/後/播放/暫停/速度）
- [ ] 至少一個即時輸入欄位
- [ ] Hover 檢查（raw data / hex / 解釋）
- [ ] 每步 1–2 句白話說明
- [ ] 顏色 legend 開頭就有
- [ ] Reveal 順序正確（容器→節點→邊→標籤→高亮，其餘 dim 0.35）
- [ ] 動畫都對應真實變化、可定格、尊重 reduced-motion
- [ ] 用了至少一種敘事模式
- [ ] 深色主題、語意色一致
- [ ] 離線可開、無 CDN、無 fetch、無音檔
- [ ] 親眼看過每個 beat，無溢出/錯位/不符

---

## 15. 通用語意色（可微調，但全篇一致）

| 語意 | 建議色 | 用途 |
|---|---|---|
| 輸入 / 明文 | `#06B6D4` 青 | 原始資料 |
| 金鑰 / 憑證 | `#F59E0B` 琥珀 | 秘密材料 |
| 輸出 / 密文 | `#10B981` 翡翠 | 結果 |
| 中間態 / 處理 | `#8B5CF6` 紫 | 暫存/推導 |
| 危險 / 竄改 | `#F43F5E` 玫瑰 | 失敗/被改 |
| 背景 | `#0B0F19` 深灰 | 畫布 |
