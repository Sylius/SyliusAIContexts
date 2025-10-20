# Prepare and Run

This step prepares your development environment and runs the test application to verify the migration progress.

**When to skip this step:**
- You already have a working test application setup
- You're doing migration on CI/CD only

**When to do this step:**
- First time setting up the migrated plugin
- After major changes to verify everything works
- Before continuing with feature-specific migration steps

---

## 1. Check Database Configuration

**Note:** The `.env` file may be located in:
- Root directory: `.env`
- Test application: `tests/TestApplication/.env`

Check which structure your plugin uses.

Verify your `.env` file has correct database URL:

```bash
# Check if DATABASE_URL is set correctly
grep DATABASE_URL .env
# Or if using test application structure:
grep DATABASE_URL tests/TestApplication/.env
```

If not exists or incorrect, create/update the appropriate `.env` file:

```env
DATABASE_URL="mysql://root:root@127.0.0.1:3306/sylius_test?serverVersion=8.0"
```

Adjust username, password, host, and database name as needed.

---

## 2. (Optional) Backup Templates

If you want to track which templates were migrated to Twig Hooks:

**Note:** If you have old structure (`src/Resources/views/`), copy from there instead.

```bash
# Copy templates to backup
cp -r templates/ old_templates/
# OR if old structure:
# cp -r src/Resources/views/ old_templates/

# Clear the templates directory
rm -rf templates/*
```

This is helpful to know what's left to migrate in later steps. As you migrate templates to Twig Hooks, you can compare with backup.

---

## 3. Setup Assets Structure

Check if asset directories exist, if not create them:

```bash
mkdir -p assets/admin assets/shop
```

### Create entrypoint files (if missing)

**`assets/admin/entrypoint.js`** (empty file):
```bash
touch assets/admin/entrypoint.js
```

**`assets/shop/entrypoint.js`** (empty file):
```bash
touch assets/shop/entrypoint.js
```

### Create controllers.json files (if missing)

**`assets/admin/controllers.json`:**
```json
{
    "controllers": [],
    "entrypoints": []
}
```

**`assets/shop/controllers.json`:**
```json
{
    "controllers": [],
    "entrypoints": []
}
```

---

## 4. Database Setup

### Create database

```bash
vendor/bin/console doctrine:database:create --if-not-exists
```

### Setup schema and data

**If your plugin has entities:**

**Option A: Plugin has migrations**
```bash
vendor/bin/console doctrine:migrations:migrate -n
```

**Option B: Plugin has no migrations**
```bash
vendor/bin/console doctrine:schema:create
```

### Load fixtures

```bash
vendor/bin/console sylius:fixtures:load -n
```

This provides test data for development.

---

## 5. Install and Build Assets

### Install dependencies

```bash
yarn install --cwd vendor/sylius/test-application
```

### Build assets

For development (with watch mode):
```bash
yarn --cwd vendor/sylius/test-application encore dev
```

Or for production build:
```bash
yarn --cwd vendor/sylius/test-application encore production
```

### Install public assets

```bash
vendor/bin/console assets:install
```

### Troubleshooting: Asset Build Errors

If `encore dev` fails with compilation errors (e.g., missing dependencies, import errors):

**Solution:** Comment out the problematic code with a TODO marker:

1. Identify the file causing the error from the webpack output
2. Open the file and comment out the broken import/code
3. Add a TODO comment: `// TODO: Fix in Assets migration step - [describe issue]`
4. Rebuild assets: `yarn --cwd vendor/sylius/test-application encore dev`

**Example:**
```javascript
// TODO: Fix in Assets migration step - add slick-carousel dependency
// import 'slick-carousel';
```

**Goal:** Ensure assets compile successfully. Missing dependencies will be properly added during the Assets migration step (7.5).

---

## 6. Start Development Server

```bash
symfony server:start
```

The application should now be accessible at `http://127.0.0.1:8000`

**Default credentials:**
- Admin: `http://127.0.0.1:8000/admin`
  - Username: `sylius@example.com`
  - Password: `sylius`
