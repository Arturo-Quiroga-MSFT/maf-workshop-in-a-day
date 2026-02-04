# Translation Guide

This repository uses a dual-branch strategy to maintain both Korean (original) and English (translation) versions.

## Branch Strategy

- **`main` branch**: Original Korean content, syncs with upstream
- **`english` branch**: English translation of all materials

## Workflow

### 1. Keeping Up-to-Date with Upstream

To sync Korean updates from the original repository:

```bash
# Switch to main branch
git checkout main

# Pull latest changes from upstream
git pull upstream main

# Push updates to your fork
git push origin main

# Switch to English branch
git checkout english

# Merge main into english (you'll have merge conflicts - that's expected)
git merge main

# Resolve conflicts by keeping your English translations
# Then commit the merge
git add .
git commit -m "Merge updates from Korean original"
git push origin english
```

### 2. Translating Documentation Files

Priority translation list:

1. ✅ **README.md** (completed)
2. ✅ **CONTRIBUTING.md** (completed)
3. ✅ **CHANGELOG.md** (completed)
4. ✅ **docs/00-setup.md** (completed)
5. ⏳ **docs/01-single-agent-with-maf.md** (pending - ~380 lines)
6. ⏳ **docs/02-ui-integration-with-maf.md** (pending)
7. ⏳ **docs/03-multi-agent-with-maf.md** (pending)
8. ⏳ **docs/04-aspire-orchestration.md** (pending)
9. ⏳ **docs/05-mcp-server-development.md** (pending)
10. ⏳ **docs/06-mcp-server-integration-with-maf.md** (pending)
11. ⏳ **docs/07-mcp-server-integration-with-copilot-studio.md** (pending)

### 3. Technical Translation Workflow (RECOMMENDED)

**Best Practice: Use `multi_replace_string_in_file` for efficiency**

Based on experience translating docs/00-setup.md, here's the proven workflow:

#### Step 1: Read the file
```bash
# Check file structure and line count
wc -l docs/filename.md
```

#### Step 2: Translate in sections
- Break the file into logical sections (headings, paragraphs)
- Translate 3-5 sections at a time using `multi_replace_string_in_file`
- This allows for batch edits while maintaining accuracy

#### Step 3: Match text exactly
- Copy the **exact Korean text** including:
  - Unicode characters
  - Whitespace
  - Line breaks
  - Punctuation
- Include 3-5 lines of context before and after the target text

#### Step 4: What to translate
✅ **TRANSLATE:**
- Headings (`## 사전 준비 사항` → `## Prerequisites`)
- Paragraphs and explanatory text
- Inline notes and warnings
- List items (non-code)
- Comments in text (not in code blocks)

❌ **DO NOT TRANSLATE:**
- Code blocks (```bash, ```powershell, etc.)
- Command-line commands
- URLs and links
- File paths
- Package names
- Variable names
- API endpoints
- Error messages (keep as-is for debugging)

#### Step 5: Commit after each file
```bash
git add docs/filename.md
git commit -m "Translate docs/filename.md to English

- Translated all Korean text to English
- Kept all code blocks, commands, and URLs unchanged
- Maintained original structure and formatting"
git push origin english
```

### 4. Example Translation Pattern

Here's a working example from docs/00-setup.md:

```javascript
// Original Korean
"oldString": "## 사전 준비 사항\n\n- 크로미움 계열 웹브라우저..."

// English Translation  
"newString": "## Prerequisites\n\n- A Chromium-based web browser..."
```

**Key Points:**
- Use `\n` for line breaks in multi-line replacements
- Match Korean Unicode exactly (use copy-paste)
- Keep markdown formatting identical
- Preserve emojis and special characters (👉, ✅, etc.)

### 5. Using AI for Translation

You can use GitHub Copilot or other AI tools to help translate:

```bash
# Example prompt for AI:
"Translate the following Korean markdown documentation to English, 
keeping all code blocks, commands, URLs, and file paths unchanged. 
Only translate the Korean text:"
```

### 6. Translation Quality Checklist

Before committing a translated file, verify:

- [ ] All Korean text is translated to English
- [ ] All code blocks remain unchanged
- [ ] All commands remain unchanged
- [ ] All URLs and links work correctly
- [ ] Markdown formatting is preserved
- [ ] Section navigation links are updated (if they reference Korean text)
- [ ] File builds/renders correctly as markdown
- [ ] Emojis and special characters are preserved
- [ ] Technical terms are translated consistently

### 7. Common Translation Patterns

| Korean | English |
|--------|---------|
| 사전 준비 사항 | Prerequisites |
| 세션 | Session |
| 워크샵 | Workshop |
| 개발 환경 | Development Environment |
| 설정 | Setup / Configuration |
| 실행 | Run / Execute |
| 확인 | Verify / Check |
| 아래 명령어를 실행 | Run the following command |
| 축하합니다 | Congratulations |

### 8. Syncing Strategy

- **Weekly or monthly**: Merge latest changes from upstream Korean version
- **After each merge**: Review and update English translations where needed
- **Conflict resolution**: Always keep English translations, update only if Korean content changed significantly

## Quick Commands Reference

```bash
# Clone your fork
git clone https://github.com/Arturo-Quiroga-MSFT/maf-workshop-in-a-day.git
cd maf-workshop-in-a-day

# Set up remotes
git remote add upstream https://github.com/Azure-Samples/maf-workshop-in-a-day-ko.git

# View remotes
git remote -v

# Switch between branches
git checkout main      # Korean original
git checkout english   # English translation

# Sync with upstream (Korean original)
git checkout main
git pull upstream main
git checkout english
git merge main
```

## Repository Structure

```
maf-workshop-in-a-day/
├── main branch (Korean - syncs with upstream)
│   └── All original Korean content
└── english branch (English - your translations)
    └── All translated English content
```

## Contributing Back to Original

If you find bugs or improvements:
1. Make fixes in the `main` branch
2. Create a PR to the upstream repository
3. Once merged upstream, pull changes back to your fork
4. Merge into `english` branch and update translations if needed

## Notes

- **Original repository**: https://github.com/Azure-Samples/maf-workshop-in-a-day-ko
- **Your fork**: https://github.com/Arturo-Quiroga-MSFT/maf-workshop-in-a-day
- **Your English version**: https://github.com/Arturo-Quiroga-MSFT/maf-workshop-in-a-day/tree/english
