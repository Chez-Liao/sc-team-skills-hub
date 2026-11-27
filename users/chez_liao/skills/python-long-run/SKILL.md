---
name: power-cycle-longrun
description: 產生 Telnet + Raspberry Pi GPIO power cycle 的 long run 壓力測試腳本（可選 UART log、可選停止條件）
argument-hint: "[測試需求描述或直接給參數]"
allowed-tools:
  - read
  - write
  - edit
  - grep
  - glob
  - exec
---

# Power Cycle Long Run 腳本產生器

當 user 要「長時間斷電重開機壓力測試」腳本時使用本 skill。
產出物是一支獨立可跑的 Python 腳本：Telnet 下檢查指令 → Pi GPIO 繼電器斷電/上電 → ping 判定 → 重連 Telnet，循環執行並記錄每輪 log 與總耗時。

> 注意：Team Skill Hub 只接受 `.md` 檔，因此 template.py 與 config 範例直接內嵌在本檔附錄，請從附錄取出原始碼使用。

## 流程

### 1. 收集參數

先檢查 user 的 prompt 與對話上下文是否已提供參數，缺的再用 `ask_user_question` 補問（每批最多 4 題；telnet 指令清單請 user 直接貼上或用 Other 自由輸入）。

**必填：**

| 參數 | 說明 | 範例 |
|------|------|------|
| DUT IP | 被測設備 telnet 目標 | `192.168.1.1` |
| telnet 帳密 | 登入帳號密碼 | `admin` / `admin` |
| Pi COM port | 連 Raspberry Pi 的序列埠（下 GPIO 指令用） | `COM3` |
| telnet 指令清單 | 登入後要依序執行的檢查指令 | 見下方範例 |
| 輸出目錄 + 腳本名稱 | 產生的 .py 放哪、叫什麼 | `power_cycle_015/XF10_longrun.py` |

**選填（有預設值）：**

| 參數 | 預設 |
|------|------|
| 波率（Pi 及 UART 共用） | `115200` |
| GPIO 腳位 | `16` |
| 斷電秒數 `power_off_seconds` | `5` |
| 上電後死等 `reboot_wait_interval` | `90` |
| 連續 ping 次數 `required_pings` | `10` |

**UART console log（問 user 要不要）：**
要的話再問 UART 的 COM port。注意 UART port 不可與 Pi 的 `ras_port` 相同。

**停止條件（問 user 要不要）：**
「測試中要不要偵測到特定狀況就停下來？條件是什麼？」
- 條件能用「整份 log 含 regex」或「某條 telnet 指令的輸出含 regex」表達 → 寫成 `stop_conditions` 規則
- 條件複雜（跨指令關聯、多個條件組合、要算狀態）→ 產生 `custom_stop_check()` 函式實作
- 不要 → 留空（`stop_conditions: []` + `custom_stop_check` 回傳 `None`）

### 2. 判斷停止條件類型

`stop_conditions` 規則格式：

```json
{ "name": "規則名", "scope": "whole_log 或 section:<telnet指令>", "pattern": "regex" }
```

範例：
- 整份 log 偵測 kernel panic → `{"name": "kernel_panic", "scope": "whole_log", "pattern": "Kernel panic|kernel BUG"}`
- `wl --all bss` 輸出有 radio down → `{"name": "bss_down", "scope": "section:wl --all bss", "pattern": "---\\s+wl\\d+[^\\n]*\\n\\s*down"}`

若 user 的條件兩種都表達不了，直接在產出腳本的 `custom_stop_check(log_path, test_count)` 填 Python 邏輯：回傳 `None` 繼續跑，回傳字串就停止並顯示原因。

### 3. 產生腳本

1. 從本檔「附錄 A：template.py」的 ```python 程式碼區塊取出**完整原始碼**
2. 寫到 user 指定的目標路徑，**只修改檔頭 `CONFIG = {...}` 區塊**填入收集到的參數（其餘邏輯不要動）
3. 若 user 的條件需要 `custom_stop_check`，在標記區塊 `# === STOP CONDITION: custom logic ===` 填入實作
4. 若 user 要 config-file 模式（多個 DUT 共用一支 script），把參數另外寫成 json（格式見附錄 B），腳本用 `python script.py --config xxx.json` 執行即可覆蓋 CONFIG

