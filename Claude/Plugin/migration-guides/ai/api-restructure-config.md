# API Platform 4.x: Restructure Configuration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
Execute only if:
- Plugin has API resources configured (check for `config/api_resources*/` or old API config directory)
- API Platform 2.x XML files exist (using `metadata-2.0.xsd` schema)

Skip if plugin has no API functionality.

## Commands

### 1. Detect Old API Configuration

```
Glob: "config/api_resources*/*.xml"
```

If files found with old structure (API Platform 2.x format), proceed with migration.

### 2. Read Current API Resource Files

For each XML file found:
```
Read {api_resource_file}
```

Check if using old format:
- Schema: `metadata-2.0.xsd`
- Single file contains both `<resource>` with `<property>` and `<collectionOperations>`

### 3. Create New Directory Structure

```
Bash: mkdir -p config/api_resources/properties config/api_resources/resources/shop
```

Expected: Directories created successfully.

### 4. Split and Migrate Each Resource

For each resource XML file (e.g., `Banner.xml`, `Ad.xml`, `Section.xml`):

#### 4a. Extract and Create Properties File

```
Read {old_resource_file}
```

Extract all `<property>` elements and create new properties file:

```
Write config/api_resources/properties/{ResourceName}.xml:
<?xml version="1.0" encoding="UTF-8"?>

<properties xmlns="https://api-platform.com/schema/metadata/properties-3.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="https://api-platform.com/schema/metadata/properties-3.0 https://api-platform.com/schema/metadata/properties-3.0.xsd"
>
    <property resource="%plugin.model.resource.class%" name="id" identifier="true" writable="false"/>
    <property resource="%plugin.model.resource.class%" name="code" identifier="false" writable="false"/>
    <!-- Add all other properties from old config -->
</properties>
```

**Key transformations:**
- Update schema: `metadata-2.0.xsd` → `properties-3.0.xsd`
- Update namespace: `metadata` → `properties`
- Convert format: `<property name="id">` → `<property resource="%class%" name="id">`
- Add `resource` attribute to each property pointing to model class

#### 4b. Extract and Create Resources File (Only if has operations)

Check if old file has `<collectionOperations>` or `<itemOperations>`.

If YES, create resources file:

```
Write config/api_resources/resources/shop/{ResourceName}.xml:
<?xml version="1.0" encoding="UTF-8"?>

<resources
    xmlns="https://api-platform.com/schema/metadata/resources-3.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="https://api-platform.com/schema/metadata/resources-3.0 https://api-platform.com/schema/metadata/resources-3.0.xsd"
>
    <resource class="%plugin.model.resource.class%" shortName="ResourceName">
        <operations>
            <!-- Convert old operations to new format -->
        </operations>
    </resource>
</resources>
```

**Operation conversion:**

Old format (API Platform 2.x):
```xml
<collectionOperation name="get_ads_banners">
    <attribute name="method">GET</attribute>
    <attribute name="path">/shop/ads/banners</attribute>
</collectionOperation>
```

New format (API Platform 4.x):
```xml
<operation name="plugin_api_shop_get_ads_banners" class="ApiPlatform\Metadata\GetCollection" uriTemplate="/shop/ads/banners">
    <normalizationContext>
        <values>
            <value name="groups">
                <values>
                    <value>plugin:shop:resource:index</value>
                </values>
            </value>
        </values>
    </normalizationContext>
</operation>
```

**Operation type mapping:**
- Collection GET → `class="ApiPlatform\Metadata\GetCollection"`
- Item GET → `class="ApiPlatform\Metadata\Get"`
- Item POST → `class="ApiPlatform\Metadata\Post"`
- Item PUT → `class="ApiPlatform\Metadata\Put"`
- Item PATCH → `class="ApiPlatform\Metadata\Patch"`
- Item DELETE → `class="ApiPlatform\Metadata\Delete"`

**Important:**
- Replace `<attribute name="path">` with `uriTemplate=`
- Remove `<attribute name="method">` (implied by operation class)
- Add normalizationContext with serialization groups following pattern: `{vendor}:shop:{resource}:index` or `{vendor}:shop:{resource}:show`
- Do NOT add validation_groups at resource level (deprecated in 3.0)

If NO operations, skip creating resources file (properties-only resources are valid).

### 5. Remove Old Directory

```
Bash: rm -rf config/api_resources.TODO_migrate_step9
```

Or remove whatever the old directory name was.

### 6. Validate Configuration

```
Bash: vendor/bin/console cache:clear
```

Expected: Cache clears without errors.

```
Bash: vendor/bin/console debug:router | grep "{plugin_prefix}_api"
```

Expected: Plugin API routes appear in the list.

## Success Criteria
- New directory structure exists: `config/api_resources/properties/` and `config/api_resources/resources/shop/`
- Properties files use `properties-3.0.xsd` schema
- Resources files use `resources-3.0.xsd` schema
- Operations converted to new format with proper classes
- Old API config directory removed
- Cache clears successfully
- API routes appear in `debug:router`

## Notes for AI
- API Platform 4.x separates properties and resources into different files and directories
- Properties go in `config/api_resources/properties/{Resource}.xml`
- Resources (operations) go in `config/api_resources/resources/shop/{Resource}.xml` or `resources/admin/{Resource}.xml`
- If a resource has no operations, only create the properties file
- Sylius auto-discovers `api_resources/` directory - no manual config needed
- Serialization groups should follow pattern: `{vendor}:shop:{resource}:index` for collections, `{vendor}:shop:{resource}:show` for items
- The `shortName` in resources must match the resource name (e.g., `Banner`, `Product`)
- Use existing Sylius examples as reference: `vendor/sylius/sylius/src/Sylius/Bundle/ApiBundle/Resources/config/api_platform/`

## Common Issues

**Issue:** Cache clear fails with "Element 'attribute' not expected"
**Fix:** Remove `<attribute name="validation_groups">` from resource level - not supported in 3.0 schema

**Issue:** Cache clear fails with "Missing child element operation"
**Fix:** If resource has no operations, don't create resources file - only create properties file

**Issue:** Routes don't appear after migration
**Fix:** Ensure directory is named `api_resources` (not `api_platform`) - Sylius auto-discovers this name

## Testing (Optional)
If MCP tools are available:
```
# Test API endpoint
Bash: curl -X GET http://localhost/api/v2/shop/{resource}/{id} -H "Accept: application/json"
```

Expected: JSON response with resource data (or 404 if resource doesn't exist).
