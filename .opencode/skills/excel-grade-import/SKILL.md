---
name: excel-grade-import
description: Build a .NET backend and JavaScript frontend demo that uploads an Excel grade sheet, validates rows, imports them into a database, and shows import results for school staff.
compatibility: opencode
metadata:
  audience: school-it-engineers
  backend: dotnet
  frontend: javascript
  database: sqlite
---

# Excel Grade Import Skill

Use this skill when the user asks for a sample project where school staff upload an Excel grade sheet and the system imports the data into a backend database.

The goal is not to build a production student information system. The goal is to generate a small, runnable, reviewable prototype that demonstrates the development workflow and gives school engineers a concrete pattern they can adapt.

## Environment Setup Requirements

Before generating code, verify and install the following tools if missing.

### macOS / Linux (Bash)

```bash
# Check and install Node.js
if ! command -v node &> /dev/null; then
    echo "Node.js not found. Installing..."
    if [[ "$OSTYPE" == "darwin"* ]]; then
        brew install node
    elif [[ -f /etc/debian_version ]]; then
        curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
        sudo apt install -y nodejs
    fi
fi
node --version
npm --version

# Check and install .NET SDK
if ! command -v dotnet &> /dev/null; then
    echo ".NET SDK not found. Installing..."
    if [[ "$OSTYPE" == "darwin"* ]]; then
        brew install dotnet-sdk
    elif [[ -f /etc/debian_version ]]; then
        sudo apt install -y dotnet-sdk-10.0
    fi
fi
dotnet --version

echo "=== Environment Check ==="
command -v code && echo "VS Code: OK" || echo "VS Code: MISSING (install from https://code.visualstudio.com/)"
command -v node && echo "Node.js: OK" || echo "Node.js: MISSING"
command -v npm && echo "npm: OK" || echo "npm: MISSING"
command -v dotnet && echo ".NET SDK: OK" || echo ".NET SDK: MISSING"
```

### Windows (PowerShell)

```powershell
# Check and install Node.js
if (-not (Get-Command node -ErrorAction SilentlyContinue)) {
    Write-Host "Node.js not found. Installing..."
    winget install OpenJS.NodeJS.LTS
    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
}
node --version
npm --version

# Check and install .NET SDK
if (-not (Get-Command dotnet -ErrorAction SilentlyContinue)) {
    Write-Host ".NET SDK not found. Installing..."
    winget install Microsoft.DotNet.SDK.10
    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
}
dotnet --version

# Check PowerShell execution policy
$policy = Get-ExecutionPolicy
if ($policy -eq "Restricted") {
    Write-Host "PowerShell execution policy is Restricted. Fixing..."
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
}

Write-Host "=== Environment Check ==="
if (Get-Command code -ErrorAction SilentlyContinue) { Write-Host "VS Code: OK" } else { Write-Host "VS Code: MISSING (install from https://code.visualstudio.com/)" }
if (Get-Command node -ErrorAction SilentlyContinue) { Write-Host "Node.js: OK" } else { Write-Host "Node.js: MISSING" }
if (Get-Command npm -ErrorAction SilentlyContinue) { Write-Host "npm: OK" } else { Write-Host "npm: MISSING" }
if (Get-Command dotnet -ErrorAction SilentlyContinue) { Write-Host ".NET SDK: OK" } else { Write-Host ".NET SDK: MISSING" }
```

If any tool is still missing after installation, refer to `docs/environment-setup.zh-TW.md` for manual installation instructions.

### Important: Running the Backend Server

The backend server (`dotnet run`) is a **blocking command** — it runs continuously and does not exit. When the agent needs to start the backend:

**macOS / Linux:**
```bash
nohup dotnet run --urls "http://localhost:5000" --project src/GradeImport.Api > /tmp/backend.log 2>&1 &
sleep 4
curl -s http://localhost:5000/api/health
```

**Windows (PowerShell):**
```powershell
Start-Process -WindowStyle Hidden -FilePath "dotnet" -ArgumentList "run --urls http://localhost:5000 --project src\GradeImport.Api"
Start-Sleep -Seconds 4
Invoke-RestMethod -Uri "http://localhost:5000/api/health"
```

