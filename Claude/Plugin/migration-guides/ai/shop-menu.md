# Shop: Menu Migration - AI Guide

This guide provides Tool commands to migrate shop menu to Sylius 2.0.

## Step 1: Find shop menu listeners/subscribers

```bash
Tool: Grep
Pattern: sylius\.menu\.shop
Path: src/
Output: files_with_matches
```

If no matches found → Skip this step

## Step 2: Find icons to update

For each file found in Step 1:

```bash
Tool: Grep
Pattern: setLabelAttribute\('icon'
Path: {file_from_step1}
Output: content
Lines: -B 2 -A 2
```

## Step 3: Update icons to Tabler

For each icon found:

```bash
Tool: Edit
File: {file}
Old: ->setLabelAttribute('icon', '{old_semantic_ui_icon}')
New: ->setLabelAttribute('icon', 'tabler:{tabler_icon_name}')
```

**Common icon conversions:**
- `'check circle outline'` → `'tabler:circle-check'`
- `'user'` → `'tabler:user'`
- `'shopping cart'` → `'tabler:shopping-cart'`
- `'heart'` → `'tabler:heart'`
- `'list'` → `'tabler:list'`

Search icons at: https://ux.symfony.com/icons

## Step 4: Add active routes (optional)

For better UX, add active routes configuration:

```bash
Tool: Edit
File: {menu_file}
Old: ->addChild('{item_name}', [
        'route' => '{route_name}',
    ])
New: ->addChild('{item_name}', [
        'route' => '{route_name}',
        'extras' => [
            'routes' => [
                ['route' => '{related_route_1}'],
                ['route' => '{related_route_2}'],
            ],
        ],
    ])
```

## Step 5: Validate

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

Expected: Cache clears successfully, no errors.

## Summary

After execution:
- ✅ Icons updated to Tabler format
- ✅ Active routes added (optional)
- ✅ Cache cleared
- ✅ No deprecated icon formats remain

## Notes

- Shop menu events (`sylius.menu.shop.account`, etc.) do NOT change
- No menu events were removed in shop (unlike admin)
- Focus is mainly on icon updates
