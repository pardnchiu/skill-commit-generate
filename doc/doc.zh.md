# commit-generate - 技術文件

> 返回 [README](./README.zh.md)

## 前置需求

- Claude Code CLI（安裝於本機並可存取 skills 目錄）
- Git ≥ 2.23（支援 `git diff --cached` 與現代 diff 行為）
- 目標專案為 git 儲存庫

## 安裝

### 放置到使用者 skills 目錄

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/pardnchiu/skill-commit-generate.git \
    ~/.claude/skills/commit-generate
```

### 放置到專案 skills 目錄

```bash
mkdir -p <project>/.claude/skills
git clone https://github.com/pardnchiu/skill-commit-generate.git \
    <project>/.claude/skills/commit-generate
```

### 驗證安裝

```bash
ls ~/.claude/skills/commit-generate/SKILL.md
```

若檔案存在，重啟 Claude Code 後即可透過 `/commit-generate` 觸發。

## 使用方式

### 基礎

```bash
# 1. 先選擇要進入 commit 的檔案
git add <file1> <file2>

# 2. 於 Claude Code 呼叫 skill
/commit-generate
```

輸出範例：

```
feat: Add Docker environment auto-detection and database path switching
feat: 新增 Docker 環境自動偵測與資料庫路徑切換機制
```

### 跨主題變更

當 staged diff 橫跨多個語意無關的模組或觸及 2+ primary tag 時：

```
⚠️ 偵測到跨主題變更，建議拆分為多次 commit：
- 認證模組重構：internal/auth/*.go
- UI 樣式調整：web/styles/*.css

若仍要合併，以下為概括描述：
refactor: Split auth module and adjust UI styles
refactor: 重構認證模組並調整 UI 樣式
```

### 無 staged 變更

```
目前沒有 staged 變更，請先執行 `git add` 選擇要提交的檔案。
```

## 參考

### Classification Tags

| Tag | 適用情境 |
|-----|----------|
| `feat` | 新功能、新 endpoint、新元件 |
| `fix` | Bug 修正、錯誤處理、nil check |
| `update` | 修改既有行為、參數調整 |
| `add` | 新增檔案 / 資源（非功能） |
| `remove` | 刪除程式碼或檔案 |
| `refactor` | 重構（行為不變） |
| `perf` | 效能優化 |
| `style` | 格式化、排版 |
| `doc` | 文件、註解 |
| `test` | 測試相關 |
| `chore` | CI/CD、工具、依賴管理 |
| `security` | 安全性修補 |
| `breaking` | 破壞性變更 |

### Tag 優先序

```
BREAKING > FEAT > FIX > SECURITY > UPDATE > REFACTOR > PERF > others
```

### Breaking Signals（命中 → `breaking`）

| 類別 | 具體訊號 |
|------|----------|
| 符號刪除 | 刪除任何 exported / public function、class、method、type、constant、enum value |
| 簽章變更 | 參數型別、順序、新增必填參數、回傳型別改變 |
| HTTP API | endpoint 刪除、URL 路徑改、response schema 欄位移除或型別變、status code 變更 |
| 設定 | 刪除既有鍵、新增必填鍵、既有鍵型別 / 格式變更 |
| Migration | `DROP COLUMN`、必填欄位加在有資料的表、型別縮窄 |
| CLI | 旗標刪除、新增必填位置參數、既有旗標語意變更 |
| Import | package / module 結構變更導致既有引用失效 |

### Security Signals（命中 → `security`，除非同時命中 Breaking）

| 類別 | 具體訊號 |
|------|----------|
| 注入漏洞 | SQL / XSS / Command / LDAP / Template injection |
| 授權 | missing auth check、privilege escalation、JWT 驗證缺失 |
| 敏感資料 | log / response / error message 移除密鑰、token、PII |
| 硬編碼 | 移除 token、預設密碼、API key |
| Web 安全 | CSRF / CORS / CSP / HSTS 修補 |
| CVE | 相依套件 CVE 修補（從 `chore: upgrade` 升級為 `security`） |

### 跨主題偵測條件

任一命中即視為跨主題：

- diff 同時觸及 2+ primary tag 類別（`feat` / `fix` / `refactor` / `breaking` / `security`）
- 變更檔案橫跨 2+ 語意無關模組（以一級目錄或套件邊界判斷）
- 單次 commit 包含 3+ 彼此無關主題

格式化、純註解補充、依賴升級等輔助性改動可併入主 commit，不視為跨主題。

### Output Format

```
tag: English one-line description
tag: 繁體中文一句話描述
```

### 格式規則

| 規則 | 說明 |
|------|------|
| 單一 Tag | 選擇最能代表核心意圖的 Tag |
| 英文 subject | imperative mood，≤ 72 字元（Add / Fix / Refactor / Remove / Update） |
| 繁體中文 body | ≤ 50 字，動詞開頭（新增、修正、重構、移除、優化） |
| 合併相關變更 | 多個小改動歸納為單一描述 |
| 單一主題優先 | 一次 commit 表達一個意圖；跨主題先警示拆分，再給概括 message |
