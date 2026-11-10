---
name: cm-test-script
description: 在 cm_test_framework 專案內開發/修改/整合測試 script (TestCase)。遵守 docs/CODING_RULES.md 規範;僅在 cm_test_framework 專案內使用。
---

# CM Test Framework 測試 Script 開發 (Team Skill Hub 版)

## 適用範圍 (重要)

本 skill **僅在 cm_test_framework 專案內**開發/整合測試 script 時使用。
若目前專案**不是** cm_test_framework(根目錄沒有 `AGENTS.md` 或沒有 `tests/`、`test_scripts/` 結構):
**不要套用本 skill**,回覆「本 skill 僅適用於 cm_test_framework 專案」即可。

---

---
name: cm-test-script
description: 在 cm_test_framework 專案內開發/修改測試 script (TestCase)。遵守 docs/CODING_RULES.md 規範,使上層網頁/APP 可呼叫。用於新增 test case、把團隊各自開發的 Python script 整合進 framework、或修改既有 test。
---

# CM Test Framework 測試 Script 開發

本專案根目錄有 `AGENTS.md`(session 啟動會自動載入),處理任何任務前也遵守它;
開發/整合 test script 時以本 skill 為準。

使用者在 `C:\Users\03015\cm_test_framework`(或 UNC 版本)開發測試 script,
需要整合進框架、被上層網頁與 APP 呼叫。**先讀 `docs/CODING_RULES.md` 並遵守**。

## 專案結構 (script / config / utility)

```
cm_test_framework/
├── tests/                # script: 每個 test case 一個檔案
│   ├── functional/       #   例: snmp_walk.py, sysdescr.py, cvc_fw_upgrade.py
│   └── stress/           #   例: cvc_fw_upgrade_loop.py, emmc_rw_test.py, reboot_test.py
│   ├── base_test.py      #   TestCase 基底 (不要改)
│   ├── fw_flow.py        #   FW 升級共用流程 (範例 utility)
│   └── _template.py      #   新 test 的複製範本
├── config/settings.py    # config: DEVICES (COM port/telnet/SNMP) + TEST_PARAMS
├── utils/                # utility: ping / wait_host_up / get_device_cfg
├── core/                 # framework 核心 (不要改)
└── scripts/              # mock_device.py / smoke_test.py (開發驗證用)
```

## 核心概念

- 每個 test 是 **TestCase plugin**:只需要實作 `execute()`,framework 負責
  執行緒、start/stop、log 存檔 (`logs/<日期>/<test_id>_<時間>.log`)、
  report (`reports/<日期>/report_*.html`)、WebSocket 即時串流。
- **不需要**自己開 server、寫 log 檔、生 report、註冊。
- script 寫完重開 server,registry 自動掃描就出現在網頁。

## 新增一個 test case 的步驟

1. 複製 `tests/_template.py` 到 `tests/functional/` 或 `tests/stress/`,
   檔名改成 `<test_id>.py`(小寫 snake_case)。
