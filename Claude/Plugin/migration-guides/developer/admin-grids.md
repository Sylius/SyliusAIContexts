# Admin: Grid Migration

## When to Execute

This step is OPTIONAL. Execute only if your plugin defines admin grids.

## Overview

Grid definitions remain largely the same in Sylius 2.0. Main changes:
- Update Sylius template paths (`@SyliusAdmin/Grid/...` → `@SyliusAdmin/shared/grid/...`)
- Verify template paths use lowercase (`Grid/Field` → `grid/field`)
- Replace deprecated `entities` filter with `entity`
- Update icon names (Semantic UI → Tabler)

## 1. Update Sylius Template Paths

Sylius 2.0 moved templates from `@SyliusAdmin/Grid/...` to `@SyliusAdmin/shared/grid/...`.

**Skip this if:**
- Your plugin doesn't reference Sylius grid templates
- All your grid templates are custom (from your own plugin)

### Update in both YAML grids and PHP Extension:

**Before (Sylius 1.x):**
```yaml
# In YAML (config/grids/*.yaml)
sylius_grid:
    grids:
        your_plugin_admin_resource:
            fields:
                image:
                    type: twig
                    options:
                        template: "@SyliusAdmin/Grid/Field/image.html.twig"
```

```php
// In PHP Extension (src/DependencyInjection/*Extension.php)
$container->prependExtensionConfig('sylius_grid', [
    'grids' => [
        'your_plugin_admin_resource' => [
            'fields' => [
                'image' => [
                    'type' => 'twig',
                    'options' => [
                        'template' => '@SyliusAdmin/Grid/Field/image.html.twig',
                    ],
                ],
            ],
        ],
    ],
]);
```

**After (Sylius 2.0):**
```yaml
# In YAML
template: "@SyliusAdmin/shared/grid/field/image.html.twig"
```

```php
// In PHP Extension
'template' => '@SyliusAdmin/shared/grid/field/image.html.twig',
```

**Common patterns to update:**
- `@SyliusAdmin/Grid/Field/...` → `@SyliusAdmin/shared/grid/field/...`
- `@SyliusAdmin/Grid/Action/...` → `@SyliusAdmin/shared/grid/action/...`
- `@SyliusAdmin/Grid/Filter/...` → `@SyliusAdmin/shared/grid/filter/...`

**Note:** Custom plugin templates like `@YourPlugin/admin/grid/field/custom.html.twig` don't need changes.

## 2. Verify Template References

After updating paths, verify that all referenced templates actually exist in Sylius 2.0 and have correct casing.

**Common issues:**

### Incorrect Casing

Sylius 2.0 uses lowercase paths. Fix any capital letters:

```diff
# Before (will fail)
- template: "@SyliusUi/Grid/Field/enabled.html.twig"

# After (correct)
+ template: "@SyliusUi/grid/field/enabled.html.twig"
```

**Common patterns:**
- `@SyliusUi/Grid/Field/` → `@SyliusUi/grid/field/`
- `@SyliusAdmin/Grid/` → `@SyliusAdmin/grid/` or `@SyliusAdmin/shared/grid/`

### Template No Longer Exists

Some Sylius templates may have been removed in 2.0. To check if a template exists:

```bash
find vendor/sylius -name "your_template.twig"
```

If not found, create a custom template in your plugin.

## 3. Restore Custom Grid Templates

If your grid configuration references custom templates, restore them from backup with proper `snake_case` naming.

**Important:** Directory names must use `snake_case` in Sylius 2.0:
- `Admin/Grid/Field` → `admin/grid/field`
- `Admin/Resource/Grid` → `admin/resource/grid`

**Steps:**

1. Check your grid configuration for custom template references in `config/grids/*.yaml` or `src/DependencyInjection/*Extension.php`

2. Find templates in `old_templates/` and restore them with `snake_case` paths:
   ```bash
   # Example: if old template was in CamelCase
   # Old: old_templates/Admin/Grid/Field/channels.html.twig
   # New: templates/admin/grid/field/channels.html.twig

   mkdir -p templates/admin/grid/field
   cp old_templates/Admin/Grid/Field/channels.html.twig templates/admin/grid/field/channels.html.twig
   ```

3. Update grid configuration to use `snake_case` paths:
   ```diff
   - template: '@YourPlugin/Admin/Grid/Field/channels.html.twig'
   + template: '@YourPlugin/admin/grid/field/channels.html.twig'
   ```

## 4. Update Deprecated `entities` Filter

The experimental `entities` filter has been removed. Replace it with `entity`:

```diff
sylius_grid:
    grids:
        your_plugin_admin_resource:
            filters:
                channels:
-                   type: entities
+                   type: entity
                    options:
-                       field: "channels.id"
+                       fields: [channels.id]
```

**Key changes:**
- Filter type: `entities` → `entity`
- Option: `field: "value"` → `fields: [value]` (string to array)

## 5. Update Icons to Tabler Format

Update icon format to use `tabler:` prefix:

```diff
- icon: pencil
+ icon: tabler:pencil
```

Find icon names at: https://ux.symfony.com/icons

## 6. Validate

Clear cache and check if grids work:

```bash
vendor/bin/console cache:clear
```

Visit your admin grid pages and verify:
- Grid displays correctly
- Filters work (especially if you changed `entities` → `entity`)
- Actions have correct Tabler icons
- Pagination works

## Common Issues

### Template not found errors

**Problem:** Error about missing `@SyliusAdmin/Grid/...` or `@SyliusAdmin/shared/grid/...` templates.

**Solution:** Most Sylius grid templates still exist in the new version at `@SyliusAdmin/shared/grid/...`, but some were removed. If a template was removed, you need to create your own.

**Recommended path for custom templates:**
```
templates/{resource}/grid/{type}/{name}.html.twig
```

Examples:
- `templates/catalog_promotion/grid/action/delete.html.twig`
- `templates/product/grid/field/image.html.twig`
- `templates/order/grid/filter/channel.html.twig`

Replace `{resource}`, `{type}` (action/field/filter), and `{name}` with your specific values.
