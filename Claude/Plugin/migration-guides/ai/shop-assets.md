# Shop Assets Migration - AI Guide

## Workflow Pattern
Each step follows: **Execute → Validate → Report**

## When to Execute
This step is OPTIONAL. Execute only if the plugin has custom JavaScript or CSS in shop.

## Commands

### 1. Check for Existing Assets

Check various possible locations:

```bash
Tool: Glob
Pattern: *.js
Path: src/Resources/public/
```

```bash
Tool: Glob
Pattern: *.css
Path: src/Resources/public/
```

```bash
Tool: Glob
Pattern: *.js
Path: assets/
```

```bash
Tool: Glob
Pattern: *.css
Path: assets/
```

If no assets found → Skip this step

### 2. Analyze JavaScript Type

For each JS file found, read and analyze:

```bash
Tool: Read
File: {path_to_js_file}
```

**Decision logic:**

- **Contains UI logic that could be replaced**?
  → Report to user: "This JavaScript can potentially be rewritten as Stimulus Controller if you know how. Would you like to migrate it as-is or attempt rewriting?"
  → Wait for user response

- **User wants to rewrite**?
  → Report detailed suggestions for Stimulus Controller

- **User wants simple migration**?
  → Proceed to Option 2 (move to assets/shop/)

### 3. Option A: Report Stimulus Opportunity

If JS is UI component:

```bash
Tool: Report to user
Message: "The JavaScript file {filename} appears to be a UI component (product gallery, image zoom, etc.).
This could be rewritten as a Stimulus Controller if you know how.
Learn more: https://docs.sylius.com/the-customization-guide/customizing-dynamic-elements

Would you like to:
1. Migrate JS as-is to assets/shop/
2. Get suggestions for Stimulus Controller structure (you'll need to implement it)"
```

Wait for user decision.

### 4. Option B: Move to assets/shop/ (Default approach)

Move JavaScript files to correct location:

```bash
Tool: Bash
Command: mkdir -p assets/shop
```

For each JS file:

```bash
Tool: Bash
Command: mv {source_path} assets/shop/{filename}
```

Possible source paths:
- `src/Resources/public/{filename}`
- `assets/{filename}` (if in wrong location)

Update entrypoint.js:

```bash
Tool: Read
File: assets/shop/entrypoint.js
```

```bash
Tool: Edit
File: assets/shop/entrypoint.js
Old: (existing content or empty)
New: import './{filename}';

(existing content if any)
```

### 5. Migrate CSS/SCSS

Check for CSS files in all locations:

```bash
Tool: Glob
Pattern: *.css
Path: src/Resources/public/
```

```bash
Tool: Glob
Pattern: *.scss
Path: src/Resources/public/
```

```bash
Tool: Glob
Pattern: *.css
Path: assets/
```

For each CSS/SCSS found:

Move to assets/shop/ (convert to SCSS if needed):

```bash
Tool: Bash
Command: mv {source_path} assets/shop/{filename}.scss
```

Update entrypoint:

```bash
Tool: Edit
File: assets/shop/entrypoint.js
Old: (existing imports)
New: import './{filename}.scss';

(existing imports)
```

### 6. Verify No Files Remain in Old Locations

Check if src/Resources/public/ still has assets:

```bash
Tool: Bash
Command: find src/Resources/public/ -type f \( -name "*.js" -o -name "*.css" -o -name "*.scss" \) 2>/dev/null || echo "No assets found"
```

If files found, report to user:

```bash
Tool: Report
Message: "Warning: Files still remain in src/Resources/public/:
{list_of_files}

ALL assets must be moved to assets/shop/.
Should I move these files as well?"
```

### 7. Validate

Rebuild assets:

```bash
Tool: Bash
Command: yarn --cwd vendor/sylius/test-application encore dev
Timeout: 120000
```

Expected: Build completes without errors

If errors occur, report to user with error details.

Install assets:

```bash
Tool: Bash
Command: vendor/bin/console assets:install
```

Expected: Assets installed successfully

Clear cache:

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

Expected: Cache cleared successfully

## Success Criteria
- All JavaScript moved to `assets/shop/`
- All CSS/SCSS moved to `assets/shop/`
- No assets remain in `src/Resources/public/`
- Assets imported in entrypoint files
- Assets build successfully with Webpack Encore
- No errors in compilation
- Functionality works in shop

## Notes for AI
- ALWAYS check multiple possible locations for assets (src/Resources/public/, assets/, inline in templates)
- **CRITICAL:** If JavaScript imports external packages, MUST create `tests/TestApplication/package.json` with dependencies
- Check for external imports using Grep pattern: `^import .* from ['"](?!\.|\@symfony)`
- Report Stimulus opportunities but DON'T force it - user decides
- If user wants to rewrite to Stimulus, provide suggestions but user must implement
- Default approach is simple migration (move to assets/shop/ + import)
- Ensure NOTHING remains in src/Resources/public/ after migration
- Check both .js and .css/.scss files
- Convert .css to .scss when moving (recommended)
- Report all findings and ask for confirmation before bulk operations
- If unsure about approach, ask user which they prefer
- Document required packages in UPGRADE.md or README.md for end users