2. 填寫:
   - 檔頭 docstring:測試名稱/作者/版本/最後修改/**歷史**(每次修改加一行)
   - `test_id`(唯一,snake_case,= 檔名)
   - `name` / `category`("Functional"|"Stress") / `description`
   - `connections`(需要的連線型態)與 `default_conn`
3. 參數:一律 `self.param("key", default)` 讀取;設備 IP/COM port/SNMP 放
   `config/settings.py` 的 `DEVICES`,`TEST_PARAMS` 放測試參數。
   **禁止硬編碼 IP / COM port / 次數在 script 內。**
4. 連線 (一律用模組化共用功能, 不要自己寫 socket/pyserial/SNMP session):
   - telnet/console: `conn = self.open_connection("telnet"|"console")`,
     `conn.send_command(cmd, expect_patterns=[...], timeout=N)` —
     **blocking 讀取一定要給 timeout**
   - SNMP: `utils.snmp.snmp_get(device_id, oid)` / `snmp_walk(device_id, root_oid)`
   - ping/等設備: `utils.net.ping` / `wait_host_up`
5. 結果: `execute()` 回傳 `("PASS"|"FAIL"|"ERROR"|"STOPPED", summary)`。
6. Log: `self.logger.info/warn/error(...)`,**不要用 print()**。
7. 停止支援:迴圈每圈 `self.check_stopped()`,停止時回傳 `"STOPPED"`。
8. 錯誤處理:預期錯誤自己 log + 回傳 FAIL/ERROR 與明確 summary。

## 整合團隊既有 script (已有現成 test script 時)

**先讀 `docs/SCRIPT_INTEGRATION.md` 並照做。** 重點:

1. 開發者的 script 一律在 `test_scripts/`(檔名含版本);先讀 script 搞懂:
   在哪執行(本機/設備 tftp)、需要哪種連線、參數、完成/失敗 marker、kill 方式。
2. 選整合方式:
   - **設備端 script**(tftp 下載執行)→ 繼承 `tests/tftp_script_test.py` 的
     `TftpScriptTestCase`(tftp 下載/chmod/執行/Stop 時 kill 內建)
   - **本機 CLI script** → 包成 `TestCase`,用
     `subprocess.run(..., text=True, capture_output=True)`,輸出寫 `self.logger`
   - 簡單邏輯 → 直接改寫成 module 內建(照 `tests/_template.py`)
3. TEST_PARAMS 填 `script_name` / `tftp_server` / `run_cmd`(含 DONE marker)/
   `done_pass` / `done_fail` / `kill_cmd` / `kill_wait`;機台差異放 `local_settings.py`。
4. 驗證: `py scripts/smoke_test.py` 全過 + 網頁能 Start/Stop + 能生 report。

## 紅線 (整合 / 開發時不可違反)

- **只新增自己的檔案**,不要修改(除非使用者明確要求):
  - 其他人的 module(`tests/` 既有的檔案)
  - 共用層:`core/`、`connections/`、`utils/`、`tests/base_test.py`、
    `tests/tftp_script_test.py`、`app.py`、`templates/`
  - `test_scripts/` 既有的 script(別人的設備端 script)
- 整合新 script 時:在 `test_scripts/` **新增**自己的 script(檔名含版本、唯一),
  在 `tests/` **新增**自己的 module(照 `_template.py`)。
- 機台差異 (COM port / 設備 IP / TFTP) 一律放 `config/local_settings.py`
  (每台機台自己維護, 不 commit; 每台 COM port 可能不同, 例: COM11/12、COM3/4)。
- 不確定是否為共用檔案時,先問使用者。

## 驗證 (改完必做)

0. **紅線自檢 (整合/開發後必做)**:執行 `git status --short`,確認
   - 只新增了自己的檔案 (狀態 `??` 是自己的新檔, `M` 只該是自己的檔案)
   - **沒有修改**共用層 / 他人的 module / `test_scripts/` 既有 script
   - 若有誤改 → 還原 (`git checkout -- <檔案>`) 並告知使用者
1. registry 掃描確認有出現:
   `py -c "from core.test_registry import registry; print([t['id'] for t in registry.list_metadata()])"`
2. 確認 framework 沒被改壞:
   `py scripts/smoke_test.py`(全部要 PASS)
3. 起 server 手動測: `py app.py` → 網頁能看到卡片、能 Start/Stop、
   有 log 檔、能生 report。

## 常見陷阱

- `test_id` 與檔名不一致、或重複 → 網頁上會被蓋掉,命名要唯一。
- `expect()` 沒給 timeout → 使用者按 Stop 會卡住。
- 在 script 自己開檔寫 log → 違反規範,要用 `self.logger`。
- 修改 `core/` 或 `tests/base_test.py` → 會影響所有 test,先問使用者。
- 設備參數要能切換:COM11/COM12、192.168.0.1/192.168.100.1 等用多個
  device profile(settings `DEVICES`)+ 網頁下拉,不要寫死在 script。

## 上層呼叫介面 (script 被誰呼叫)

| Method | Path | 說明 |
|---|---|---|
| POST | `/api/tests/<test_id>/start` | 啟動 (body `{device_id, params}`) |
| POST | `/api/tests/<test_id>/stop` | 停止 |
| GET | `/api/state` | 所有 test 狀態 |
| GET | `/api/tests/<test_id>/log?offset=N` | 增量 log |
| POST | `/api/report` | 產生 report |

WebSocket: `start_test` / `stop_test` / `generate_report`(收到)
與 `state_update` / `log_line` / `test_finished` / `report_done`(推送)。
