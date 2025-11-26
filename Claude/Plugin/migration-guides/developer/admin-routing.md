# Admin: Routing Migration

This step updates routing configuration to match Sylius 2.0 conventions. The main changes are switching to `shared` template sets and replacing deprecated `vars` configuration with the new Twig Hooks system.

**When to skip this step:**
- Your plugin doesn't use Sylius resource routing (no `type: sylius.resource`)
- You don't reference `@SyliusAdmin` templates in routing

**When to do this step:**
- You use Sylius CRUD routes with template references

## 1. Update Admin Route Templates

In `config/routes/admin.yaml`, update the routing configuration:

**Before (Sylius 1.x):**
```yaml
plugin_admin_resource:
    resource: |
        alias: plugin.resource
        section: admin
        templates: "@SyliusAdmin\Crud"
        redirect: update
        grid: plugin_resource
        vars:
            all:
                subheader: plugin.ui.manage_resources
                templates:
                    form: "@Plugin/admin/resource/_form.html.twig"
            index:
                icon: 'check circle outline'
    type: sylius.resource
```

**After (Sylius 2.0):**
```yaml
plugin_admin_resource:
    resource: |
        alias: plugin.resource
        section: admin
        templates: "@SyliusAdmin\shared\crud"
        redirect: update
        grid: plugin_resource
        vars:
            all:
                hook_prefix: 'plugin.admin.resource'
    type: sylius.resource
```

**Key changes:**
- `templates: "@SyliusAdmin\Crud"` → `templates: "@SyliusAdmin\shared\crud"`
- Remove `vars.all.subheader` and `vars.{action}.subheader` - no longer used
- Remove `vars.all.templates.form` - replaced by Twig Hooks
- Remove `vars.all.icon` and `vars.{action}.icon` (e.g., `vars.index.icon`) - no longer used
- Add `vars.all.hook_prefix` - for Twig Hooks integration

**Note:** Do not modify action configuration (`only`, `except`) - preserve the original configuration as it reflects the plugin's needs.

## 2. Validate Routes

Check that routes load correctly:

```bash
vendor/bin/console debug:router
```

You should see your routes listed without errors. Verify that routes match your expectations.

**Note:** Form templates and other customizations will be migrated to Twig Hooks in the Templates step.
