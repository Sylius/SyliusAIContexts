# Admin Templates: Add Custom Live Component - AI Guide

This guide replaces Basic Live Component with Custom Live Component for custom actions.

## Prerequisites

- Twig Hooks already created
- Basic Live Component already configured (execute `admin-templates-live-basic.md` first)

## When to Use This

Only use custom component when you need:
- Custom actions (e.g., auto-generate slug from name)
- Field dependencies
- Custom form behavior beyond basic validation

## Step 1: Extract resource information

```bash
Tool: Read
File: config/routes/admin.yaml
```

Extract:
- Resource name
- Hook prefix

```bash
Tool: Read
File: config/services.xml
```

Extract:
- Plugin namespace
- Repository service ID
- Model class parameter

## Step 2: Detect what custom logic is needed

```bash
Tool: Read
File: src/Form/Type/{Resource}Type.php
```

Identify which fields might need custom actions:
- Slug field → likely needs generation from name
- Preview field → might need render action
- Dependent fields → might need update actions

## Step 3: Create FormComponent directory

```bash
Tool: Bash
Command: mkdir -p src/Twig/Component/{Resource}
```

## Step 4: Create custom FormComponent class

```bash
Tool: Write
File: src/Twig/Component/{Resource}/FormComponent.php
```

Content pattern:
```php
<?php

declare(strict_types=1);

namespace Plugin\Twig\Component\{Resource};

use Plugin\Model\{Resource}Interface;
use Sylius\Bundle\UiBundle\Twig\Component\ResourceFormComponentTrait;
use Sylius\Bundle\UiBundle\Twig\Component\TemplatePropTrait;
use Sylius\Component\Product\Generator\SlugGeneratorInterface;
use Sylius\Component\Resource\Repository\RepositoryInterface;
use Symfony\Component\Form\FormFactoryInterface;
use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\Attribute\LiveAction;
use Symfony\UX\LiveComponent\Attribute\LiveArg;

#[AsLiveComponent]
class FormComponent
{
    /** @use ResourceFormComponentTrait<{Resource}Interface> */
    use ResourceFormComponentTrait;
    use TemplatePropTrait;

    public function __construct(
        RepositoryInterface ${resource}Repository,
        FormFactoryInterface $formFactory,
        string $resourceClass,
        string $formClass,
        protected readonly SlugGeneratorInterface $slugGenerator,
    ) {
        $this->initialize(${resource}Repository, $formFactory, $resourceClass, $formClass);
    }

    #[LiveAction]
    public function generate{Resource}Slug(#[LiveArg] string $localeCode): void
    {
        $this->formValues['translations'][$localeCode]['slug'] =
            $this->slugGenerator->generate($this->formValues['translations'][$localeCode]['name']);
    }
}
```

**Important replacements:**
- `Plugin` → actual plugin namespace
- `{Resource}` → resource name (PascalCase)
- `{resource}` → resource name (camelCase)
- Add/remove dependencies as needed
- Add/remove LiveAction methods as needed

**Common LiveAction patterns:**

For slug generation:
```php
#[LiveAction]
public function generateSlug(#[LiveArg] string $localeCode): void
{
    $this->formValues['translations'][$localeCode]['slug'] =
        $this->slugGenerator->generate($this->formValues['translations'][$localeCode]['name']);
}
```

For field dependencies:
```php
#[LiveAction]
public function updateDependentField(): void
{
    // Custom logic based on $this->formValues
}
```

## Step 5: Update service definition

```bash
Tool: Edit
File: config/services/twig_component.xml
Old: <service
            id="plugin.admin.twig.component.{resource}.form"
            class="Sylius\Bundle\UiBundle\Twig\Component\ResourceFormComponent"
        >
            <argument type="service" id="plugin.repository.{resource}" />
            <argument type="service" id="form.factory" />
            <argument>%plugin.model.{resource}.class%</argument>
            <argument>Plugin\Form\Type\{Resource}Type</argument>

            <tag name="sylius.live_component.admin" key="plugin:admin:{resource}:form" />
        </service>
New: <service
            id="plugin.admin.twig.component.{resource}.form"
            class="Plugin\Twig\Component\{Resource}\FormComponent"
        >
            <argument type="service" id="plugin.repository.{resource}" />
            <argument type="service" id="form.factory" />
            <argument>%plugin.model.{resource}.class%</argument>
            <argument>Plugin\Form\Type\{Resource}Type</argument>
            <argument type="service" id="sylius.generator.slug" />

            <tag name="sylius.live_component.admin" key="plugin:admin:{resource}:form" />
        </service>
```

**Important:**
- Change class to your custom FormComponent
- Add additional `<argument>` tags for any extra dependencies your custom component needs
- Keep the same service ID and tag

## Step 6: Validate PHP syntax

```bash
Tool: Bash
Command: php -l src/Twig/Component/{Resource}/FormComponent.php
```

Expected output: `No syntax errors detected`

## Step 7: Clear cache and validate

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

Check if custom component is registered:

```bash
Tool: Bash
Command: vendor/bin/console debug:container plugin.admin.twig.component.{resource}.form --show-arguments
```

Expected: Should show your custom class with all arguments.

Check hooks still work:

```bash
Tool: Bash
Command: vendor/bin/console debug:twig-hooks | grep -i "{resource}"
```

## Summary Checklist

After executing this guide:
- ✅ Custom FormComponent PHP class created
- ✅ LiveAction methods defined
- ✅ Service definition updated to use custom class
- ✅ Additional dependencies added to service
- ✅ PHP syntax validated
- ✅ Cache cleared
- ✅ Service validated

## What You Get

- ✅ All benefits of Basic Live Component (live validation)
- ✅ Custom actions (slug generation, etc.)
- ✅ Full control over form behavior
- ✅ Reusable actions across different forms

## Testing Custom Actions

After implementation, test in browser:
1. Open create/update form
2. Fill in fields that trigger custom actions
3. Verify actions execute (e.g., slug appears)
4. Check browser console for errors

## Common Custom Actions

### Slug Generation

Generates slug from name field:
```php
#[LiveAction]
public function generateSlug(#[LiveArg] string $localeCode): void
{
    $this->formValues['translations'][$localeCode]['slug'] =
        $this->slugGenerator->generate(
            $this->formValues['translations'][$localeCode]['name']
        );
}
```

### Clear Field

Clears a specific field:
```php
#[LiveAction]
public function clearDescription(): void
{
    $this->formValues['description'] = '';
}
```

### Calculate Field

Calculates value based on other fields:
```php
#[LiveAction]
public function calculateTotal(): void
{
    $this->formValues['total'] =
        $this->formValues['quantity'] * $this->formValues['price'];
}
```

## Troubleshooting

### Class not found error

Check:
1. Namespace correct in PHP file?
2. Class name matches file name?
3. Directory structure correct?

### Action not executing

Check:
1. Method has `#[LiveAction]` attribute?
2. Method is public?
3. Arguments have `#[LiveArg]` attribute?
4. Browser console for JavaScript errors?

### Service not loading

Check:
1. All constructor dependencies available in container?
2. Service definition has all required arguments?
3. Cache cleared after changes?
