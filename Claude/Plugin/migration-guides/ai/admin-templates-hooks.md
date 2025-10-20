# Admin Templates: Create Twig Hooks - AI Guide

This guide provides Tool commands to create Twig Hooks structure for admin forms.

## Prerequisites

Hook prefix must be configured in routing (done in routing migration step).

## Step 1: Find ALL resources that need migration

**CRITICAL:** You must migrate ALL resources with admin forms, not just resources with custom templates in `old_templates/`.

### Find all resources:

```bash
Tool: Read
File: config/routing/admin.yaml
```

**Extract ALL resources** that have `hook_prefix` defined. Each resource with `hook_prefix` needs Twig Hooks migration.

**Example:** If routing has:
```yaml
plugin_resource_one:
    vars:
        all:
            hook_prefix: 'plugin.admin.resource_one'

plugin_resource_two:
    vars:
        all:
            hook_prefix: 'plugin.admin.resource_two'
```

Then you have **2 resources** to migrate: `resource_one` and `resource_two`.

**Important notes:**
- Count ALL resources with `hook_prefix` in routing
- Old templates in `old_templates/` are NOT relevant for this count
- You must execute Steps 2-15 FOR EACH resource found
- Do NOT skip resources that don't have custom templates

### Verify with Form Types:

Cross-check by finding all Form Types:

```bash
Tool: Glob
Pattern: *Type.php
Path: src/Form/Type/
```

Each Form Type (excluding translation types) typically corresponds to one resource that needs migration.

## Step 2: Check if twig_hooks config is imported

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

If no imports section exists:

```bash
Tool: Read
File: config/config.yaml
```

Then prepend at the top:

```bash
Tool: Edit
File: config/config.yaml
Old: (first line of file)
New: imports:
    - { resource: "twig_hooks/**/*.yaml" }

(first line of file)
```

## Step 3: Extract resource info (FOR EACH RESOURCE)

**Execute Steps 3-15 FOR EACH resource found in Step 1.**

For the current resource you're migrating:

```bash
Tool: Read
File: config/routes/admin.yaml
```

Extract:
- Resource name from `alias: plugin.resource_name`
- Hook prefix from `vars.all.hook_prefix`

## Step 3: Detect form fields

```bash
Tool: Read
File: src/Form/Type/{Resource}Type.php
```

Identify all `->add('field_name')` calls. These are the fields you need to create templates for.

## Step 4: Detect translation fields

```bash
Tool: Grep
Pattern: ResourceTranslationsType
Path: src/Form/Type/
Output: content
```

If found, check which fields are translatable by reading the translation type.

## Step 5: Create twig_hooks directory

```bash
Tool: Bash
Command: mkdir -p config/twig_hooks/admin/{resource}
```

## Step 6: Create create.yaml hook configuration

**Note:** This creates hooks WITHOUT Live Components. If you want Live Components, execute `admin-templates-live-basic.md` after this guide.

```bash
Tool: Write
File: config/twig_hooks/admin/{resource}/create.yaml
```

Content pattern (basic example without translations):
```yaml
sylius_twig_hooks:
    hooks:
        '{prefix}.admin.{resource}.create.content.form.sections':
            general:
                template: '@PluginName/admin/{resource}/form/sections/general.html.twig'
                priority: 100

        '{prefix}.admin.{resource}.create.content.form.sections.general':
            default:
                enabled: false
            code:
                template: '@PluginName/admin/{resource}/form/sections/general/code.html.twig'
                priority: 300
            enabled:
                template: '@PluginName/admin/{resource}/form/sections/general/enabled.html.twig'
                priority: 200
```

**If resource has translations**, add translations section:
```yaml
        '{prefix}.admin.{resource}.create.content.form.sections':
            general:
                template: '@PluginName/admin/{resource}/form/sections/general.html.twig'
                priority: 100
            translations:  # ← Add this only if resource has translations
                template: '@PluginName/admin/{resource}/form/sections/translations.html.twig'
                priority: 0

        '{prefix}.admin.{resource}.create.content.form.sections.translations':
            # Add one hook per translatable field with descending priority
            name:
                template: '@PluginName/admin/{resource}/form/sections/translations/name.html.twig'
                priority: 300
            slug:
                template: '@PluginName/admin/{resource}/form/sections/translations/slug.html.twig'
                priority: 200
```

