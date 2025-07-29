# Step 8: Testing and Validation

**MANDATORY TESTING PHASE**: Before removing old folders, ALL tests must pass. This ensures the migration is successful.

## Complete Migration Process

Follow these steps in the correct order:

### 1. Install Dependencies
```bash
composer install
```

### 2. Create Database
```bash
APP_ENV=test vendor/bin/console doctrine:database:create
```

### 3. Run Migrations
```bash
APP_ENV=test vendor/bin/console doctrine:schema:create --no-interaction
```

### 4. Install Yarn Dependencies (in TestApplication directory)
```bash
yarn --cwd vendor/sylius/test-application install
```

### 5. Install Assets
```bash
APP_ENV=test vendor/bin/console assets:install -vvv
```

### 6. Create Node Modules Symlink
```bash
rm -f node_modules
ln -sf vendor/sylius/test-application/node_modules node_modules
```

### 7. Build Assets (in TestApplication directory)
```bash
yarn --cwd vendor/sylius/test-application build
```

## Mandatory Test Suite

**CRITICAL**: Run ALL tests and ensure they pass before proceeding.

### Test Each Component Systematically

Run tests in this exact order and ensure ALL pass:

#### 1. ECS (Easy Coding Standard)
```bash
vendor/bin/ecs check
```
If issues found, fix them:
```bash
vendor/bin/ecs check --fix
```

#### 2. PHPStan (Static Analysis)
```bash
vendor/bin/phpstan analyse
```

#### 3. PHPUnit (Unit Tests)
```bash
APP_ENV=test vendor/bin/phpunit
```

#### 4. PHPSpec (Specification Tests)
```bash
vendor/bin/phpspec run
```

#### 5. Behat (Acceptance Tests)

First, prepare the test environment:
```bash
APP_ENV=test vendor/bin/console cache:warmup -vvv
APP_ENV=test vendor/bin/console sylius:fixtures:load -n
```

Start the test server:
```bash
APP_ENV=test symfony server:start --port=8080 --daemon
```

Run Behat tests:
```bash
APP_ENV=test vendor/bin/behat --no-interaction
```

Stop the server when done:
```bash
symfony server:stop
```

## Test Failure Handling

**Important**: If ANY test fails, you must fix the issue before proceeding.

### Common Issues and Solutions

#### Container Build Failures
- **Issue**: Container fails to build after migration
- **Solutions**:
  1. Missing bundles in `bundles.php` - ensure all composer dependencies are registered
  2. Duplicate configuration imports - remove duplicate imports
  3. Service configuration syntax - use proper PHP configurator syntax

#### Database Issues
- **Issue**: Database connection problems or missing tables
- **Solutions**:
  1. Check database configuration in `.env.local` files
  2. Ensure migrations run successfully
  3. Verify custom plugin migrations are included

#### Asset Compilation Issues
- **Issue**: Assets fail to compile
- **Solutions**:
  1. Check entrypoint.js files have correct imports
  2. Verify asset directory structure is correct
  3. Use relative paths instead of problematic aliases

## Success Criteria

Migration is successful when:
- [ ] ✅ ECS passes (coding standards)
- [ ] ✅ PHPStan passes (static analysis)
- [ ] ✅ PHPUnit passes (unit tests)
- [ ] ✅ PHPSpec passes (specification tests)
- [ ] ✅ Behat passes (acceptance tests)
- [ ] ✅ Assets compile successfully
- [ ] ✅ Database migrations work
- [ ] ✅ Test server starts without errors

## Clean Up Old Files

**ONLY after ALL tests pass**, remove old files:

```bash
# Remove old test application (if it still exists)
rm -rf tests/Application

# Remove node symlink script (if exists)
rm -f bin/create_node_symlink.php
```

## Final Verification

After cleanup, run a final test to ensure everything still works:

```bash
# Quick verification
composer install
APP_ENV=test vendor/bin/console cache:clear
yarn --cwd vendor/sylius/test-application build
vendor/bin/phpspec run
```

## Commit Final Changes

After successful testing and cleanup:

```bash
git add .
git add -u  # Stage deletions
git commit -m "feat: complete testApplication migration and cleanup

- All tests passing (ECS, PHPStan, PHPUnit, PHPSpec, Behat)
- Assets compiling successfully
- Database migrations working
- Remove old test Application directory
- Migration to testApplication 1.4 complete

✅ Migration validated and complete"
```

## Migration Complete

Congratulations! Your plugin has been successfully migrated to TestApplication 1.4.

### What Changed
- ✅ Using centralized TestApplication instead of custom test app
- ✅ Modern asset compilation with TestApplication webpack
- ✅ Standardized testing environment
- ✅ Reduced maintenance overhead
- ✅ Better CI/CD integration

### Next Steps
- Monitor CI/CD pipeline to ensure it works with the new setup
- Update documentation if needed
- Consider removing any old migration-related files or scripts

## Troubleshooting

If you encounter issues after migration:

1. **Check git history** - each step was committed separately for easy rollback
2. **Review test outputs** - error messages usually indicate the specific issue
3. **Verify file permissions** - ensure TestApplication files are readable
4. **Check environment variables** - verify database and other configs are correct

The migration is now complete and your plugin is ready for use with TestApplication 1.4!
