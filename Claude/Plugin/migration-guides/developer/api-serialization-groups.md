# API: Update Serialization Groups

This step updates serialization group naming to follow Sylius 2.0 conventions with proper plugin-specific prefixes.

**When to skip this step:**
- Your plugin doesn't have API functionality
- No serialization groups are configured

**When to do this step:**
- You have API resources with serialization groups in XML/YAML or PHP attributes

## Overview of Changes

Sylius 2.0 requires all plugin serialization groups to use a vendor/plugin-specific prefix to avoid naming conflicts.

**Old format (no prefix or wrong prefix):**
```
shop:banner:read
admin:product:index
sylius:shop:custom:read  ← Wrong! (sylius prefix is for core only)
```

**New format (with plugin prefix):**
```
bitbag_sylius_banner:shop:banner:index
bitbag_sylius_banner:admin:banner:show
acme_shop:shop:product:create
```

## 1. Determine Your Plugin Prefix

Your plugin prefix should match your service parameter naming.

Check `config/services.xml` or `config/packages/*.yaml` for parameter names like:
```
%bitbag_sylius_banner_plugin.model.banner.class%
```

Extract the prefix: `bitbag_sylius_banner` (everything before `_plugin`)

**Prefix format**: `{vendor}_{plugin_name}` without the "_plugin" suffix

Examples:
- BitBag SyliusBannerPlugin → `bitbag_sylius_banner`
- Acme ShopExtensionPlugin → `acme_shop_extension`

## 2. Check Current Serialization Configuration

Serialization can be configured in two ways:

### Option A: XML/YAML Files

Check `config/serialization/` directory:
```bash
ls config/serialization/
```

### Option B: PHP Attributes

Check entity files for `#[Groups]` attributes:
```bash
grep -r "Groups" src/Entity/
```

**Important:** Use only ONE method (XML or PHP attributes), not both.

## 3. Update Serialization Groups

### If using XML (Recommended)

**Before:**

`config/serialization/Banner.xml`:
```xml
<?xml version="1.0" ?>

<serializer xmlns="http://symfony.com/schema/dic/serializer-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/serializer-mapping
            https://symfony.com/schema/dic/serializer-mapping/serializer-mapping-1.0.xsd"
>
    <class name="%bitbag_sylius_banner_plugin.model.banner.class%">
        <attribute name="id">
            <group>shop:banner:read</group>
        </attribute>
        <attribute name="path">
            <group>shop:banner:read</group>
        </attribute>
    </class>
</serializer>
```

**After:**

```xml
<?xml version="1.0" ?>

<serializer xmlns="http://symfony.com/schema/dic/serializer-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/serializer-mapping
            https://symfony.com/schema/dic/serializer-mapping/serializer-mapping-1.0.xsd"
>
    <class name="%bitbag_sylius_banner_plugin.model.banner.class%">
        <attribute name="id">
            <group>bitbag_sylius_banner:shop:banner:index</group>
        </attribute>
        <attribute name="path">
            <group>bitbag_sylius_banner:shop:banner:index</group>
        </attribute>
    </class>
</serializer>
```

### If using PHP Attributes

**Before:**

```php
use Symfony\Component\Serializer\Annotation\Groups;

class Banner implements BannerInterface
{
    #[Groups(['shop:banner:read'])]
    protected ?int $id = null;

    #[Groups(['shop:banner:read'])]
    protected ?string $path = null;
}
```

**After:**

```php
use Symfony\Component\Serializer\Annotation\Groups;

class Banner implements BannerInterface
{
    #[Groups(['bitbag_sylius_banner:shop:banner:index'])]
    protected ?int $id = null;

    #[Groups(['bitbag_sylius_banner:shop:banner:index'])]
    protected ?string $path = null;
}
```

## 4. Group Naming Convention

Follow this pattern for group names:

```
{plugin_prefix}:{section}:{resource}:{action}
```

**Components:**
- `plugin_prefix`: Your plugin prefix (e.g., `bitbag_sylius_banner`)
- `section`: `shop` or `admin`
- `resource`: Singular lowercase resource name (e.g., `banner`, `section`, `ad`)
- `action`: Operation type

**Action types:**
| API Operation | Group Action | Description |
|--------------|-------------|-------------|
| GetCollection | `index` | List/collection endpoint |
| Get (item) | `show` | Single item endpoint |
| Post | `create` | Create new resource |
| Put/Patch | `update` | Update existing resource |
| Delete | `delete` | Delete resource |

