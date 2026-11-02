---
name: mt-code
description: 當有build flow 不清楚 有關出MT code 或 MT bundle burn in image 的問題
---

# Sercomm XF10 與 XF10_MT 分支差異分析、Bundle Burn-in Image 產出機制與 Jenkins 出 Code 流程全指南
 
---
 
## 1. 概述與分支定位 (Overview & Branch Architecture)
 
在 Sercomm XF10 (Broadcom BCM68375 平台) 專案中，主要維護兩個最核心的代碼分支：
 
| 分支名稱 | 主要定位 | 目標受眾 | 維護策略 |
| :--- | :--- | :--- | :--- |
| **`XF10`** | **R&D 主要研發主線** (Official Development Line) | 研發人員、功能開發、客戶官方 Release 驗證 | 包含最新功能、完整安全機制 (FIT 加密、RootFS 加密、SMC RPC 服務等)。 |
| **`XF10_MT`** | **工廠測試專用出 Code 分支** (Manufacturing Test / Factory Line) | 工廠產線、TFE 測試工程師、量產燒錄 | 從 `XF10` 挑選 (cherry-pick) 必要與穩定的 commit，並針對工廠產線需求進行客製（如預載對齊之 RDK Bootloader、允許進入 U-Boot console、去除冗餘開機初始化以加速燒錄等）。 |
 
```
              ┌──────────────────────────────────────────────┐
              │                XF10 (R&D 主線)                │
              │  - 完整 SMC RootFS / FIT 加密機制              │
              │  - 原始碼即時編譯 Bootloader                  │
              │  - 鎖定 U-Boot Console (PROD)                │
              └──────────────────────┬───────────────────────┘
                                     │
                             cherry-pick 必要 commits
                                     │
                                     ▼
              ┌──────────────────────────────────────────────┐
              │              XF10_MT (工廠產線分支)            │
              │  - 預置 Pre-built RDK Bootloader (對齊客戶)    │
              │  - PROD 開啟 U-Boot Console 除錯權限          │
              │  - 停用 RootFS / FIT Header 加密 (簡化/提速)   │
              │  - TFE 產線優化 (移除空 LUN、減少 bootdelay)  │
              └──────────────────────────────────────────────┘
```
 
---
 
## 2. 核心代碼與架構差異詳解 (Deep-Dive Analysis)
 
透過比對 `origin/XF10` 與 `origin/XF10_MT`（共計 115 個檔案變更，621 行新增，4081 行刪除），發現以下幾項關鍵差異：
 
### 2.1 Bootloader 對齊機制：Pre-built RDK Bootloader
* **問題背景**：
  工廠在燒錄出貨用的 **Burn-in Bundle Image (`burn_image.bin`)** 時，系統內部具備雙 Bank 架構：
  * **Bank 1**：放 MT (工廠測試) 韌體。
  * **Bank 2**：放 RDK (正式交付給客戶 Comcast 的系統) 韌體。
  
  **客戶端 (Comcast) 已經有正式發布驗證過的 Release Bootloader**。如果 Bank 2 的 RDK 開機時使用的是由 MT 本地原始碼臨時編譯出來的 Bootloader，一旦原始碼有些微差異（如 SVN 版本、環境變數、簽名 key 等不一致），將導致開機失敗或與客戶端正式環境脫節。
* **XF10 vs XF10_MT 做法差異**：
  * **`XF10`**：
    打包時**完全使用本地編譯**出來的 `armbl_lun.bin`。無論 Bank 1 還是 Bank 2 均指向同一個剛編譯出的 `armbl_lun.bin`。
  * **`XF10_MT`**：
    在 `sc-bootloader/input/` 目錄中直接存放了經過官方驗證且 Release 給客戶的 **Pre-built Bootloader 二進位檔案**：
    * `armbl_lun_rdk_dev.bin` / `armbl_lun_rdk_prod.bin` (ARM Trusted Firmware / Bootloader)
    * `tee_rdk_dev.bin` / `tee_rdk_prod.bin` (OP-TEE OS)
    * `u-boot-nodtb_rdk_dev.bin` / `u-boot-nodtb_rdk_prod.bin` (U-Boot)
  * **打包時替換**：
    在 `auto_build_bb_dev.sh` / `auto_build_bb_prod.sh` 中，`burn_image.par` 會透過腳本動態替換 Bank 2 的 armbl：
    ```bash
    # XF10_MT 的 auto_build_bb_dev.sh
    if [ ! -f ".skip_rdk_upgrade" ]; then
      sed -i '/&D40000/s/:armbl_lun\.bin    /:armbl_lun_rdk_dev.bin /' burn_image_dev.par
    fi
    $TOP/$TOOL/scBind -i burn_image_dev.par -o burn_image.bin
    ```
    此舉確保了 `burn_image.bin` 在 0xD40000 (Bank 2 ARMBL) 燒錄進去的是**客戶官方對齊的 Bootloader**！
 