**Important:**
- Replace `{prefix}`, `{resource}`, `@PluginName` with actual values
- Add hook for each field detected in Step 3/4
- Always set `default: enabled: false` for sub-hooks
- Use descending priority (300, 200, 100, etc.)
- Translations section is OPTIONAL - only add if resource has translations

## Step 7: Create update.yaml hook configuration

```bash
Tool: Bash
Command: cp config/twig_hooks/admin/{resource}/create.yaml config/twig_hooks/admin/{resource}/update.yaml
```

Then replace create with update:

```bash
Tool: Edit
File: config/twig_hooks/admin/{resource}/update.yaml
Old: .create.content
New: .update.content
ReplaceAll: true
```

## Step 8: Create template directories

```bash
Tool: Bash
Command: mkdir -p templates/admin/{resource}/form/sections/general templates/admin/{resource}/form/sections/translations
```

## Step 9: Create main section template (general)

```bash
Tool: Write
File: templates/admin/{resource}/form/sections/general.html.twig
```

Content:
```twig
<div class="card mb-3">
    <div class="card-header">
        <div class="card-title">
            {{ 'sylius.ui.general'|trans }}
        </div>
    </div>
    <div class="card-body">
        <div class="row">
            {% hook 'general' %}
        </div>
    </div>
</div>
```

## Step 10: Create field templates for general section

For each non-translatable field:

```bash
Tool: Write
File: templates/admin/{resource}/form/sections/general/{field_name}.html.twig
```

Content pattern:
```twig
{% set form = hookable_metadata.context.form %}

<div class="col-12">
    {{ form_row(form.{field_name}, sylius_test_form_attribute('{field_name}')) }}
</div>
```

**Column width options:**
- `col-12` - full width
- `col-md-6` - half width on medium screens and up

## Step 11: (Optional) Create translations section template

**Only if resource has translations** detected in Step 4:

```bash
Tool: Write
File: templates/admin/{resource}/form/sections/translations.html.twig
```

Content:
```twig
{% import '@SyliusAdmin/shared/helper/translations.html.twig' as translations %}

{% set form = hookable_metadata.context.form %}
{% set prefixes = hookable_metadata.prefixes %}

<div class="card mb-3">
    <div class="card-header">
        <div class="card-title">{{ 'sylius.ui.translations'|trans }}</div>
    </div>
    <div class="card-body">
        <div class="row">
            {{ translations.with_hook(form.translations, prefixes, null, { accordion_flush: true }) }}
        </div>
    </div>
</div>
```

## Step 12: (Optional) Create field templates for translations

**Only if resource has translations**. For each translatable field:

```bash
Tool: Write
File: templates/admin/{resource}/form/sections/translations/{field_name}.html.twig
```

Content pattern:
```twig
{% set form = hookable_metadata.context.form %}

<div class="col-12">
    {{ form_row(form.{field_name}, sylius_test_form_attribute('{field_name}')) }}
</div>
```

## Step 13: Migrate existing custom templates to Bootstrap 5 (OPTIONAL)

**Note:** This step is OPTIONAL and only applies if the resource has custom template overrides in `old_templates/`.

**This does NOT affect which resources need Twig Hooks migration** - all resources need Twig Hooks regardless of old_templates.

If the current resource has existing Semantic UI custom templates in `old_templates/`, convert them:

### Find all admin templates

```bash
Tool: Glob
Pattern: **/*.twig
Path: templates/admin/
```

### For each template file, replace Semantic UI with Bootstrap 5

```bash
Tool: Edit
File: {template_path}
```

**Common conversions:**
- `ui segment` → `card`
- `class="header"` inside segment → `class="card-header"`
- `class="content"` inside segment → `class="card-body"`
- `<h*` inside card-header → add `class="card-title"`
- `ui grid` → `row`
- `column` / `wide column` → `col-12` / `col-md-6`
- `ui form` → (remove class, forms auto-styled)
- `ui button` → `btn btn-primary`
- `ui message` → `alert alert-info`
- `ui labeled input` → `input-group`

## Step 14: Validate

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

```bash
Tool: Bash
Command: vendor/bin/console debug:twig-hooks | grep -i "{resource}"
```

Expected output: Your hooks should appear in the list with correct hierarchy.

## Summary Checklist

After executing this guide:
- ✅ Twig hooks config imported in config.yaml
- ✅ Hook configuration created for create and update actions
- ✅ Section templates created (general, and translations if applicable)
- ✅ Field templates created for all form fields
- ✅ `default: enabled: false` set for all sub-hook groups
- ✅ Bootstrap 5 classes used throughout
- ✅ Cache cleared and hooks validated

