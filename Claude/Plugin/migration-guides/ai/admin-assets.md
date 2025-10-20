# Admin Assets Migration - AI Guide

## Workflow Pattern
Each step follows: **Execute → Validate → Report**

## When to Execute
This step is OPTIONAL. Execute only if the plugin has custom JavaScript or CSS in admin panel.

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

- **Contains form interactions or UI logic that could be replaced**?
  → Report to user: "This JavaScript can potentially be rewritten as Live Component or Stimulus Controller if you know how. Would you like to migrate it as-is or attempt rewriting?"
  → Wait for user response

- **User wants to rewrite**?
  → Report detailed suggestions and ask which approach (Live Component or Stimulus)

- **User wants simple migration**?
  → Proceed to Option 3 (move to assets/admin/)

### 3. Option A: Report Live Component Opportunity

If JS could be Live Component:

```bash
Tool: Report to user
Message: "The JavaScript file {filename} contains logic that could be rewritten as a Live Component.
Live Components work with forms or standalone - not limited to forms!
Learn more: https://symfony.com/bundles/ux-live-component/current/index.html
See also: admin-templates.md for examples

Would you like to:
1. Migrate JS as-is to assets/admin/
2. Get suggestions for Live Component rewrite (you'll need to implement it)"
```

Wait for user decision.

### 4. Option B: Report Stimulus Opportunity

If JS is UI component:

```bash
Tool: Report to user
Message: "The JavaScript file {filename} appears to be a UI component (modal, dropdown, etc.).
This could be rewritten as a Stimulus Controller if you know how.
Learn more: https://symfony.com/bundles/StimulusBundle/current/index.html

Would you like to:
1. Migrate JS as-is to assets/admin/
2. Get suggestions for Stimulus Controller structure (you'll need to implement it)"
```

Wait for user decision.

### 5. Option C: Move to assets/admin/ (Default approach)

Move JavaScript files to correct location:

```bash
Tool: Bash
Command: mkdir -p assets/admin
```

For each JS file:

```bash
Tool: Bash
Command: mv {source_path} assets/admin/{filename}
```

Possible source paths:
- `src/Resources/public/{filename}`
- `assets/{filename}` (if in wrong location)

Update entrypoint.js:

```bash
Tool: Read
File: assets/admin/entrypoint.js
```

```bash
Tool: Edit
File: assets/admin/entrypoint.js
Old: (existing content or empty)
New: import './{filename}';

(existing content if any)
```

### 6. Migrate CSS/SCSS

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

Move to assets/admin/ (convert to SCSS if needed):

```bash
Tool: Bash
Command: mv {source_path} assets/admin/{filename}.scss
```

Update entrypoint:

```bash
Tool: Edit
File: assets/admin/entrypoint.js
Old: (existing imports)
New: import './{filename}.scss';

(existing imports)
```

### 7. Verify No Files Remain in Old Locations

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

ALL assets must be moved to assets/admin/ or assets/shop/.
Should I move these files as well?"
```

### 8. Declare JavaScript Dependencies (If Needed)

**IMPORTANT:** If your JavaScript imports external packages (like Swiper, Chart.js, etc.), you must declare them in `tests/TestApplication/package.json`.

Check if JavaScript uses external imports:

```bash
Tool: Grep
Pattern: ^import .* from ['"](?!\.|\@symfony)
Path: assets/admin/
Output: content
-n: true
```

If external imports found (packages not starting with `.` or `@symfony`), create package.json:

```bash
Tool: Write
File: tests/TestApplication/package.json
Content:
{
    "dependencies": {
        "package-name": "^version"
    }
}
```

**Examples of packages that need declaration:**
- `swiper` - Modern carousel library
- `chart.js` - Charting library
- `flatpickr` - Date picker
- `tom-select` - Select enhancement (if not already in Sylius)

**How it works:**
1. Plugin declares dependency in `tests/TestApplication/package.json`
2. Test application merge script reads it during `yarn install`
3. End users must install the same packages (document in UPGRADE.md or README.md)

**Check if package.json already exists:**

```bash
Tool: Bash
Command: ls -la tests/TestApplication/package.json 2>/dev/null || echo "Does not exist"
```

If exists, merge dependencies:

```bash
Tool: Edit
File: tests/TestApplication/package.json
Old: "dependencies": {
New: "dependencies": {
        "new-package": "^version",
```

### 9. Validate

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
- All JavaScript moved to `assets/admin/` or `assets/shop/`
- All CSS/SCSS moved to `assets/admin/` or `assets/shop/`
- No assets remain in `src/Resources/public/`
- Assets imported in entrypoint files
- Assets build successfully with Webpack Encore
- No errors in compilation
- Functionality works in admin panel

## Notes for AI
- ALWAYS check multiple possible locations for assets (src/Resources/public/, assets/, inline in templates)
- **CRITICAL:** If JavaScript imports external packages, MUST create `tests/TestApplication/package.json` with dependencies
- Check for external imports using Grep pattern: `^import .* from ['"](?!\.|\@symfony)`
- Report Live Component and Stimulus opportunities but DON'T force them - user decides
- If user wants to rewrite to Live Component/Stimulus, provide suggestions but user must implement
- Default approach is simple migration (move to assets/admin/ + import)
- Ensure NOTHING remains in src/Resources/public/ after migration
- Check both .js and .css/.scss files
- Convert .css to .scss when moving (recommended)
- Report all findings and ask for confirmation before bulk operations
- If unsure about approach, ask user which they prefer
- Document required packages in UPGRADE.md or README.md for end users
