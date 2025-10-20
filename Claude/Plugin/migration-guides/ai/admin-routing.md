# Routing Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
This step is OPTIONAL. Execute only if you use Sylius resource routing (`type: sylius.resource`).

## Commands

### 1. Find Routing Files
```
Glob: "config/routes/*.yaml"
```

For each routing file found, read and check if it uses `type: sylius.resource`.

### 2. Update Admin Routes to shared/crud Templates

```
Read {routing_file}
```

Look for routes with:
- `templates: "@SyliusAdmin\\Crud"`
- `templates: "@SyliusAdmin\\crud"`

```
Edit {routing_file}:
Replace:
- templates: "@SyliusAdmin\\Crud"
+ templates: "@SyliusAdmin\\shared\\crud"

OR

- templates: "@SyliusAdmin\\crud"
+ templates: "@SyliusAdmin\\shared\\crud"
```

### 3. Add except: [show] if Not Needed

If the route doesn't need a show action, add `except: [show]` after the `section:` line:

```
Edit {routing_file}:
Add after "section: admin":
        except: [show]
```

### 4. Remove Deprecated vars Configuration

Remove these deprecated vars that are no longer used in Sylius 2.0:

```
Edit {routing_file}:
Remove if present:
- vars.all.subheader (no longer used)
- vars.{action}.subheader (e.g., vars.index.subheader, vars.create.subheader - no longer used)
- vars.all.icon (no longer used)
- vars.{action}.icon (e.g., vars.index.icon, vars.create.icon - no longer used)
- vars.all.templates.form (replaced by Twig Hooks)
```

### 5. Add hook_prefix for Twig Hooks

Add `hook_prefix` to `vars.all` for Twig Hooks integration:

```
Edit {routing_file}:
Add to vars.all:
                hook_prefix: '{plugin_prefix}.admin.{resource_name}'
```

Example:
```yaml
vars:
    all:
        hook_prefix: 'your_plugin.admin.resource'
```

The hook_prefix should follow pattern: `{vendor_plugin}.{section}.{resource}`

### 6. Update Shop Routes to shared Templates

If you have Shop routes:

```
Read {routing_file}
```

Look for routes with `templates: "@SyliusShop\\{name}"`:

```
Edit {routing_file}:
Replace:
- templates: "@SyliusShop\\product"
+ templates: "@SyliusShop\\shared\\product"

- templates: "@SyliusShop\\{any_name}"
+ templates: "@SyliusShop\\shared\\{any_name}"
```

### 7. Validate Routes

```
Bash: vendor/bin/console debug:router 2>&1
```

Expected: Routes listed without errors. Check that:
- Your plugin routes appear in the list
- No errors about missing templates
- Routes match expectations (especially if you used `except`)

## Success Criteria
- All `@SyliusAdmin\\Crud` changed to `@SyliusAdmin\\shared\\crud`
- All `@SyliusShop\\{name}` changed to `@SyliusShop\\shared\\{name}`
- Added `except: [show]` where appropriate
- Removed deprecated vars: subheader, icon, templates.form
- Added `hook_prefix` to vars.all
- `debug:router` shows routes without errors

## Notes for AI
- This step is OPTIONAL - skip if no `type: sylius.resource` routes found
- Multiple routing files may exist in config/routes/ directory
- The `except: [show]` is optional but commonly used
- hook_prefix format: `{vendor_plugin}.{section}.{resource}` (use snake_case)
- Form templates will be migrated to Twig Hooks in Template Migration step
- Icon configuration moved to menu configuration (handled in Admin Menu step)
- Subheader configuration is completely deprecated

## Testing (Optional)
If MCP tools (playwright/chrome-devtools) are available, you can test the routing changes:
- Navigate to admin panel
- Click on menu items to verify routes work
- Check that index/create/update pages load correctly
- Report any 404 or template errors
