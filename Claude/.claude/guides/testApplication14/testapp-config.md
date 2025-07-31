# Step 2: TestApplication Configuration

**IMPORTANT**: This step must be completed BEFORE migrating custom content (now Step 4). You need the configuration structure in place before copying custom content into it.

Set up the core TestApplication configuration files that define bundles, services, and routing.

## 1. Create tests/TestApplication/config/bundles.php

**Critical**: All composer dependencies must be registered in bundles.php.

```php
<?php

declare(strict_types=1);

$bundles = [
    // Register ALL plugin dependencies from composer.json (require and require-dev)
    // which provide Symfony Bundles and are NOT present in sylius/test-application/config/bundles.php

    // Always register your plugin's bundle:
    YourVendor\YourPlugin\YourPluginBundle::class => ['all' => true],
 
    // Add other plugin dependencies as needed:
    // Vendor\Dependency\SomeOtherBundle::class => ['all' => true],
];

return $bundles;
```

## 2. Create tests/TestApplication/config/config.yaml

**Check TestApplication package configuration and remove duplicates**.

**NOTE**: In Step 4 (custom content migration), you will add any custom configuration from your old Application to this file. Keep all configuration consolidated in this single config.yaml file.

```yaml
imports:
    - { resource: "@YourPlugin/config/config.yaml" } # or "@YourPlugin/Resources/config/config.yaml" if using old structure
    - { resource: "services_test.php" }

# Only include plugin-specific configuration that differs from core Sylius
# Remove duplicates like sylius_shop, sylius_api, framework, etc.

# Add your plugin-specific sylius resource mappings overrides here if needed
# sylius_resource:
#     resources:
#         your_resource:
#             classes:
#                 model: Tests\YourPlugin\Entity\YourResource
#                 repository: YourPlugin\Repository\YourResourceRepository

# IMPORTANT: Only add the doctrine mapping section below if you have custom entities!
# If you don't have custom entities, remove the entire doctrine section.
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

# If the plugin provides its own API Platform configuration in a directory other than api_resources
# parameters:
#    your_plugin_api_platform_mapping_path: '%kernel.project_dir%/../../../config/api_platform/'

# api_platform:
#     mapping:
#         paths:
#             - '%your_plugin_api_platform_mapping_path%'
```

## 3. Create tests/TestApplication/config/routes.yaml

```yaml
your_plugin:
    resource: "@YourPlugin/config/routes.yaml"
```

## 4. Create tests/TestApplication/config/services_test.php

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

## Important Configuration Notes

### Bundle Registration
- Package which you have in composer.json should be in bundles.php
- Only register bundles that are NOT already in the base TestApplication

### Service Configuration
- If service configuration files (e.g., services.yaml) are already imported in the TestApplication package, do not import them again
- If a services.yaml file contains only comments and no actual service definitions, do not create that file

### Doctrine Configuration
- Only include doctrine mapping if you have custom entities
- Choose between attribute or XML mapping based on your entities
- Adjust paths and prefixes according to your namespace structure

## Completion Checklist

Verify you have created:
- [ ] `bundles.php` with all necessary bundles registered
- [ ] `config.yaml` with plugin-specific configuration
- [ ] `routes.yaml` with plugin route imports
- [ ] `services_test.php` with test-specific services
- [ ] Doctrine mapping (only if you have custom entities)

## Commit Changes

After creating all configuration files:

```bash
git add tests/TestApplication/config/
git commit -m "feat: configure testApplication files

- Add bundles.php with plugin bundle registration
- Configure config.yaml with plugin-specific settings
- Set up routing configuration
- Add test-specific service configuration"
```

## Next Step

Continue to [database-config.md](database-config.md) to configure database connection.