---
 
### 2.2 U-Boot 除錯權限：MT PROD 允許進入 U-Boot Console
* **`XF10`**：在正式 PROD build 下，U-Boot 嚴格禁用 autoboot 中斷與互動式 CLI，防止一般使用者或外部進入 bootloader。
* **`XF10_MT`**（Commit `519d88baf`: `[BUGFIX][Bootloader]MT PROD bootloader should be able to get into u-boot`）：
  修改了 `bootloaders/u-boot-2024.04/common/autoboot.c`、`cli.c` 與 `spl.c`。
  **即使在 PROD 模式下，MT 的 Bootloader 依然允許產線測試設備或工程師透過 UART 終端按下按鍵中斷開機並進入 U-Boot 指令列**，這對於產線校正、硬體測試、讀寫暫存器與除錯至關重要。
 
---
 
### 2.3 安全架構與加密機制 (SMC FIT / dm-crypt RootFS Encryption)
這是兩者代碼量差距最大的一處（相差 4000 多行）：
 
| 安全功能 | `XF10` (R&D 主線) | `XF10_MT` (工廠分支) | 差異原因 |
| :--- | :--- | :--- | :--- |
| **FIT Image Encryption** | **啟用** (`bcm_uboot_fit_img_enc = uboot kernel dtb atf optee`) | **停用** (註解掉) | 工廠測試無需每一映像檔均 AES 加密，停用以加快編譯與測試穩定度。 |
| **FIT Header Encryption** | **啟用** (`bcm_sec_fit_hdr_cypher = a2k3`) | **停用** (無 `encrypt_fit_hdr` 步驟) | 避免需要額外 `a2k3` 金鑰維護與解密開銷。 |
| **RootFS 加密 (dm-crypt)** | **啟用** (`docker_run_encrypt_verity_rootfs.sh`, `dm.txt`, `rootfs.enc`) | **停用** (直接使用 raw `rootfs.img`) | 工廠若測試加密 rootfs，損壞時無法掛載除錯；MT 採用純 raw squashfs。 |
| **Kernel SMC Crypto RPC** | **啟用** (`bcm_rpc_ba_decrypt`, `BA_SVC_SEC_AES_CRYPT`) | **移除** | 核心無需向 SMC 請求解密 rootfs 金鑰。 |
| **U-Boot SMC 解密功能** | **包含** `bcm_sec_smc_decrypt_fit` 等 | **移除** | U-Boot 簡化，僅執行標準簽章驗證。 |
| **PROD 簽章機制** | 本地腳本 / HSM 簽名 | 調用遠端 HSM 伺服器 (`hsm_linux_amd64 -s 172.31.7.64 -p 9443`) | 產線正式簽章直接走公司內部 HSM 服務。 |
 
---
 
### 2.4 工廠產線開機速度優化 (TFE Time Reduction)
Commit `81e6a813c` (`[TFE]Reduce encryption time`) 與 `0c21b3ab7` (`Reduce bootdelay`) 專為工廠產線所做：
1. **精簡 Flash Partition Map (`dyn_lun.ini`)**：
   * 在 `XF10` 中，`burn_image.bin` 包含了大量空白的 LUN 分區（如 `boot_data`, `nvram`, `nvram2`, `App_Data`, `App_Storage`, `App_Logging`, `Cri_store`, `NCri_store`, `data2`）。
   * 在 `XF10_MT` 中，將這些空白 LUN 從 `burn_image.bin` 中移除，使得 **燒錄檔大幅縮小，產線燒錄時間縮短數分鐘**。
   * 移除的分區改由 Linux 啟動服務腳本 `targets/fs.src/etc/init.d/create-vfbio-luns.sh` 於**開機時動態自動建立**。
