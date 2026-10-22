---
name: docsis-cm-evidence-collect
description: 收集 DOCSIS CM 的設備身分、SNMP、RDK 查詢與既有 RF 頻譜證據，產生可追溯且可去識別的診斷資料包。適用於 CM 問題初查與交接；不包含設備設定、重啟或主動啟動頻譜量測。
metadata:
  version: 1.0.0
---

# DOCSIS CM 證據採集

目標是取得可供同事重現與判讀的證據，保留原始值、採集位置、時間及失敗結果。命令範例適用於已確認支援 Net-SNMP 的 shell；其他平台先確認相應工具。

## 輸入與操作界線

- 需要：授權的目標設備、連線路徑、CM 或 CMTS 角色、韌體／build、採集範圍與保存位置。只詢問缺少且影響執行的資訊。
- 透過使用者既有的安全設定取得認證，不將 community、密碼、token、私鑰寫入命令示例、報告或分享檔。
- 預設只讀。不得自行 SET、重啟、停止服務、變更 RF／診斷模式或觸發量測。這些操作需有適用流程及該次授權。
- 即使只讀 walk 也可能增加負載或與量測客戶端爭用；從小範圍開始，遇到延遲或不穩定便減少範圍，保留錯誤，不用大量重試掩蓋問題。

## 1. 確認網路位置與設備身分

1. 區分外部 console host、RG Linux shell、CM diagnostic console 與 CMTS。能 SSH 到 console host，不代表該處可以存取 CM 管理網路。
2. 若需經 RG 存取 CM，使用現場已核對的 console mapping；先確認提示字元、工具與設備身分，再執行 Linux 命令。不得把某台設備的主機別名或連接埠當成通用設定。
3. 記錄採集端 UTC 時間／時區、設備時間與可能偏差、設備角色、型號、HW／SW／build，以及授權範圍。設備唯一識別保留於受控原始包；分享版使用一致代碼。
4. 使用 SNMP system 與介面資料交叉確認。若回應設備與要求不一致，停止；timeout 先檢查路由、ACL、認證與 vantage point，不能直接判定設備故障。

以下是 POSIX shell 的只讀範例。`CM_TARGET` 須由操作員確認；SNMP 版本與認證應先由安全的 Net-SNMP client profile 設定。本例使用現場核准的 v2c；若需 v3，改用已驗證的 v3 profile，不降級認證。

```sh
: "${CM_TARGET:?請先設定已確認的目標設備}"
date -u
command -v snmpget
command -v snmpwalk
snmpget -v2c -t 3 -r 0 -On "$CM_TARGET" .1.3.6.1.2.1.1.1.0
```

## 2. 依問題選擇採集範圍

逐項執行並記錄開始／結束時間、exit status、stderr 及回應；不要把失敗時的空檔當成功。

| 範圍 | 數字 OID／方式 | 用途 |
|---|---|---|
| System | `.1.3.6.1.2.1.1` | 身分、sysUpTime 與系統資訊；內容可能含內部名稱或位置 |
| ifTable | `.1.3.6.1.2.1.2.2.1` | 介面身分與狀態，供 instance 對照 |
| DOCS-IF-MIB | `.1.3.6.1.2.1.10.127` | 經典 DOCSIS RF、狀態與計數器；不是所有新世代資料的全集 |
| DOCS-IF3／IF31／PNM | 依目標版本的 MIB 選取相關 subtree | OFDM／OFDMA、擴充狀態、PNM；先確認 CM／CMTS 角色 |
| Vendor spectrum | 僅限現場已有文件的精確 subtree | 補充私有格式；不以整棵 enterprise walk 作預設 |
| RDK data model | 已確認存在的確切 path | 對照 SNMP 與 RDK 值；不臆造 dmcli path |

```sh
snmpwalk -v2c -t 3 -r 0 -On "$CM_TARGET" .1.3.6.1.2.1.1
snmpwalk -v2c -t 3 -r 0 -On "$CM_TARGET" .1.3.6.1.2.1.2.2.1
snmpwalk -v2c -t 3 -r 0 -On "$CM_TARGET" .1.3.6.1.2.1.10.127
```

若某工具不存在，使用已確認可用的等效只讀方法；不要把 client 缺少 GETBULK 工具記成 CM 不支援 GETBULK。`noSuchObject`、`noSuchInstance`、空表、timeout 與權限拒絕分開記錄；目前回應不足以單獨證明平台永遠不支援該能力。

## 3. 保存與交接

- 保留 console 原始副本；另產生去除 ANSI、NUL、提示字元的解析副本。Hex-STRING 可能跨行，必須保留續行，不能只留下以 OID 開頭的行。
- 若只取得 spectrum 控制／狀態或檔名，明確記為「未取得波形／檔案內容」；不為了補齊資料自行啟動量測。
- 經跳板轉檔時，使用現場核准的傳輸方式，記錄來源、目的地、大小與 SHA-256。優先受保護傳輸；若現場只能 TFTP，限定受控網段，分享前移除端點資訊。
- 產生來源 manifest：檔名、雜湊、時間、採集範圍、命令類型、執行結果、工具／MIB 版本與缺項。
- 原始包留在受控位置。另製分享包，移除認證、MAC／序號、內部 IP／hostname、帳號、客戶識別與本機路徑；保留一致的設備／介面代碼及關係。檢查文字、JSON、檔名、封存檔及圖片，不覆寫原始證據。

## 完成判據與輸出

報告提供：設備／環境摘要、已取得項目、失敗或未執行項目及原因、檔案 manifest、分享版去識別說明，以及下一步建議。只有身分確認、來源可追溯、實際檔案存在並核對雜湊的項目才算採集成功；採集完成不等於 RF 正常或 DOCSIS 驗收通過。

## 參考與適用性

本 Skill 為原創整理的採集流程，不附帶實驗室存取資料或 vendor 私有 MIB 定義。設備與版本差異須現場核對。

- [Net-SNMP snmpcmd](https://www.net-snmp.org/docs/man/snmpcmd.html)
- [Net-SNMP snmpwalk](https://www.net-snmp.org/docs/man/snmpwalk.html)
- [IF-MIB／RFC 2863](https://www.rfc-editor.org/rfc/rfc2863)
