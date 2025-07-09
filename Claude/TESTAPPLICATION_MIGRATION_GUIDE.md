# Sylius TestApplication Migration Guide

This guide documents how to migrate a Sylius plugin from using a traditional `tests/Application` setup to the centralized Sylius TestApplication framework.

## Overview 

The Sylius TestApplication eliminates the need to maintain custom test applications for each plugin, reducing maintenance overhead while providing a standardized testing environment.

## Prerequisites

- Sylius plugin with existing `tests/Application` setup
- Composer dependency management
- Basic understanding of Behat, PHPUnit, and Symfony configuration

## Migration Steps

### 1. Update composer.json

Add the TestApplication dependency and configure paths:

```json
{
    "require-dev": {
        "sylius/test-application": "^2.0.0@alpha"
    },
    "config": {
        "allow-plugins": {
            "symfony/flex": true,
            "symfony/runtime": false
        }
    },
    "extra": {
        "public-dir": "vendor/sylius/test-application/public"
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\YourPlugin\\": [
                "tests/TestApplication/src",
                "tests/Behat",
                "tests/Integration",
                "tests/Unit"
            ]
        }
    }
}
```

**Important Notes:**
- Use `"sylius/test-application": "^2.0.0@alpha"` but expect to get v2.1.x due to Sylius compatibility requirements
- **Remove the entire `scripts` section** - node symlink scripts and other custom scripts are no longer needed with TestApplication
- Ensure `public-dir` is in the `extra` section, not `config` section
- Update `autoload-dev` to include all necessary test paths

### 2. Create TestApplication Directory Structure

Create the following structure:

```
tests/TestApplication/
├── config/
│   ├── config.yaml
│   ├── routes.yaml
│   ├── services.yaml
│   └── services_test.php
├── src/
│   └── Entity/
│       └── [Your custom entities if needed]
├── templates/
│   └── .gitkeep
├── .env
├── .env.test
├── .env.local (not committed)
├── .env.test.local (not committed)
└── bundles.php
```

### 3. Configure Database Connection

**Important**: Before proceeding, configure your database connection for the TestApplication.

Ask the user for their database configuration:
- Database host and port
- Database credentials (username/password)  
- Database name pattern

Create the appropriate `.env.local` and `.env.test.local` files with the user's specific database configuration.

### 4. Configure TestApplication Files

#### tests/TestApplication/bundles.php

**Critical**: All composer dependencies must be registered in bundles.php

```php
<?php

declare(strict_types=1);

$bundles = [
    // Register ALL dependencies from composer.json require-dev
    YourVendor\YourPlugin\YourPluginBundle::class => ['all' => true],
    // Add other plugin dependencies as needed
];

return $bundles;
```

#### tests/TestApplication/config/config.yaml

**Check TestApplication package configuration and remove duplicates**:

```yaml
imports:
    - { resource: "@YourPlugin/config/config.yaml" }
    - { resource: "services.yaml" }
    - { resource: "services_test.php" }

# Only include plugin-specific configuration that differs from core Sylius
# Remove duplicates like sylius_shop, sylius_api, framework, etc.

# Add your plugin-specific entity mappings here if needed
# sylius_customer:
#     resources:
#         customer:
#             classes:
#                 model: Tests\YourPlugin\Entity\Customer
#                 repository: YourPlugin\Repository\CustomerRepository

twig:
    paths:
        '%kernel.project_dir%/../../../tests/TestApplication/templates': ~

# Only add doctrine mapping if you have custom entities
# Use type: attribute for modern Doctrine attributes or type: xml for XML mapping
doctrine:
    orm:
        entity_managers:
            default:
                mappings:
                    TestApp:
                        is_bundle: false
                        type: attribute  # or 'xml' for XML mapping
                        dir: '%kernel.project_dir%/../../../tests/TestApplication/src/Entity'  # for attributes
                        # dir: '%kernel.project_dir%/../../../tests/TestApplication/config/doctrine'  # for XML
                        prefix: Tests\YourPlugin  # for attributes
                        # prefix: Tests\YourPlugin\Entity  # for XML mapping
```

#### tests/TestApplication/config/routes.yaml
```yaml
your_plugin:
    resource: "@YourPlugin/config/routes.yaml"
```

#### tests/TestApplication/config/services.yaml

Main application services:

```yaml
services:
    # Add your plugin-specific services here
    app.your_service:
        class: Tests\YourPlugin\Service\YourService
        arguments:
            - "@sylius.repository.product"
```

#### tests/TestApplication/config/services_test.php

Test-specific services only:

```php
<?php

declare(strict_types=1);

use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return function (ContainerConfigurator $container) {
    $env = $_ENV['APP_ENV'] ?? 'dev';

    if (str_starts_with($env, 'test')) {
        $container->import('../../../vendor/sylius/sylius/src/Sylius/Behat/Resources/config/services.xml');
        $container->import('@YourPlugin/tests/Behat/Resources/services.xml');
        
    }
};
```

#### tests/TestApplication/.env

