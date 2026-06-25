# Windows OpenCode Skill Tutorial Screenshots

這個資料夾用來放 Windows 上示範「OpenCode 使用 SKILL.md 產生成績 Excel 匯入 demo」的截圖。

請盡量照編號和階段放圖，之後 Codex 可以依照檔名幫你判斷每張圖適合放在手冊哪裡。

## Folder Structure

```text
screenshots/windows-opencode-skill-tutorial/
├── 01-environment/       # 安裝與環境檢查
├── 02-opencode-skill/    # SKILL.md 與專案規格
├── 03-agent-plan/        # agent 讀規格、產生 plan
├── 04-agent-build/       # agent 產生檔案、專案結構
├── 05-run-backend/       # dotnet run、health API、Swagger
├── 06-frontend-upload/   # 前端頁面、選檔、上傳
├── 07-results/           # 成功匯入、重複匯入、錯誤 Excel
├── 08-troubleshooting/   # 錯誤訊息與修正流程
└── 99-extra/             # 不確定用途或備用截圖
```

## Naming Rule

建議檔名格式：

```text
<step-number>-<short-description>.png
```

範例：

```text
01-vscode-open-folder.png
02-node-version-check.png
03-opencode-start.png
04-skill-md-opened.png
05-agent-plan-generated.png
06-project-files-created.png
07-dotnet-run-health-ok.png
08-upload-page-opened.png
09-valid-excel-success.png
10-duplicate-import-error.png
11-invalid-excel-row-errors.png
```

## Required Screenshots

至少建議截這些：

### 01 Environment

- `01-vscode-open-folder.png`
  - VS Code 開啟 Windows 專案資料夾。
- `02-node-npm-version.png`
  - 顯示 `node --version` 和 `npm --version`。
- `03-dotnet-version.png`
  - 顯示 `dotnet --version`。
- `04-opencode-version-or-start.png`
  - 顯示 OpenCode 已啟動或版本資訊。

### 02 OpenCode Skill

- `01-skill-md-location.png`
  - 檔案樹中看到 `SKILL.md`。
- `02-skill-md-content.png`
  - 開啟 `SKILL.md`，看到需求規格。

### 03 Agent Plan

- `01-first-prompt.png`
  - 你要求 agent 讀 skill 並先產生 plan 的 prompt。
- `02-agent-plan-output.png`
  - agent 產生的實作計畫。

### 04 Agent Build

- `01-agent-generating-files.png`
  - agent 正在建立檔案或修改檔案。
- `02-generated-project-tree.png`
  - 產生後的前端、後端、sample data 專案結構。

### 05 Run Backend

- `01-dotnet-run.png`
  - 執行 `dotnet run --urls http://localhost:5000`。
- `02-health-api-ok.png`
  - `http://localhost:5000/api/health` 回傳 OK。
- `03-swagger-page.png`
  - 如果有 Swagger，截 API 文件頁。

### 06 Frontend Upload

- `01-upload-page.png`
  - 前端上傳頁面。
- `02-file-selected.png`
  - 選擇 Excel 檔案後的畫面。

### 07 Results

- `01-valid-import-success.png`
  - 乾淨資料庫下，正確 Excel 匯入成功，最好是成功 4、失敗 0。
- `02-duplicate-import-error.png`
  - 重複匯入錯誤。你目前那張圖就很適合放這裡。
- `03-invalid-excel-row-errors.png`
  - 錯誤 Excel 的逐列錯誤訊息。
- `04-imported-grades-table.png`
  - 已匯入成績列表。

### 08 Troubleshooting

- `01-powershell-policy-error.png`
  - 如果遇到 PowerShell execution policy 錯誤。
- `02-node-not-found.png`
  - 如果遇到 `npm` 或 `node` 不存在。
- `03-dotnet-not-found.png`
  - 如果遇到 `dotnet` 不存在。
- `04-port-5000-in-use.png`
  - 如果遇到 5000 port 被占用。

## What To Do If You Are Not Sure

如果不知道截圖要放哪裡，先丟到：

```text
screenshots/windows-opencode-skill-tutorial/99-extra/
```

之後請 Codex 幫你重新分類。

## Screenshot Quality

盡量做到：

- 畫面要包含 VS Code 左側檔案樹或 terminal，讓人知道現在在哪一步。
- 如果是瀏覽器畫面，要露出網址列。
- 如果是錯誤訊息，要截完整錯誤文字。
- 如果是 agent 輸出，要截到 prompt 和 agent 回應的關鍵段落。
- 不要截到真實學生資料、API key、token、帳號密碼。

