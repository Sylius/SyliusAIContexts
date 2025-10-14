# Shop: Template Migration

This step converts shop templates from Semantic UI to Bootstrap 5 and integrates them with Sylius 2.0 Twig Hooks system.

**When to skip this step:**
- Your plugin has no shop templates
- Templates are already using Bootstrap 5 and Twig Hooks

**When to do this step:**
- You have shop pages using Semantic UI
- You want to make shop templates customizable in end application

---

## 📊 Overview & Approach

Shop templates are simpler than admin forms:
- Usually custom pages (product list, resource details, etc.)
- No forms or Live Components (typically)
- Focus on content presentation

### Two Approaches:

**A. Extend Existing Sylius Hooks** (for adding content to existing pages)
- Add your content to checkout, cart, product pages, etc.
- Use existing Sylius hook structure
- Example: Add custom checkbox to checkout

**B. Custom Hooks for Custom Pages** (RECOMMENDED for new pages)
- Create your own hook hierarchy
- Full control over structure
- End application can customize everything
- Example: Resource show page, custom product listing

---

## ✅ RECOMMENDED: Custom Hooks for Custom Pages

Use this when creating new pages in your plugin (like resource detail page, custom listings, etc.).

### 1. Create main page template

`templates/shop/resource/show.html.twig`:
```twig
{% extends '@SyliusShop/shared/layout/base.html.twig' %}

{% block title %}{{ resource.name }} | {{ parent() }}{% endblock %}

{% block content %}
    {% hook 'plugin.shop.resource.show' with { resource: resource } %}
{% endblock %}
```

**Key points:**
- Extend Sylius base layout
- Use `{% hook %}` tag to make content customizable
- Pass variables needed by sub-templates

### 2. Create hook configuration

```bash
mkdir -p config/twig_hooks/shop/resource
```

`config/twig_hooks/shop/resource/show.yaml`:
```yaml
sylius_twig_hooks:
    hooks:
        'plugin.shop.resource.show':
            content:
                template: '@Plugin/shop/resource/show/content.html.twig'
                priority: 0

        'plugin.shop.resource.show.content':
            header:
                template: '@Plugin/shop/resource/show/content/header.html.twig'
                priority: 100
            main:
                template: '@Plugin/shop/resource/show/content/main.html.twig'
                priority: 0
```

**Hook naming convention:**
- `plugin.shop.{page}.{action}` - top level
- `plugin.shop.{page}.{action}.{section}` - sections
- Use priorities to control order (higher = first)

### 3. Create section templates

`templates/shop/resource/show/content.html.twig`:
```twig
<div class="container my-5">
    {% hook 'header' %}
    {% hook 'main' %}
</div>
```

`templates/shop/resource/show/content/header.html.twig`:
```twig
{% set resource = hookable_metadata.context.resource %}

<div class="row mb-4">
    <div class="col-12">
        <h1>{{ resource.name }}</h1>
    </div>
</div>
```

`templates/shop/resource/show/content/main.html.twig`:
```twig
{% set resource = hookable_metadata.context.resource %}

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
- Access variables via `hookable_metadata.context.{variable}`
- Use Bootstrap 5 classes: `container`, `row`, `col-*`, `card`
- Each section in separate template = customizable

### 4. Import Twig Hooks config

```yaml
# config/config.yaml
imports:
    - { resource: "twig_hooks/**/*.yaml" }
```

### 5. Validate

```bash
vendor/bin/console cache:clear
vendor/bin/console debug:twig-hooks | grep -i "shop"
```

---

## 📝 Alternative: Extend Existing Sylius Hooks

Use this to add content to existing Sylius pages (checkout, cart, product, etc.).

### 1. Find existing Sylius hooks

**Option A: Symfony Profiler (recommended)**
- Open the page in browser
- Open Symfony Profiler → Twig Hooks tab
- See all available hooks

**Option B: Grep vendor files**
```bash
grep -r "sylius_twig_hooks:" vendor/sylius/sylius/src/Sylius/Bundle/ShopBundle/config/twig_hooks/
```

### 2. Create hook configuration

`config/twig_hooks/shop/checkout/complete.yaml`:
```yaml
sylius_twig_hooks:
    hooks:
        'sylius_shop.checkout.complete.content.form':
            custom_checkbox:
                template: '@Plugin/shop/checkout/complete/content/form/custom.html.twig'
                priority: 50
```

### 3. Create template

`templates/shop/checkout/complete/content/form/custom.html.twig`:
```twig
{% set form = hookable_metadata.context.form %}

<div class="mb-3">
    {{ form_row(form.custom_field) }}
</div>
```

**When to use this:**
- Adding fields to existing forms
- Injecting content into existing pages
- Extending without replacing

**When NOT to use this:**
- Creating new pages → Use custom hooks
- Need full control → Use custom hooks

---

## 🎨 Bootstrap 5 Migration

Sylius 2.0 uses Bootstrap 5 instead of Semantic UI. Use standard Bootstrap classes in your templates.

**Common patterns:**
- Container: `container`, `container-fluid`
- Grid: `row`, `col-*`
- Cards: `card`, `card-header`, `card-body`
- Buttons: `btn btn-primary`

---

## 🔍 Finding Existing Sylius Hooks

**For developers (manual):**
- Open shop page in browser
- Open Symfony Profiler → Twig Hooks tab
- See all available hooks with their structure
- Copy hook name and use in your config

**For AI:**
```bash
grep -r "sylius_twig_hooks:" vendor/sylius/sylius/src/Sylius/Bundle/ShopBundle/config/twig_hooks/ | grep -i "{keyword}"
```

---

## ✅ Summary

**Recommended approach for shop:**
1. **Custom pages:** Use custom hooks (full control)
2. **Extending existing pages:** Extend Sylius hooks (minimal changes)
3. **Bootstrap 5:** Use modern classes
4. **No Live Components needed:** Shop is usually static content

**Key principles:**
- Custom hooks for new pages = maximum flexibility
- Extend Sylius hooks for small additions
- Bootstrap 5: `container`, `row`, `col-*`, `card` classes
- Pass all needed variables through hook context
- Each section = separate template = customizable

**What end applications can do:**
- Disable any section
- Reorder sections
- Override templates
- Add new sections
- Change priorities
