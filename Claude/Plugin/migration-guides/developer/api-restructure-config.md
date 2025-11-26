# API: Restructure Configuration

This step migrates API Platform configuration from version 2.x to 4.x format. The main change is splitting resource definitions into separate files for properties and operations.

**When to skip this step:**
- Your plugin doesn't have API functionality
- No API resource configuration files exist

**When to do this step:**
- You have API resources configured with API Platform 2.x format (single XML files with both properties and operations)

## Overview of Changes

API Platform 4.x introduces a clearer separation of concerns:

**Old structure (2.x):**
```
config/api_resources/
└── Resource.xml  (contains both properties AND operations)
```

**New structure (4.x):**
```
config/api_resources/
├── properties/
│   └── Resource.xml  (property metadata only)
└── resources/
    ├── shop/
    │   └── Resource.xml  (operations only)
    └── admin/
        └── Resource.xml  (operations only)
```

## 1. Create New Directory Structure

Create the directories for the new structure:

```bash
mkdir -p config/api_resources/properties
mkdir -p config/api_resources/resources/shop
mkdir -p config/api_resources/resources/admin  # if you have admin API
```

## 2. Split Resource Configuration

For each resource, you need to create two separate files.

### Example: Banner Resource

**Before (API Platform 2.x) - Single file:**

`config/api_resources/Banner.xml`:
```xml
<?xml version="1.0" ?>

<resources xmlns="https://api-platform.com/schema/metadata"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="https://api-platform.com/schema/metadata
           https://api-platform.com/schema/metadata/metadata-2.0.xsd"
>
    <resource class="%plugin.model.banner.class%" shortName="Banner">
        <attribute name="validation_groups">sylius</attribute>

        <collectionOperations>
            <collectionOperation name="get_banners">
                <attribute name="method">GET</attribute>
                <attribute name="path">/shop/banners</attribute>
            </collectionOperation>
        </collectionOperations>

        <property name="path" writable="false" />
        <property name="alt" writable="false" />
        <property name="id" identifier="true" writable="false" />
    </resource>
</resources>
```

**After (API Platform 4.x) - Two files:**

`config/api_resources/properties/Banner.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>

<properties xmlns="https://api-platform.com/schema/metadata/properties-3.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="https://api-platform.com/schema/metadata/properties-3.0 https://api-platform.com/schema/metadata/properties-3.0.xsd"
>
    <property resource="%plugin.model.banner.class%" name="id" identifier="true" writable="false"/>
    <property resource="%plugin.model.banner.class%" name="path" writable="false"/>
    <property resource="%plugin.model.banner.class%" name="alt" writable="false"/>
</properties>
```

`config/api_resources/resources/shop/Banner.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>

<resources
    xmlns="https://api-platform.com/schema/metadata/resources-3.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="https://api-platform.com/schema/metadata/resources-3.0 https://api-platform.com/schema/metadata/resources-3.0.xsd"
>
    <resource class="%plugin.model.banner.class%" shortName="Banner">
        <operations>
            <operation name="plugin_api_shop_get_banners" class="ApiPlatform\Metadata\GetCollection" uriTemplate="/shop/banners">
                <normalizationContext>
                    <values>
                        <value name="groups">
                            <values>
                                <value>plugin:shop:banner:index</value>
                            </values>
                        </value>
                    </values>
                </normalizationContext>
            </operation>
        </operations>
    </resource>
</resources>
```

## 3. Key Changes to Note

### Schema Updates

- **Properties:** `metadata-2.0.xsd` → `properties-3.0.xsd`
- **Resources:** `metadata-2.0.xsd` → `resources-3.0.xsd`

### Properties Format

Old:
```xml
<property name="id" identifier="true" writable="false" />
```

New:
```xml
<property resource="%plugin.model.resource.class%" name="id" identifier="true" writable="false"/>
```

Each property must now include the `resource` attribute pointing to the model class.

### Operations Format

Old:
```xml
<collectionOperation name="get_items">
    <attribute name="method">GET</attribute>
    <attribute name="path">/shop/items</attribute>
</collectionOperation>
```

New:
```xml
<operation name="plugin_api_shop_get_items" class="ApiPlatform\Metadata\GetCollection" uriTemplate="/shop/items">
    <normalizationContext>
        <values>
            <value name="groups">
                <values>
                    <value>plugin:shop:item:index</value>
                </values>
            </value>
        </values>
    </normalizationContext>
</operation>
```

### Operation Type Mapping

| Old Operation Type | New Class |
|-------------------|-----------|
| Collection GET | `ApiPlatform\Metadata\GetCollection` |
| Item GET | `ApiPlatform\Metadata\Get` |
| POST | `ApiPlatform\Metadata\Post` |
| PUT | `ApiPlatform\Metadata\Put` |
| PATCH | `ApiPlatform\Metadata\Patch` |
| DELETE | `ApiPlatform\Metadata\Delete` |

### Serialization Groups

Follow the Sylius convention with vendor prefix:

- Collection: `{vendor}:shop:{resource}:index`
- Item: `{vendor}:shop:{resource}:show`

Example:
- `bitbag:shop:banner:index`
- `sylius:admin:product:show`

## 4. Resources Without Operations

If a resource has no API operations (properties only), you only need to create the properties file. You can skip creating the resources file.

Example for a read-only resource accessible only through relations:

`config/api_resources/properties/BannerTranslation.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>

<properties xmlns="https://api-platform.com/schema/metadata/properties-3.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="https://api-platform.com/schema/metadata/properties-3.0 https://api-platform.com/schema/metadata/properties-3.0.xsd"
>
    <property resource="%plugin.model.banner_translation.class%" name="id" identifier="false" writable="false"/>
    <property resource="%plugin.model.banner_translation.class%" name="locale" identifier="true" writable="false"/>
    <property resource="%plugin.model.banner_translation.class%" name="title" writable="false"/>
</properties>
```

No resources file needed.

## 5. Remove Old Configuration

After successfully migrating all resources, remove the old directory:

```bash
rm -rf config/api_resources.OLD  # or whatever the old directory was named
```

## 6. Validate Changes

Clear the cache and verify routes are registered:

```bash
vendor/bin/console cache:clear
vendor/bin/console debug:router | grep your_plugin_api
```

You should see your API routes listed.

## 7. Test API Endpoints

Test the migrated endpoints:

```bash
# Test GET collection
curl -X GET http://localhost/api/v2/shop/banners -H "Accept: application/json"

# Test GET item
curl -X GET http://localhost/api/v2/shop/banners/1 -H "Accept: application/json"
```

## Important Notes

1. **No manual configuration needed:** Sylius auto-discovers the `config/api_resources/` directory
2. **Validation groups:** Don't add `<attribute name="validation_groups">` at resource level in 4.x - it's not supported
3. **Empty operations:** If `<operations>` tag is empty, remove the resources file entirely
4. **Identifier selection:** Use `code` as identifier when available, otherwise `id`

## Reference

For more examples, check Sylius core API configuration:
```
vendor/sylius/sylius/src/Sylius/Bundle/ApiBundle/Resources/config/api_platform/
```
