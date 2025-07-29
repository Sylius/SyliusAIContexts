# Step 1: Preparation

This step prepares your project for testApplication 1.4 migration by updating dependencies, creating directory structure, and updating git configuration.

## 1. Update composer.json

Update your `composer.json` with testApplication dependency and configuration:

### Dependencies
```json
{
    "require": {
        "php": "^8.1",
        "sylius/sylius": "^1.14"
    },
    "require-dev": {
        "sylius/test-application": "^1.14.x-dev"
    }
}
```

### Configuration
```json
{
    "config": {
        "allow-plugins": {
            "symfony/flex": true,
            "symfony/runtime": true
        }
    },
    "extra": {
        "public-dir": "vendor/sylius/test-application/public"
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\YourPlugin\\": "tests/TestApplication/src"
        }
    }
}
```

**Important Notes:**
- Ensure `public-dir` is in the `extra` section, not `config` section
- Update `autoload-dev` to include test paths
- Dependencies should be in alphabetical order

## 2. Create TestApplication Directory Structure

Create the required directory structure:

```bash
mkdir -p tests/TestApplication/config
mkdir -p tests/TestApplication/src/Entity
mkdir -p tests/TestApplication/templates
touch tests/TestApplication/templates/.gitkeep
```

## 3. Update .gitignore

Add TestApplication local files to `.gitignore`:

```gitignore
/var/
/tests/TestApplication/.env.local
/tests/TestApplication/.env.test.local
/tests/TestApplication/.env.*.local
```

## Completion

After completing these steps:

1. Run `composer install` to install new dependencies
2. Verify the directory structure was created correctly
3. **Commit your changes:**

```bash
git add .
git commit -m "feat: prepare project for testApplication 1.4 migration

- Update composer.json with testApplication 1.4 dependency
- Add testApplication configuration and autoload paths
- Create TestApplication directory structure
- Update .gitignore for TestApplication files"
```

## Next Step

Continue to [custom-content.md](custom-content.md) to copy custom content from the old Application.