**Verification:** After starting the backend, always verify it is running:
```bash
# macOS/Linux
curl -s http://localhost:5000/api/health

# Windows
Invoke-RestMethod -Uri "http://localhost:5000/api/health"
```

If the health check returns `{"status":"ok"}`, the backend is ready.

## Default Stack

Unless the user gives a different stack, use:

- Backend: ASP.NET Core Web API with C#
- Runtime: .NET 10 (or .NET 8 LTS if preferred)
- Database: SQLite for local demo (see Demo Database Scope below)
- Frontend: plain HTML, CSS, and JavaScript, or a small Vite app if the repository already uses one
- Excel parsing: ExcelDataReader library, supports both `.xls` and `.xlsx` formats
  - **IMPORTANT for Taiwanese .xls files:** Set fallback encoding to Big5/CP950 to avoid garbled Chinese characters:
    ```csharp
    var config = new ExcelReaderConfiguration
    {
        FallbackEncoding = System.Text.Encoding.GetEncoding(950)
    };
    using var reader = ExcelReaderFactory.CreateReader(stream, config);
    ```
- API style: REST endpoints returning JSON
- Test style: focused unit tests for validation logic and at least one integration-style import test when practical

## Demo Database Scope

This demo uses **SQLite** as a local-only database. It is not connected to any production system.

- The SQLite database is for demonstrating the workflow only.
- For production, engineers should write additional API endpoints to connect to the school's existing database (SQL Server, MariaDB, MySQL, etc.).
- Do not assume Docker is available. Use SQLite as the default for the demo.

## User Scenario

The target user is a normal school staff member or teacher, not an engineer.

They should be able to:

1. Open a web page.
2. Select or drag an Excel file.
3. Click upload.
4. See whether import succeeded.
5. See which rows failed and why.
6. See imported grade records in a simple result table.

## Required Excel Template

The demo Excel file should follow a school grade sheet style rather than a generic one-row-header spreadsheet. Use fake data only. The expected structure is:

### File Structure

The Excel file has a specific layout with metadata rows at the top:

| Row | Content | Description |
|-----|---------|-------------|
| 0 | 學期資訊 | e.g., "114學年度第二學期 成績評量表" |
| 1 | 班級資訊 | e.g., "上課班級：大學部土木系一年級甲班" |
| 2 | 教師資訊 | e.g., "授課教師：李龍盛 (A0360)" |
| 3 | 課程資訊 | e.g., "課程名稱：(11423460317) 基礎程式設計" |
| 4 | 評量項目名稱 | Score category names |
| 5 | 百分比權重 | Score weights |
| 6 | 欄位標題 | Column headers |
| 7+ | 學生資料 | Student data rows |

### Column Headers (Row 6)

| Column | Header | Description |
|--------|--------|-------------|
| 0 | 編號 | Row index |
| 1 | 學號 | Student ID (required) |
| 2 | 中文姓名 | Student Chinese name (required) |
| 3 | 學籍狀況 | Enrollment status |
| 4 | 分數 | Score - 課堂參與討論 |
| 5 | 分數 | Score - 小考 |
| 6 | 分數 | Score - 書面報告 |
| 7 | 分數 | Score - 口頭報告 |
| 8 | 分數 | Score - 操作實作 |
| 9 | 分數 | Score - 實習 |
| 10 | 分數 | Score - 作業習題演練 |
| 11 | 分數 | Score - 檔案記錄 |
| 12 | 分數 | Score - 口試 |
| 13 | 分數 | Score - 其他1 |
| 14 | 分數 | Score - 其他2 |
| 15 | 分數 | Score - 其他3 |
| 16 | 分數 | Score - 其他4 |
| 17 | 分數 | Score - 其他5 |
| 18 | 分數 | Score - 其他6 |
| 19 | 分數 | Score - 期中考 |
| 20 | 分數 | Score - 期末考 |
| 21 | 學期成績 | Semester total grade |
| 22 | 不及格 | Fail flag |

