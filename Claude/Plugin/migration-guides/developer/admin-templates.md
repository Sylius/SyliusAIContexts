# Admin: Template Migration

This step converts templates from Semantic UI to Bootstrap 5 and integrates them with Sylius 2.0 Twig Hooks system.

**When to skip this step:**
- Your plugin has no admin templates
- Templates are already using Bootstrap 5 and Twig Hooks

**When to do this step:**
- You have admin forms (all resources need Twig Hooks in Sylius 2.0)
- You want to make templates customizable in end application

**Important:** Execute this migration for ALL resources with admin forms, not just resources with custom templates. In Sylius 2.0, all admin forms require Twig Hooks configuration.

---

## 📊 Overview & Recommendations

### Two Independent Decisions:

####Decision 1: Template Structure
- **Static Templates** - All fields in one file, not customizable by end application
- **Twig Hooks with sub-hooks** - Each field in separate template, full flexibility

#### Decision 2: Form Type
- **Normal Forms** - Traditional forms without interactivity
- **Live Components** - Server-side interactivity (live validation, dynamic actions)

### 🎯 Recommended Approach

**For 90% of cases: Twig Hooks + Basic Live Component**

✅ **Why this is recommended:**
- End application can customize every field
- Live validation out of the box (no extra work!)
- Uses Sylius's built-in `ResourceFormComponent` - no custom code needed
- Maximum flexibility with minimal effort

**When to add Custom Live Component:**
- You need custom actions (e.g., auto-generate slug from name)
- You have field dependencies
- You need custom form behavior

**When to skip Live Components:**
- Very simple forms where live validation isn't needed
- Performance-critical scenarios (rare)

---

## 🎯 Decision Guide

```
Do you want end applications to customize your forms?
├─ NO → Use Static Templates (not recommended)
└─ YES → Use Twig Hooks
    │
    Do you want live validation for free?
    ├─ YES → Add Basic Live Component (recommended)
    │   │
    │   Do you need custom actions (slug generation, etc)?
    │   ├─ NO → ✅ You're done! (Twig Hooks + Basic Live Component)
    │   └─ YES → Add Custom Live Component
    │
    └─ NO → Just use Twig Hooks (normal forms)
```

---

## ✅ RECOMMENDED: Twig Hooks + Basic Live Component

This is the recommended approach for most plugins. Follow these steps:

### 1. Add hook prefix to routing

Already done in Routing step:
```yaml
# config/routes/admin.yaml
vars:
    all:
        hook_prefix: 'plugin.admin.resource'
```

This creates hook: `plugin.admin.resource.{action}.content.form.sections`

### 2. Import Twig Hooks config

```yaml
# config/config.yaml
imports:
    - { resource: "twig_hooks/**/*.yaml" }
```

### 3. Configure Live Component service

**First, configure the service so you know what to reference in hooks.**

`config/services/twig_component.xml`:
```xml
<service
    id="plugin.admin.twig.component.resource.form"
    class="Sylius\Bundle\UiBundle\Twig\Component\ResourceFormComponent"
>
    <argument type="service" id="plugin.repository.resource" />
    <argument type="service" id="form.factory" />
    <argument>%plugin.model.resource.class%</argument>
    <argument>Plugin\Form\Type\ResourceType</argument>

    <tag name="sylius.live_component.admin" key="plugin:admin:resource:form" />
</service>
```

**Key value:** The `key` attribute (`plugin:admin:resource:form`) is what you'll reference in hook configuration.

### 4. Create hook configuration

```bash
mkdir -p config/twig_hooks/admin/resource
```

