---
name: docsis-pnm-interpret
description: 判讀 DOCSIS／PNM 的 SNMP、RDK data model、RF 與計數器資料，核對單位、instance、平台角色與時間差，輸出有證據的診斷線索。適用於既有輸出的分析；不以單一數值宣告根因或完整認證通過。
metadata:
  version: 1.0.0
---

# DOCSIS／PNM 數據判讀與 RDK 對照

以實際輸出和相符版本的 MIB／data model 為依據。分開描述「觀察」「推論」「缺口」，不把歷史例子當成目前設備狀態。

## 1. 建立可比較的上下文

確認設備型號／晶片、韌體／build、DOCSIS mode、CM 或 CMTS 角色、採集位置與時間、原始 OID／資料模型 path、ASN.1 型別及 MIB／data model 版本。缺少單位定義時保留 raw value 並標為未確認，不猜測倍率。

SNMP table index、ifIndex、channel ID、RDK instance number 是不同識別；不可因數字相同就配對。利用介面類型、channel ID、頻率、方向、狀態及同時段資料建立對照；一對多或時間不同的對應須註明。

## 2. 先判斷取到哪一層資料

每項使用以下描述之一，並附證據：

- 已取得值：有對應 instance 與可解碼的回應；不代表量測準確度或規範符合性已通過。
- 部分取得：只有控制、狀態、摘要、檔名或未解碼 bytes。
- 本次不可取得：角色不適用、instance 不存在、timeout、權限／工具限制；分別記原因，不籠統寫成不支援。
- 未執行：未查詢或缺少必要環境，不能列 PASS。

例如 file status ready、檔名存在或 HTTP success，不證明檔案已下載或 payload 正確。CM 上查不到 CMTS 接收功率／UTSC 等 CMTS-side 物件，應先核對 agent 角色。

## 3. 單位與物理意義

下列是已知物件型別的判讀範例；套用前仍核對目標 MIB。範例數字是人工示例。

| 資料 | 判讀方式 | 不可直接推論 |
|---|---|---|
| `docsIfDownChannelPower`／TenthdBmV | raw 123 → 12.3 dBmV | 不代表總頻段功率或所有 OFDM power 定義 |
| `docsIfSigQSignalNoise`／TenthdB | raw 415 → 41.5 dB | 不因名稱相近就與 RxMER 或 per-subcarrier MER 等同 |
| OFDMA Tx power | 若目標物件型別為 QuarterdBmV，raw 76 → 19 dBmV | 不對全部 upstream power 一律除以 4；須核對 reference bandwidth |
| Frequency | 依物件 Units 換算 Hz／kHz／MHz | PLC、center、zero frequency 與 band edge 不可互換 |
| Equalization／PNM octets | 依格式解碼 tap、header 與樣本 | 有 octet string 不代表有可直接解讀的 MER／頻譜 |
| dBmV 與 dBm | 在 75 Ω、相同量測定義下，dBm = dBmV − 約 48.75 | 不套用到不同阻抗、PSD 或不同積分頻寬的比較 |

RDK 可能已做單位轉換、四捨五入、快取或聚合。`dmcli eRT getv` 僅作已確認 path 的查詢示例；先查設備可用 namespace／component，不掃不存在的路徑。比較表必須並列 raw OID、SNMP raw／unit、RDK path／raw／unit、時間及轉換方式。

## 4. 計數器與趨勢

1. 優先收集同一設備、介面及 counter 的兩次以上讀值，記錄實際時間差、sysUpTime／restart／discontinuity 指標。
2. 無 reset 或 discontinuity 時，rate = delta / elapsed_seconds。counter 下降先檢查 reboot、reset、instance 重建及 wrap；只有寬度已知且可排除 reset／多次 wrap 時才作 modulo 修正，否則該區間標為不可比較。
3. Corrected／uncorrectable、lost sync、T3／T4 event 要與時間窗、channel 狀態及原始事件訊息一起看。沒有適當分母，不計算 error percentage；不要把累積值當當下速率。
4. 誤碼比例需採用同範圍、同時間窗且定義相容的 delta，避免重複累加 total 與其子項。計數器單位是 codeword、packet 或 event 也要分清。

## 5. 形成診斷線索

- Power／SNR／MER：比較同設備的時間趨勢及相同類型 channel；判定門檻來自適用產品／operator／測試 Profile，不硬編通用 PASS 值。
- Channel loss／partial service：對照 bonding、鎖定狀態、事件與當時可用的 RF plan；某個介面 up 不代表所有必要 channel 均正常。
- T3／T4 或 upstream power 上升：列為需進一步驗證的線索，與 CMTS 側狀態／RF／provisioning 交叉核對；不能僅憑單筆事件斷定 ingress、線材或 CMTS 根因。
- OFDM／OFDMA：先確認 channel、profile、PLC／subcarrier 範圍與 power 定義，避免拿不同頻寬／profile 的值直接比較。
- 若需要原始採集或頻譜來源檢查，依當前可用工具執行；不依賴特定其他 Skill 已安裝。

## 輸出格式與完成判據

提供：

1. 設備／版本／採集時窗與來源清單。
2. 對照表：物件／instance、原始值、型別／單位、換算值、RDK 對應、證據位置與限制。
3. 直接觀察、待驗證假設及缺少資料。
4. 最小的下一步驗證及適用判定依據。

未確認單位／instance／時間的值不得用於絕對比較或通過判定。分享前移除認證、客戶及設備唯一識別與內部端點；原始證據留在受控位置。

## 公開參考

本 Skill 是原創判讀工作法，不重製完整規範或內部映射表。實作符合性仍以目標版次文件與實測為準。

- [Broadband Forum TR-181 Device Data Model](https://device-data-model.broadband-forum.org/)
- [IF-MIB／RFC 2863：counter discontinuity 與介面識別](https://www.rfc-editor.org/rfc/rfc2863)
- [CableLabs specifications](https://www.cablelabs.com/specifications)
