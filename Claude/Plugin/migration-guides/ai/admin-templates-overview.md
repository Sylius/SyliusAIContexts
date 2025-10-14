# Admin Templates: Overview & Decision Guide - AI

This guide helps AI decide which template migration approach to use.

## Recommended Approach

**For 90% of cases: Twig Hooks + Basic Live Component**

Execute in this order:
1. `admin-templates-hooks.md` - Create Twig Hooks structure
2. `admin-templates-live-basic.md` - Add Basic Live Component

Only if custom actions needed:
3. `admin-templates-live-custom.md` - Replace with Custom Live Component

## Decision Tree for AI

```
Does plugin have admin form templates?
├─ NO → Skip this step
└─ YES → Execute admin-templates-hooks.md
    │
    ├─ Execute admin-templates-live-basic.md (recommended)
    │   │
    │   Does plugin need custom actions (slug generation, etc)?
    │   ├─ NO → ✅ Done!
    │   └─ YES → Execute admin-templates-live-custom.md
    │
    └─ (Not recommended) Static templates with single hook
        → See "Alternative: Static Templates" section in admin-templates-hooks.md
```

## Detection Commands

### Check if plugin has admin templates

```bash
Tool: Glob
Pattern: **/*.twig
Path: templates/admin/
```

If templates found → Proceed with migration

### Check if plugin has form templates

```bash
Tool: Grep
Pattern: form_row|form_widget|form_start
Path: templates/admin/
Output: files_with_matches
```

If matches found → Forms exist, proceed

### Detect form fields

```bash
Tool: Read
File: src/Form/Type/{Resource}Type.php
```

Look for `->add()` calls to identify all form fields.

### Detect translation fields

```bash
Tool: Grep
Pattern: ResourceTranslationsType|translations
Path: src/Form/Type/
Output: content
```

If found → Has translatable fields

## Execution Order

1. **Always execute:** `admin-templates-hooks.md`
   - Creates Twig Hooks structure
   - Migrates to Bootstrap 5
   - Makes templates customizable

2. **Recommended:** `admin-templates-live-basic.md`
   - Adds live validation
   - No custom PHP code needed
   - Uses Sylius's ResourceFormComponent

3. **Optional:** `admin-templates-live-custom.md`
   - Only if custom actions needed
   - Replaces basic with custom component
   - Requires custom PHP code

## Files to Create

After executing guides, you should have:

```
config/
├── twig_hooks/admin/{resource}/
│   ├── create.yaml
│   └── update.yaml
├── services/
│   └── twig_component.xml
templates/admin/{resource}/
├── form.html.twig
└── form/sections/
    ├── general.html.twig
    ├── general/
    │   ├── code.html.twig
    │   └── enabled.html.twig
    ├── translations.html.twig
    └── translations/
        ├── name.html.twig
        └── slug.html.twig
src/Twig/Component/{Resource}/
└── FormComponent.php (only if custom actions needed)
```

## Validation After Each Step

After `admin-templates-hooks.md`:
```bash
Tool: Bash
Command: vendor/bin/console cache:clear && vendor/bin/console debug:twig-hooks | grep -i "{resource}"
```

After `admin-templates-live-basic.md`:
```bash
Tool: Bash
Command: vendor/bin/console debug:container | grep "twig.component.{resource}"
```

After `admin-templates-live-custom.md`:
```bash
Tool: Bash
Command: vendor/bin/console debug:container {component_service_id}
```

## Common Patterns

### Resource name extraction

From routing file:
```yaml
# config/routes/admin.yaml
your_plugin_admin:
    resource: |
        alias: your_plugin.resource  # resource name: "resource"
```

### Hook prefix extraction

From routing file:
```yaml
# config/routes/admin.yaml
vars:
    all:
        hook_prefix: 'your_plugin.admin.resource'
```

This creates hooks like:
- `{prefix}.{action}.content`
- `{prefix}.{action}.content.form.sections`
- `{prefix}.{action}.content.form.sections.general`

## Summary

- **Default:** Execute hooks + basic live component
- **Advanced:** Add custom live component if needed
- **Validate:** After each step, ensure no errors
- **Report:** Success or errors to user
