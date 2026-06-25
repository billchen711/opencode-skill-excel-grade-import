# Agent Project Brief: School Excel Grade Import Demo

This repository is for a teacher-facing coding-agent workflow demo.

The main goal is to prepare reusable Markdown instructions/specifications that a coding agent such as OpenCode, MimoCode, or a similar tool can read, then use to generate a small sample system.

This project intentionally excludes local LLM procurement/deployment research unless the user explicitly asks for it again.

## Current Assignment

The teacher wants a Markdown-based specification or skill file that school engineers can use with a coding agent.

The demo should show the workflow:

1. Open VS Code.
2. Open OpenCode, MimoCode, or a similar coding-agent tool beside the editor.
3. Provide a reusable Markdown skill/spec.
4. Ask the agent to read the spec and generate an implementation plan first.
5. Review the plan.
6. Let the agent generate a sample frontend/backend system.
7. Run and test the generated system.
8. Capture screenshots or perform a live demo.

The point is not only to build one finished upload system. The point is to show school engineers how to write enough specification context for a coding agent to generate a usable starting point.

## People and Audience

- Primary audience for the Markdown/spec workflow: school IT engineers.
- End user of the demo system: ordinary school staff or teachers, not engineers.
- Student's role: prepare the spec, demo process, and sample workflow. Do not claim production architecture ownership.
- Engineer's role: review, adapt, harden, and maintain any generated code.

## Demo System

The sample system should demonstrate:

- A web page where school staff can upload an Excel grade sheet.
- A backend API that receives the file.
- Excel row parsing and validation.
- Import of valid data into a database.
- A result page or API response showing success count, failure count, and row-level errors.
- A simple way to view imported grade records.

This is a prototype/demo, not a production student information system.

## Preferred Stack

Use this stack unless the user or teacher changes it:

- Backend: C# ASP.NET Core Web API.
- Runtime: .NET 10 (current installed version); .NET 8 LTS is also acceptable.
- Frontend: simple JavaScript, HTML, and CSS.
- Database: SQLite for local demo. Production environments should connect to SQL Server, MariaDB, MySQL, or the school's existing database via API.
- Excel parsing: use a maintained .NET package for `.xlsx` files.
- API style: REST endpoints returning JSON.

### Demo Database Scope

This demo uses **SQLite** as a local-only database. It is not connected to any production system. The purpose is to demonstrate the AI coding agent workflow, not to build a production-ready student information system.

For production deployment, engineers should:
- Write additional API endpoints to connect to the school's existing database.
- Replace SQLite with SQL Server, MariaDB, MySQL, or the school's preferred database.
- Add authentication, authorization, audit logging, and backup strategies.

## Environment Setup Requirements

Before running this demo, ensure the following are installed:

### Required Software

| Software | Purpose | Install Guide |
|----------|---------|---------------|
| VS Code | Code editor | https://code.visualstudio.com/ |
| Node.js + npm | Required for OpenCode/MimoCode | https://nodejs.org/ |
| .NET SDK | Backend runtime | https://dotnet.microsoft.com/download |
| OpenCode or MimoCode | AI coding agent | See agent documentation |

### Platform Notes

#### macOS
```bash
# Install .NET SDK via Homebrew
brew install dotnet-sdk
```