### Score Category Names (Row 4)

The actual score categories are defined in Row 4:
- Col 4: 課堂參與討論
- Col 5: 小考
- Col 6: 書面報告
- Col 7: 口頭報告
- Col 8: 操作實作
- Col 9: 實習
- Col 10: 作業習題演練
- Col 11: 檔案記錄
- Col 12: 口試
- Col 13: 其他1
- Col 14: 其他2
- Col 15: 其他3
- Col 16: 其他4
- Col 17: 其他5
- Col 18: 其他6
- Col 19: 期中考
- Col 20: 期末考

### Score Weights (Row 5)

The weights are defined in Row 5:
- Col 4: 10% (課堂參與討論)
- Col 10: 30% (作業習題演練)
- Col 13: 20% (其他1)
- Col 19: 20% (期中考)
- Col 20: 20% (期末考)

### Validation Rules

When parsing the Excel file:

1. **Skip metadata rows (0-6)**: These are header information, not student data.
2. **Required fields for each student row**:
   - 學號 (StudentId) - required
   - 中文姓名 (StudentName) - required
   - 學期成績 (Semester Grade) - required, must be numeric
3. **Optional fields**:
   - 學籍狀況 (Enrollment Status)
   - Individual score columns (4-20)
   - 不及格 (Fail flag)
4. **Error messages should be in Chinese**:
   - Good: `第 7 列缺少學號，請補上後重新上傳。`
   - Avoid: `ValidationError: StudentId required`

### Metadata Extraction

The system should extract metadata from the header rows:
- **Semester**: From Row 0 (e.g., "114學年度第二學期")
- **Class**: From Row 1 (e.g., "大學部土木系一年級甲班")
- **Teacher**: From Row 2 (e.g., "李龍盛")
- **Course**: From Row 3 (e.g., "基礎程式設計")
- **Course Code**: From Row 3 (e.g., "11423460317")

## Database Design

Implement a simple schema with these entities or equivalent tables:

- `ImportBatches`
  - `Id`
  - `FileName`
  - `Semester`
  - `ClassName`
  - `TeacherName`
  - `CourseName`
  - `CourseCode`
  - `UploadedAt`
  - `TotalRows`
  - `SuccessRows`
  - `FailedRows`
- `Students`
  - `Id`
  - `StudentId`
  - `Name`
  - `EnrollmentStatus`
  - `CreatedAt`
- `Grades`
  - `Id`
  - `StudentId`
  - `ImportBatchId`
  - `Score04` through `Score20` (17 score columns)
  - `SemesterGrade`
  - `FailFlag`
  - `CreatedAt`

Use indexes or constraints to prevent duplicate grade rows for the same student and import batch.

## Backend Requirements

**Port:** Use port 5000 for local development. Update `launchSettings.json` if needed:
```json
"profiles": {
    "GradeImport.Api": {
        "commandName": "Project",
        "applicationUrl": "http://localhost:5000"
    }
}
```

**Swagger / OpenAPI:**
- For .NET 8: Use `AddSwaggerGen()` / `UseSwagger()` / `UseSwaggerUI()`
- For .NET 9+: Use `AddOpenApi()` / `MapOpenApi()` (Swashbuckle is no longer included by default)

Implement these endpoints unless the user asks otherwise:

- `GET /api/health`
  - Returns a simple health status.
- `POST /api/grade-imports`
  - Accepts multipart form upload.
  - Accepts both `.xls` and `.xlsx` files.
  - Validates file size. Use a conservative limit such as 5 MB for the demo unless specified.
  - Parses rows (skip first 7 rows as headers).
  - Validates each student row.
  - Saves valid rows in a transaction.
  - Returns import summary and row-level errors.
- `GET /api/grade-imports`
  - Returns all import batches.
- `GET /api/grade-imports/{id}`
  - Returns a prior import summary and imported rows.
- `GET /api/grades`
  - Returns imported grades for demo verification.

The `POST /api/grade-imports` response should include:

