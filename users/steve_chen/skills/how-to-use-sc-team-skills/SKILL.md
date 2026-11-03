---
name: how-to-use-sc-team-skills
description: 使用 Sercomm Team Skill Hub、安裝與更新 Devin team-skills Plugin、驗證同步或排除安裝問題時使用。
---
# How to Use SC Team Skills

## 日常使用
1. 登入 Team Skill Hub。
2. 選擇需要的 Skill；本 Skill 為系統必備，不能停用。
3. 等待下方 GitHub 操作進度監視器顯示同步完成。
4. 在 Windows PowerShell 執行：

```powershell
devin plugins update
devin plugins list
```

5. 回到 Devin Desktop，執行 Reload，選 Local 或 Remote SSH，建立 New Session。
6. 輸入 `/team-skills:`，確認啟用的技能已出現。

## 第一次安裝
1. 在 Skill Hub 確認 GitHub Remote / main 狀態正常。
2. 到 Devin Web：Customize → Plugins → Add plugin → From repository。
3. Repository 使用 Skill Hub 顯示的 GitHub Repository。
4. Subdirectory 使用 `users/<你的名字>`。
5. 安裝後執行 Reindex plugins。
6. 本機執行 `devin plugins update`，再 Reload Devin Desktop。

## 快速驗證
```powershell
devin --version
devin plugins list
devin plugins update
```

預期 Plugin 清單可看到 `team-skills`；新 Session 輸入 `/team-skills:` 可看到已啟用技能。

## 排錯順序
1. GUI：確認 Skill 顯示為已啟用。
2. SQLite DB：查看三端同步狀態中的個人啟用清單。
3. GitHub Remote：確認 main 有 `users/<名字>/.devin-plugin/plugin.json` 與 `skills/`。
4. Devin Web：移除卡住的舊安裝，重新 Add from repository，然後 Reindex。
5. Devin Local：執行 `devin plugins update`，Reload，建立 New Session。

若 GitHub 正常但 Devin Web 仍停在 Installing，提供 Repository、Subdirectory、Git commit SHA 與畫面狀態給 Devin Support。
