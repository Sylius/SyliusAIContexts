# Cleanup - AI Guide

This guide helps review and clean up migration artifacts, commented code, and TODO markers.

## Workflow Pattern
Each step follows: **Execute → Validate → Report**

---

## Step 1: Find all TODO markers

Search for TODO comments left during migration:

```bash
Tool: Grep
Pattern: TODO
Path: ./
Glob: **/*.php
Output: content
Context: -C 2
```

**For each TODO found:**
- Read the context
- Check if the migration step was completed
- If completed → proceed to removal
- If not completed → report to user

---

## Step 2: Find all commented code blocks

Search for large commented blocks that might be migration leftovers:

```bash
Tool: Grep
Pattern: ^[ \t]*\/\*\*?$
Path: ./
Glob: **/*.php
Output: content
Context: -A 5
```

Look specifically for:
- Commented `sylius_ui` configuration
- Commented event subscribers
- Commented old template code
- Commented routing

**Common patterns to search:**

```bash
Tool: Grep
Pattern: \/\*.*sylius_ui
Path: src/
Output: content
Context: -C 3
```

```bash
Tool: Grep
Pattern: \/\*.*Migrate.*Twig Hooks
Path: src/
Output: content
Context: -C 3
```

---

## Step 3: Review each finding

For each commented block or TODO:

1. **Read the full file context:**
```bash
Tool: Read
File: {file_path}
```

2. **Determine if it's obsolete:**
   - Was this migrated to Twig Hooks? → Remove
   - Was this replaced by Live Component? → Remove
   - Was this moved to assets/? → Remove
   - Is this still needed? → Keep or update TODO

3. **If obsolete, remove:**
```bash
Tool: Edit
File: {file_path}
Old: {commented_block_with_todo}
New: (empty or updated code)
```

---

## Step 4: Check for orphaned template files

Look for old template files that should have been removed:

```bash
Tool: Bash
Command: find templates/ -name "_*.html.twig" 2>/dev/null || echo "None found"
```

Files like:
- `_javascripts.html.twig` (replaced by assets or Live Components)
- `_form.html.twig` (replaced by Twig Hooks)
- `_breadcrumb.html.twig` (integrated into new structure)

**For each orphaned template:**

```bash
Tool: Read
File: templates/{path}/_*.html.twig
```

Check if it's still used:
```bash
Tool: Grep
Pattern: {filename}
Path: ./
Output: files_with_matches
```

If not used anywhere → remove:
```bash
Tool: Bash
Command: git rm templates/{path}/_*.html.twig
```

---

## Step 5: Validate after cleanup

Clear cache and check for errors:

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

Expected: No errors

Check container compiles:

```bash
Tool: Bash
Command: vendor/bin/console debug:container --parameters 2>&1 | head -20
```

Expected: No service configuration errors

---

## Step 6: Final verification

Run a quick sanity check:

```bash
Tool: Grep
Pattern: TODO|FIXME|XXX
Path: src/
Output: count
```

Report to user:
- How many TODOs remain (should be minimal or zero)
- Which files still have commented code (if any)
- Which templates were removed

---

## Summary Checklist

After executing this guide:
- ✅ All migration-related TODOs reviewed
- ✅ Obsolete commented code removed
- ✅ Orphaned template files removed
- ✅ Container compiles without errors
- ✅ Cache clears successfully

## Notes for AI

- DON'T remove TODOs that are unrelated to migration
- DON'T remove commented code that documents why something was done a certain way
- DO remove:
  - TODOs about "Migrate to Twig Hooks" if migration is complete
  - Commented sylius_ui blocks if replaced with Twig Hooks
  - Old template includes if replaced by assets or Live Components
- ALWAYS validate after removal
- If unsure whether to remove something, ask the user