- `batchId`
- `totalRows`
- `successRows`
- `failedRows`
- `errors`, each containing row number, field name, and user-readable message

Example error message style:

- Good: `第 7 列缺少學號，請補上後重新上傳。`
- Avoid: `ValidationError: StudentId required`

## Frontend Requirements

Create a simple user interface with:

- Page title: `成績 Excel 匯入`
- Three tabs: Upload, Import History, Imported Grades
- **IMPORTANT: Frontend must be served via HTTP** (not file://). Browser security blocks API calls from file:// URLs. Options:
  - Serve static files from the backend (recommended)
  - Or use a simple dev server: `npx serve frontend`

### Tab 1: Upload
- File upload control (supports drag and drop)
- Upload button
- Loading state while uploading
- Import summary after upload
- Error table with row number, field, message

### Tab 2: Import History (匯入紀錄)
- Call `GET /api/grade-imports` to get all import batches
- Display a table with columns: Import Time, File Name, Semester, Course, Total Rows, Success, Failed
- Each row should be clickable to show batch detail
- Clicking a row calls `GET /api/grade-imports/{id}` and shows the grades from that batch

### Tab 3: Imported Grades (已匯入成績)
- Call `GET /api/grades` to get all imported grades
- Display a table with columns: 學號, 中文姓名, 學期成績, 不及格
- Show "尚無匯入成績" if empty

The UI should be plain and work-focused. Avoid marketing copy. Do not make the user read technical instructions before uploading.

## Security and Privacy Rules

Because grade data can include personal information:

- Use fake data in the sample project.
- Do not send uploaded grade files to external AI APIs.
- Do not log raw student grade rows in application logs.
- Do not store uploaded files permanently unless the user explicitly asks for file archiving.
- Validate file extension and content type, but do not rely only on content type.
- Reject macro-enabled files such as `.xlsm` in the demo.
- Keep database connection strings in configuration or environment variables, not hardcoded source code.

## Output Expectations

When asked to build the project, produce:

- A runnable backend project
- A runnable frontend
- Database schema or EF Core migrations
- A sample Excel file or CSV-to-XLSX generation script
- A README with setup, run, and test commands
- A short explanation of project structure
- A short list of what is demo-only and what would need hardening for production

## Work Plan for the Agent

Before editing code, first produce a plan that includes:

1. Project structure.
2. Backend endpoints.
3. Database schema.
4. Excel validation rules.
5. Frontend screens.
6. Test and demo commands.

After the plan is approved, implement in small steps:

1. Scaffold backend.
2. Add database models and migrations.
3. Add Excel parsing and validation service.
4. Add import API.
5. Add frontend upload page.
6. Add sample data.
7. Run build and tests.
8. Fix compile or runtime errors.
9. Provide final run instructions.

## Verification Checklist

Do not call the work complete until these checks pass or are explicitly documented as blocked:

- Backend starts successfully.
- Frontend opens successfully.
- Sample Excel upload succeeds.
- Invalid sample Excel produces row-level errors.
- Imported rows can be viewed from the UI or `GET /api/grades`.
- Build command succeeds.
- Tests pass, if tests were created.

## Suggested First Prompt

Use this prompt in OpenCode after this skill is available:

```text
Please use the excel-grade-import skill. Read the skill content first, then produce an implementation plan only. Do not modify files yet.

I need a school staff Excel grade import demo:
- Backend: C# ASP.NET Core Web API.
- Frontend: simple JavaScript page with 3 tabs (Upload, Import History, Imported Grades).
- Database: SQLite.
- The Excel file has a specific format with 7 header rows (semester, class, teacher, course info, score categories, weights, column headers).
- Student data starts from Row 7.
- Required fields: 學號 (StudentId), 中文姓名 (StudentName), 學期成績 (Semester Grade).
- 17 score columns (Col 4-20) for different assessment types.
- Use ExcelDataReader to support both .xls and .xlsx formats.
- Use fake data only, no real student records.

List the files, API endpoints, tables, validation rules, run and test commands. Wait for my approval before implementation.
```
