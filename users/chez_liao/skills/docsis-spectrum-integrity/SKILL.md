---
name: docsis-spectrum-integrity
description: 檢查 DOCSIS RF 頻譜的真實來源、新鮮度、診斷模式、binary 解碼、單位、頻率幾何及 Hold 聚合，判斷資料是否可供 ingress 分析或 AI 使用。適用於既有 capture 與採集流程審查；不自行改動 RF 或診斷設定。
metadata:
  version: 1.0.1
---

# DOCSIS RF 頻譜資料可信度

數值合理、曲線漂亮或 API 回成功，皆不足以證明資料是當下的真實 RF。把來源、時間、格式與聚合分開驗證；來源未確認時保留資料供診斷，但不得宣稱可供真實 RF 推論、模型訓練或效能評估。

## 1. 確認資料契約與平台

取得設備／build、provider／endpoint 類型、當次 request、原始 response／bytes、時間戳、capture ID、診斷模式證據與格式定義。沒有現場存取權時，以已提供資料作離線審查，列出缺口。

每個 provider 分別確認：raw encoding、端序、符號、倍率、單位、阻抗／reference、樣本數及頻率定義。不得因 SNMP 與 JSON-RPC 畫出相似形狀，就認定它們的 raw values、參數或來源標記相同。私有協定只依有權使用的版本文件／實作解碼，不把某一 firmware 的契約套到全部產品。

## 2. 來源與新鮮度關卡

- 讀取平台已確認的診斷／測試／快取模式；名稱與數值依該版本定義，不預設某個 enum 對所有設備一致。若平台以 0 表示 disabled，仍需先確認這個定義。
- 在開始、結束，以及長時間／分段量測期間取得相應狀態證據。非 live 模式、讀取失敗、過期或未知狀態均不能通過真實 RF 來源關卡；不自行將模式改成 disabled。
- 識別 square-wave 等合成波形、frozen／lock-data 快取、校正 offset curve。它們可以用於相應測試，不能標成現場 RF ingress。
- 前後模式值相同只能降低疑慮，無法排除中間切換後又恢復。完整來源保證應使用與 payload 原子綁定的 mode／capture generation，或可核對的獨占／全程事件證據；缺少時標為「部分核對、仍有 race gap」，不寫成已完整驗證。
- 核對量測時間、request／response 時序、sequence、cache age 與裝置時鐘偏差。樣本 hash 改變只能支持內容有變，不能單獨證明 live；hash 相同也不能單獨證明 replay。

## 3. 格式、數量與頻率軸

1. 依格式驗證 header、payload length、bin／segment 數量、有限數值、缺漏及重複。Hex-STRING 的 console 換行不是 segment 邊界，清理時保留續行 bytes。
2. 只有在契約明定 signed 16-bit、big-endian、0.01 dBmV 時，才使用：u = 256×byte0 + byte1；s = u−65536（u≥32768），否則 s=u；amplitude_dBmV = s/100。這不是通用 spectrum decoder。
3. 明確區分 first-bin、center、edge、span、bin spacing。若契約定義 N 個包含兩端點的 bin，spacing = span/(N−1)；若採 FFT center／半開區間，可能是 span/N。檢查實際 MIB／API 定義，不能在畫圖時默默修正 header 或換成另一公式。
4. 檢查 segment index／center 與 bin offset，處理重疊／接縫時保留原始資料與處理紀錄。每種 request 的 fftSize／span 是否真的生效，須從 response geometry 驗證，不能只看 request 已送出。
5. Window、ENBW、resolution bandwidth、bin spacing 各自核對，不能直接視為相同。格式可解析但 header 語意不符時，分項列出，整體不得寫全數通過。

## 4. 單位與聚合

- 在 75 Ω 及相同量測定義下，dBm = dBmV − 約 48.75。先核對 provider 契約；不可把 PSD、單 bin、segment total、不同頻寬或不同阻抗直接比較。
- Max Hold／Min Hold 是在相同幾何的有效完整 sweep 上，逐 bin 取最大／最小；不是選出一整條「最大 sweep」，也不是從 Max Hold 推導 Min Hold。
- 同一批 sweep 的 Max[i] 應不小於 Min[i]。此排序是必要檢查，並不足以證明聚合正確；還需對照原始 sweep、sample count 與時間窗。
- Average power 應在同一 reference 的線性功率域取平均，再轉回 dB；不要用 dB 值算術平均冒充平均功率。
- 裝置、provider、頻率幾何、mode 或 capture generation 改變時，分開聚合，不能混入舊樣本。若做 smoothing、clipping、補點或 repair，保留原始版本並在輸出標記，不能將修補後資料冒充未加工原始量測。
- 用完整 sweep 的實際完成時間計算有效間隔與 window coverage；requested polling interval 不等於採樣速率。未滿足該次 window／最低樣本數時，標為 incomplete，不用較短設定值宣稱已達成快速取樣。

## 5. 輸出與處理結果

### 證據欄位規則

