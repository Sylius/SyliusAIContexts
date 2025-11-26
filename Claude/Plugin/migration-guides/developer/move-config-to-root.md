# Move Config to Root (Optional)

Starting from Sylius 2.0, it's recommended to move all configuration, translations, and templates from `src/Resources/` to the root `config/`, `templates/`, and `translations/` directories — following the standard Symfony 7+ structure.

**About the old structure:**
The `src/Resources/` directory structure is a legacy approach that is still supported by both Symfony 7 and Sylius 2.0, but it's not recommended for modern development.

**When to skip this step:**
- If you don't have a `src/Resources/` directory
- If you prefer to keep the legacy structure (still works, but not recommended)

**When to do this step:**
- You want to follow modern Symfony 7+ conventions
- You're planning long-term maintenance and upgrades

## 1. Move Configuration Files

Move all configuration files from `src/Resources/config/` to `config/`:

```bash
# Example structure to create
mkdir -p config/{routes,services,validation,doctrine,grids,api_resources,twig_hooks}

# Move files (adjust paths based on your plugin)
mv src/Resources/config/* config/
```

**Note:** Preserve the directory structure. For example:
- `src/Resources/config/routes/admin.yaml` → `config/routes/admin.yaml`
- `src/Resources/config/doctrine/model/*.xml` → `config/doctrine/model/*.xml`

## 2. Move Translations

Move translation files:

```bash
mv src/Resources/translations/* translations/
```

## 3. Move Templates

Move template files:

```bash
mv src/Resources/views/* templates/
```

**Note:** Template naming conventions (lowercase folders, snake_case files) will be addressed in a later step.

## 4. Update Plugin Class

Add the `getPath()` method to your Plugin class (e.g., `src/YourPlugin.php`):

```php
public function getPath(): string
{
    return \dirname(__DIR__);
}
```

This tells Symfony where to find your plugin's root directory.

## 5. Update Extension Class

Update the FileLocator path in your Extension class (e.g., `src/DependencyInjection/YourExtension.php`):

```diff
- $loader = new XmlFileLoader($container, new FileLocator(__DIR__ . '/../Resources/config'));
+ $loader = new XmlFileLoader($container, new FileLocator(__DIR__ . '/../../config'));
```

## 6. Update Resource Paths in Config Files

Update all references from old paths to new paths:

**In your plugin's config files:**
```diff
- resource: "@YourPlugin/Resources/config/routes/admin.yaml"
+ resource: "@YourPlugin/config/routes/admin.yaml"
```

**In test application config:**
```diff
- { resource: "@YourPlugin/Resources/config/config.yaml" }
+ { resource: "@YourPlugin/config/config.yaml" }
```

Search for all occurrences:
```bash
grep -r "Resources/config" config/
grep -r "Resources/config" tests/TestApplication/config/
```

**In test application .env and .env.test:**

Update to use bundle notation instead of relative paths in both files:

```env
# Old (relative paths):
SYLIUS_TEST_APP_CONFIGS_TO_IMPORT="../../../../tests/TestApplication/config/config.yaml"
SYLIUS_TEST_APP_ROUTES_TO_IMPORT="../../../../tests/TestApplication/config/routes.yaml"

# New (bundle notation):
SYLIUS_TEST_APP_CONFIGS_TO_IMPORT="@YourPlugin/tests/TestApplication/config/config.yaml"
SYLIUS_TEST_APP_ROUTES_TO_IMPORT="@YourPlugin/tests/TestApplication/config/routes.yaml"
```

Replace `YourPlugin` with your actual plugin name (e.g., `@AcmeMyAwesomePlugin`).

Apply these changes to both `.env` and `.env.test` files.

## 7. Clean Up Old Directory

After moving all files:

- If `src/Resources/public/` still exists - **leave it** (will be moved in Asset Migration step)
- If `src/Resources/` is now empty - delete it:
  ```bash
  rm -rf src/Resources/
  ```

## 8. Clear Cache and Validate

```bash
vendor/bin/console cache:clear
```

Check if the application works correctly.

---

## Troubleshooting

### Doctrine Mapping Error: "invalid directory path"

**Symptom:**
```
File mapping drivers must have a valid directory path,
however the given path [.../Resources/config/doctrine/model] seems to be incorrect!
```

**When this happens:**
- Your plugin extends `AbstractResourceBundle`
- After moving files, Doctrine still looks in the old `Resources/config/doctrine/model/` path

**Why:**
`AbstractResourceBundle` has a hardcoded path to `Resources/config/doctrine/model/`. When you move files to `config/`, you need to override this.

**Solution:**

Add the `getConfigFilesPath()` method to your Plugin class:

```php
protected function getConfigFilesPath(): string
{
    return sprintf(
        '%s/config/doctrine/%s',
        $this->getPath(),
        strtolower($this->getDoctrineMappingDirectory()),
    );
}
```

**Full example:**

```php
final class YourPlugin extends AbstractResourceBundle
{
    use SyliusPluginTrait;

    public function getSupportedDrivers(): array
    {
        return [
            SyliusResourceBundle::DRIVER_DOCTRINE_ORM,
        ];
    }

    public function getPath(): string
    {
        return \dirname(__DIR__);
    }

    protected function getConfigFilesPath(): string
    {
        return sprintf(
            '%s/config/doctrine/%s',
            $this->getPath(),
            strtolower($this->getDoctrineMappingDirectory()),
        );
    }
}
```
