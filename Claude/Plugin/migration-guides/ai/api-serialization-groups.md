# API: Update Serialization Groups

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
Execute only if:
- Plugin has API resources configured
- Serialization groups are used (either in XML/YAML config or PHP attributes)

Skip if plugin has no API functionality.

## Commands

### 1. Detect Existing Serialization Configuration

Check for XML/YAML serialization config:
```
Glob: "config/serialization/*.xml"
Glob: "config/serialization/*.yaml"
Glob: "config/serialization/*.yml"
```

Check for PHP Groups attributes in entities:
```
Bash: grep -r "Groups" src/Entity/*.php
```

If neither found, skip this step (no serialization configured).

### 2. Determine Plugin Prefix

Extract plugin identifier from service parameters:
```
Grep: pattern="model\\..*\\.class%" path="config"
```

Look for pattern like `%vendor_plugin_name.model.resource.class%`

Example:
- `%bitbag_sylius_banner_plugin.model.banner.class%` → prefix is `bitbag_sylius_banner`
- `%acme_shop_plugin.model.product.class%` → prefix is `acme_shop`

**Format**: `{vendor}_{plugin_name}` (without "_plugin" suffix)

### 3a. Update XML Serialization Groups (if XML exists)

For each XML file in `config/serialization/`:

```
Read config/serialization/{Resource}.xml
```

Check current group format. Old formats to update:
- Without prefix: `shop:resource:read` or `admin:resource:read`
- Wrong prefix: `sylius:shop:resource:read`
- Legacy format: `shop:customName:read`

```
Edit config/serialization/{Resource}.xml:
Replace old group patterns with new format:
- Old: <group>shop:resource:read</group>
- New: <group>{plugin_prefix}:shop:{resource}:index</group>

- Old: <group>admin:resource:read</group>
- New: <group>{plugin_prefix}:admin:{resource}:index</group>

- Old: <group>shop:resource:write</group>
- New: <group>{plugin_prefix}:shop:{resource}:create</group>
```

**Group naming convention:**
- Collection GET: `{prefix}:shop:{resource}:index` or `{prefix}:admin:{resource}:index`
- Item GET: `{prefix}:shop:{resource}:show` or `{prefix}:admin:{resource}:show`
- POST: `{prefix}:shop:{resource}:create` or `{prefix}:admin:{resource}:create`
- PUT/PATCH: `{prefix}:shop:{resource}:update` or `{prefix}:admin:{resource}:update`

### 3b. Add PHP Groups Attributes (if no XML/YAML)

If no XML/YAML serialization config exists, add Groups attributes to entity classes:

```
Read src/Entity/{Resource}.php
```

```
Edit src/Entity/{Resource}.php:
Add at top of class:
use Symfony\Component\Serializer\Annotation\Groups;

Add to each property that should be serialized:
    #[Groups(['{plugin_prefix}:shop:{resource}:index', '{plugin_prefix}:shop:{resource}:show'])]
    protected ?int $id = null;

    #[Groups(['{plugin_prefix}:shop:{resource}:index', '{plugin_prefix}:shop:{resource}:show'])]
    protected ?string $name = null;
```

### 4. Create Missing Serialization Files

For resources that have API operations but no serialization config:

```
Glob: "config/api_resources/resources/**/*.xml"
```

For each resource XML file found, check if corresponding serialization exists:

```
Bash: test -f config/serialization/{ResourceName}.xml && echo "exists" || echo "missing"
```

If missing, create serialization XML:

```
Write config/serialization/{ResourceName}.xml:
<?xml version="1.0" ?>

<serializer xmlns="http://symfony.com/schema/dic/serializer-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/serializer-mapping
            https://symfony.com/schema/dic/serializer-mapping/serializer-mapping-1.0.xsd"
>
    <class name="%{plugin_prefix}_plugin.model.{resource}.class%">
        <attribute name="id">
            <group>{plugin_prefix}:shop:{resource}:show</group>
        </attribute>
        <!-- Add other attributes from entity -->
    </class>
</serializer>
```

**Important:** Read the entity class to determine all properties to include.

### 5. Update API Resource Files

Ensure API resource operations use the same serialization groups:

```
Read config/api_resources/resources/shop/{Resource}.xml
```

Check `<normalizationContext>` groups match serialization config:

```
Edit config/api_resources/resources/shop/{Resource}.xml:
Update operation groups to match serialization:

For GetCollection:
<value>{plugin_prefix}:shop:{resource}:index</value>

For Get (item):
<value>{plugin_prefix}:shop:{resource}:show</value>
```

Do the same for admin resources if they exist.

### 6. Validate Configuration

```
Bash: vendor/bin/console cache:clear
```

Expected: Cache clears without errors.

```
Bash: vendor/bin/console debug:router | grep "{plugin_prefix}_api"
```

Expected: API routes appear in the list.

## Success Criteria
- All serialization groups use correct prefix: `{plugin_prefix}:{section}:{resource}:{action}`
- API resource operations reference the same groups as serialization config
- Cache clears successfully
- API routes registered correctly

## Notes for AI
- **Prefix format**: Use the plugin service parameter prefix (e.g., `bitbag_sylius_banner` from `%bitbag_sylius_banner_plugin.model.banner.class%`)
- **Never use** `sylius:` prefix for plugin resources - that's reserved for Sylius core
- **Group naming**: `{vendor_plugin}:{section}:{resource}:{action}`
  - section: `shop` or `admin`
  - resource: lowercase singular (e.g., `banner`, `product`, `order`)
  - action: `index`, `show`, `create`, `update`, `delete`
- **Choose XML or PHP attributes**, not both - plugin should use one consistently
- If XML serialization exists, update it; don't add PHP attributes
- If no serialization exists, prefer XML format (matches Sylius convention)
- **Relation attributes** (like `section`, `ads`, `banners`) often don't need groups - they can be empty `<attribute name="section"></attribute>`

## Common Issues

**Issue:** Groups in XML but also in PHP attributes
**Fix:** Remove one - keep only XML or PHP attributes, not both

**Issue:** Serialization group mismatch between XML and API operations
**Fix:** Ensure exact match - copy group name from serialization to API resource

**Issue:** Wrong prefix used (e.g., `sylius:` instead of plugin prefix)
**Fix:** Replace `sylius:` with `{plugin_prefix}:` for all plugin resources

**Issue:** API returns empty response
**Fix:** Check that entity properties have serialization groups assigned

## Testing (Optional)
If API endpoints are available:
```
# Test API endpoint returns data
Bash: curl -X GET http://localhost/api/v2/shop/{resource}/{id} -H "Accept: application/json"
```

Expected: JSON response with all properties that have the serialization group.