Structure with proper sections:

```
###> sylius/test-application ###
CONFIGS_TO_IMPORT="@YourPlugin/tests/TestApplication/config/config.yaml"
ROUTES_TO_IMPORT="@YourPlugin/tests/TestApplication/config/routes.yaml"
TEST_APP_BUNDLES_PATH="tests/TestApplication/bundles.php"
###< sylius/test-application ###

###> doctrine/doctrine-bundle ###
DATABASE_URL=mysql://root:root@127.0.0.1:3306/plugin_name_%kernel.environment%?serverVersion=8.0
###< doctrine/doctrine-bundle ###

# Add plugin-specific environment variables here
###> your-plugin ###
YOUR_PLUGIN_SETTING=value
###< your-plugin ###
```

#### tests/TestApplication/.env.test

Same structure as .env - default values:

```
###> sylius/test-application ###
CONFIGS_TO_IMPORT="@YourPlugin/tests/TestApplication/config/config.yaml"
ROUTES_TO_IMPORT="@YourPlugin/tests/TestApplication/config/routes.yaml"
TEST_APP_BUNDLES_PATH="tests/TestApplication/bundles.php"
###< sylius/test-application ###

# Add plugin-specific environment variables here
###> your-plugin ###
YOUR_PLUGIN_SETTING=value
###< your-plugin ###
```

#### tests/TestApplication/.env.local 

**Not committed** - your specific overrides:

```
###> doctrine/doctrine-bundle ###
DATABASE_URL=mysql://root:root@127.0.0.1:3306/plugin_name_%kernel.environment%?serverVersion=8.0
###< doctrine/doctrine-bundle ###

# Add plugin-specific environment overrides here
###> your-plugin ###
YOUR_PLUGIN_SETTING=custom_value
###< your-plugin ###
```

#### tests/TestApplication/.env.test.local

**Not committed** - your specific test overrides:

```
###> doctrine/doctrine-bundle ###
DATABASE_URL=mysql://root:root@127.0.0.1:3306/plugin_name_%kernel.environment%?serverVersion=8.0
###< doctrine/doctrine-bundle ###

# Add plugin-specific environment overrides here
###> your-plugin ###
YOUR_PLUGIN_SETTING=custom_value
###< your-plugin ###
```

#### tests/TestApplication/templates/.gitkeep

Create empty templates directory to satisfy Twig configuration:

```bash
mkdir -p tests/TestApplication/templates
touch tests/TestApplication/templates/.gitkeep
```

### 5. Update behat.yml.dist

Update the Behat configuration to use TestApplication:

```yaml
default:
    extensions:
        Behat\MinkExtension:
            base_url: "https://127.0.0.1:8080/"
            sessions:
                panther:
                    panther:
                        options:
                            webServerDir: '%paths.base%/vendor/sylius/test-application/public'

        FriendsOfBehat\SymfonyExtension:
            bootstrap: vendor/sylius/test-application/config/bootstrap.php
            kernel:
                class: Sylius\TestApplication\Kernel
```

### 6. Update phpunit.xml.dist

Update PHPUnit configuration:

```xml
<phpunit bootstrap="vendor/sylius/test-application/config/bootstrap.php">
    <php>
        <env name="KERNEL_CLASS" value="Sylius\TestApplication\Kernel" />
        <env name="APP_ENV" value="test"/>
    </php>
    <!-- Your test configuration -->
</phpunit>
```

### 7. Update .gitignore

Add TestApplication local files to .gitignore:

```gitignore
/tests/TestApplication/.env.local
/tests/TestApplication/.env.test.local
/tests/TestApplication/.env.*.local
```

### 8. Update GitHub Actions Workflow

Update `.github/workflows/build.yml`:

**Key Changes:**
- Use `vendor/bin/console` instead of `bin/console`
- Use migrations instead of `doctrine:schema:create`
- Move Chrome and server startup to just before Behat

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

# ... other testing steps (ECS, PHPStan, PHPUnit, etc.) ...

- name: Run Chrome Headless
  run: google-chrome-stable --enable-automation --disable-background-networking --no-default-browser-check --no-first-run --disable-popup-blocking --disable-default-apps --allow-insecure-localhost --disable-translate --disable-extensions --no-sandbox --enable-features=Metal --headless --remote-debugging-port=9222 --window-size=2880,1800 --proxy-server='direct://' --proxy-bypass-list='*' http://127.0.0.1 > /dev/null 2>&1 &

- name: Run webserver
  run: symfony server:start --port=8080 --daemon && symfony -V

