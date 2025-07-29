# Step 5: Test Configuration Updates

Update Behat and PHPUnit configurations to use TestApplication instead of the old test application.

## 1. Update behat.yml.dist

Update the Behat configuration to use TestApplication:

```yaml
imports:
    - vendor/sylius/sylius/src/Sylius/Behat/Resources/config/suites.yml
    - tests/Behat/Resources/suites.yml

default:
    extensions:
        DMore\ChromeExtension\Behat\ServiceContainer\ChromeExtension: ~

        FriendsOfBehat\MinkDebugExtension:
            directory: etc/build
            clean_start: false
            screenshot: true

        Behat\MinkExtension:
            files_path: "%paths.base%/vendor/sylius/sylius/src/Sylius/Behat/Resources/fixtures/"
            base_url: "https://127.0.0.1:8080/"
            default_session: symfony
            javascript_session: chrome
            sessions:
                symfony:
                    symfony: ~
                chrome:
                    chrome:
                        api_url: http://127.0.0.1:9222
                        validate_certificate: false
                panther:
                    panther:
                        options:
                            webServerDir: '%paths.base%/vendor/sylius/test-application/public' # if using Symfony Panther
            show_auto: false

        FriendsOfBehat\SymfonyExtension:
            bootstrap: vendor/sylius/test-application/config/bootstrap.php
            kernel:
                class: Sylius\TestApplication\Kernel

        FriendsOfBehat\VariadicExtension: ~

        FriendsOfBehat\SuiteSettingsExtension:
            paths:
                - "features"
```

**Key changes:**
- `bootstrap: vendor/sylius/test-application/config/bootstrap.php`
- `class: Sylius\TestApplication\Kernel`
- `webServerDir: '%paths.base%/vendor/sylius/test-application/public'` (if using Panther)

## 2. Update phpunit.xml.dist

Update PHPUnit configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://schema.phpunit.de/9.5/phpunit.xsd"
         colors="true"
         bootstrap="vendor/sylius/test-application/config/bootstrap.php">
    <testsuites>
        <testsuite name="Test Suite">
            <directory>tests</directory>
        </testsuite>
    </testsuites>

    <php>
        <ini name="error_reporting" value="-1" />

        <server name="KERNEL_CLASS_PATH" value="/tests/Application/AppKernel.php" />
        <server name="IS_DOCTRINE_ORM_SUPPORTED" value="true" />

        <env name="KERNEL_CLASS" value="Sylius\TestApplication\Kernel" />
        <env name="APP_ENV" value="test"/>
        <env name="SHELL_VERBOSITY" value="-1" />
    </php>
</phpunit>
```

**Key changes:**
- `bootstrap="vendor/sylius/test-application/config/bootstrap.php"`
- `<env name="KERNEL_CLASS" value="Sylius\TestApplication\Kernel" />`
- `<env name="APP_ENV" value="test"/>`

## Configuration Notes

### Behat Configuration
- Ensure all session configurations point to the correct TestApplication paths
- Keep existing extension configurations that work with your plugin
- The base URL should match your test server configuration

### PHPUnit Configuration
- Bootstrap path must point to TestApplication
- Kernel class must be `Sylius\TestApplication\Kernel`
- Environment variables should be set for test environment

### Common Issues
- Verify that the bootstrap path exists after installing dependencies
- Ensure the Kernel class is available in the TestApplication package
- Check that test suites can find your test files

## Completion Checklist

Verify you have updated:
- [ ] `behat.yml.dist` with TestApplication bootstrap and kernel
- [ ] `phpunit.xml.dist` with TestApplication configuration
- [ ] All paths point to `vendor/sylius/test-application/`
- [ ] Environment variables are set correctly

## Commit Changes

After updating both configuration files:

```bash
git add behat.yml.dist phpunit.xml.dist
git commit -m "feat: update test configurations for testApplication

- Update behat.yml.dist to use TestApplication bootstrap and kernel
- Configure phpunit.xml.dist for TestApplication environment
- Set correct paths for test application assets and configuration"
```

## Next Step

Continue to [github-actions.md](github-actions.md) to update your CI/CD workflow.