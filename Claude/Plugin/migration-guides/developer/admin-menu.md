# Admin: Menu Migration

This step updates menu configuration for Sylius 2.0, including icon migration to Tabler and handling removed menu events.

**When to skip this step:**
- Your plugin doesn't add items to Sylius admin menu
- You don't have any menu builders, listeners, or event subscribers

**When to do this step:**
- You use EventSubscriber or Listener to add menu items
- You have custom menu builders
- You use icons in menu items

## 1. Check for Removed Menu Events

The following menu events were **removed** in Sylius 2.0 and replaced by Twig Hooks:

- `sylius.menu.admin.customer.show`
- `sylius.menu.admin.order.show`
- `sylius.menu.admin.product.form`
- `sylius.menu.admin.product.update`
- `sylius.menu.admin.product_variant.form`
- `sylius.menu.admin.promotion.update`

**If you use any of these events:**
- Remove the event subscriber/listener
- Migrate functionality to Twig Hooks (will be covered in Templates step)

**Main menu events still work:**
- `sylius.menu.admin.main` ✅
- `sylius.menu.shop.account` ✅

## 2. Add Active Routes (Optional but Recommended)

To make menu items stay active on related pages (e.g., edit/create pages), add `extras` with routes:

```php
$item
    ->addChild('resource', [
        'route' => 'plugin_admin_resource_index',
        'extras' => [
            'routes' => [
                ['route' => 'plugin_admin_resource_create'],
                ['route' => 'plugin_admin_resource_update'],
            ],
        ],
    ])
    ...
    ;
```

## 3. Update Menu Icons to Tabler

If your menu listener/subscriber/builder sets icon via `setLabelAttribute('icon', ...)`, update from Semantic UI to Tabler format:

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

## 4. Validate

Clear cache and check if menu displays correctly:

```bash
vendor/bin/console cache:clear
```

Access your admin panel and verify:
- Menu items appear in correct location
- Icons display correctly (Tabler icons)
- Menu items are active on correct pages

---

**Important Notes:**
- Event names in EventSubscriber/Listener (`sylius.menu.admin.main`) **do not change**
- Removed menu events must be replaced with Twig Hooks