```

### 9. Fix Asset Compilation Issues

If @vendor alias doesn't work, use relative paths. Future TestApplication versions will have @vendor alias support.

**Important**: Don't modify vendor files - use relative paths instead.

### 10. Complete Migration Process

Follow these steps in the correct order:

1. **Install dependencies**:
   ```bash
   composer install
   ```

2. **Create database**:
   ```bash
   APP_ENV=test vendor/bin/console doctrine:database:create
   ```

3. **Run migrations**:
   ```bash
   APP_ENV=test vendor/bin/console doctrine:migrations:migrate --no-interaction
   ```

4. **Install yarn dependencies** (in TestApplication directory):
   ```bash
   yarn --cwd vendor/sylius/test-application install
   ```

5. **Install assets**:
   ```bash
   APP_ENV=test vendor/bin/console assets:install -vvv
   ```

6. **Change symlink node_modules to new path**:
   ```bash
   rm -f node_modules
   ln -sf vendor/sylius/test-application/node_modules node_modules
   ```

7. **Build assets** (in TestApplication directory):
   ```bash
   yarn --cwd vendor/sylius/test-application build
   ```

8. **Start server in test environment**:
   ```bash
   APP_ENV=test symfony server:start --port=8080 --daemon
   ```

## MANDATORY TESTING PHASE

**CRITICAL**: Before removing old folders, ALL tests must pass. This is mandatory and scope cannot be changed.

### Test Each Component Systematically

Run tests in this exact order and ensure ALL pass:

1. **ECS (Easy Coding Standard)**:
   ```bash
   vendor/bin/ecs check
   ```

2. **PHPStan (Static Analysis)**:
   ```bash
   vendor/bin/phpstan analyse
   ```

3. **PHPUnit (Unit Tests)**:
   ```bash
   APP_ENV=test vendor/bin/phpunit
   ```

4. **Behat (Acceptance Tests)**:
   ```bash
   APP_ENV=test vendor/bin/behat --no-interaction
   ```

**Important**: If ANY test fails, you must fix the issue before proceeding. Do not delete old folders until ALL tests pass.

### 11. Remove Old Files and Directories

**ONLY after ALL tests pass**, remove old files:

```bash
# Remove old test application
rm -rf tests/Application

# Remove node symlink script (if exists)
rm -f bin/create_node_symlink.php
```

## Common Issues and Solutions

### Container Build Failures

**Issue**: Container fails to build after migration

**Root Causes and Solutions:**
1. **Missing bundles in bundles.php**: Package which you have in composer.json should be in bundles.php
2. **Duplicate configuration imports**: In some way you import twice state machine configuration - remove duplicate imports
3. **Service configuration syntax**: Use proper PHP configurator syntax with `service()` and `expr()` functions

### Service Organization Issues

**Issue**: Services in wrong files

**Solution**: 
- Main application services go in `services.yaml`
- Test-specific services go in `services_test.php`

### Database Configuration Issues

**Issue**: Database connection problems

**Solution**: Always ask about database configuration and create proper `.env.local` files with user-specific settings.

### Configuration Bloat

**Issue**: Too much unnecessary configuration

**Solution**: 
- Check TestApplication package configuration and remove duplicates
- Remove configurations like `sylius_shop`, `sylius_api`, `framework`, `sylius_state_machine_abstraction`, `sylius_twig_hooks`, `fos_elastica` that are provided by core
- Keep only plugin-specific configuration

### Environment File Organization

**Issue**: Huge mess in env files

**Solution**: 
- Structure `.env` and `.env.test` with proper sections
- Keep default values in `.env` and `.env.test`
- Put customizations in `.env.local` and `.env.test.local`
- Follow the pattern: `###> package-name ###` sections

## Post-Migration Checklist

- [ ] All composer dependencies registered in bundles.php
- [ ] No duplicate configuration imports
- [ ] Services properly organized (services.yaml vs services_test.php)
- [ ] Database configuration asked and configured
- [ ] Environment files properly structured with sections
- [ ] Webpack builds without vendor file modifications
- [ ] Templates directory exists with .gitkeep
- [ ] **ECS passes**
- [ ] **PHPStan passes**
- [ ] **PHPUnit passes**
- [ ] **Behat passes (all directories)**
- [ ] GitHub Actions workflow updated
- [ ] `.gitignore` updated for local env files
- [ ] Old `tests/Application` directory removed
- [ ] `scripts` section removed from composer.json

## Benefits of Migration

- ✅ Eliminates custom test application maintenance
- ✅ Standardized testing environment across all plugins
- ✅ Reduced repository size and complexity
- ✅ No more node symlink management
- ✅ Automatic updates with TestApplication releases
- ✅ Simplified composer.json configuration
- ✅ Modern environment file organization

## Critical Success Factors

1. **Bundle Registration**: All composer dependencies must be in bundles.php
2. **Configuration Cleanup**: Remove duplicates, check TestApplication package config
3. **Service Organization**: Separate main services from test-specific services
4. **Environment Setup**: Ask about database config, structure env files properly
5. **Asset Compilation**: Fix webpack paths without modifying vendor files
6. **Mandatory Testing**: ALL tests must pass before removing old folders
7. **Systematic Approach**: Test each directory individually to identify issues

---

*This guide is based on successful migrations of Sylius plugins from tests/Application to TestApplication. All challenges encountered during migrations are documented with their solutions. The mandatory testing phase ensures a reliable migration process.*
