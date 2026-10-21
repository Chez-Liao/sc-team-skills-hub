---
name: yocto-bitbake-troubleshooter
description: 當遇到 Yocto / BitBake 編譯失敗、修改 .bb 或 .bbappend 檔案、或處理 QA Issue (如 installed-vs-shipped) 時使用。
---

# Yocto BitBake 診斷與修復指南

1. **定位 Log 檔案**：
   - 優先讀取 `WORKDIR/temp/log.do_compile` 或 `log.do_install`。
2. **常見 QA Issue 處理**：
   - `installed-vs-shipped`: 檢查 `FILES:${PN}` (或舊版語法 `FILES_${PN}`) 是否涵蓋安裝路徑。
   - `ldflags`: 確認 Makefile 中未覆寫 `LDFLAGS`，必要時加上 `TARGET_CC_ARCH += "${LDFLAGS}"`。
3. **修改驗證**：
   - 修改 recipe 後執行 `bitbake <recipe> -c cleansstate && bitbake <recipe>` 驗證。
