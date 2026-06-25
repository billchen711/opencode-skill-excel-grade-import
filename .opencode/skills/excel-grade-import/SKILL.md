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
    # Refresh PATH
    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
}
node --version
npm --version

# Check and install .NET SDK
if (-not (Get-Command dotnet -ErrorAction SilentlyContinue)) {
    Write-Host ".NET SDK not found. Installing..."
    winget install Microsoft.DotNet.SDK.10
    # Refresh PATH
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
Start-Process -FilePath "dotnet" -ArgumentList "run --urls http://localhost:5000 --project src\GradeImport.Api" -WindowStyle Normal
Start-Sleep -Seconds 4
Invoke-RestMethod -Uri "http://localhost:5000/api/health"
```

**Windows alternative — run in a new PowerShell window:**
```powershell
Start-Process powershell -ArgumentList "-NoExit", "-Command", "cd '$PWD\src\GradeImport.Api'; dotnet run --urls http://localhost:5000"
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
- Excel parsing: NPOI library, supports both `.xls` and `.xlsx` formats
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

Create or document a sample Excel file with these columns:

- `StudentId` required, text, unique student identifier
- `StudentName` required, text
- `CourseCode` required, text
- `CourseName` required, text
- `Semester` required, text such as `113-2`
- `Score` required, number from 0 to 100

Supported formats: `.xls` (Excel 97-2003) and `.xlsx` (modern Excel)

Reject or report rows with:

- Missing required values
- Score outside 0 to 100
- Non-numeric score
- Duplicate grade row for the same `StudentId`, `CourseCode`, and `Semester`
- Unexpected empty rows mixed into the file

Do not use real student data in the generated sample. Use fake sample data only.

## Database Design

Implement a simple schema with these entities or equivalent tables:

- `Students`
  - `Id`
  - `StudentId`
  - `Name`
  - `CreatedAt`
- `Courses`
  - `Id`
  - `CourseCode`
  - `CourseName`
  - `CreatedAt`
- `GradeImportBatches`
  - `Id`
  - `FileName`
  - `UploadedAt`
  - `TotalRows`
  - `SuccessRows`
  - `FailedRows`
- `Grades`
  - `Id`
  - `StudentId`
  - `CourseId`
  - `Semester`
  - `Score`
  - `ImportBatchId`
  - `CreatedAt`

Use indexes or constraints to prevent duplicate grade rows for the same student, course, and semester.

## Backend Requirements

Implement these endpoints unless the user asks otherwise:

- `GET /api/health`
  - Returns a simple health status.
- `POST /api/grade-imports`
  - Accepts multipart form upload.
  - Accepts both `.xls` (Excel 97-2003) and `.xlsx` files.
  - Validates file size. Use a conservative limit such as 5 MB for the demo unless specified.
  - Parses rows.
  - Validates each row.
  - Saves valid rows in a transaction.
  - Returns import summary and row-level errors.
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

- Good: `第 5 列缺少學號，請補上後重新上傳。`
- Avoid: `ValidationError: StudentId required`

## Frontend Requirements

Create a simple user interface with:

- Page title: `成績 Excel 匯入`
- File upload control
- Upload button
- Loading state while uploading
- Import summary after upload
- Error table with row number, field, message
- Imported grade preview table

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
- Frontend: simple JavaScript page.
- Database: SQLite.
- Upload .xls or .xlsx grade files, validate rows, import to database, show success and error results.
- Use fake data only, no real student records.

List the files, API endpoints, tables, validation rules, run and test commands. Wait for my approval before implementation.
```
