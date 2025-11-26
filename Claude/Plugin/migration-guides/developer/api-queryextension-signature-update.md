# API: Update QueryExtension Signatures

This step updates API Platform 2.x QueryExtension method signatures to API Platform 4.x format.

**When to skip this step:**
- Your plugin doesn't have custom QueryExtension classes

**When to do this step:**
- You have custom QueryExtension classes implementing `QueryCollectionExtensionInterface` or `QueryItemExtensionInterface`

## Overview of Changes

API Platform 4.x changed the method signature for QueryExtensions to pass an `Operation` object instead of operation name string:

| API Platform 2.x                       | API Platform 4.x                               |
|----------------------------------------|------------------------------------------------|
| `string $operationName = null`         | `?Operation $operation = null`                 |
| Check: `'shop_get' === $operationName` | Check: `$operation?->getName() === 'shop_get'` |

**Key changes:**
- Parameter type changed from `string` to `?Operation`
- Parameter name changed from `$operationName` to `$operation`
- Must import `ApiPlatform\Metadata\Operation`
- Access operation name via `$operation?->getName()`

## 1. Identify Files to Migrate

Check if you have QueryExtensions:
```bash
find src -name "*Extension.php" -path "*/QueryExtension/*"
find src -name "*Extension.php" -path "*/Doctrine/ORM/*"
```

## 2. Update Method Signatures

### Signature Changes

**For `QueryCollectionExtensionInterface`:**

**Before (API Platform 2.x):**
```php
public function applyToCollection(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    string $operationName = null,
    array $context = [],
): void
```

**After (API Platform 4.x):**
```php
use ApiPlatform\Metadata\Operation;

public function applyToCollection(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    ?Operation $operation = null,
    array $context = [],
): void
```

**For `QueryItemExtensionInterface`:**

**Before (API Platform 2.x):**
```php
public function applyToItem(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    array $identifiers,
    string $operationName = null,
    array $context = [],
): void
```

**After (API Platform 4.x):**
```php
use ApiPlatform\Metadata\Operation;

public function applyToItem(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    array $identifiers,
    ?Operation $operation = null,
    array $context = [],
): void
```

**Required changes:**
1. Add import: `use ApiPlatform\Metadata\Operation;`
2. Change parameter type: `string $operationName = null` → `?Operation $operation = null`
3. Update any references to `$operationName` in method body

## 3. Example Migrations

### Example 1: Extension That Applies to All Operations

This is the simplest case - only signature needs updating.

**Before (API Platform 2.x):**

`src/Doctrine/ORM/QueryExtension/EnabledExtension.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;
use Vendor\Plugin\Entity\ProductInterface;

final class EnabledExtension implements QueryCollectionExtensionInterface
{
    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        string $operationName = null,
        array $context = [],
    ): void {
        if (!is_a($resourceClass, ProductInterface::class, true)) {
            return;
        }

        $rootAlias = $queryBuilder->getRootAliases()[0];
        $queryBuilder->andWhere(sprintf('%s.enabled = :enabled', $rootAlias))
            ->setParameter('enabled', true);
    }
}
```

**After (API Platform 4.x):**

```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;
use Vendor\Plugin\Entity\ProductInterface;

final class EnabledExtension implements QueryCollectionExtensionInterface
{
    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        if (!is_a($resourceClass, ProductInterface::class, true)) {
            return;
        }

        $rootAlias = $queryBuilder->getRootAliases()[0];
        $queryBuilder->andWhere(sprintf('%s.enabled = :enabled', $rootAlias))
            ->setParameter('enabled', true);
    }
}
```