Create `config/twig_hooks/admin/resource/create.yaml`:
```yaml
sylius_twig_hooks:
    hooks:
        'plugin.admin.resource.create.content':
            form:
                component: 'plugin:admin:resource:form'  # ← Value from service tag key
                props:
                    resource: '@=_context.resource'
                    form: '@=_context.form'
                    template: '@SyliusAdmin/shared/crud/common/content/form.html.twig'
                configuration:
                    render_rest: false
                priority: 0

        'plugin.admin.resource.create.content.form.sections':
            general:
                template: '@Plugin/admin/resource/form/sections/general.html.twig'
                priority: 100

        'plugin.admin.resource.create.content.form.sections.general':
            default:
                enabled: false
            code:
                template: '@Plugin/admin/resource/form/sections/general/code.html.twig'
                priority: 300
            enabled:
                template: '@Plugin/admin/resource/form/sections/general/enabled.html.twig'
                priority: 200
```

**Key points:**
- Component uses `@SyliusAdmin/shared/crud/common/content/form.html.twig` - Sylius's default form wrapper
- Only create custom `form.html.twig` if you need custom form theme or modifications
- `default: enabled: false` - prevents double rendering when using sub-hooks

### 5. Create section templates

**Main section** (`templates/admin/resource/form/sections/general.html.twig`):
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

**Field template** (`templates/admin/resource/form/sections/general/code.html.twig`):
```twig
{% set form = hookable_metadata.context.form %}

<div class="col-12">
    {{ form_row(form.code, sylius_test_form_attribute('code')) }}
</div>
```

### 6. (Optional) Handle translations with Sylius helper

**Only if your resource has translations** and you want accordion UI:

Add to hook configuration:
```yaml
'plugin.admin.resource.create.content.form.sections':
    translations:
        template: '@Plugin/admin/resource/form/sections/translations.html.twig'
        priority: 0
```

Create translations section (`templates/admin/resource/form/sections/translations.html.twig`):
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

This helper automatically:
- Creates accordion for translations
- Allows sub-hooks for each translation field
- Handles locale switching

Add sub-hooks for translation fields:
```yaml
'plugin.admin.resource.create.content.form.sections.translations':
    name:
        template: '@Plugin/admin/resource/form/sections/translations/name.html.twig'
        priority: 300
    slug:
        template: '@Plugin/admin/resource/form/sections/translations/slug.html.twig'
        priority: 200
```

Create translation field templates (`templates/admin/resource/form/sections/translations/name.html.twig`):
```twig
{% set form = hookable_metadata.context.form %}

<div class="col-12">
    {{ form_row(form.name, sylius_test_form_attribute('name')) }}
</div>
```

### 7. Copy for update action

Copy `create.yaml` to `update.yaml` and change `create` to `update` in hook names.

### 8. Validate

```bash
vendor/bin/console cache:clear
vendor/bin/console debug:twig-hooks
```

Check that your hooks appear in the list.

**That's it!** Forms are now migrated to Bootstrap 5 with Twig Hooks and Live Components.

---

## 🔧 Advanced: Custom Form Template

**Only create custom `form.html.twig` if you need:**
- Custom form theme
- Custom form attributes
- Additional wrapper markup

Example (like in this plugin for custom form theme):

`templates/admin/resource/form.html.twig`:
```twig
{% form_theme form '@Plugin/admin/resource/form_theme.html.twig' %}

<div class="container-xl" {{ attributes }}>
    {{ form_start(form, {'attr': {'class': 'ui loadable form', 'novalidate': 'novalidate'}}) }}
        <div class="row">
            {% if hookable_metadata.configuration.method is defined %}
                <input type="hidden" name="_method" value="{{ hookable_metadata.configuration.method }}" />
            {% endif %}
            {{ form_errors(form) }}
            {{ form_widget(form._token) }}

            {% hook 'form' with { form, resource } %}
        </div>
    {{ form_end(form, {render_rest: hookable.configuration.render_rest|default(false)}) }}
</div>
```

Then update hook configuration to use your custom template:
```yaml
template: '@Plugin/admin/resource/form.html.twig'
```

---

## 🔧 Advanced: Custom Live Component Actions

If you need custom actions (e.g., auto-generate slug from name), follow these additional steps:

**Before You Start:** Check if you have existing JavaScript assets that already implement this logic. If you do, consider whether the JavaScript should be:
1. Rewritten as Live Component actions (this section)
2. Migrated to Stimulus Controller (see Assets Migration)
3. Left as-is and migrated to assets/admin/

