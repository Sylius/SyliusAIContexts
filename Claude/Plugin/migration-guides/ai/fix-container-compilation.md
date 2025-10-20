# Fix Container Compilation - AI Guide

This guide provides Tool commands to fix service container compilation after Sylius 2.0 update.

## Prerequisites

Composer dependencies already updated to Sylius 2.0.

## Philosophy

**IMPORTANT - Read this first:**

1. ✅ **Try to fix first** - If service has replacement in UPGRADE-2.0.md → update service ID
2. ⏳ **Comment only if needed** - Only comment out code with no replacement or too complex to fix

**DO NOT comment out services that just need a new ID!**

## Step 1: Test container compilation

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

**If successful:** Skip this guide, proceed to next step.

**If errors:** Continue with steps below.

---

## Step 2: Identify error types

Look at the error output from Step 1:

### Error Type A: "Service 'sylius.xyz' not found"

→ Execute Step 3 (Check Sylius UPGRADE file)

### Error Type B: "Class 'XYZ' not found"

→ Execute Step 4 (Comment out deprecated code)

### Error Type C: Other errors

→ Execute Step 5 (Fix service definitions)

---

## Step 3: Check Sylius UPGRADE file for service changes

```bash
Tool: Read
File: vendor/sylius/sylius/UPGRADE-2.0.md
```

**Important:** This file contains a large table showing:
- Removed services
- Replacement services
- Migration instructions

Search for the service name from the error in this file.

**Common patterns:**
- `sylius.repository.*` → Usually unchanged
- `sylius.factory.*` → May be renamed or restructured
- `sylius.ui.*` → Removed, replaced by Twig Hooks
- `sylius.grid.*` → May be restructured

### If service replacement found → FIX IT NOW

**DO NOT comment it out - just update the service ID!**

```bash
Tool: Grep
Pattern: {old_service_id}
Path: src/
Output: files_with_matches
```

Also search in config:

```bash
Tool: Grep
Pattern: {old_service_id}
Path: config/
Output: files_with_matches
```

For each file found:

```bash
Tool: Read
File: {file_from_grep}
```

```bash
Tool: Edit
File: {file_from_grep}
Old: {old_service_id}
New: {new_service_id}
```

Re-test after fix:

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

### If no replacement (service removed) → Comment it out

**Only if UPGRADE-2.0.md says service was removed with no replacement:**

```bash
Tool: Edit
File: {file_with_broken_service}
Old: {code_block_using_service}
New: // TODO: Fix in appropriate step (check which feature this is)
// {code_block_using_service}
```

---

## Step 4: Comment out what cannot be fixed now

**ONLY comment out code that:**
- Has no replacement (removed Sylius features)
- Is too complex to fix now
- Will be migrated in later steps

**DO NOT comment out services that just need new ID** - those should be fixed in Step 3!

**IMPORTANT:** Always add TODO comments with each commented code block explaining:
- What needs to be fixed
- In which step it will be fixed

### 4.1: Comment out sylius_ui event configurations

```bash
Tool: Grep
Pattern: sylius_ui
Path: src/DependencyInjection/
Output: files_with_matches
```

If found:

```bash
Tool: Read
File: src/DependencyInjection/{PluginName}Extension.php
```

```bash
Tool: Edit
File: src/DependencyInjection/{PluginName}Extension.php
Old: $container->prependExtensionConfig('sylius_ui', [
New: // TODO: Migrate to Twig Hooks in Step (Templates)
/*
$container->prependExtensionConfig('sylius_ui', [
```

And at the end of the sylius_ui block:

```bash
Tool: Edit
File: src/DependencyInjection/{PluginName}Extension.php
Old: ]);
New: ]);
*/
```

Check for sylius_ui config files:

```bash
Tool: Glob
Pattern: sylius_ui.yaml
Path: config/packages/
```

If found, comment out:

```bash
Tool: Edit
File: config/packages/sylius_ui.yaml
Old: sylius_ui:
New: # TODO: Migrate to Twig Hooks in Step (Templates)
# sylius_ui:
```

### 4.2: Comment out API service definitions

If you see errors related to API Platform services:

```bash
Tool: Grep
Pattern: api_platform|API Platform|DataProvider|DataPersister
Path: config/services.xml
Output: files_with_matches
```

For each found file:

```bash
Tool: Read
File: {file_from_grep}
```

```bash
Tool: Edit
File: {file_from_grep}
Old: <service id="app.api.{name}">
New: <!-- TODO: Migrate to API Platform 4.x in Step 9 -->
<!--
<service id="app.api.{name}">
```

Close the comment after the service definition.

### 4.3: Comment out UI Block event listeners

```bash
Tool: Grep
Pattern: use Sylius\\Bundle\\UiBundle\\Block
Path: src/
Output: files_with_matches
```

If found, comment out entire class:

```bash
Tool: Edit
File: {file_from_grep}
Old: <?php
New: <?php
// TODO: Migrate to Twig Hooks in Step (Templates)
/*
```

Add closing comment at end of file.

### 4.4: Any other broken code

If container still won't compile, identify the problematic service/class from error message and comment it out with TODO.

**IMPORTANT NOTE FOR AI:**
- Every commented code block MUST have a TODO comment
- TODO should specify which migration step will fix it
- Track all TODOs - they will be verified in final cleanup step
- **First try to fix (Step 3) - only comment if no fix available**
- Don't comment services that just need new ID!

---

## Step 5: Re-test container compilation

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

**Expected:** Command succeeds without errors.

**If still failing:**
1. Read error message
2. Identify the problematic service/class
3. Comment it out with TODO (Step 4.4)
4. Try cache:clear again
5. Repeat until container compiles

**Goal:** Container must compile. Don't try to fix features now - just comment them out.

---

## Step 6: Validate no critical errors

```bash
Tool: Bash
Command: vendor/bin/console debug:container 2>&1 | head -20
```

Should show container statistics without errors.

---

## Summary Checklist

After executing this guide:
- ✅ Container compiles successfully (`cache:clear` works)
- ✅ All broken code commented out with TODO comments
- ✅ sylius_ui configurations commented out
- ✅ API service definitions commented out (if applicable)
- ✅ Missing Sylius services identified in UPGRADE-2.0.md
- ✅ TODO comments specify which step will fix each issue

## What You've Fixed

- ✅ Service container compiles
- ✅ Removed blocking errors
- ⏳ Runtime features may still be broken (will fix in later steps)
- 📝 All TODOs tracked for later steps

## Important Notes for AI

**TODO Tracking:**
- Every commented code block has a TODO comment
- TODOs specify which migration step will fix the issue
- In final cleanup step (Step 10), verify all TODOs are resolved
- If any TODOs remain, report them to user