2. **縮短 Bootdelay**：
   * `env_smc_XF10_*.conf` 中的 `bootdelay` 調整，減少產線重啟等待時間。
 
---
 
### 2.5 DTS、硬體與周邊驅動差異
 
| 項目 | `XF10` | `XF10_MT` |
| :--- | :--- | :--- |
| **`kernel/dts/68375/XF10.dts`** | `model = "XF10_DVT"`, 記憶體配置為 **LPDDR5** (32Gb, 2750MHz) | `model = "XF10"`, 記憶體配置為 **DDR4** (2048MB, 1600MHz) |
| **`targets/XF10/XF10` Board ID** | `BRCM_BOARD_ID="XF10_DVT"` | `BRCM_BOARD_ID="XF10_EVT3"` |
| **Reset Button (`reset_button.c`)** | 曾被修改為短按切換 LED 狀態 (`switch_led_status` 呼叫 `/etc/sc_led.sh`) | 保留標準行為：呼叫 `board_util_restoredefault(BTN_ACT_RESET)` 進行系統還原重啟 |
| **Ethernet PHY Tx Power** | 包含特定 PHY tx-power 調整 | 配合最新 Broadcom CS 修正 (如 CS00012468303、CS00012467826) |
 
---
 
## 3. Bundle Burn-in Image 與相關 Image 產出機制 (Packaging Architecture)
 
產出完整映像檔的架構由以下三個層級構成：
 
```
[源碼編譯] 
  make PROFILE=...
      ├── targets/XF10/bcmXF10_armbl_lun.bin (MT ARMBL)
      ├── targets/XF10/rootfs.img           (MT RootFS)
      ├── bootloaders/obj/uboot/u-boot-nodtb.bin
      └── bootloaders/obj/optee/tee.bin
                               │
[Post-Gen 收集]                ▼
  auto_build_post_gen.sh 複製至 sc-bootloader/input/
      ├── MT 編譯檔案
      └── RDK 檔案 (vmlinux_rdk.bin.lzo, rootfs_rdk.img, prebuilt bootloaders)
                               │
[sc-bootloader 打包]            ▼
  auto_build_bb_dev.sh / auto_build_bb_prod.sh
      ├── mkimage       ──> bootfs.bin (MT) & bootfs_rdk.bin (RDK)
      ├── gen4_svn_composer ──> svn_mt.bin & svn_sec_rdk.bin
      ├── vfbnize       ──> dyn_lun.bin (包含 LUN 3, 4, 5, 6, 8, 11, 18, 19, 20)
      ├── scBind (smc_loader.par)  ──> smc_bootl.bin
      ├── scBind (burn_image_*.par)──> burn_image.bin
      └── mkimage (*.pkgts)        ──> XF10_upgrade_mt.bin & XF10_upgrade_sec_rdk.bin
```
 
### 3.1 核心產出檔案清單
 
| 檔案名稱 | 產出工具 | 存放位置 (targets/...) | 用途與說明 |
| :--- | :--- | :--- | :--- |
| **`${VERSION}_burn_image.bin`** | `scBind` | `emmc/` (Jenkins: `bundle_image/`) | **工廠產線燒錄用全鏡像檔**，直接燒錄於 eMMC User Data 分區，內含 SMC OS、U-Boot Env、雙 Bank Bootloader、雙 Bank 系統。 |
| **`${VERSION}_smc_bootl.bin`** | `scBind` | `emmc/` (Jenkins: `bundle_image/`) | **SMC Bootloader**，工廠燒錄於 eMMC Boot0 / Boot1 分區。 |
| **`${VERSION}_XF10_upgrade_mt.bin`** | `mkimage` | `web_upgrade/` | **MT 韌體升級檔**，用於 Web UI 或 CLI 升級 Bank 1。 |
| **`${VERSION}_XF10_upgrade_sec_rdk.bin`** | `mkimage` | `web_upgrade/` | **RDK 韌體升級檔**，用於 Web UI 或 CLI 升級 Bank 2。 |
 
---
 
