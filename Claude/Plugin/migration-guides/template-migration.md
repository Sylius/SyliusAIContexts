# Template Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Create Hook Configuration
```
Bash: mkdir -p config/twig_hooks/admin/{resource_name}
```

### 2. Create Hook Files
For each resource:
```
Write config/twig_hooks/admin/{resource}/create.yaml:
sylius_twig_hooks:
    hooks:
        'sylius_admin.{resource}.create.content.form.sections':
            general:
                template: '@{PluginName}/admin/{resource}/form/sections/general.html.twig'
                priority: 100

Write config/twig_hooks/admin/{resource}/update.yaml:
sylius_twig_hooks:
    hooks:
        'sylius_admin.{resource}.update.content.form.sections':
            general:
                template: '@{PluginName}/admin/{resource}/form/sections/general.html.twig'
                priority: 100
```

### 3. Convert Semantic UI to Bootstrap 5
```
Grep: "class=\"ui " --include="*.html.twig"
```
For each occurrence:
```
Edit {template}:
- class="ui form" → class="needs-validation"
- class="ui button" → class="btn btn-primary"
- class="ui segment" → class="card"
- class="ui grid" → class="row"
- class="column" → class="col-md-6"
- class="ui dropdown" → class="form-select"
- class="ui input" → class="form-control"
- class="ui message" → class="alert alert-info"
```

### 4. Create Template Sections
```
Write templates/admin/{resource}/form/sections/general.html.twig:
{% set form = hookable_metadata.context.form %}

<div class="card mb-3">
    <div class="card-header">
        <h3 class="card-title">General Information</h3>
    </div>
    <div class="card-body">
        <div class="row">
            <div class="col-md-6">
                {{ form_row(form.{field}, {'attr': {'class': 'form-control'}}) }}
            </div>
        </div>
    </div>
</div>
```

### 5. Update Icons
```
Grep: "icon" --include="*.html.twig"
```
Replace icons:
```
Edit {template}:
- <i class="images outline icon"></i> → <i class="tabler-icon tabler-icon-photo"></i>
- <i class="adversal icon"></i> → <i class="tabler-icon tabler-icon-ad"></i>
```

### 6. Update Config to Import Hooks
```
Edit config/config.yaml:
Add to imports:
    - { resource: "twig_hooks/**/*.yaml" }
```

### 7. Validate
```
Bash: vendor/bin/console debug:twig-hooks
```
Expected: Plugin hooks appear in list

### 8. Commit Changes
```
Bash: git add .
Bash: git commit -m "Convert templates to use Twig hooks and Bootstrap 5"
```