# Menu Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
This step is OPTIONAL. Execute only if the plugin adds menu items to Sylius admin/shop menus.

## Commands

### 1. Check for Removed Menu Events

```
Grep: "sylius.menu.admin.customer.show|sylius.menu.admin.order.show|sylius.menu.admin.product.form|sylius.menu.admin.product.update|sylius.menu.admin.product_variant.form|sylius.menu.admin.promotion.update" --include="*.php"
```

If any of these events are found, they were removed in Sylius 2.0. Note the files for manual migration to Twig Hooks (will be covered in Template Migration step).

### 2. Find Menu Event Subscribers/Listeners

```
Grep: "sylius.menu.admin.main|sylius.menu.shop.account" --include="*.php"
```

This finds EventSubscribers and Listeners that add menu items.

### 3. Update Menu Icons to Tabler

For each file found, search for `setLabelAttribute('icon'`:

```
Read {menu_file}
Grep: "setLabelAttribute\('icon'" path={menu_file} output_mode="content" -n=true
```

Update Semantic UI icons to Tabler format:

```
Edit {menu_file}:
Replace icon values:
- 'check circle outline' → 'tabler:circle-check'
- 'inbox' → 'tabler:inbox'
- 'images outline' → 'tabler:photo'
- 'adversal' → 'tabler:ad'
- 'external alternate' → 'tabler:layout-grid'
- 'shopping cart' → 'tabler:shopping-cart'
- 'users' → 'tabler:users'
- 'tags' → 'tabler:tags'
```

Format: `'tabler:{icon-name}'`

For finding icon names, search at: https://ux.symfony.com/icons

### 4. Add Active Routes (Optional)

If menu items should stay active on related pages (e.g., create/update), add `extras` configuration:

```
Edit {menu_file}:
In the addChild() call, add extras array:
'extras' => [
    'routes' => [
        ['route' => '{plugin_prefix}_admin_{resource}_create'],
        ['route' => '{plugin_prefix}_admin_{resource}_update'],
    ],
],
```

Example:
```php
$item->addChild('resource', [
    'route' => 'your_plugin_admin_resource_index',
    'extras' => [
        'routes' => [
            ['route' => 'your_plugin_admin_resource_create'],
            ['route' => 'your_plugin_admin_resource_update'],
        ],
    ],
]);
```

### 5. Validate

```
Bash: vendor/bin/console cache:clear 2>&1
```

Expected: Cache clears successfully without errors.

Check that menu displays correctly in admin panel with Tabler icons.

## Success Criteria
- Removed menu events identified (if any)
- Semantic UI icons replaced with Tabler format (`tabler:{icon-name}`)
- Active routes added for better UX (optional)
- Cache clears successfully
- Menu displays correctly with new icons

## Notes for AI
- This step is OPTIONAL - skip if plugin doesn't add menu items
- Event names (`sylius.menu.admin.main`) DO NOT change
- Only icon format changes: Semantic UI → Tabler
- Main menu events still work: `sylius.menu.admin.main`, `sylius.menu.shop.account`
- Removed events (customer.show, order.show, product.form, etc.) must be migrated to Twig Hooks
- Can be implemented via EventSubscriber or Listener (kernel.event_listener tag)
- Active routes in extras are optional but improve UX
