# Translation Template - Quick Reference

Use this template when translating each documentation file.

## Pre-Translation Checklist

- [ ] On `english` branch: `git checkout english`
- [ ] File path: `docs/[filename].md`
- [ ] Line count: `wc -l docs/[filename].md`
- [ ] File read: Scan structure and sections

## Translation Process

### 1. Identify Sections
Break the file into logical sections:
- Title and introduction
- Each main heading (`##`)
- Sub-sections
- Conclusion/navigation

### 2. Batch Translate
Use `multi_replace_string_in_file` with 3-5 replacements per batch:

```json
{
  "filePath": "/Users/arturoquiroga/GITHUB/AQ_MAF_WORKSHOP_KO/docs/[filename].md",
  "oldString": "[exact Korean text with context]",
  "newString": "[English translation with same formatting]"
}
```

### 3. Translation Rules

✅ **ALWAYS TRANSLATE:**
- Headings: `# 제목` → `# Title`
- Paragraphs: Full sentences to English
- Lists: Bullet/numbered list items
- Notes/Warnings: `> **NOTE**: 내용` → `> **NOTE**: Content`
- Navigation: `다음 단계` → `Next step`

❌ **NEVER TRANSLATE:**
- Code blocks: ` ```bash`, ` ```powershell`, ` ```python`
- Commands: `dotnet build`, `git push`, `npm install`
- Paths: `/Users/...`, `./docs/...`, `C:\...`
- URLs: `https://...`, `http://...`
- Package names: `Microsoft.Agent.Framework`, `react`, `express`
- Environment variables: `$REPOSITORY_ROOT`, `%PATH%`
- File names: `appsettings.json`, `Program.cs`

## Common Korean → English Patterns

```
개발 환경 설정 → Development Environment Setup
사전 준비 사항 → Prerequisites
시작하기 → Getting Started
아키텍처 → Architecture
세션 목표 → Session Goals
실행 → Run/Execute
확인 → Verify/Check
설치 → Install
생성 → Create
수정 → Modify/Edit
삭제 → Delete
추가 → Add
리소스 → Resources
워크샵 → Workshop
프로젝트 → Project
애플리케이션 → Application
축하합니다 → Congratulations
다음 단계로 이동 → Proceed to the next step
아래 명령어를 실행 → Run the following command
아래와 같이 → Similar to / As follows
```

## Post-Translation Checklist

- [ ] All Korean text translated
- [ ] Code blocks unchanged
- [ ] Commands unchanged
- [ ] URLs working
- [ ] Markdown rendering correctly
- [ ] Navigation links updated (if needed)
- [ ] Commit with descriptive message
- [ ] Push to `english` branch

## Commit Message Template

```bash
git add docs/[filename].md
git commit -m "Translate docs/[filename].md to English

- Translated all Korean text to English
- Kept all code blocks, commands, and URLs unchanged
- Maintained original structure and formatting
- Preserved technical accuracy and examples"
git push origin english
```

## Verification Commands

```bash
# Check file was modified
git status docs/[filename].md

# Review changes
git diff docs/[filename].md

# Verify line changes are reasonable
git diff docs/[filename].md --stat

# View rendered markdown (in VS Code)
# Cmd+Shift+V or Ctrl+Shift+V
```

## Estimated Time per File

- Small (< 100 lines): 10-15 minutes
- Medium (100-300 lines): 20-30 minutes  
- Large (300-500 lines): 30-45 minutes
- Extra Large (> 500 lines): 45-60 minutes

## Notes

- Work section by section, don't try to translate everything at once
- Use `multi_replace_string_in_file` for efficiency (3-5 replacements per call)
- Copy-paste Korean text exactly (including Unicode)
- Preserve all whitespace and formatting
- Test markdown rendering after translation
- Commit frequently (after each file)
