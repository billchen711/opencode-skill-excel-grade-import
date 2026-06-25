# OpenCode Skill: Excel Grade Import

A reusable skill file for OpenCode / MimoCode that generates a school grade Excel import system.

## What This Skill Does

When loaded into a coding agent, this skill generates:
- ASP.NET Core Web API backend
- HTML/JavaScript frontend for uploading Excel files
- SQLite database with grade data
- Row-level validation with Chinese error messages
- Sample Excel files for testing

## How to Use

### Option 1: Clone This Repo

```bash
git clone https://github.com/YOUR_USERNAME/opencode-skill-excel-grade-import.git
cd opencode-skill-excel-grade-import
```

Then open OpenCode and say:

```
Please use the excel-grade-import skill. Read the skill content first, then produce an implementation plan only.
```

### Option 2: Copy to Your Project

Copy the `.opencode/` folder into your project root:

```bash
cp -r .opencode /path/to/your/project/
```

OpenCode will automatically discover the skill.

## Requirements

- .NET 10 SDK (or .NET 8 LTS)
- Node.js + npm
- OpenCode or MimoCode

## Skill File

- `.opencode/skills/excel-grade-import/SKILL.md`

## License

MIT
