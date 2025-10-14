# Admin Templates: Add Basic Live Component - AI Guide

This guide adds Basic Live Component to existing Twig Hooks structure for live validation.

## Prerequisites

- Twig Hooks already created (execute `admin-templates-hooks.md` first)
- Hook configuration files exist

## What This Adds

- ✅ Live form validation
- ✅ No custom PHP code needed
- ✅ Uses Sylius's `ResourceFormComponent`

## Step 1: Extract resource information

```bash
Tool: Read
File: config/routes/admin.yaml
```

Extract:
- Resource name from `alias: plugin.resource_name`
- Hook prefix from `vars.all.hook_prefix`

```bash
Tool: Read
File: config/services.xml
```

Extract:
- Plugin namespace
- Repository service ID (usually `plugin.repository.{resource}`)
- Model class parameter (usually `%plugin.model.{resource}.class%`)

## Step 2: Detect Form Type

```bash
Tool: Glob
Pattern: *Type.php
Path: src/Form/Type/
```

Find the form type for your resource (e.g., `ResourceType.php`).

## Step 3: Check if twig_component.xml exists

```bash
Tool: Glob
Pattern: twig_component.xml
Path: config/services/
```

If not found, create it:

```bash
Tool: Write
File: config/services/twig_component.xml
```

Content:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
    </services>
</container>
```

## Step 4: Add Live Component service definition

**IMPORTANT: Do this BEFORE updating hook configuration, so you know what key to reference.**

```bash
Tool: Edit
File: config/services/twig_component.xml
Old:     </services>
New:         <service
            id="plugin.admin.twig.component.{resource}.form"
            class="Sylius\Bundle\UiBundle\Twig\Component\ResourceFormComponent"
        >
            <argument type="service" id="plugin.repository.{resource}" />
            <argument type="service" id="form.factory" />
            <argument>%plugin.model.{resource}.class%</argument>
            <argument>Plugin\Form\Type\{Resource}Type</argument>

            <tag name="sylius.live_component.admin" key="plugin:admin:{resource}:form" />
        </service>
    </services>
```

**Important:** Replace placeholders:
- `plugin.admin.twig.component.{resource}.form` → actual service ID
- `plugin.repository.{resource}` → actual repository service ID
- `%plugin.model.{resource}.class%` → actual model parameter
- `Plugin\Form\Type\{Resource}Type` → actual form type class with full namespace
- `plugin:admin:{resource}:form` → actual component key (colon-separated, this is what you'll use in hooks)

## Step 5: Import twig_component.xml in services.xml

```bash
Tool: Read
File: config/services.xml
```

Check if twig_component.xml is already imported. If not:

```bash
Tool: Edit
File: config/services.xml
Old: <imports>
New: <imports>
        <import resource="services/twig_component.xml" />
```

If no imports section exists, add it after the opening `<container>` tag.

## Step 6: Update hook configuration to use Live Component

```bash
Tool: Read
File: config/twig_hooks/admin/{resource}/create.yaml
```

Find the top-level content hook (e.g., `{prefix}.admin.{resource}.create.content`).

```bash
Tool: Edit
File: config/twig_hooks/admin/{resource}/create.yaml
Old: '{prefix}.admin.{resource}.create.content.form.sections':
New: '{prefix}.admin.{resource}.create.content':
            form:
                component: 'plugin:admin:{resource}:form'
                props:
                    resource: '@=_context.resource'
                    form: '@=_context.form'
                    template: '@SyliusAdmin/shared/crud/common/content/form.html.twig'
                configuration:
                    render_rest: false
                priority: 0

        '{prefix}.admin.{resource}.create.content.form.sections':
```

**Important:**
- Replace placeholders with actual values from Step 1
- Use `@SyliusAdmin/shared/crud/common/content/form.html.twig` - Sylius's default form wrapper
- Component key must match the key from service tag in Step 4

## Step 7: Update update.yaml similarly

```bash
Tool: Read
File: config/twig_hooks/admin/{resource}/update.yaml
```

Apply same changes as create.yaml:

```bash
Tool: Edit
File: config/twig_hooks/admin/{resource}/update.yaml
Old: '{prefix}.admin.{resource}.update.content.form.sections':
New: '{prefix}.admin.{resource}.update.content':
            form:
                component: 'plugin:admin:{resource}:form'
                props:
                    resource: '@=_context.resource'
                    form: '@=_context.form'
                    template: '@SyliusAdmin/shared/crud/common/content/form.html.twig'
                configuration:
                    render_rest: false
                priority: 0

        '{prefix}.admin.{resource}.update.content.form.sections':
```

## Step 8: Validate

Clear cache:

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

Check if component service is registered:

```bash
Tool: Bash
Command: vendor/bin/console debug:container | grep "twig.component.{resource}"
```

Expected: Service should appear in the list.

Check hooks still work:

```bash
Tool: Bash
Command: vendor/bin/console debug:twig-hooks | grep -i "{resource}"
```

Expected: Hooks should include the component configuration.

## Summary Checklist

After executing this guide:
- ✅ twig_component.xml created/updated
- ✅ Live Component service defined (BEFORE hook configuration)
- ✅ Service file imported
- ✅ Hook configuration updated to use component with Sylius's default form template
- ✅ Cache cleared
- ✅ Component service validated

## What You Get

- ✅ Live form validation (validates fields as user types)
- ✅ Form state persisted across interactions
- ✅ No page reload needed for validation
- ✅ Using Sylius's default form template - no custom template needed!
- ✅ All using Sylius's built-in component - no custom code!

## Next Steps

- If you need custom actions (slug generation, etc.) → Execute `admin-templates-live-custom.md`
- Otherwise → You're done! Test the forms in browser

## Notes on Form Template

- By default, use `@SyliusAdmin/shared/crud/common/content/form.html.twig`
- Only create custom `form.html.twig` if you need:
  - Custom form theme
  - Custom form attributes
  - Additional wrapper markup

If you need custom form template, create it and update `template:` prop in hook configuration.

## Troubleshooting

### Service not found error

Check:
1. Repository service ID correct?
2. Model parameter exists in parameters?
3. Form Type class namespace correct?
4. twig_component.xml imported in services.xml?

### Component not loading

Check:
1. Component key format: `plugin:admin:resource:form` (colon-separated, all lowercase)
2. Service has correct tag: `sylius.live_component.admin`
3. Key in service matches key in hook configuration?
4. Cache cleared?

### Hooks not rendering

Check:
1. Template path correct in component props?
2. Using `@SyliusAdmin/shared/crud/common/content/form.html.twig`?
3. `render_rest: false` set?
4. Hook priority set (usually 0 for top-level)?