**Examples:**
- `bitbag_sylius_banner:shop:banner:index` - List banners in shop
- `bitbag_sylius_banner:shop:banner:show` - Get single banner in shop
- `bitbag_sylius_banner:admin:section:create` - Create section in admin
- `bitbag_sylius_banner:admin:ad:update` - Update ad in admin

## 5. Handle Relations

For related entities (like `section`, `banners`, `ads`), you have two options:

### Option 1: No serialization group (empty)
```xml
<attribute name="section">
</attribute>
```

Related entity won't be serialized (only IRI will be returned).

### Option 2: Add nested group
```xml
<attribute name="section">
    <group>bitbag_sylius_banner:shop:banner:index</group>
</attribute>
```

Related entity will be embedded with its own serialization groups.

## 6. Update API Resource Operations

Ensure API resource operations reference the same groups.

`config/api_resources/resources/shop/Banner.xml`:

**Before:**
```xml
<operation name="bitbag_api_shop_get_banners" class="ApiPlatform\Metadata\GetCollection" uriTemplate="/shop/banners">
    <normalizationContext>
        <values>
            <value name="groups">
                <values>
                    <value>shop:banner:read</value>
                </values>
            </value>
        </values>
    </normalizationContext>
</operation>
```

**After:**
```xml
<operation name="bitbag_api_shop_get_banners" class="ApiPlatform\Metadata\GetCollection" uriTemplate="/shop/banners">
    <normalizationContext>
        <values>
            <value name="groups">
                <values>
                    <value>bitbag_sylius_banner:shop:banner:index</value>
                </values>
            </value>
        </values>
    </normalizationContext>
</operation>
```

## 7. Create Missing Serialization Files

If you have API resources without serialization configuration, create the files.

For each resource in `config/api_resources/resources/`, create a corresponding file in `config/serialization/`:

`config/serialization/Ad.xml`:
```xml
<?xml version="1.0" ?>

<serializer xmlns="http://symfony.com/schema/dic/serializer-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/serializer-mapping
            https://symfony.com/schema/dic/serializer-mapping/serializer-mapping-1.0.xsd"
>
    <class name="%bitbag_sylius_banner_plugin.model.ad.class%">
        <attribute name="id">
            <group>bitbag_sylius_banner:shop:ad:show</group>
        </attribute>
        <attribute name="name">
            <group>bitbag_sylius_banner:shop:ad:show</group>
        </attribute>
        <attribute name="code">
            <group>bitbag_sylius_banner:shop:ad:show</group>
        </attribute>
        <!-- Add other attributes -->
    </class>
</serializer>
```

## 8. Validate Changes

Clear the cache:
```bash
vendor/bin/console cache:clear
```

Verify routes are registered:
```bash
vendor/bin/console debug:router | grep your_plugin_api
```

## 9. Test API Endpoints

Test that serialization works correctly:

```bash
# Test GET collection
curl -X GET http://localhost/api/v2/shop/banners -H "Accept: application/json"

# Test GET item
curl -X GET http://localhost/api/v2/shop/banners/1 -H "Accept: application/json"
```

Verify that:
- Response includes all properties with the serialization group
- Related entities are handled correctly (embedded or IRI-only)
- No 500 errors or serialization exceptions

## Important Notes

1. **Never use `sylius:` prefix** for plugin resources - it's reserved for Sylius core
2. **Be consistent**: Use the same prefix across all your serialization groups
3. **Match exactly**: API operation groups must match serialization config groups exactly
4. **One method only**: Don't mix XML and PHP attributes in the same project
5. **Follow the pattern**: Always use `{prefix}:{section}:{resource}:{action}` format

## Common Mistakes

❌ **Wrong:**
```xml
<group>sylius:shop:banner:index</group>  <!-- Using sylius prefix for plugin -->
<group>bitbag:shop:banner:read</group>   <!-- Inconsistent action naming -->
<group>shop:banner:index</group>         <!-- Missing plugin prefix -->
```

✅ **Correct:**
```xml
<group>bitbag_sylius_banner:shop:banner:index</group>
<group>bitbag_sylius_banner:shop:banner:show</group>
<group>bitbag_sylius_banner:admin:banner:create</group>
```

## Reference

For more examples, check Sylius core serialization:
```
vendor/sylius/sylius/src/Sylius/Bundle/ApiBundle/Resources/config/serialization/
```
