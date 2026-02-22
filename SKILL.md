---
name: commit-generate
description: Generate one-sentence commit message in Traditional Chinese from git changes. Uses staged diff (git diff --cached) if any files are staged, otherwise falls back to full diff (git diff).
---

# Commit Message Generator

從 git diff 產生單句繁體中文 commit message。

## Input

依據是否有 staged 檔案，自動選擇 diff 來源。

## Steps

1. 使用 Bash 工具執行 `git diff --cached --name-only` 檢查是否有 staged 檔案
2. **若有 staged 檔案**：執行 `git diff --cached` 取得 staged diff
3. **若無 staged 檔案**：執行 `git diff` 取得工作區 diff
4. 若兩者輸出皆為空，回應：「目前沒有任何變更。」並停止
5. 依據 diff 內容，套用下方規則產生 commit message
6. 輸出 commit message（純文字，不加額外說明）

## Classification Tags

| Tag | 適用情境 |
|-----|----------|
| `feat` | 新功能、新 endpoint、新元件 |
| `fix` | Bug 修正、錯誤處理、nil check |
| `update` | 修改既有行為、參數調整 |
| `add` | 新增檔案/資源（非功能） |
| `remove` | 刪除程式碼或檔案 |
| `refactor` | 重構（行為不變） |
| `perf` | 效能優化 |
| `style` | 格式化、排版 |
| `doc` | 文件、註解 |
| `test` | 測試相關 |
| `chore` | CI/CD、工具、依賴管理 |
| `security` | 安全性修補 |
| `breaking` | 破壞性變更 |

## Output Format
```
tag: 一句話描述變更內容（繁體中文）
```

## Rules

1. **單一 Tag** — 選擇最能代表本次變更核心意圖的 Tag
2. **Tag 優先序** — `BREAKING` > `FEAT` > `FIX` > `SECURITY` > `UPDATE` > `REFACTOR` > `PERF` > others
3. **簡潔** — 不超過 50 字，動詞開頭（新增、修正、重構、移除、優化）
4. **合併相關變更** — 多個小改動歸納為單一描述

## Examples
```
feat: 新增 Docker 環境自動偵測與資料庫路徑切換機制
fix: 修正使用者登入時 token 過期未正確處理的問題
refactor: 重構訂單模組並抽離共用驗證邏輯
update: 調整 API timeout 為 30 秒並加入 context 傳遞
chore: 升級 Go 版本至 1.22 並更新相依套件
breaking: 移除舊版 API v1 endpoint，需更新所有 client
security: 修補 SQL injection 漏洞並加入參數化查詢
perf: 優化批次查詢減少 N+1 問題
```
