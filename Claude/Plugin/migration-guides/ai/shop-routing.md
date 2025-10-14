# Shop: Routing Migration - AI Guide

This guide provides Tool commands to migrate shop routing configuration to Sylius 2.0.

## Step 1: Check if shop routing exists

```bash
Tool: Read
File: config/routes/shop.yaml
```

If file doesn't exist → Skip this step

## Step 2: Update template paths

```bash
Tool: Edit
File: config/routes/shop.yaml
Old: templates: "@SyliusShop
New: templates: "@SyliusShop\shared
```

This updates old `@SyliusShop\product` to `@SyliusShop\shared\product`

## Step 3: Remove deprecated vars

If `vars.all.subheader` exists:

```bash
Tool: Edit
File: config/routes/shop.yaml
Old: subheader: (any value)
(delete entire line)
```

## Step 4: Add hook_prefix if using custom templates

If routing uses custom templates and will use Twig Hooks:

```bash
Tool: Edit
File: config/routes/shop.yaml
Old: vars:
            all:
New: vars:
            all:
                hook_prefix: 'plugin.shop.{resource}'
```

Replace `{resource}` with actual resource name from `alias:` value.

## Step 5: Validate

```bash
Tool: Bash
Command: vendor/bin/console debug:router | grep shop
```

Expected: Shop routes should load without errors.

## Summary

After execution:
- ✅ Template paths updated to `@SyliusShop\shared\*`
- ✅ Deprecated `subheader` removed
- ✅ Hook prefix added (if needed)
- ✅ Routes validated