**Changes:**
1. Added `use ApiPlatform\Metadata\Operation;`
2. Changed `string $operationName = null` to `?Operation $operation = null`
3. Method body unchanged (doesn't use operation name)

### Example 2: Extension That Checks Operation Name

**Before (API Platform 2.x):**

`src/Doctrine/ORM/QueryExtension/Shop/ChannelBasedExtension.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension\Shop;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;
use Sylius\Component\Channel\Context\ChannelContextInterface;
use Vendor\Plugin\Entity\ProductInterface;

final class ChannelBasedExtension implements QueryCollectionExtensionInterface
{
    public function __construct(
        private ChannelContextInterface $channelContext,
    ) {
    }

    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        string $operationName = null,
        array $context = [],
    ): void {
        if (!is_a($resourceClass, ProductInterface::class, true)) {
            return;
        }

        // Only apply to shop operations
        if ('shop_get_products' !== $operationName) {
            return;
        }

        $channel = $this->channelContext->getChannel();
        $rootAlias = $queryBuilder->getRootAliases()[0];

        $queryBuilder
            ->andWhere(sprintf('%s.channel = :channel', $rootAlias))
            ->setParameter('channel', $channel);
    }
}
```

**After (API Platform 4.x):**

```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension\Shop;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;
use Sylius\Component\Channel\Context\ChannelContextInterface;
use Vendor\Plugin\Entity\ProductInterface;

final class ChannelBasedExtension implements QueryCollectionExtensionInterface
{
    public function __construct(
        private ChannelContextInterface $channelContext,
    ) {
    }

    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        if (!is_a($resourceClass, ProductInterface::class, true)) {
            return;
        }

        // Only apply to shop operations
        if ($operation?->getName() !== 'shop_get_products') {
            return;
        }

        $channel = $this->channelContext->getChannel();
        $rootAlias = $queryBuilder->getRootAliases()[0];

        $queryBuilder
            ->andWhere(sprintf('%s.channel = :channel', $rootAlias))
            ->setParameter('channel', $channel);
    }
}
```

**Changes:**
1. Added `use ApiPlatform\Metadata\Operation;`
2. Changed `string $operationName = null` to `?Operation $operation = null`
3. Changed `'shop_get_products' !== $operationName` to `$operation?->getName() !== 'shop_get_products'`

### Example 3: Item Extension

**Before (API Platform 2.x):**

`src/Doctrine/ORM/QueryExtension/Shop/LocaleBasedExtension.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension\Shop;

use ApiPlatform\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;
use Sylius\Component\Locale\Context\LocaleContextInterface;
use Vendor\Plugin\Entity\TranslatableInterface;

final class LocaleBasedExtension implements QueryItemExtensionInterface
{
    public function __construct(
        private LocaleContextInterface $localeContext,
    ) {
    }

    public function applyToItem(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        array $identifiers,
        string $operationName = null,
        array $context = [],
    ): void {
        if (!is_a($resourceClass, TranslatableInterface::class, true)) {
            return;
        }

        $locale = $this->localeContext->getLocaleCode();
        $rootAlias = $queryBuilder->getRootAliases()[0];

        $queryBuilder
            ->innerJoin(sprintf('%s.translations', $rootAlias), 'translation')
            ->andWhere('translation.locale = :locale')
            ->setParameter('locale', $locale);
    }
}
```

**After (API Platform 4.x):**

```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension\Shop;

use ApiPlatform\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;
use Sylius\Component\Locale\Context\LocaleContextInterface;
use Vendor\Plugin\Entity\TranslatableInterface;

final class LocaleBasedExtension implements QueryItemExtensionInterface
{
    public function __construct(
        private LocaleContextInterface $localeContext,
    ) {
    }

    public function applyToItem(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        array $identifiers,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        if (!is_a($resourceClass, TranslatableInterface::class, true)) {
            return;
        }

        $locale = $this->localeContext->getLocaleCode();
        $rootAlias = $queryBuilder->getRootAliases()[0];

        $queryBuilder
            ->innerJoin(sprintf('%s.translations', $rootAlias), 'translation')
            ->andWhere('translation.locale = :locale')
            ->setParameter('locale', $locale);
    }
}
```

**Changes:**
1. Added `use ApiPlatform\Metadata\Operation;`
2. Changed `string $operationName = null` to `?Operation $operation = null`
3. Method body unchanged (doesn't use operation name)

### Example 4: Extension Implementing Both Interfaces

**Before (API Platform 2.x):**

`src/Doctrine/ORM/QueryExtension/TaxonBasedExtension.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;
use Vendor\Plugin\Entity\ProductInterface;

final class TaxonBasedExtension implements
    QueryCollectionExtensionInterface,
    QueryItemExtensionInterface
{
    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        string $operationName = null,
        array $context = [],
    ): void {
        $this->addWhere($queryBuilder, $resourceClass, $context);
    }

    public function applyToItem(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        array $identifiers,
        string $operationName = null,
        array $context = [],
    ): void {
        $this->addWhere($queryBuilder, $resourceClass, $context);
    }

    private function addWhere(QueryBuilder $queryBuilder, string $resourceClass, array $context): void
    {
        if (!is_a($resourceClass, ProductInterface::class, true)) {
            return;
        }

        $taxonCode = $context['filters']['taxon'] ?? null;
        if (null === $taxonCode) {
            return;
        }

        $rootAlias = $queryBuilder->getRootAliases()[0];
        $queryBuilder
            ->innerJoin(sprintf('%s.taxons', $rootAlias), 'taxon')
            ->andWhere('taxon.code = :taxonCode')
            ->setParameter('taxonCode', $taxonCode);
    }
}
```

**After (API Platform 4.x):**

```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;
use Vendor\Plugin\Entity\ProductInterface;

final class TaxonBasedExtension implements
    QueryCollectionExtensionInterface,
    QueryItemExtensionInterface
{
    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        $this->addWhere($queryBuilder, $resourceClass, $context);
    }

    public function applyToItem(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        array $identifiers,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        $this->addWhere($queryBuilder, $resourceClass, $context);
    }

    private function addWhere(QueryBuilder $queryBuilder, string $resourceClass, array $context): void
    {
        if (!is_a($resourceClass, ProductInterface::class, true)) {
            return;
        }

        $taxonCode = $context['filters']['taxon'] ?? null;
        if (null === $taxonCode) {
            return;
        }

        $rootAlias = $queryBuilder->getRootAliases()[0];
        $queryBuilder
            ->innerJoin(sprintf('%s.taxons', $rootAlias), 'taxon')
            ->andWhere('taxon.code = :taxonCode')
            ->setParameter('taxonCode', $taxonCode);
    }
}
```

**Changes:**
1. Added `use ApiPlatform\Metadata\Operation;`
2. Changed `string $operationName = null` to `?Operation $operation = null` in **both** `applyToCollection()` and `applyToItem()` methods
3. Method bodies unchanged (don't use operation name)

## 4. Common Operation Name Check Patterns

If your extension checks the operation name, update as follows:

### Check if operation matches specific name:
```php
// Before:
if ('shop_get_products' === $operationName) { }

// After:
if ($operation?->getName() === 'shop_get_products') { }
```

### Check if operation is NOT a specific name:
```php
// Before:
if ('admin_get_all' !== $operationName) { return; }

// After:
if ($operation?->getName() !== 'admin_get_all') { return; }
```

### Check if operation exists:
```php
// Before:
if (null !== $operationName) { }

// After:
if (null !== $operation) { }
```

### Check multiple operations:
```php
// Before:
if (in_array($operationName, ['shop_get_products', 'shop_search_products'], true)) { }

// After:
if (in_array($operation?->getName(), ['shop_get_products', 'shop_search_products'], true)) { }
```

## 5. Service Registration

**No changes needed** to service registration. QueryExtensions are auto-tagged.

Example service configuration (remains the same):
```xml
<service id="vendor.plugin.doctrine.orm.query_extension.enabled"
         class="Vendor\Plugin\Doctrine\ORM\QueryExtension\EnabledExtension"
>
    <tag name="api_platform.doctrine.orm.query_extension.collection" priority="10"/>
</service>
```

## 6. Validate Changes

Clear the cache:
```bash
vendor/bin/console cache:clear
```

Verify extensions are registered:
```bash
vendor/bin/console debug:container | grep query.*extension
```

Test API endpoints:
```bash
# Test collection endpoint
curl -X GET http://localhost/api/v2/shop/products -H "Accept: application/json"

# Test item endpoint
curl -X GET http://localhost/api/v2/shop/products/1 -H "Accept: application/json"
```

## Important Notes

1. **Import required:** Always add `use ApiPlatform\Metadata\Operation;`
2. **Signature change:** `string $operationName = null` → `?Operation $operation = null`
3. **Null-safe operator:** Use `$operation?->getName()` to safely access operation name
4. **Both methods:** If implementing both interfaces, update both `applyToCollection()` and `applyToItem()`
5. **Service registration:** No changes needed - extensions are auto-tagged
6. **Backward compatible:** The `= null` default maintains same behavior

## Additional Operation Methods

The `Operation` object provides additional useful methods:

```php
// Get operation name
$operation?->getName()

// Get resource class
$operation?->getClass()

// Get operation short name
$operation?->getShortName()

// Check operation type
$operation instanceof \ApiPlatform\Metadata\Get
$operation instanceof \ApiPlatform\Metadata\GetCollection
```

## Reference

For more examples, check Sylius core QueryExtensions:
```
vendor/sylius/sylius/src/Sylius/Bundle/ApiBundle/Doctrine/ORM/QueryExtension/
```

Example files:
- `Shop/Channel/ChannelBasedExtension.php` - Collection extension checking operation
- `Shop/Product/EnabledExtension.php` - Simple extension applying to all operations
- `Common/TranslationOrderNameAndLocaleExtension.php` - Extension using both interfaces
