# Prepare and Run - AI Guide

This guide provides Tool commands to prepare development environment and run test application.

## Prerequisites

- Container compiles successfully (Step 2 completed)
- Configuration moved to root if applicable (Step 4 completed)

---

## Step 1: Check database configuration

**Note:** The `.env` file may be located in:
- Root directory: `.env`
- Test application: `tests/TestApplication/.env`

Check which structure the plugin uses first.

```bash
Tool: Bash
Command: grep DATABASE_URL .env 2>/dev/null || grep DATABASE_URL tests/TestApplication/.env 2>/dev/null || echo "Not found"
```

**If not found or needs update:**

```bash
Tool: Read
File: .env
```
or
```bash
Tool: Read
File: tests/TestApplication/.env
```

Check if DATABASE_URL exists. If not:

```bash
Tool: Edit
File: .env  (or tests/TestApplication/.env)
Old: (end of file)
New:
DATABASE_URL="mysql://root:root@127.0.0.1:3306/sylius_test?serverVersion=8.0"
```

Or if exists but incorrect, update it.

---

## Step 2: Backup templates (Always execute)

Templates need to be backed up to track migration progress.

### Detect templates location

```bash
Tool: Bash
Command: test -d templates && echo "new structure" || test -d src/Resources/views && echo "old structure" || echo "no templates"
```

**If new structure (templates/):**

```bash
Tool: Bash
Command: cp -r templates/ old_templates/ && rm -rf templates/*
```

**If old structure (src/Resources/views/):**

```bash
Tool: Bash
Command: cp -r src/Resources/views/ old_templates/ && rm -rf src/Resources/views/*
```

This allows tracking which templates were migrated to Twig Hooks.

---

## Step 3: Setup assets structure

### Check if assets directories exist

```bash
Tool: Bash
Command: ls -la assets/admin assets/shop 2>&1 || echo "Not found"
```

**If not found:**

```bash
Tool: Bash
Command: mkdir -p assets/admin assets/shop
```

### Create entrypoint.js files if missing

```bash
Tool: Bash
Command: test -f assets/admin/entrypoint.js || touch assets/admin/entrypoint.js
```

```bash
Tool: Bash
Command: test -f assets/shop/entrypoint.js || touch assets/shop/entrypoint.js
```

### Create controllers.json files if missing

Check if exists:

```bash
Tool: Bash
Command: test -f assets/admin/controllers.json && echo "exists" || echo "missing"
```

**If missing:**

```bash
Tool: Write
File: assets/admin/controllers.json
```

Content:
```json
{
    "controllers": [],
    "entrypoints": []
}
```

Same for shop:

```bash
Tool: Bash
Command: test -f assets/shop/controllers.json && echo "exists" || echo "missing"
```

**If missing:**

```bash
Tool: Write
File: assets/shop/controllers.json
```

Content:
```json
{
    "controllers": [],
    "entrypoints": []
}
```

---

## Step 4: Database setup

### Create database

```bash
Tool: Bash
Command: vendor/bin/console doctrine:database:create --if-not-exists
```

### Detect if plugin has entities

```bash
Tool: Glob
Pattern: *.php
Path: src/Entity/
```

**If entities found:**

### Check if plugin has migrations

```bash
Tool: Bash
Command: test -d src/Migrations && echo "has migrations" || echo "no migrations"
```

**If has migrations:**

```bash
Tool: Bash
Command: vendor/bin/console doctrine:migrations:migrate -n
```

**If no migrations:**

```bash
Tool: Bash
Command: vendor/bin/console doctrine:schema:create
```

### Load fixtures (always)

**Execute this regardless of migrations/schema:**

```bash
Tool: Bash
Command: vendor/bin/console sylius:fixtures:load -n
```

---

## Step 5: Install and build assets

### Install yarn dependencies

```bash
Tool: Bash
Command: yarn install --cwd vendor/sylius/test-application
Timeout: 300000
```

### Build assets (dev mode)

```bash
Tool: Bash
Command: yarn --cwd vendor/sylius/test-application encore dev
Timeout: 300000
```

### Install public assets

```bash
Tool: Bash
Command: vendor/bin/console assets:install
```

### Handle asset build errors (if any)

If `encore dev` fails with compilation errors:

**Check the error output and identify the problematic file/import.**

**Solution:** Comment out the broken code with TODO:
- Add TODO comment explaining the issue
- Comment out the problematic import/code
- Specify it will be fixed in Assets migration step (7.5)

**Rebuild assets:**
```bash
Tool: Bash
Command: yarn --cwd vendor/sylius/test-application encore dev
Timeout: 300000
```

**Goal:** Assets must compile successfully. Missing dependencies and broken code will be properly fixed in Assets migration step.

---

## Step 6: Start development server

```bash
Tool: Bash
Command: symfony server:start -d
```

Application should be accessible at `http://127.0.0.1:8000`

**Default credentials:**
- Admin: `http://127.0.0.1:8000/admin`
  - Username: `sylius@example.com`
  - Password: `sylius`

---

## Summary Checklist

After executing this guide:
- ✅ Database created and configured
- ✅ Assets structure set up
- ✅ Schema/migrations applied
- ✅ Fixtures loaded
- ✅ Assets built
- ✅ Development server running

## Notes for AI

- Step 2 (Backup templates) should ALWAYS be executed - it's non-destructive and helps track migration progress
- Don't stop server after starting - let it run in background
- If any step fails, report error to user
- Assets build may take time - use appropriate timeout
- Fixtures load may prompt for confirmation - use `-n` flag