### 3.2 `burn_image.bin` 的 Flash Map 佈局 (Flash Layout)
 
`burn_image_dev.par` 與 `burn_image_prod.par` 定義了 eMMC 燒錄的絕對偏移位置：
 
```
偏移位置 (Offset)          內容 (Content)                                備註
-----------------------------------------------------------------------------------------------------
0x00000000 (&000000)   smc_os-68375B0.bin.prodkey.hsm_signed.flash   SMC OS (Primary)
0x00500000 (&500000)   smc_os-68375B0.bin.prodkey.hsm_signed.flash   SMC OS (Secondary 備援)
0x00A00000 (&A00000)   ubootenv_lun.bin                              U-Boot Environment (Primary)
0x00A20000 (&A20000)   ubootenv_lun.bin                              U-Boot Environment (Secondary)
0x00A40000 (&A40000)   meminit_lun.bin                               Memory Init (Bank 1)
0x00B40000 (&B40000)   armbl_lun.bin                                 ★ Bank 1 ARMBL (MT 原始碼編譯)
0x00C40000 (&C40000)   meminit_lun.bin                               Memory Init (Bank 2)
0x00D40000 (&D40000)   armbl_lun_rdk_dev.bin / prod.bin              ★ Bank 2 ARMBL (Pre-built RDK 客戶端)
0x00E80000 (&E80000)   dyn_lun.bin (由 vfbnize 產生)                  動態 LUN 容器 (見下方細項)
```
 
#### `dyn_lun.bin` 內部包含的 LUN (Dynamic LUNs)：
* **LUN 3 (`bootfs1`)**：Bank 1 MT Kernel / DTB / OP-TEE (`bootfs.bin`)
* **LUN 4 (`rootfs1`)**：Bank 1 MT Root File System (`rootfs.img`)
* **LUN 5 (`bootfs2`)**：Bank 2 RDK Kernel / DTB / OP-TEE (`bootfs_rdk.bin` 或 DUAL_MT 模式下的 `bootfs.bin`)
* **LUN 6 (`rootfs2`)**：Bank 2 RDK Root File System (`rootfs_rdk.img` 或 DUAL_MT 模式下的 `rootfs.img`)
* **LUN 8 (`defaults`)**：預設設定分區
* **LUN 11 (`data`)**：使用者資料分區
* **LUN 18 (`crashdump`)**：當機日誌分區
* **LUN 19 (`svn1`)**：Bank 1 Secure Version Manifest (`svn_mt.bin`)
* **LUN 20 (`svn2`)**：Bank 2 Secure Version Manifest (`svn_sec_rdk.bin` 或 `svn_mt.bin`)
 
---
 
## 4. Jenkins 自動化出 Code 流程全解析 (Jenkins Pipeline Guide)
 
Jenkinsfile 採用 Declarative Pipeline，運行於帶有 `docker` label 的 Node，並在名為 **`ApplePie`** 的 Docker 容器中執行編譯與打包工作。
 
### 4.1 Jenkins 參數說明 (Parameters)
 
| 參數名稱 | 預設值 | 可選項目 | 說明 |
| :--- | :--- | :--- | :--- |
| **`BRANCH`** | `XF10_MT` | 字串 | Gerrit 分支名稱。 |
| **`REVISION`** | *(空)* | Commit SHA / Tag | 欲編譯的基礎版本，若留空則為該 Branch 之 HEAD。 |
| **`PREVIOUS_RELEASE_COMMIT`** | *(空)* | Commit SHA / Tag | 上一個 Release Commit，用於產出兩版本間的 Release Notes。留空會自動搜尋 first-parent history。 |
| **`REVIEW_COMMITS`** | *(空)* | 多行文字 | 欲臨時 cherry-pick 的 Gerrit Ref (如 `refs/changes/69/4169/1`) 或 Commit SHA。 |
| **`REVERT_COMMITS`** | *(空)* | 多行文字 | 欲在套用 Review 前先 revert 的 Commit SHA。 |
| **`TARGET`** | `XF10` | `XF10`, `XF10_CTC` | 編譯硬體 Profile。 |
| **`WIFI_MODE`** | `WLTEST` | `WLTEST`, `NORMAL` | Wi-Fi 模式：<br>• `WLTEST`：產線量測模式 (對應 auto_build 參數 3=`WLTEST`, 4=`VERSION`)<br>• `NORMAL`：正式 Wi-Fi 模式 (對應 auto_build 參數 3=`VERSION`, 4=`NONE`) |
| **`VERSION`** | `M5.04.D2.003` | 字串 | MT 韌體版本號碼。 |
| **`BUILD_TYPE`** | `DEV` | `DEV`, `PROD` | 金鑰類型 (開發測試用 DEV 或是量產 PROD)。 |
| **`CERT_TYPE`** | `PROD_CERT` | `PROD_CERT`, `DEV_CERT` | 憑證類型 (僅在 PROD 模式下有效)。 |
| **`RDK_VMLINUX`** | *(Stashed)* | 檔案上傳 | 客戶端 RDK Kernel 檔案 (`vmlinux_rdk.bin.lzo`)。 |
| **`RDK_ROOTFS`** | *(Stashed)* | 檔案上傳 | 客戶端 RDK RootFS 檔案 (`rootfs_rdk.img`)。 |
 