See the [Assets Migration guide](admin-assets.md) for analyzing existing JavaScript before adding custom Live Component actions.

### 1. Create custom PHP Component

`src/Twig/Component/Resource/FormComponent.php`:
```php
<?php

declare(strict_types=1);

namespace Plugin\Twig\Component\Resource;

use Plugin\Model\ResourceInterface;
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
    /** @use ResourceFormComponentTrait<ResourceInterface> */
    use ResourceFormComponentTrait;
    use TemplatePropTrait;

    public function __construct(
        RepositoryInterface $resourceRepository,
        FormFactoryInterface $formFactory,
        string $resourceClass,
        string $formClass,
        protected readonly SlugGeneratorInterface $slugGenerator,
    ) {
        $this->initialize($resourceRepository, $formFactory, $resourceClass, $formClass);
    }

    #[LiveAction]
    public function generateSlug(#[LiveArg] string $localeCode): void
    {
        $this->formValues['translations'][$localeCode]['slug'] =
            $this->slugGenerator->generate($this->formValues['translations'][$localeCode]['name']);
    }
}
```

### 2. Update service definition

Change the service class to your custom component:

```xml
<service
    id="plugin.admin.twig.component.resource.form"
    class="Plugin\Twig\Component\Resource\FormComponent"
>
    <argument type="service" id="plugin.repository.resource" />
    <argument type="service" id="form.factory" />
    <argument>%plugin.model.resource.class%</argument>
    <argument>Plugin\Form\Type\ResourceType</argument>
    <argument type="service" id="sylius.generator.slug" />

    <tag name="sylius.live_component.admin" key="plugin:admin:resource:form" />
</service>
```

### 3. Use the action in templates

You can now call your custom action from templates with data attributes.

---

## 📝 Alternative: Twig Hooks without Live Components

If you don't want Live Components (not recommended), don't add the component configuration. Your hooks will work with normal forms.

**Hook configuration without component:**
```yaml
sylius_twig_hooks:
    hooks:
        'plugin.admin.resource.create.content.form.sections':
            general:
                template: '@Plugin/admin/resource/form/sections/general.html.twig'
                priority: 100

        'plugin.admin.resource.create.content.form.sections.general':
            default:
                enabled: false
            code:
                template: '@Plugin/admin/resource/form/sections/general/code.html.twig'
                priority: 300
```

**What you lose:**
- No live validation
- No live form state
- Form submits like traditional forms

**What you keep:**
- Fully customizable via Twig Hooks
- Bootstrap 5 styling
- End applications can override any field

---

## 📝 Alternative: Static Templates (Not Recommended)

If you want the simplest approach with no customization, create one template with all form fields hardcoded. **You still need to use Twig Hooks**, but only a single hook pointing to one template file.

**Single template example** (`templates/admin/resource/form/sections/all_fields.html.twig`):
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
        </div>
    </div>
</div>

{{ form_rest(form) }}
```

**Hook configuration** (`config/twig_hooks/admin/resource/create.yaml`):
```yaml
sylius_twig_hooks:
    hooks:
        'plugin.admin.resource.create.content.form.sections':
            general:
                enabled: false
            all_fields:
                template: '@Plugin/admin/resource/form/sections/all_fields.html.twig'
                priority: 0
```

Copy to `update.yaml` and change `create` to `update`.

**Why not recommended:**
- End application **cannot customize** individual fields
- No flexibility for end users
- Hard to maintain for complex forms

---

## 🔍 Finding Existing Sylius Hooks

**For developers:**
- Open admin page in browser
- Open Symfony Profiler → Twig Hooks tab
- See all available hooks with their structure

---

## 🎨 Bootstrap 5 Migration

Sylius 2.0 uses Bootstrap 5 instead of Semantic UI. Use standard Bootstrap classes in your templates.

**Common patterns:**
- Container: `container`, `container-fluid`
- Grid: `row`, `col-*`
- Cards: `card`, `card-header`, `card-body`
- Buttons: `btn btn-primary`
