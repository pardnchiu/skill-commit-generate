---
name: commit-generate
description: Generate one-sentence commit message in Traditional Chinese from staged changes. Use when user wants a commit message generated from currently staged git files (git diff --cached).
---

# Commit Message Generator

從 git diff 產生單句繁體中文 commit message。

## Input

執行 `git diff --cached` 取得目前已 stage 的變更。若無 staged 變更，提示使用者先執行 `git add`。

## Steps

1. 使用 Bash 工具執行 `git diff --cached` 取得 staged diff
2. 若輸出為空，回應：「目前沒有 staged 的變更，請先執行 `git add <file>` 後再試。」並停止
3. 依據 diff 內容，套用下方規則產生 commit message
4. 輸出 commit message（純文字，不加額外說明）

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