#### Windows
```powershell
# Install .NET SDK
winget install Microsoft.DotNet.SDK.10

# If PowerShell execution policy blocks OpenCode:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### Linux
```bash
# Install .NET SDK
sudo apt install dotnet-sdk-10.0
```

For detailed installation steps, see `docs/environment-setup.zh-TW.md`.

## Important Existing Files

Agent-facing skill:

- `.opencode/skills/excel-grade-import/SKILL.md`

Human-readable guide:

- `README.md`
- `docs/environment-setup.zh-TW.md`

Do not assume other reference documents exist unless they are present in the repository.

## Required Excel Template

Use fake data only. Do not use real student records.

The demo should follow a school grade sheet style rather than a generic one-row-header spreadsheet.

Expected layout:

- Rows 0-3: metadata such as semester, class, teacher, course name, and course code.
- Row 4: score category names.
- Row 5: score weights.
- Row 6: column headers.
- Row 7 and after: student grade rows.

Required student fields:

- `學號` / `StudentId`: required text.
- `中文姓名` / `StudentName`: required text.
- `學期成績` / `SemesterGrade`: required number from 0 to 100.

Optional student fields:

- `學籍狀況` / `EnrollmentStatus`.
- Individual score columns, such as homework, midterm, final, and other assessment categories.
- `不及格` / `FailFlag`.

Validation rules:

- Missing required values should fail.
- Score outside 0 to 100 should fail.
- Non-numeric score should fail.
- Duplicate grade row for the same student and import batch should fail or be handled explicitly.
- Unexpected empty rows should be ignored or reported clearly.

User-facing error messages should be understandable to school staff.

Good:

```text
第 5 列缺少學號，請補上後重新上傳。
```

Avoid:

```text
ValidationError: StudentId required
```

## Suggested Database Schema

Use these entities or equivalent tables:

### Students

- `Id`
- `StudentId`
- `Name`
- `EnrollmentStatus`
- `CreatedAt`

### GradeImportBatches

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

### Grades

- `Id`
- `StudentId`
- `ImportBatchId`
- `SemesterGrade`
- Individual score columns or a normalized score-detail table
- `FailFlag`
- `CreatedAt`

Add indexes or constraints to prevent duplicate grade rows for the same student and import batch.

## Suggested API Endpoints

### `GET /api/health`

Return simple backend health status.

### `POST /api/grade-imports`

Accept multipart upload and import a grade sheet.

Behavior:

- Accept `.xlsx` and `.xls` in the demo unless the implementation explicitly supports only `.xlsx`.
- Reject `.xlsm` and macro-enabled files.
- Validate file size, such as 5 MB for demo.
- Parse metadata rows, header rows, and student rows.
- Validate each row.
- Save valid rows in a transaction.
- Return summary and row-level errors.

Expected JSON response fields:

- `batchId`
- `totalRows`
- `successRows`
- `failedRows`
- `errors`

Each error should include:

- row number
- field name
- user-readable message

### `GET /api/grade-imports/{id}`

Return an import batch summary and imported rows.

### `GET /api/grade-imports`

Return import history for demo verification.

### `GET /api/grades`

Return imported grades for demo verification.

## Frontend Requirements

Build a simple, work-focused interface.

Required UI elements:

- Page title: `成績 Excel 匯入`
- File upload control.
- Upload button.
- Loading state during upload.
- Import summary after upload.
- Error table with row number, field, and message.
- Imported grade preview table.

The interface is for general school staff, so avoid engineering jargon.

## Privacy and Security Rules

Grade data can include personal information.

For this demo:

- Use fake data only.
- Do not send uploaded grade files to external AI APIs.
- Do not log raw student grade rows.
- Do not permanently store uploaded files unless explicitly requested.
- Keep database connection strings in configuration or environment variables.
- Validate file extension and content type, but do not rely only on content type.
- Reject macro-enabled files in the demo.

For production, engineers must add:

- authentication
- authorization
- audit logs
- retention policy
- database backup strategy
- stricter upload scanning
- deployment-specific security review

## Agent Workflow Rules

When a coding agent is asked to work on this project:

1. Read this `AGENTS.md`.
2. Read `.opencode/skills/excel-grade-import/SKILL.md`.
3. Produce an implementation plan first.
4. Do not modify files until the plan is approved by the user.
5. Keep the first version small and runnable.
6. Prefer demo clarity over production complexity.
7. Use fake data.
8. Run build/test commands after implementation when the required tools are installed.
9. Document any blocked checks, such as missing .NET SDK or Docker.

Do not:

- Build a full production student information system.
- Add local LLM procurement discussion.
- Use real student data.
- Hide validation or import failures.
- Leave the user without run commands.

## Required Deliverables for the Demo

The final generated demo should include:

- Backend project.
- Frontend upload page.
- Database schema or migrations.
- Sample fake Excel file or a script that creates it.
- README with setup, run, and test commands.
- Clear demo steps.
- Screenshots or a screenshot checklist.
- Notes explaining what is demo-only and what production would still need.

## Live Demo Script

Suggested live sequence:

1. Open the folder in VS Code.
2. Show `AGENTS.md`.
3. Show `.opencode/skills/excel-grade-import/SKILL.md`.
4. Start OpenCode or MimoCode.
5. Ask the agent to read the skill/spec and produce only a plan.
6. Review the plan.
7. Approve implementation.
8. Run build/test commands if available.
9. Open the upload page.
10. Upload a valid fake Excel grade sheet.
11. Show successful import result.
12. Upload or test an invalid sheet.
13. Show row-level errors.
14. Show imported grades through UI, API, or database view.

## Screenshot Checklist

Capture these if preparing a report:

1. VS Code project folder.
2. `AGENTS.md`.
3. `.opencode/skills/excel-grade-import/SKILL.md`.
4. Coding agent running beside VS Code.
5. First prompt asking for a plan.
6. Generated plan.
7. Generated file tree.
8. Build success or documented blocker.
9. Upload page.
10. Successful import result.
11. Invalid row error table.
12. Imported grade preview.

## Suggested First Prompt for a Coding Agent

```text
Read AGENTS.md and .opencode/skills/excel-grade-import/SKILL.md first.

I need a demo project for school IT engineers. The demo should show how a coding agent can generate a small system where ordinary school staff upload an Excel grade sheet, the backend validates rows, imports valid data into a database, and returns readable import results.

Use C# ASP.NET Core Web API for the backend, prefer .NET 8, use simple JavaScript for the frontend, and target MariaDB/MySQL with SQLite fallback if needed. Use fake data only.

First, produce an implementation plan only. Do not modify files yet. The plan must include project structure, endpoints, database schema, Excel validation rules, frontend screens, run commands, test commands, and demo steps.
```

## Future Tasks for the Student

1. Confirm with the teacher whether the final demo should target .NET 8 or .NET 10.
2. Confirm whether the school prefers MariaDB, MySQL, SQL Server, or another database.
3. Decide whether the demo should actually run locally or only show the generated plan/code workflow.
4. Install or prepare .NET SDK before live backend execution.
5. Install or prepare Docker/MariaDB if database demo must run locally.
6. Generate or prepare fake Excel sample files.
7. Use OpenCode first to produce a plan from the skill/spec.
8. Take screenshots step by step.
9. Test both valid and invalid Excel import.
10. Prepare a short explanation for the teacher: the deliverable is a reusable spec workflow plus a prototype demo, not a production school system.
