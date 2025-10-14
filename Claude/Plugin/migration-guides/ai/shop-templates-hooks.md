# Shop Templates: Create Custom Hooks - AI Guide

This guide provides Tool commands to create custom Twig Hooks for shop pages.

## Prerequisites

Shop pages exist in `templates/shop/`

## Step 1: Check if twig_hooks config is imported

```bash
Tool: Read
File: config/config.yaml
```

Look for `twig_hooks/**/*.yaml` in imports. If not found, add it:

```bash
Tool: Edit
File: config/config.yaml
Old: imports:
New: imports:
    - { resource: "twig_hooks/**/*.yaml" }
```

## Step 2: Identify shop pages

```bash
Tool: Glob
Pattern: *.twig
Path: templates/shop/
```

For each page found (e.g., `show.html.twig`, `index.html.twig`), create hook configuration.

## Step 3: Detect page context variables

```bash
Tool: Read
File: templates/shop/{page}/{action}.html.twig
```

Look for:
- Variables used in template (e.g., `{{ resource.name }}` → variable is `resource`)
- Extended layout (should be `@SyliusShop/shared/layout/base.html.twig`)
- Blocks used

## Step 4: Create twig_hooks directory

```bash
Tool: Bash
Command: mkdir -p config/twig_hooks/shop/{page}
```

## Step 5: Create hook configuration

```bash
Tool: Write
File: config/twig_hooks/shop/{page}/{action}.yaml
```

Content pattern:
```yaml
sylius_twig_hooks:
    hooks:
        'plugin.shop.{page}.{action}':
            content:
                template: '@PluginName/shop/{page}/{action}/content.html.twig'
                priority: 0

        'plugin.shop.{page}.{action}.content':
            header:
                template: '@PluginName/shop/{page}/{action}/content/header.html.twig'
                priority: 100
            main:
                template: '@PluginName/shop/{page}/{action}/content/main.html.twig'
                priority: 0
```

**Important replacements:**
- `{page}` → page name (e.g., `resource`, `products`)
- `{action}` → action name (e.g., `show`, `index`)
- `plugin.shop.{page}.{action}` → your hook prefix
- `@PluginName` → actual plugin template namespace

## Step 6: Update main page template

```bash
Tool: Edit
File: templates/shop/{page}/{action}.html.twig
Old: {% block content %}
(existing content)
{% endblock %}
New: {% block content %}
    {% hook 'plugin.shop.{page}.{action}' with { {detected_variables} } %}
{% endblock %}
```

Pass all variables that sub-templates will need.

**Example:**
```twig
{% hook 'plugin.shop.resource.show' with { resource: resource } %}
```

## Step 7: Create template directories

```bash
Tool: Bash
Command: mkdir -p templates/shop/{page}/{action}/content
```

## Step 8: Create content wrapper template

```bash
Tool: Write
File: templates/shop/{page}/{action}/content.html.twig
```

Content:
```twig
<div class="container my-5">
    {% hook 'header' %}
    {% hook 'main' %}
</div>
```

**Bootstrap 5 classes:**
- `container` - centered content with responsive width
- `container-fluid` - full width content
- `my-5` - margin top/bottom (spacing)

## Step 9: Create section templates

### Header template

```bash
Tool: Write
File: templates/shop/{page}/{action}/content/header.html.twig
```

Content pattern:
```twig
{% set resource = hookable_metadata.context.{resource_name} %}

<div class="row mb-4">
    <div class="col-12">
        <h1>{{ resource.name }}</h1>
    </div>
</div>
```

### Main template

```bash
Tool: Write
File: templates/shop/{page}/{action}/content/main.html.twig
```

Content pattern:
```twig
{% set resource = hookable_metadata.context.{resource_name} %}

<div class="row">
    <div class="col-12">
        <div class="card">
            <div class="card-body">
                {{ resource.content|raw }}
            </div>
        </div>
    </div>
</div>
```

**Key points:**
- Access variables via `hookable_metadata.context.{variable_name}`
- Use Bootstrap 5 classes
- Each section = separate template = customizable

## Step 10: Migrate existing content to Bootstrap 5

If old templates use Semantic UI, convert:

```bash
Tool: Glob
Pattern: **/*.twig
Path: templates/shop/
```

For each template:

```bash
Tool: Edit
File: {template_path}
```

**Common conversions:**
- `ui container` → `container`
- `ui segment` → `card` (with `card-body`)
- `ui grid` → `row`
- `column` → `col-12`, `col-md-6`
- `ui button` → `btn btn-primary`
- `ui message` → `alert alert-info`
- `ui header` → `<h1>`, `<h2>`

## Step 11: Validate

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

```bash
Tool: Bash
Command: vendor/bin/console debug:twig-hooks | grep -i "shop.{page}"
```

Expected: Your shop hooks should appear with correct hierarchy.

## Summary Checklist

After executing this guide:
- ✅ Twig hooks config imported
- ✅ Hook configuration created for each shop page
- ✅ Main page template updated to use hooks
- ✅ Section templates created (content, header, main)
- ✅ Bootstrap 5 classes used throughout
- ✅ Variables passed through hook context
- ✅ Cache cleared and hooks validated

## Common Shop Page Patterns

### Product Listing

```yaml
'plugin.shop.product.index.content':
    grid:
        template: '@Plugin/shop/product/index/content/grid.html.twig'
        priority: 0
```

### Detail Page

```yaml
'plugin.shop.resource.show.content':
    header:
        template: '@Plugin/shop/resource/show/content/header.html.twig'
        priority: 100
    main:
        template: '@Plugin/shop/resource/show/content/main.html.twig'
        priority: 50
    sidebar:
        template: '@Plugin/shop/resource/show/content/sidebar.html.twig'
        priority: 0
```

### Content Page

```yaml
'plugin.shop.page.show.content':
    breadcrumbs:
        template: '@Plugin/shop/page/show/content/breadcrumbs.html.twig'
        priority: 100
    body:
        template: '@Plugin/shop/page/show/content/body.html.twig'
        priority: 0
```

## Troubleshooting

### Hooks not appearing

Check:
1. twig_hooks/**/*.yaml imported in config.yaml?
2. YAML syntax correct?
3. Cache cleared?

### Variables not accessible

Check:
1. Variables passed in `{% hook %}` call?
2. Using `hookable_metadata.context.{var}`?
3. Variable names match?

### Bootstrap styles not applied

Check:
1. Using correct Bootstrap 5 classes?
2. Not mixing with old Semantic UI classes?
3. Layout extends `@SyliusShop/shared/layout/base.html.twig`?