---
 
### 4.2 雙模式自動偵測：MT + RDK vs DUAL MT
 
Jenkinsfile 的 `stage('Prepare Uploaded RDK Files')` 會檢查使用者上傳的檔案：
 
1. **MT + RDK 模式 (雙系統模式)**：
   * 同時上傳了 `RDK_VMLINUX` 與 `RDK_ROOTFS`。
   * `RDK_MODE` 設為 `"RDK"`。
   * 將檔案複製進容器的 `sc-bootloader/input/` 目錄。
   * 執行 `auto_build.sh` 時，自動輸入 2 個 `y`：
     ```bash
     BUILD_INPUT='y
     y'
     ```
     （1: 允許執行 post_gen；2: 允許清理 input 暫存並保留 RDK 檔案）
2. **DUAL MT 模式 (雙 Bank MT 模式)**：
   * 兩者皆未上傳 (留空)。
   * `RDK_MODE` 設為 `"DUAL_MT"`。
   * 執行 `auto_build.sh` 時，自動輸入 3 個 `y`：
     ```bash
     BUILD_INPUT='y
     y
     y'
     ```
     （1: 允許執行 post_gen；2: 允許清理 input 暫存；3: 確認略過 RDK 模式，產生 `.skip_rdk_upgrade`，雙 Bank 均使用 MT 韌體）
3. **錯誤防護**：
   * 若只上傳其中一個檔案（例如只有 Kernel 沒有 RootFS），Jenkins 會直接中斷並拋出 FATAL ERROR，避免產出不完整的無效韌體。
 
---
 
### 4.3 Pipeline 核心 Stages 執行時序
 
```
Stage 1: Prepare Build Configuration
   ├── 檢查參數合法性 (Regex 檢查)
   ├── 映射 WIFI_MODE 至 auto_build.sh 參數順序
   └── 確認 ApplePie 容器正在運作
 
Stage 2: Prepare Uploaded RDK Files
   ├── 解開 Stashed 檔案
   └── 判斷並設定 RDK_MODE 為 "RDK" 或 "DUAL_MT"
 
Stage 3 & 4: Create Build Folder & Clone MT Code
   ├── 於容器中建立 /opt/comcast/XF10_MT/${VERSION}/${BUILD_NUMBER}
   └── 自 Gerrit clone 6.3.0GA 分支
 
Stage 5, 6, 7: Checkout Base Revision, Revert & Cherry-Pick Reviews
   ├── Checkout 目標分支或 Tag
   ├── 依序執行 git revert (支援非 merge commit)
   └── 依序 fetch 並 cherry-pick Gerrit refs (如 refs/changes/...)
 
Stage 8: Generate Release Notes
   └── 遍歷 first-parent commit 歷史，自動產出 RELEASE_NOTES.txt
 
Stage 9 & 10: Prepare RDK Bank & Save Build Information
   ├── 將 RDK 映像檔放置至 sc-bootloader/input/ (若為 RDK 模式)
   └── 產出包含完整 Commit SHA 與編譯參數的 BUILD_INFO.txt
 
Stage 11: Check ApplePie Docker-in-Docker (DinD)
   └── 確保容器具備 --privileged 且內層 docker daemon 運作正常 (Broadcom 編譯所需)
 
Stage 12: Build MT Image
   ├── 傳入 BUILD_INPUT ('y\ny' 或 'y\ny\ny')
   ├── 執行 sh auto_build.sh ${TARGET} NONE ${ARG3} ${ARG4} ${BUILD_TYPE} [${CERT_TYPE}]
   └── 驗證 targets/... 下的 upgrade_mt, smc_bootl, burn_image 等檔案是否存在
 
Stage 13 & 14: Verify Bootloader Binaries & Generated Images
   └── 提取並重命名 armbl_lun, tee.bin, u-boot-nodtb 並計算 MD5
 
Stage 15 & 16: Collect & Archive Jenkins Artifacts
   └── 歸檔至 Jenkins 產物目錄
```
 
