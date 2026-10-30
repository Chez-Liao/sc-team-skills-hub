---
name: labbridge-test-report
description: 使用 LabBridge 核准的固定測試套餐或配方，追蹤持久工作、復原與證據，整理失敗及未測範圍；不以 AI 手動 Console 取代正常測試程序。
---

# LabBridge 測試與報告

## 連線與必要輸入

組織私有入門說明須提供 LabBridge 的 HTTPS MCP URL、本人帳號、短效 token、核准網路與設備分享權限。token 由私有環境變數 `LABBRIDGE_MCP_TOKEN` 提供，不寫入聊天、共用專案或此技能。Hub 啟用不會建立 MCP 連線或增加 operator 權限。

先讀當前 tools/list，確認 `list_recipes`、`run_recipe`、`get_run`、`list_runs`、`get_run_evidence`／`read_run_evidence`；套餐流程另需 `start_test_package`、`get_test_package_run`。缺少工具時，要求維護者確認版本，不改用任意命令或第二套派工。

## 工作流程

1. 確認使用者是要求「查現有結果」或「執行新測試」。摘要要求不授權新 run。查明確 deviceId、產品 profile／image、online／ready、simulation、核准套餐或 recipe 的精確版本，以及現有 recovery lock。
2. 只在明確執行要求、適用版本及安全條件齊備時，以 `start_test_package` 或 `run_recipe` 提交；使用 UUID idempotency key。版本、命令、順序、profile、判據及證據輸出由既有固定腳本／服務決定，不由 AI 每次臨場操作或解讀後改判。
3. 逾時先查原 run，重查提交需沿用 key，不換 key 重跑設備命令。queued 不代表已執行。節點能力、資格、ACL、整機／CM／RG／共用資源租約及復原狀態由服務核對，不修改它們來強行開始。
4. 使用 `get_test_package_run`、`get_run`、`list_runs` 追蹤。讀取索引證據與 checksum，保留 run／device／profile／image／版本／日期。分頁或序號 gap 必須註明，不將缺失的輸出當完整證據。
5. 取消只依使用者要求；active cancellation 仍可能需要帶租約代次的復原確認。未知命令結果、斷線、證據缺失或復原失敗保留原鎖，交由既有例外流程處理，不重送命令或清鎖。

## 設備與驗收界線

一般測試要求不能推論為授權 defaults、重開、升級／刷機、OTP、factory provisioning、RF／語音／電源或其他實體控制。這些工作必須有特定設備、適用 profile、明確授權、固定前置與復原程序。唯讀檢查仍可能占用共享資源。

自動復原僅能使用當前設備／映像／Console policy 已核准、且當次具有證據的固定流程；不得將某一產品的安全復原資格推廣到其他設備。不要啟動第二個 Server 共用 state，或另建一份設備鎖資料庫。

## 報告格式

簡述日期、設備／profile／image、recipe／package 版本、run ID、實際結果、cleanup／recovery、證據連結與下一步。保留 PASS／FAIL／BLOCKED／INCONCLUSIVE／NOT_TESTED，不因摘要語言而重判。

分開「adapter 存在」「可執行」「來源部分／完整對應」「實際已跑」「完整來源驗收」。固定來源分母，不合併不同產品或透過拆分子檢查提高通過率。缺少設備、規格、資格或授權時逐項說明；PASS 字串或單一子檢查不能證明完整產品驗收。

判定狀態與執行證據必須分開。BLOCKED 不代表已開始執行，不能將 PASS + FAIL + BLOCKED 相加當作「實際已跑」。執行數只能依逐項開始／完成記錄或服務提供的明確執行計數；若只有狀態彙總而沒有執行記錄，寫「實際執行數未確認」。INCONCLUSIVE、缺失證據或可執行數也不能自動換算為已跑數。即使輸出執行數，也須另外保留證據完整性與原始狀態。

## 使用例

`/team-skills:labbridge-test-report 整理指定設備最近套餐結果，列出失敗、未測、復原狀態與證據；不要重跑。`

安裝與工具發現只驗證可載入。硬體、環境、腳本及完整驗收須另有當次證據，不能由本技能的發布狀態推定。
