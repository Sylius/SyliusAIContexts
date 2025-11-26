# Migration to Sylius 2.x

> **Note:** It's recommended to migrate directly from 1.14 to 2.1. Migrating 1.14 → 2.0 → 2.1 may be more time-consuming than going straight to 2.1.

## Step 1: Update composer.json

1. Update Sylius to version 2.x:
   ```bash
   composer require sylius/sylius:^2.1
   ```
2. Update related packages to versions compatible with Sylius 2.0+
3. Remove packages that don't support Sylius 2.0+

## Step 2: Green container

Goal: Get the application container running without errors.

### 2.1 Configuration changes

Apply all changes described in the **"To start off"** section of [UPGRADE-2.0.md](https://github.com/Sylius/Sylius/blob/2.0/UPGRADE-2.0.md#to-start-off):

- Update `config/packages/_sylius.yaml` (payment gateway config)
- Update `config/packages/security.yaml` (renamed firewalls, user checkers)
- Update `config/routes/sylius_shop.yaml` and `config/routes/sylius_api.yaml`
- Update `config/bundles.php` (remove old bundles, add new UX bundles)
- Add new Messenger transport variables to `.env`

### 2.2 Fix build errors

Fix errors iteratively until the container is green:

1. Clear cache or rebuild container:
   ```bash
   bin/console cache:clear
   ```
2. Identify the error (usually constructor signature changes or renamed classes)
3. Find the fix in `UPGRADE-2.0.md`
4. Apply the fix
5. Repeat until no errors

### 2.3 Run migrations

```bash
bin/console doctrine:migrations:migrate --no-interaction
```

## Step 3: Templates and assets

There are two approaches to migrating templates and assets:

### Option A: In-place modification

Directly modify existing files based on the new Sylius 2.x structure. This approach can become confusing in larger projects as it's harder to track what has been migrated and what hasn't.

### Option B: Rename and copy (recommended for larger projects)

This approach preserves old files for reference, making it easier to track migration progress and transfer customizations.

1. Rename old files and folders by adding `_old` suffix. This preserves your existing configuration and customizations, allowing you to reference them while migrating and selectively transfer your changes to the new structure:
   - `assets` → `assets_old`
   - `templates` → `templates_old`
   - `package.json` → `package.json_old`
   - `webpack.config.js` → `webpack.config.js_old`

2. Copy fresh files from [Sylius-Standard](https://github.com/Sylius/Sylius-Standard) repository (use the branch matching your target Sylius version):
   - `assets/`
   - `templates/`
   - `package.json`
   - `webpack.config.js`

3. Your customizations from `*_old` files will be migrated to the new structure as needed during the upgrade process (e.g., when updating templates or views).

### Common steps (both options)

4. Adjust the following files:
   - `assets/admin/entrypoint.js`
   - `assets/shop/entrypoint.js`
   - `package.json`
   - `.babelrc`

5. Ensure that `assets/admin/entrypoint.js` and `assets/shop/entrypoint.js` contain:
   ```js
   import './bootstrap';
   ```

6. Use the following `package.json`:
   ```json
   {
     "license": "MIT",
     "scripts": {
       "build": "encore dev",
       "build:prod": "encore production",
       "watch": "encore dev --watch"
     },
     "dependencies": {
       "@sylius-ui/admin": "file:vendor/sylius/sylius/src/Sylius/Bundle/AdminBundle",
       "@sylius-ui/shop": "file:vendor/sylius/sylius/src/Sylius/Bundle/ShopBundle",
       "@sylius/admin-bundle": "file:vendor/sylius/sylius/src/Sylius/Bundle/AdminBundle/Resources/assets",
       "@sylius/shop-bundle": "file:vendor/sylius/sylius/src/Sylius/Bundle/ShopBundle/Resources/assets",
       "@symfony/ux-autocomplete": "file:vendor/symfony/ux-autocomplete/assets",
       "@symfony/ux-live-component": "file:vendor/symfony/ux-live-component/assets",
       "core-js": "^3.47.0"
     },
     "devDependencies": {
       "@hotwired/stimulus": "^3.0.0",
       "@symfony/stimulus-bridge": "^3.2.0",
       "@symfony/webpack-encore": "^5.0.1",
       "file-loader": "^6.0.0",
       "tom-select": "^2.2.2"
     }
   }
   ```

7. Use the following `.babelrc`:
   ```json
   {
     "presets": [
       ["@babel/preset-env", {
         "useBuiltIns": "usage",
         "corejs": "3.47.0"
       }]
     ],
     "plugins": [
       ["@babel/plugin-transform-object-rest-spread", {
         "useBuiltIns": true
       }]
     ]
   }
   ```

8. Add the following import to load twig hooks configuration (e.g., in `config/packages/_sylius.yaml` or another file where configs are imported):
   ```yaml
   imports:
       - { resource: '../twig_hooks/**/*.yaml' }
   ```
   This will load all twig hooks configuration files. It's recommended to organize them in a directory structure like `config/twig_hooks/admin/` or `config/twig_hooks/shop/`.

9. Create base twig hooks for loading custom stylesheets and javascripts:

   **`config/twig_hooks/admin/base.yaml`:**
   ```yaml
   sylius_twig_hooks:
       hooks:
           'sylius_admin.base#stylesheets':
               app_styles:
                   template: 'admin/stylesheets.html.twig'
           'sylius_admin.base#javascripts':
               app_javascripts:
                   template: 'admin/javascripts.html.twig'
   ```

   **`config/twig_hooks/shop/base.yaml`:**
   ```yaml
   sylius_twig_hooks:
       hooks:
           'sylius_shop.base#stylesheets':
               app_styles:
                   template: 'shop/stylesheets.html.twig'
           'sylius_shop.base#javascripts':
               app_javascripts:
                   template: 'shop/javascripts.html.twig'
   ```

10. Create the stylesheet and javascript templates:

    **`templates/admin/stylesheets.html.twig`:**
    ```twig
    {{ encore_entry_link_tags('app-admin-entry', null, 'app.admin') }}
    ```

    **`templates/admin/javascripts.html.twig`:**
    ```twig
    {{ encore_entry_script_tags('app-admin-entry', null, 'app.admin') }}
    ```

    **`templates/shop/stylesheets.html.twig`:**
    ```twig
    {{ encore_entry_link_tags('app-shop-entry', null, 'app.shop') }}
    ```

    **`templates/shop/javascripts.html.twig`:**
    ```twig
    {{ encore_entry_script_tags('app-shop-entry', null, 'app.shop') }}
    ```

11. Ensure that the entry names (e.g., `app-admin-entry`, `app-shop-entry`) and build names (e.g., `app.admin`, `app.shop`) are configured in `config/packages/framework.yaml` under `framework.assets` and in `config/packages/webpack_encore.yaml` under `webpack_encore.builds`.

## Step 4: Plugins

### 4.1 Update plugins

For each plugin:
1. Check if the plugin has an upgrade file (e.g., `UPGRADE.md`, `CHANGELOG.md`) and review what has changed
2. Update the plugin to a version compatible with Sylius 2.x
3. Apply any required configuration changes according to the plugin's upgrade guide

### 4.2 Add plugin assets

Sylius 2.0 changed the approach to plugin assets. Previously, plugins handled their assets in various ways. Now there is a unified approach: if a plugin has its own assets, they should be explicitly imported in the entrypoint files.

Import plugin assets in the appropriate entrypoint file (`assets/shop/entrypoint.js` or `assets/admin/entrypoint.js`):

```js
import './bootstrap';

import '@vendor/sylius/custom-plugin/assets/shop/entrypoint';
```

Repeat for each plugin that provides assets.

## Step 5: Admin

### 5.1 CRUD resources

#### 5.1.1 Update route configuration

For each admin CRUD resource, update the route configuration:

**Before (Sylius 1.x):**
```yaml
app_resource:
    resource: |
        alias: app.resource
        section: admin
        templates: "@SyliusAdmin/Crud"
        redirect: update
        grid: app_resource
        vars:
            all:
                header: app.ui.resource
                templates:
                    form: "@SyliusAdmin/Resource/_form.html.twig"
            index:
                icon: some icon
    type: sylius.resource
```

**After (Sylius 2.x):**
```yaml
app_resource:
    resource: |
        alias: app.resource
        section: admin
        except: [show]
        templates: "@SyliusAdmin/shared/crud"
        redirect: update
        grid: app_resource
        vars:
            all:
                hook_prefix: 'app.admin.resource'
    type: sylius.resource
```

Key changes:
- `templates: "@SyliusAdmin/Crud"` → `templates: "@SyliusAdmin/shared/crud"`
- Remove `header`, `templates.form`, `icon` from `vars` (no longer supported)
- Add `hook_prefix` in `vars.all`
- Add `except: [show]` if you don't have a show template for this resource

#### 5.1.2 Create twig hooks configuration

Create twig hooks configuration files for create and update actions. They should look the same, just with different action names.

**`config/twig_hooks/admin/resource/create.yaml`:**
```yaml
sylius_twig_hooks:
    hooks:
        'app.admin.resource.create.content.form.sections':
            general:
                template: 'admin/resource/form/sections/general.html.twig'
                priority: 0
```

**`config/twig_hooks/admin/resource/update.yaml`:**
```yaml
sylius_twig_hooks:
    hooks:
        'app.admin.resource.update.content.form.sections':
            general:
                template: 'admin/resource/form/sections/general.html.twig'
                priority: 0
```

#### 5.1.3 Create form templates

Create a single template with all form fields. There's no need to split it into multiple twig hooks if no one will extend it.

**`templates/admin/resource/form/sections/general.html.twig`:**
```twig
{% import '@SyliusAdmin/shared/helper/translations.html.twig' as translations %}
{% set form = hookable_metadata.context.form %}

<div class="card mb-3">
    <div class="card-header">
        <div class="card-title">
            {{ 'sylius.ui.general'|trans }}
        </div>
    </div>
    <div class="card-body">
        <div class="row">
            <div class="col-12">
                {{ form_row(form.fieldName) }}
            </div>
        </div>
    </div>
</div>

{# If the resource has translations #}
<div class="card mb-3">
    <div class="card-header">
        <div class="card-title">{{ 'sylius.ui.translations'|trans }}</div>
    </div>
    <div class="card-body">
        <div class="row">
            {{ translations.default(form.translations) }}
        </div>
    </div>
</div>
```

#### 5.1.4 Add LiveComponent to a form (optional)

To make a form reactive using Symfony UX LiveComponent:

1. Register the LiveComponent service (e.g., in `config/services/twig_component.yaml`):
   ```yaml
   services:
       app.admin.twig.component.resource.form:
           class: Sylius\Bundle\UiBundle\Twig\Component\ResourceFormComponent
           arguments:
               - '@app.repository.resource'
               - '@form.factory'
               - '%app.model.resource.class%'
               - 'App\Form\Type\ResourceType'
           tags:
               - { name: 'sylius.live_component.admin', key: 'app:admin:resource:form' }
   ```

2. Update your twig hooks to use the component instead of a template:

   **`config/twig_hooks/admin/resource/create.yaml`:**
   ```yaml
   sylius_twig_hooks:
       hooks:
           'app.admin.resource.create.content':
               form:
                   component: 'app:admin:resource:form'
                   props:
                       resource: '@=_context.resource'
                       form: '@=_context.form'
                       template: '@SyliusAdmin/shared/crud/common/content/form.html.twig'
                   priority: 0
           'app.admin.resource.create.content.form.sections':
               general:
                   template: 'admin/resource/form/sections/general.html.twig'
                   priority: 0
   ```

   **`config/twig_hooks/admin/resource/update.yaml`:**
   ```yaml
   sylius_twig_hooks:
       hooks:
           'app.admin.resource.update.content':
               form:
                   component: 'app:admin:resource:form'
                   props:
                       resource: '@=_context.resource'
                       form: '@=_context.form'
                       template: '@SyliusAdmin/shared/crud/common/content/form.html.twig'
                   priority: 0
           'app.admin.resource.update.content.form.sections':
               general:
                   template: 'admin/resource/form/sections/general.html.twig'
                   priority: 0
   ```

### 5.2 Menu

(TODO)

## Common issues

### 1. "No locale has been set and current locale is undefined" error

If you encounter this error:
```
An error occurred during rendering the "form" hook in the "sylius_admin.common.create.content" hookable.
An exception has been thrown during the rendering of a template ("No locale has been set and current locale is undefined.")
```

This happens because `resource` is not defined in the default form template. To fix this, copy the default form template and modify it:

1. Copy `@SyliusAdmin/shared/crud/common/content/form.html.twig` to your templates (e.g., `templates/admin/resource/form.html.twig`)

2. In the copied file, change:
```twig
{% if resource is not defined %}
    {% set resource = hookable_metadata.context.resource|default(null) %}
{% endif %}
```
to:
```twig
{% set resource = hookable_metadata.context.resource|default(null) %}
```

3. Update your create.yaml to include the form hook:
```yaml
sylius_twig_hooks:
    hooks:
        'app.admin.resource.create.content':
            form:
                template: 'admin/resource/form.html.twig'
                priority: 0
        'app.admin.resource.create.content.form.sections':
            general:
                template: 'admin/resource/form/sections/general.html.twig'
                priority: 0
```