---
 
### 4.4 Jenkins 產物 (Artifacts) 結構清單
 
編譯完成後，Jenkins 會在 `artifacts/` 目錄歸檔以下結構：
 
```
artifacts/
├── bundle_image/                         # ★ 工廠燒錄用映像檔
│   ├── ${VERSION}_burn_image.bin         # eMMC User Data 全燒錄鏡像檔
│   └── ${VERSION}_smc_bootl.bin          # eMMC Boot Partition SMC 開機載入檔
├── web_upgrade/                          # ★ Web 升級檔
│   ├── ${VERSION}_XF10_upgrade_mt.bin    # Bank 1 MT 升級檔
│   └── ${VERSION}_XF10_upgrade_sec_rdk.bin # Bank 2 RDK 升級檔 (RDK 模式時產出)
├── bootloader/                           # ★ 開機載入器個別二進位檔
│   ├── armbl_lun_${dev/prod}.bin
│   ├── tee_${dev/prod}.bin
│   └── u-boot-nodtb_${dev/prod}.bin
├── rtecdc/                               # ★ Wi-Fi 韌體與量產測試工具
│   ├── rtecdc.bin                        # Broadcom Wi-Fi RAM Firmware
│   └── rtecdc.exe                        # Wi-Fi 產線測試執行檔
├── emmc/                                 # 其他原始 emmc partition 檔案
├── RELEASE_NOTES.txt                     # 本次 Release 詳細 Commit 清單與變更說明
├── BUILD_INFO.txt                        # 完整編譯環境、時間、Git Commit、指令摘要
└── MD5SUMS.txt                           # 所有產物的 MD5 校驗清單
```
 
---
 
## 5. 常見問題與維護注意事項 (Troubleshooting & Best Practices)
 
1. **RDK Bootloader 更新時的注意事項**：
   * 當客戶（Comcast）更新了官方 release 的 Bootloader 版本（例如升級至 0010 版）：
   * **務必將最新的 prebuilt bootloader 檔案同步更新至 `XF10_MT` 分支的 `sc-bootloader/input/` 目錄**（包含 `armbl_lun_rdk_*.bin`、`tee_rdk_*.bin`、`u-boot-nodtb_rdk_*.bin`）。
   * 否則，產線產出的 `burn_image.bin` 燒錄出的 Bank 2 仍會停留在舊版的客戶 Bootloader。
2. **Wi-Fi 模式挑選**：
   * 產線校準與吞吐量量測，務必選擇 `WIFI_MODE: WLTEST`，此時 Broadcom 驅動會掛載 `config_pcie_fdap_mfgtest` 且產出 `WLTEST` 版 armbl 與 rtecdc。
   * 出貨正式連線測試，請選擇 `WIFI_MODE: NORMAL`。
3. **Jenkins DinD (Docker-in-Docker) 檢查失敗**：
   * Broadcom 的底層編譯腳本需要執行內層 docker。
   * 若 Stage 11 出現 `FATAL: ApplePie was not created with --privileged`，必須在宿主機以 `--privileged` 重新啟動 `ApplePie` 容器。
4. **Gerrit Cherry-pick 失敗**：
   * 若在 Jenkins 的 `REVIEW_COMMITS` 傳入了有衝突的 Patchset，Pipeline 會在 `Stage 7` 自動執行 `git cherry-pick --abort` 並印出衝突檔案清單，保護目錄不被弄髒。