### 4. 驗證與交付

1. `python -m py_compile <產出檔>` 確認語法
2. 告訴 user：檔案位置、執行方式、log 會寫到哪、停止條件是什麼

## 注意事項（gotchas）

- **COM port 不可重複**：`ras_port`（Pi）與 `uart_log.port`（DUT UART）必須是不同的 COM port
- **GPIO export 重複執行**：每次 power cycle 都會 `echo > export`，Pi 端回 `device or resource busy` 是正常的，不影響功能
- **COM port 是「切電源時才開」**：template 已實作 open → export → off → sleep → on → close，不需常駐連線
- **telnet 指令輸出擷取**：`telnet_read_all` 用 idle_timeout=3 秒判斷指令結束；若某條指令輸出特別慢（如 dmesg），可把 timeout 調大
- **ping 旗標**：template 已處理 Windows `-n` / Linux `-c` 差異
- **config json 含明碼密碼**：提醒 user 不要把含密碼的 config commit 到 git
- **測試計數延續**：`timer_log_file` 存在時會自動接續第 N 次計數；要重新計數就刪掉該檔

---

## 附錄 A：template.py 完整原始碼

複製以下 python 區塊的**全部內容**作為產出腳本的基底，只改 `CONFIG = {...}`：

```python
#!/usr/bin/env python3
"""
Power Cycle Long Run Stress Test - Telnet + Raspberry Pi GPIO Power Cycle
==========================================================================
產生工具: power-cycle-longrun skill

使用方式:
    python <script>.py                  # 使用檔頭 CONFIG
    python <script>.py --config x.json  # JSON 覆蓋 CONFIG（deep merge）

流程（每輪）:
    Telnet 下檢查指令 -> 停止條件檢查 -> exit -> Pi GPIO 斷電/上電
    -> 死等 -> ping 連續 N 次 -> 重連 Telnet -> 下一輪
"""

import copy
import json
import os
import re
import sys
import threading
import time
import socket
import subprocess
import platform
import serial
from datetime import datetime


# ===================== CONFIG（產生腳本時填寫這個區塊） =====================
CONFIG = {
    # --- DUT / Telnet ---
    "ip_address": "192.168.1.1",
    "username": "admin",
    "password": "admin",
    "reboot_wait_interval": 90,   # power cycle 後死等秒數
    "required_pings": 10,         # 需要連續成功 ping 的次數

    # --- Raspberry Pi GPIO（經 COM port 下 shell 指令控制繼電器）---
    "ras_port": "COM3",
    "ras_baudrate": 115200,
    "relay_gpio": 16,             # high trigger
    "power_off_seconds": 5,       # 斷電秒數

    # --- Telnet 登入後依序執行的指令 ---
    "telnet_commands": [
        "cat /etc/SC_MT_VERSION",
        "lspci",
        "cat /debug/gpio",
        "ifconfig",
        "wl --all status",
        "md5sum /data/psi_wifi",
        "md5sum /data/.kernel_nvram.setting",
        "dmesg | grep -i WLC_E_BSSCFG_UP",
        "wl --all bss",
        "wl -p bss",
        "wl -p isup",
        "wl -p revinfo",
        "wl -p radio",
        "wl -p ap",
        "wl -p apsta",
        "cat /tmp/wl2_hapd.conf",
    ],

    # --- UART console log（選配）---
    # enabled=true 時會開啟 port 記錄 DUT UART 輸出到 uart_run_N.log
    # 注意：port 不可與 ras_port 相同
    "uart_log": {
        "enabled": False,
        "port": "COM4",
        "baudrate": 115200,
    },

    # --- 停止條件（選配）---
    # 每條規則: {"name": str, "scope": "whole_log" | "section:<telnet指令>", "pattern": "regex"}
    # 任一規則命中 -> 印出原因、寫入 log、停止測試（不再 power cycle）
    # 留空 [] 表示不啟用
    "stop_conditions": [],

    # --- 輸出 ---
    "output_dir": ".",            # telnet_run_N.txt / uart_run_N.log / timer log 放這
    "timer_log_file": "reboot_timer_log.txt",
}
# ==============================================================================


# === STOP CONDITION: custom logic (user defined) ===
# 回傳 None = 繼續跑；回傳字串 = 停止原因（會顯示並終止程式）。
# 需要跨指令關聯、多條件組合等複雜判斷時，在這裡寫 Python 邏輯。
# log_path 為當輪的 telnet_run_N.txt 完整路徑。
def custom_stop_check(log_path, test_count):
    return None
# === END custom_stop_check ===


# ---------- config 載入 ----------
def _deep_merge(base, override):
    out = dict(base)
    for k, v in override.items():
        if isinstance(v, dict) and isinstance(out.get(k), dict):
            out[k] = _deep_merge(out[k], v)
        else:
            out[k] = v
    return out


def load_config():
    cfg = copy.deepcopy(CONFIG)
    if "--config" in sys.argv:
        idx = sys.argv.index("--config")
        if idx + 1 >= len(sys.argv):
            sys.exit("❌ --config 需要搭配 JSON 檔路徑")
        with open(sys.argv[idx + 1], "r", encoding="utf-8") as f:
            cfg = _deep_merge(cfg, json.load(f))
    return cfg


CFG = load_config()

# 展平成模組常數，讓下方流程程式碼與範本一致
IP_ADDRESS = CFG["ip_address"]
USERNAME = CFG["username"]
PASSWORD = CFG["password"]
REBOOT_WAIT_INTERVAL = CFG["reboot_wait_interval"]
REQUIRED_PINGS = CFG["required_pings"]
RAS_PORT = CFG["ras_port"]
RAS_BAUDRATE = CFG["ras_baudrate"]
RELAY_GPIO = CFG["relay_gpio"]
POWER_OFF_SECONDS = CFG["power_off_seconds"]
TELNET_COMMANDS = CFG["telnet_commands"]
UART_LOG = CFG.get("uart_log", {})
STOP_CONDITIONS = CFG.get("stop_conditions", [])
OUTPUT_DIR = CFG.get("output_dir", ".")
TIMER_LOG_FILE = os.path.join(OUTPUT_DIR, CFG.get("timer_log_file", "reboot_timer_log.txt"))


# ---------- GPIO / Power 控制 ----------
def open_ras():
    return serial.Serial(RAS_PORT, baudrate=RAS_BAUDRATE, bytesize=8,
                         parity='N', stopbits=1, timeout=1)


def gpio_export(ser):
    ser.write(b"echo %d > /sys/class/gpio/export\n" % RELAY_GPIO)
    time.sleep(0.1)
    ser.write(b"echo out > /sys/class/gpio/gpio%d/direction\n" % RELAY_GPIO)
    time.sleep(0.1)


def power_off(ser):
    ser.write(b"echo 0 > /sys/class/gpio/gpio%d/value\n" % RELAY_GPIO)
    time.sleep(0.1)


def power_on(ser):
    ser.write(b"echo 1 > /sys/class/gpio/gpio%d/value\n" % RELAY_GPIO)
    time.sleep(0.1)  # high trigger


def power_cycle():
    """開 COM port -> gpio export -> 斷電 POWER_OFF_SECONDS 秒 -> 上電 -> 關閉 COM port"""
    ser = open_ras()
    try:
        time.sleep(0.5)  # 等 Pi serial shell ready
        gpio_export(ser)
        power_off(ser)
        time.sleep(POWER_OFF_SECONDS)
        power_on(ser)
        time.sleep(2)
    finally:
        ser.close()


# ---------- UART log（選配）----------
def get_timestamp():
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f")[:-2]


def uart_reader(ser, logfile, stop_event):
    """背景 thread：把 UART 輸出加上時間戳寫進 logfile，直到 stop_event 被 set"""
    with open(logfile, "a", encoding="utf-8", errors="ignore") as f:
        while not stop_event.is_set():
            try:
                line = ser.readline()
            except Exception:
                break
            if line:
                f.write(f"[{get_timestamp()}] {line.decode(errors='ignore')}")
                if not line.endswith(b"\n"):
                    f.write("\n")
                f.flush()


def start_uart(uart_ser, test_count):
    """回傳 (stop_event, thread)；uart 未啟用時回傳 (None, None)"""
    if uart_ser is None:
        return None, None
    uart_log_path = os.path.join(OUTPUT_DIR, f"uart_run_{test_count}.log")
    stop_event = threading.Event()
    t = threading.Thread(target=uart_reader, args=(uart_ser, uart_log_path, stop_event),
                         daemon=True)
    t.start()
    return stop_event, t


def stop_uart(uart_stop, uart_thread):
    if uart_stop is not None:
        uart_stop.set()
    if uart_thread is not None:
        uart_thread.join(timeout=3)


# ---------- ping / 計時統計 ----------
def ping(host):
    """常規 Ping 檢查 (Windows/Linux 相容)"""
    param = "-n" if platform.system().lower() == "windows" else "-c"
    command = ["ping", param, "1", "-w", "1500", host]
    try:
        output = subprocess.run(
            command, stdout=subprocess.PIPE, stderr=subprocess.PIPE
        )
        return output.returncode == 0
    except Exception:
        return False


def get_next_test_count():
    """讀取計時 Log 檔案來判斷目前是第幾次測試，以延續計數"""
    if not os.path.exists(TIMER_LOG_FILE):
        return 1
    try:
        with open(TIMER_LOG_FILE, "r", encoding="utf-8") as f:
            lines = f.readlines()
            for line in reversed(lines):
                if "第" in line and "次" in line:
                    start = line.find("第") + 1
                    end = line.find("次")
                    return int(line[start:end].strip()) + 1
    except Exception:
        pass
    return 1


def log_timer(count, duration):
    """寫入總時間統計 Log"""
    log_msg = f"第 {count} 次 從power_cycle 到 telnet 成功花了 {duration:.2f} 秒\n"
    print(f"🎉 [統計結算] 寫入總表: {log_msg.strip()}")
    try:
        with open(TIMER_LOG_FILE, "a", encoding="utf-8") as f:
            f.write(log_msg)
    except Exception as e:
        print(f"❌ 寫入總計時檔案失敗: {e}")


# ---------- Telnet ----------
def telnet_connect(host, user, pwd):
    """建立 Telnet 連線並送出帳號密碼，成功回傳 socket，失敗回傳 None"""
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(5)
        s.connect((host, 23))
        time.sleep(1)
        s.sendall(user.encode("ascii") + b"\n")
        time.sleep(1)
        s.sendall(pwd.encode("ascii") + b"\n")
        time.sleep(2)
        return s
    except Exception as e:
        print(f"  ⚠️ Telnet 連線失敗: {e}")
        return None


def telnet_read_all(s, idle_timeout=2.0):
    """持續讀取 Telnet 資料，直到 idle_timeout 秒內沒有新資料為止"""
    data = b""
    s.settimeout(0.5)
    last_rx = time.time()
    while (time.time() - last_rx) < idle_timeout:
        try:
            chunk = s.recv(4096)
            if not chunk:
                break
            data += chunk
            last_rx = time.time()
        except socket.timeout:
            pass
        except Exception:
            break
    return data


def clean_telnet_output(data):
    """移除 Telnet IAC 控制序列並解碼為文字"""
    IAC = 255
    out = bytearray()
    i = 0
    while i < len(data):
        if data[i] == IAC:
            i += 1
            if i >= len(data):
                break
            cmd = data[i]
            i += 1
            if cmd == IAC:                    # 逸出的 0xFF
                out.append(IAC)
            elif cmd in (251, 252, 253, 254): # WILL/WONT/DO/DONT，後面帶 1 byte option
                i += 1
            elif cmd == 250:                  # SB ... IAC SE
                while i + 1 < len(data) and not (data[i] == IAC and data[i + 1] == 240):
                    i += 1
                i += 2
        else:
            out.append(data[i])
            i += 1
    return out.decode("utf-8", errors="ignore")


def run_telnet_commands(s, file_obj):
    """進 shell 後依序下達 TELNET_COMMANDS，輸出寫入 Log 檔"""
    s.sendall(b"sh\n")
    telnet_read_all(s, idle_timeout=1.5)

    for cmd in TELNET_COMMANDS:
        print(f"  └─ Telnet 執行: {cmd}")
        file_obj.write(f"[{get_timestamp()}] === [Telnet CMD] {cmd} ===\n")
        s.sendall(cmd.encode("ascii") + b"\n")
        output = clean_telnet_output(telnet_read_all(s, idle_timeout=3.0))
        file_obj.write(output if output.endswith("\n") else output + "\n")
        file_obj.flush()


# ---------- 停止條件 ----------
def _extract_cmd_section(txt, cmd):
    """從 log 文字中取出指定 [Telnet CMD] 區塊的輸出，取不到回傳 None"""
    m = re.search(r"=== \[Telnet CMD\] " + re.escape(cmd) + r" ===\n(.*?)(?==== \[Telnet CMD\]|\Z)",
                  txt, re.S)
    return m.group(1) if m else None


def evaluate_stop(log_path, test_count):
    """檢查當輪 log 是否觸發停止條件。

    回傳 None = 繼續跑；回傳字串 = 停止原因。
    先跑 config 的 stop_conditions regex 規則，再跑 custom_stop_check()。
    """
    reason = None

    if STOP_CONDITIONS:
        try:
            with open(log_path, "r", encoding="utf-8", errors="replace") as f:
                txt = f.read()
        except Exception as e:
            print(f"  ⚠️ 讀取 log 檢查停止條件失敗: {e}")
            txt = ""

        for rule in STOP_CONDITIONS:
            scope = rule.get("scope", "whole_log")
            pattern = rule.get("pattern", "")
            if not pattern:
                continue
            if scope == "whole_log":
                target = txt
            elif scope.startswith("section:"):
                target = _extract_cmd_section(txt, scope[len("section:"):]) or ""
            else:
                target = txt
            if re.search(pattern, target):
                reason = f"rule[{rule.get('name', '?')}] scope={scope} pattern={pattern!r}"
                break

    if reason is None:
        try:
            custom_reason = custom_stop_check(log_path, test_count)
        except Exception as e:
            print(f"  ⚠️ custom_stop_check 發生錯誤: {e}")
            custom_reason = None
        if custom_reason:
            reason = str(custom_reason)

    return reason


def main():
    os.makedirs(OUTPUT_DIR, exist_ok=True)

    print("\n" + "="*60)
    print(" 機器重啟生命週期壓力測試 - Telnet 指令 + Pi GPIO Power Cycle")
    print(f" 目標 IP: {IP_ADDRESS}")
    print(f" Pi 控制 Port: {RAS_PORT} (GPIO{RELAY_GPIO})")
    if UART_LOG.get("enabled"):
        print(f" UART log: {UART_LOG.get('port')} -> uart_run_N.log")
    if STOP_CONDITIONS:
        print(f" 停止條件: {len(STOP_CONDITIONS)} 條 regex 規則")
    print(f" 統計總表: {TIMER_LOG_FILE}")
    print("="*60 + "\n")

    # UART port 初始化（選配）
    uart_ser = None
    if UART_LOG.get("enabled"):
        uart_port = UART_LOG.get("port", "")
        if not uart_port:
            sys.exit("❌ uart_log.enabled=true 但未設定 uart_log.port")
        if uart_port.upper() == RAS_PORT.upper():
            sys.exit("❌ uart_log.port 不能與 ras_port 相同")
        uart_ser = serial.Serial(uart_port, baudrate=UART_LOG.get("baudrate", 115200),
                                 bytesize=8, parity='N', stopbits=1, timeout=1)
        print(f"[{get_timestamp()}] UART log 啟用: {uart_port}")

    test_count = get_next_test_count()

    # 初始步驟：確保網路就緒
    success_count = 0
    print(f"[{get_timestamp()}] 【初始步驟】等待網路就緒 (連續 Ping {REQUIRED_PINGS} 次)...")
    while success_count < REQUIRED_PINGS:
        if ping(IP_ADDRESS):
            success_count += 1
            if success_count < REQUIRED_PINGS:
                time.sleep(1)
        else:
            success_count = 0
            time.sleep(2)

    print(f"[{get_timestamp()}] 【初始步驟】網路已通！建立首次 Telnet 連線...")

    # 初始 Telnet 連線（重試直到成功）
    tn_socket = None
    while tn_socket is None:
        tn_socket = telnet_connect(IP_ADDRESS, USERNAME, PASSWORD)
        if tn_socket is None:
            print(f"[{get_timestamp()}] 首次 Telnet 失敗，5 秒後重試...")
            time.sleep(5)

    print(f"[{get_timestamp()}] 【初始步驟】Telnet 已連線！進入無限測試循環...\n")

    # 主測試循環
    while True:
        telnet_log_filename = os.path.join(OUTPUT_DIR, f"telnet_run_{test_count}.txt")
        telnet_file = open(telnet_log_filename, "a", encoding="utf-8", errors="ignore")

        # 啟動本輪 UART log（選配）
        uart_stop, uart_thread = start_uart(uart_ser, test_count)

        print(f"▶️ [測試第 {test_count} 次] 狀態: 透過 Telnet 下達檢查指令...")

        # 動作 A：Telnet 依序下達指令並記錄輸出
        telnet_file.write(f"[{get_timestamp()}] === [Python Script] 開始執行 Telnet 檢查指令 ===\n")
        try:
            run_telnet_commands(tn_socket, telnet_file)
            telnet_file.flush()
        except Exception as e:
            print(f"  ⚠️ Telnet 下指令時發生錯誤: {e}")

        # 動作 A2：停止條件檢查（命中 -> 停止，不再 power cycle）
        stop_reason = evaluate_stop(telnet_log_filename, test_count)
        if stop_reason:
            msg = f"第 {test_count} 次測試觸發停止條件: {stop_reason}"
            print("\n" + "!"*60)
            print(f"🛑 {msg}")
            print(f"   詳細 Log: {telnet_log_filename}")
            print("   停止測試，不再執行 power cycle。")
            print("!"*60 + "\n")
            telnet_file.write(f"[{get_timestamp()}] === [Python Script] {msg} -> 停止測試，不再 power cycle ===\n")
            telnet_file.flush()
            try:
                tn_socket.sendall(b"exit\n")
                time.sleep(1)
            except Exception:
                pass
            try:
                tn_socket.close()
            except Exception:
                pass
            stop_uart(uart_stop, uart_thread)
            telnet_file.close()
            sys.exit(1)

        # 動作 A3：正常 -> 送 exit 並關閉 Telnet
        try:
            tn_socket.sendall(b"exit\n")
            time.sleep(1)
        except Exception as e:
            print(f"  ⚠️ Telnet exit 時發生錯誤: {e}")
        try:
            tn_socket.close()
        except Exception:
            pass

        # 💡 計時起點（從斷電開始算）
        reboot_start_time = time.time()

        telnet_file.write(f"[{get_timestamp()}] === [Python Script] Telnet 已結束，透過 Pi GPIO 執行 power cycle，開始計時 ===\n")
        telnet_file.flush()

        # 動作 B：透過 Raspberry Pi GPIO 斷電 -> 上電
        print(f"🔌 [測試第 {test_count} 次] 狀態: Pi GPIO{RELAY_GPIO} 斷電 {POWER_OFF_SECONDS}s 後上電...")
        try:
            power_cycle()
        except Exception as e:
            print(f"  ⚠️ Power cycle 發生錯誤: {e}")
            telnet_file.write(f"[{get_timestamp()}] === [Python Script] Power cycle 錯誤: {e} ===\n")

        # 動作 C：強制等待
        print(f"⏳ [測試第 {test_count} 次] 狀態: 重啟中，強制死等 {REBOOT_WAIT_INTERVAL} 秒...")
        time.sleep(REBOOT_WAIT_INTERVAL)

        # 動作 D：常規 Ping 通 N 次
        print(f"🔍 [測試第 {test_count} 次] 狀態: 等待結束，開始常規 Ping 檢查...")
        success_count = 0

        while success_count < REQUIRED_PINGS:
            if ping(IP_ADDRESS):
                success_count += 1
                telnet_file.write(f"[{get_timestamp()}] === [Python Script] Ping 成功! 當前連續成功次數: {success_count}/{REQUIRED_PINGS} ===\n")
                telnet_file.flush()
                print(f"  └─ Ping 成功! 當前進度: {success_count}/{REQUIRED_PINGS}")
                if success_count < REQUIRED_PINGS:
                    time.sleep(1)
            else:
                if success_count > 0:
                    print(f"  ⚠️ Ping 中斷！次數歸零重新計算。")
                success_count = 0
                time.sleep(2)

        # 動作 E：重新建立 Telnet 連線（重試直到成功）
        print(f"🔑 [測試第 {test_count} 次] 狀態: Ping 達標，正在嘗試 Telnet 登入...")

        tn_socket = None
        while tn_socket is None:
            tn_socket = telnet_connect(IP_ADDRESS, USERNAME, PASSWORD)
            if tn_socket is None:
                print(f"  ⚠️ Telnet 未就緒，5 秒後重試...")
                time.sleep(5)

        # 💡 計時終點
        end_time = time.time()
        total_elapsed_time = end_time - reboot_start_time

        # 紀錄總時間到計時總表
        log_timer(test_count, total_elapsed_time)

        # 結束本輪 UART log 與獨立 Log
        stop_uart(uart_stop, uart_thread)

        telnet_file.write(f"[{get_timestamp()}] === [Python Script] 第 {test_count} 次測試 Telnet 登入成功，本份 Log 結束 ===\n")
        telnet_file.close()

        print(f"✅ [測試第 {test_count} 次] 完畢。Telnet 結果已儲存為 {telnet_log_filename}\n" + "-"*40 + "\n")

        test_count += 1
        time.sleep(2)


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\n[使用者中止程式。]")
        sys.exit(0)
```

---

## 附錄 B：config json 範例（選配，config-file 模式用）

```json
{
  "ip_address": "192.168.1.1",
  "username": "admin",
  "password": "admin",
  "reboot_wait_interval": 90,
  "required_pings": 10,
  "ras_port": "COM3",
  "ras_baudrate": 115200,
  "relay_gpio": 16,
  "power_off_seconds": 5,
  "telnet_commands": [
    "cat /etc/SC_MT_VERSION",
    "lspci",
    "cat /debug/gpio",
    "ifconfig",
    "wl --all status",
    "dmesg | grep -i WLC_E_BSSCFG_UP",
    "wl --all bss",
    "wl -p isup",
    "cat /tmp/wl2_hapd.conf"
  ],
  "uart_log": {
    "enabled": false,
    "port": "COM4",
    "baudrate": 115200
  },
  "stop_conditions": [
    {
      "name": "kernel_panic",
      "scope": "whole_log",
      "pattern": "Kernel panic|kernel BUG"
    },
    {
      "name": "bss_down",
      "scope": "section:wl --all bss",
      "pattern": "---\\s+wl\\d+[^\\n]*\\n\\s*down"
    }
  ],
  "output_dir": ".",
  "timer_log_file": "reboot_timer_log.txt"
}
```
