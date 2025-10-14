# Shop Templates: Overview & Decision Guide - AI

This guide helps AI decide which shop template migration approach to use.

## Two Approaches

**A. Extend Existing Sylius Hooks** - Add content to existing pages (checkout, cart, product)
- Use when: Adding small features to Sylius pages
- Example: Add custom checkbox to checkout

**B. Custom Hooks for Custom Pages** - Create new pages with own hooks (RECOMMENDED)
- Use when: Creating new pages in your plugin
- Example: Resource show page, custom listings
- Execute: `shop-templates-hooks.md`

## Decision Tree for AI

```
Does plugin have shop templates?
├─ NO → Skip this step
└─ YES → Check template type
    │
    Are these custom pages (new pages in plugin)?
    ├─ YES → Execute shop-templates-hooks.md
    │
    └─ NO → Extending existing Sylius pages?
        ├─ YES → Use approach A (simple extension, no separate guide needed)
        └─ UNCLEAR → Execute shop-templates-hooks.md (safer default)
```

## Detection Commands

### Check if plugin has shop templates

```bash
Tool: Glob
Pattern: **/*.twig
Path: templates/shop/
```

If templates found → Proceed

### Detect template type

```bash
Tool: Grep
Pattern: extends '@SyliusShop
Path: templates/shop/
Output: content
```

Look at extends:
- `@SyliusShop/shared/layout/base.html.twig` → Custom page (use approach B)
- `@SyliusShop/checkout/` or similar → Extending existing (use approach A)

### Check if hook configuration exists

```bash
Tool: Glob
Pattern: **/*.yaml
Path: config/twig_hooks/shop/
```

If exists → Already migrated or partially migrated

## Execution Order

### For Custom Pages (Most Common)

1. Execute: `shop-templates-hooks.md`
   - Creates hook configuration
   - Creates section templates
   - Migrates to Bootstrap 5

### For Extending Existing Pages (Less Common)

1. Find existing Sylius hooks:
```bash
Tool: Bash
Command: grep -r "sylius_twig_hooks:" vendor/sylius/sylius/src/Sylius/Bundle/ShopBundle/config/twig_hooks/ | grep -i "{page_name}"
```

2. Create hook configuration to extend those hooks (manual, no separate guide)

## Files to Create (Custom Pages)

```
config/
└── twig_hooks/shop/{page}/
    └── show.yaml (or index.yaml, etc.)
templates/shop/{page}/
├── show.html.twig (main template)
└── show/
    ├── content.html.twig
    └── content/
        ├── header.html.twig
        └── main.html.twig
```

## Validation

After `shop-templates-hooks.md`:

```bash
Tool: Bash
Command: vendor/bin/console cache:clear && vendor/bin/console debug:twig-hooks | grep -i "shop"
```

Expected: Your shop hooks should appear in the list.

## Common Patterns

### Custom page main template

```twig
{% extends '@SyliusShop/shared/layout/base.html.twig' %}

{% block content %}
    {% hook 'plugin.shop.{page}.{action}' with { resource: resource } %}
{% endblock %}
```

### Hook naming for custom pages

- `plugin.shop.{page}.{action}` - Top level
- `plugin.shop.{page}.{action}.content` - Content wrapper
- `plugin.shop.{page}.{action}.content.header` - Header section
- `plugin.shop.{page}.{action}.content.main` - Main section

### Extending existing Sylius pages

Find hook like: `sylius_shop.checkout.complete.content.form`

Add your content:
```yaml
'sylius_shop.checkout.complete.content.form':
    your_feature:
        template: '@Plugin/shop/checkout/your_feature.html.twig'
        priority: 50
```

## Summary

- **Default:** Create custom hooks for custom pages
- **Extending:** Find Sylius hooks and inject content
- **No Live Components:** Shop is usually static content
- **Bootstrap 5:** Use modern classes
- **Validate:** After hooks created, ensure no errors