- 每個欄位標示 `provided`（輸入提供）、`observed`（本次實際量測／查詢）、`derived`（附計算與依據）、`not_provided` 或 `not_computed`。檔案能讀取不等於來源已驗證；不得以 `verified_*` 或其他文字取代未提供的證據。
- `raw_file_hash` 使用 `{algorithm, value, status, source}`。沒有實際 digest 時 `value=null`，依情況寫 `not_provided` 或 `not_computed`；只有本次確實對指定原始檔 bytes 執行計算並取得 digest 才可標 `computed`。若引用輸入 manifest，標 `provided` 及其來源，不能聲稱自行計算或獨立核驗。`hash_changed=true` 不是 digest；對重建 hex bytes 計算的 hash 也不是原始檔案 hash。
- 有 digest 時，`value` 必須逐字完整保留來源 digest；SHA-256 應為完整 64 個十六進位字元。不要截短、遮成前綴／後綴或使用省略號，也不因回答長度限制而縮寫校驗值。輸入 digest 缺字、縮寫或格式不符時記為 invalid／incomplete，不自行補字；它不能充當可用的校驗值。未獨立核驗的完整值仍只標 `provided`，保留其來源。
- 未提供的 provider、format version、build、阻抗、segment count、mode-read timestamps 等欄位保留未知。不可因資料來自 DOCSIS、只有一段 payload 或有 sweep 時間，就補成 75 Ω、單 segment 或已觀測到 mode-read times。使用原子綁定欄位時註明它是輸入聲明還是本次觀測，沒有查詢就不寫成自己已實測。
- `segments` 沒有明示的 segment count／index 或等價結構證據時，固定輸出 `null / not_provided`；單一 payload 不能推導成 1，也不以 `derived` 標籤包裝這個假設。不要輸出全域 `validation_errors` 欄位；格式檢查只使用 `format_errors`，其餘未驗證資訊列入各自的 gaps。
- 可指派報告用代碼，但標為 `analyst_assigned`；capture A／B／C 不代表三台設備。capture generation 不直接冒充未提供的 capture ID，completion time 不直接冒充未提供的 capture start。
- 輸出前逐欄回查來源鍵。若輸入沒有 `capture_start`，該欄固定為 `null / not_provided`；第一個 sweep 的 completion=0 也不例外。另列 `sweep_completion_range` 描述最早／最晚完成時間及其 derived 覆蓋長度。若只有完成時間而沒有明示 capture end，`capture_end` 也保持未知。不可為了填滿範本把「推導的完成時間範圍」改標成「輸入提供的採集起訖」。
- 將 `format_errors`、`source_gaps`、`timing_gaps`、`unverified_metadata` 分開列出。格式未發現錯誤可寫 `format_errors=[]`，但不能因格式正常就寫全域 `validation_errors: none` 或「全部已驗證」。推導的欄位要能追溯原始輸入，假設不得當成 observed。
- 不擴張量測範圍：沒有完整且不重疊的 bin 積分頻寬／覆蓋證據時，不把所列 bin 功率和稱為全頻段總功率；缺 first-bin／center 時只能計算已由契約確定的 spacing，不能捏造絕對頻率軸。缺少來源證據不妨礙交付有限的離線解碼結果，但必須限制接受用途。

逐 capture 輸出以下欄位；原始證據留在受控位置，對外使用一致代碼並去除敏感端點／識別：

```text
device_alias / firmware_build / capture_id
capture_start (explicit source or null) / capture_end (explicit source or null) / clock_basis
sweep_completion_range (derived when available; not capture start/end)
provider / format_version / raw_file_hash
source_class: live | synthetic | cached | unknown
source_evidence / mode_read_times / race_gap
raw_encoding / normalized_unit / impedance_reference
frequency_geometry / bins / segments (explicit source or null)
aggregation / sample_count / effective_sweep_interval / window_coverage
transforms_or_repairs
format_errors / source_gaps / timing_gaps / unverified_metadata
result: accepted_for_stated_use | rejected | incomplete
intended_use / reasons / remaining_checks
```

`accepted_for_stated_use` 只用於本次用途的必要檢查全部有證據支持；不代表整台設備符合 DOCSIS 或取得認證。合成／快取／未知來源，以及無法排除模式切換的資料，不得通過要求真實 RF 來源的用途。測試資料可明確標記後用於 parser／transport 測試，但不得混入正式訓練、評估或線上推論資料集。

若現有資料不足，交付已完成的離線檢查與最小補證清單。寫入設定、切換模式、搶占量測資源或重啟服務，必須另外有適用流程與授權；Skill 本身不授予設備變更權限。

## 公開參考與適用性

本 Skill 為原創資料品質檢查流程，不附 vendor 私有 enum／API 定義、內部拓撲或測試紀錄。

- [CableLabs specifications：按目標版本取得 MIB／頻譜格式](https://www.cablelabs.com/specifications)
- [Net-SNMP snmpcmd：數值輸出與型別](https://www.net-snmp.org/docs/man/snmpcmd.html)