## What You Have Now

- ✅ Fully customizable forms (Twig Hooks)
- ✅ Bootstrap 5 styling
- ✅ Using Twig Hooks WITHOUT Live Components
- ✅ End applications can customize every field

## Next Steps

**Recommended:** Execute `admin-templates-live-basic.md`
- Adds live validation for free
- Uses Sylius's default form template (`@SyliusAdmin/shared/crud/common/content/form.html.twig`)
- No custom code needed

**Optional:** Skip Live Components if you don't need live validation

## Troubleshooting

### Hooks not appearing in debug:twig-hooks

Check:
1. Cache cleared?
2. twig_hooks/**/*.yaml imported in config.yaml?
3. YAML syntax correct (no tabs, proper indentation)?

### Form fields not rendering

Check:
1. `default: enabled: false` set for sub-hooks?
2. Field name in template matches form field name?
3. `hookable_metadata.context.form` used to access form?

### Double rendering of form

This means `default: enabled: false` is missing. Add it to all sub-hook groups.

---

## Alternative: Static Templates (Not Recommended)

If you want simplest approach with no customization, create one template with all fields. **You still must use Twig Hook**, just a single one.

### Step 1: Create single template file

```bash
Tool: Write
File: templates/admin/{resource}/form/sections/all_fields.html.twig
```

Content:
```twig
{% set form = hookable_metadata.context.form %}

<div class="card mb-3">
    <div class="card-header">
        <div class="card-title">{{ 'sylius.ui.general'|trans }}</div>
    </div>
    <div class="card-body">
        <div class="row">
            <div class="col-12">
                {{ form_row(form.code, sylius_test_form_attribute('code')) }}
            </div>
            <div class="col-12">
                {{ form_row(form.enabled, sylius_test_form_attribute('enabled')) }}
            </div>
            {# Add all other fields here #}
        </div>
    </div>
</div>

{{ form_rest(form) }}
```

### Step 2: Create hook configuration

```bash
Tool: Write
File: config/twig_hooks/admin/{resource}/create.yaml
```

Content:
```yaml
sylius_twig_hooks:
    hooks:
        '{prefix}.admin.{resource}.create.content.form.sections':
            general:
                enabled: false
            all_fields:
                template: '@PluginName/admin/{resource}/form/sections/all_fields.html.twig'
                priority: 0
```

### Step 3: Copy to update.yaml

```bash
Tool: Bash
Command: cp config/twig_hooks/admin/{resource}/create.yaml config/twig_hooks/admin/{resource}/update.yaml
```

```bash
Tool: Edit
File: config/twig_hooks/admin/{resource}/update.yaml
Old: .create.content
New: .update.content
ReplaceAll: true
```

**Why not recommended:**
- End application cannot customize individual fields
- No flexibility for end users
- Hard to maintain for complex forms

---

## Step 15: Test in Browser (if MCP browser available)

If MCP browser tools are available (mcp__playwright or mcp__chrome-devtools):
- Navigate to admin create/update forms for the resource
- Take screenshots to verify forms render correctly
- Check that all fields display, Bootstrap 5 styling is applied, and translations work

---

## Notes for AI

### Common Mistake: Only Migrating Resources with Custom Templates

**WRONG APPROACH:**
1. Check `old_templates/` for custom form templates
2. Only migrate resources that have custom templates
3. Skip resources without custom templates

**CORRECT APPROACH:**
1. Read `config/routing/admin.yaml`
2. Find ALL resources with `hook_prefix` defined
3. Migrate EVERY resource found, regardless of `old_templates/`
4. Optionally check `old_templates/` for custom styling to preserve (Step 13)

**Why this matters:**
- Sylius 2.0 requires Twig Hooks for ALL admin forms
- `old_templates/` only contains custom template overrides from Sylius 1.x
- Most resources used default Sylius templates in 1.x (no custom templates)
- But ALL resources still need Twig Hooks in 2.0

**Example:**
If a plugin has 3 resources (`ad`, `banner`, `section`) but only `banner` has a custom template in `old_templates/Admin/Banner/_form.html.twig`:
- ❌ WRONG: Migrate only `banner`
- ✅ CORRECT: Migrate all 3 resources (`ad`, `banner`, `section`)

### Workflow Summary

1. **Step 1:** Find ALL resources (count them from routing file)
2. **Steps 2-15:** Execute FOR EACH resource
3. **Verification:** Check you migrated the same number of resources as found in Step 1
