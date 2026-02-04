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
4. ⏳ **docs/00-setup.md** (in progress - needs translation)
5. ⏳ **docs/01-single-agent-with-maf.md** (pending)
6. ⏳ **docs/02-ui-integration-with-maf.md** (pending)
7. ⏳ **docs/03-multi-agent-with-maf.md** (pending)
8. ⏳ **docs/04-aspire-orchestration.md** (pending)
9. ⏳ **docs/05-mcp-server-development.md** (pending)
10. ⏳ **docs/06-mcp-server-integration-with-maf.md** (pending)
11. ⏳ **docs/07-mcp-server-integration-with-copilot-studio.md** (pending)

### 3. Translation Approach

For each file:
1. Open the file in the `english` branch
2. Keep all code snippets, commands, and file paths unchanged
3. Translate:
   - Headings
   - Paragraphs
   - Comments (Korean text in the documentation)
   - UI messages (when mentioned)
4. **DO NOT translate**:
   - Code
   - Commands
   - URLs
   - File paths
   - Package names

### 4. Using AI for Translation

You can use GitHub Copilot or other AI tools to help translate:

```bash
# Example prompt for AI:
"Translate the following Korean markdown documentation to English, 
keeping all code blocks, commands, URLs, and file paths unchanged. 
Only translate the Korean text:"
```

### 5. Syncing Strategy

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
