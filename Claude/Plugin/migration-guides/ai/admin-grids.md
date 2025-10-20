# Admin Grid Migration - AI Guide

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
This step is OPTIONAL. Execute only if the plugin defines admin grids.

## Commands

### 1. Check if Grids Exist

```bash
Tool: Glob
Pattern: *.yaml
Path: src/Resources/config/grids/
```

If files found → Proceed with migration

Also check for Grid PHP classes:

```bash
Tool: Glob
Pattern: *.php
Path: src/Grid/
```

### 2. Update Sylius Template Paths

Search for old Sylius Grid template references:

```bash
Tool: Grep
Pattern: "@SyliusAdmin/Grid/"
Path: config/grids/
Output: files_with_matches
```

For each file found:

```bash
Tool: Read
File: {grid_file}
```

Update template paths:

```bash
Tool: Edit
File: {grid_file}
Old: "@SyliusAdmin/Grid/Field/
New: "@SyliusAdmin/shared/grid/field/
```

```bash
Tool: Edit
File: {grid_file}
Old: "@SyliusAdmin/Grid/Action/
New: "@SyliusAdmin/shared/grid/action/
```

```bash
Tool: Edit
File: {grid_file}
Old: "@SyliusAdmin/Grid/Filter/
New: "@SyliusAdmin/shared/grid/filter/
```

**Also check PHP Extension files:**

```bash
Tool: Grep
Pattern: "@SyliusAdmin/Grid/"
Path: src/DependencyInjection/
Output: files_with_matches
```

Apply same replacements to PHP files.

### 3. Restore Custom Grid Templates

If the plugin has custom grid field/action/filter templates, restore them from backup.

**Important:** Sylius 2.0 requires `snake_case` directory names. Convert `CamelCase` to `snake_case`:
- `Admin/Grid/Field` → `admin/grid/field`
- `Admin/Resource/Grid` → `admin/resource/grid`

**Find custom templates:**

```bash
Tool: Bash
Command: find old_templates -path "*/grid/field/*.twig" -o -path "*/grid/action/*.twig" -o -path "*/grid/filter/*.twig"
```

For each template found:

**Read old template:**

```bash
Tool: Read
File: old_templates/{path_to_template}
```

**Create directory structure (use snake_case):**

```bash
Tool: Bash
Command: mkdir -p templates/{snake_case_path}
```

Example: If old path was `old_templates/Admin/Grid/Field`, create `templates/admin/grid/field`

**Write to new location:**

```bash
Tool: Write
File: templates/{snake_case_path}/{filename}.html.twig
Content: {content from old file}
```

**Update grid configuration if needed:**

If grid config used `CamelCase` paths, search and update:

```bash
Tool: Grep
Pattern: "@.*Plugin/.*[A-Z].*\.twig"
Path: {grid_config_file}
Output: content
```

For each CamelCase path found:

```bash
Tool: Edit
File: {grid_config_file}
Old: template: '@PluginName/Admin/Grid/Field/{filename}.html.twig'
New: template: '@PluginName/admin/grid/field/{filename}.html.twig'
```

### 4. Replace Deprecated `entities` Filter

Search for deprecated `entities` filter:

```bash
Tool: Grep
Pattern: "type: entities"
Path: config/grids/
Output: files_with_matches
```

For each file found:

```bash
Tool: Read
File: {grid_file}
```

Look for filter configuration like:

```yaml
filters:
    channels:
        type: entities
        options:
            field: "channels.id"
            form_options:
                class: "%app.model.channel.class%"
```

Replace with:

```bash
Tool: Edit
File: {grid_file}
Old: (the entire filter block with "type: entities" and "field:")
New: (same block but with "type: entity" and "fields:")
```

**Pattern:**
- Filter type: `entities` → `entity`
- Option: `field: "value"` → `fields: [value]` (string to array)
- Keep all other options unchanged

### 5. Update Icons to Tabler Format

Search for icon definitions:

```bash
Tool: Grep
Pattern: "icon:"
Path: config/grids/
Output: content
-n: true
```

For each icon found, check if it needs updating (no `tabler:` prefix).

**In YAML files:**

```bash
Tool: Edit
File: {grid_file}
Old: icon: plus
New: icon: tabler:plus
```

```bash
Tool: Edit
File: {grid_file}
Old: icon: pencil
New: icon: tabler:pencil
```

```bash
Tool: Edit
File: {grid_file}
Old: icon: trash
New: icon: tabler:trash
```

**Common mappings:**
- `icon: plus` → `icon: tabler:plus`
- `icon: pencil` → `icon: tabler:pencil`
- `icon: trash` → `icon: tabler:trash`
- `icon: eye` → `icon: tabler:eye`
- `icon: download` → `icon: tabler:download`

**In PHP Grid classes:**

```bash
Tool: Grep
Pattern: "setOptions.*icon"
Path: src/Grid/
Output: files_with_matches
```

For each file:

```bash
Tool: Read
File: {grid_class_file}
```

```bash
Tool: Edit
File: {grid_class_file}
Old: ->setOptions(['icon' => 'plus'])
New: ->setOptions(['icon' => 'tabler:plus'])
```

Apply same mapping as for YAML files.

### 6. Validate

```bash
Tool: Bash
Command: vendor/bin/console cache:clear
```

Expected: Cache clears successfully without errors.

Check that grid configs are loaded:

```bash
Tool: Bash
Command: vendor/bin/console debug:config sylius_grid
```

Expected: Your grids appear in the configuration.

### 7. Test in Browser (if MCP browser available)

If MCP browser tools are available (mcp__playwright or mcp__chrome-devtools):
- Navigate to admin grid pages for the resource
- Take screenshots to verify grids render correctly
- Check that all columns display, icons show properly, and actions work

## Success Criteria
- All Sylius template paths updated to `@SyliusAdmin/shared/grid/...`
- All custom grid templates restored from `old_templates/` to `templates/`
- All `entities` filters replaced with `entity` (and `field:` → `fields:`)
- All icons updated to Tabler format (`tabler:icon-name`)
- Cache clears successfully
- Grid configuration loads without errors
- Grid pages display correctly in browser (test via MCP)

## Common Issues

### Template not found errors

**Problem:** Error about missing `@SyliusAdmin/Grid/...` or `@SyliusAdmin/shared/grid/...` templates.

**Solution:** Most Sylius grid templates still exist in the new version, but some were removed. If a template was removed, create a custom template.

**Recommended path:**
```
templates/{resource}/grid/{type}/{name}.html.twig
```

Examples:
- `templates/catalog_promotion/grid/action/delete.html.twig`
- `templates/product/grid/field/image.html.twig`
- `templates/order/grid/filter/channel.html.twig`

Use the Write tool to create the custom template based on Sylius examples or your requirements.
