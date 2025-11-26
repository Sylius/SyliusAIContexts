# Shop: Menu Migration

This step updates shop menu configuration for Sylius 2.0, including icon migration to Tabler.

**When to skip this step:**
- Your plugin doesn't add items to shop menus
- You don't have any shop menu builders, listeners, or event subscribers

**When to do this step:**
- You use EventSubscriber or Listener to add shop menu items
- You have shop menu customizations
- You use icons in shop menu items

## 1. Add Active Routes (Optional but Recommended)

To make menu items stay active on related pages:

```php
$item
    ->addChild('resource', [
        'route' => 'plugin_shop_resource_index',
        'extras' => [
            'routes' => [
                ['route' => 'plugin_shop_resource_show'],
            ],
        ],
    ])
    ...
    ;
```

## 2. Update Menu Icons to Tabler

If your menu sets icon via `setLabelAttribute('icon', ...)`, update from Semantic UI to Tabler format:

**Before (Sylius 1.x with Semantic UI):**
```php
->setLabelAttribute('icon', 'check circle outline');
```

**After (Sylius 2.0 with Tabler):**
```php
->setLabelAttribute('icon', 'tabler:circle-check');
```

**Tabler icon format:** `'tabler:{icon-name}'`

**Search for icons:**
- https://ux.symfony.com/icons (recommended - search by keyword)

## 3. Validate

Clear cache and check if menu displays correctly:

```bash
vendor/bin/console cache:clear
```

Access your shop and verify:
- Menu items appear in correct location
- Icons display correctly (Tabler icons)
- Menu items are active on correct pages

---

**Important Notes:**
- Event names in EventSubscriber/Listener (e.g., `sylius.menu.shop.account`) **do not change**
- Shop menu events are simpler than admin (no removed events in shop)
