# Step 6: GitHub Actions Update

Update your CI/CD workflow to work with TestApplication, using migrations instead of schema creation and moving Chrome/server startup before Behat.

## Key Changes Required

Update `.github/workflows/build.yml` with these critical changes:

### 1. Use `vendor/bin/console` instead of `bin/console`
### 2. Use migrations instead of `doctrine:schema:create`
### 3. Move Chrome and server startup to just before Behat

## Updated Workflow Steps

Replace the relevant sections in your workflow:

### Install JS Dependencies
```yaml
- name: Install JS dependencies
  run: yarn install --cwd vendor/sylius/test-application
```

### Database Preparation (Use Migrations)
```yaml
- name: Prepare test application database
  run: |
      vendor/bin/console doctrine:database:create -vvv
      vendor/bin/console doctrine:migrations:migrate --no-interaction -vvv
```

**Important**: Change from `doctrine:schema:create` to `doctrine:migrations:migrate`

### Asset Preparation
```yaml
- name: Prepare test application assets
  run: |
      vendor/bin/console assets:install -vvv
      yarn --cwd vendor/sylius/test-application encore prod
```

### Cache Warmup
```yaml
- name: Prepare test application cache
  run: vendor/bin/console cache:warmup -vvv
```

### Load Fixtures
```yaml
- name: Load fixtures in test application
  run: vendor/bin/console sylius:fixtures:load -n
```

### Other Testing Steps
Keep your existing steps for ECS, PHPStan, PHPUnit, etc. as they are.

### Chrome and Server (Move Before Behat)
```yaml
# ... other testing steps (ECS, PHPStan, PHPUnit, etc.) ...

- name: Run Chrome Headless
  run: google-chrome-stable --enable-automation --disable-background-networking --no-default-browser-check --no-first-run --disable-popup-blocking --disable-default-apps --allow-insecure-localhost --disable-translate --disable-extensions --no-sandbox --enable-features=Metal --headless --remote-debugging-port=9222 --window-size=2880,1800 --proxy-server='direct://' --proxy-bypass-list='*' http://127.0.0.1 > /dev/null 2>&1 &

- name: Run webserver
  run: symfony server:start --port=8080 --daemon && symfony -V

- name: Run Behat
  run: vendor/bin/behat --colors --strict -vvv --no-interaction || vendor/bin/behat --colors --strict -vvv --no-interaction --rerun
```

## Complete Example Section

Here's how the database and asset preparation section should look:

```yaml
        - name: Install JS dependencies
          run: yarn install --cwd vendor/sylius/test-application

        - name: Prepare test application database
          run: |
              vendor/bin/console doctrine:database:create -vvv
              vendor/bin/console doctrine:migrations:migrate --no-interaction -vvv

        - name: Prepare test application assets
          run: |
              vendor/bin/console assets:install -vvv
              yarn --cwd vendor/sylius/test-application encore prod

        - name: Prepare test application cache
          run: vendor/bin/console cache:warmup -vvv

        - name: Load fixtures in test application
          run: vendor/bin/console sylius:fixtures:load -n

        - name: Validate composer.json
          run: composer validate --ansi --strict

        - name: Validate database schema
          run: vendor/bin/console doctrine:schema:validate

        # ... other testing steps (ECS, PHPStan, PHPUnit, PHPSpec) ...

        - name: Run Chrome Headless
          run: google-chrome-stable --enable-automation --disable-background-networking --no-default-browser-check --no-first-run --disable-popup-blocking --disable-default-apps --allow-insecure-localhost --disable-translate --disable-extensions --no-sandbox --enable-features=Metal --headless --remote-debugging-port=9222 --window-size=2880,1800 --proxy-server='direct://' --proxy-bypass-list='*' http://127.0.0.1 > /dev/null 2>&1 &

        - name: Run webserver
          run: symfony server:start --port=8080 --daemon && symfony -V

        - name: Run Behat
          run: vendor/bin/behat --colors --strict -vvv --no-interaction || vendor/bin/behat --colors --strict -vvv --no-interaction --rerun
```

## Important Notes

### Migration vs Schema Create
- **Old**: `doctrine:schema:create --no-interaction -vvv`
- **New**: `doctrine:migrations:migrate --no-interaction -vvv`

### Console Path
- All console commands now use `vendor/bin/console`
- TestApplication provides the console application

### Asset Management
- JavaScript dependencies are installed in TestApplication directory
- Assets are built using TestApplication's webpack configuration
- Use `yarn --cwd vendor/sylius/test-application` for all yarn commands

## Completion Checklist

Verify you have updated:
- [ ] JS dependencies installation path
- [ ] Database preparation to use migrations
- [ ] Asset preparation commands
- [ ] Console commands to use `vendor/bin/console`
- [ ] Chrome and server startup moved before Behat
- [ ] All yarn commands use `--cwd vendor/sylius/test-application`

## Commit Changes

After updating the GitHub Actions workflow:

```bash
git add .github/workflows/build.yml
git commit -m "feat: update GitHub Actions for testApplication

- Use vendor/bin/console for all console commands
- Switch from doctrine:schema:create to migrations
- Update asset building to use TestApplication webpack
- Move Chrome and server startup before Behat tests
- Update yarn commands to use TestApplication directory"
```

## Next Step

Continue to [asset-migration.md](asset-migration.md) to migrate your plugin assets.