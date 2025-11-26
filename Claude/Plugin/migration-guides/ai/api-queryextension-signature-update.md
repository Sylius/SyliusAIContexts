# API: Update QueryExtension Signatures

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
Execute only if:
- Plugin has custom QueryExtension classes (check `src/Doctrine/ORM/QueryExtension/` directory or similar)
- Plugin implements `QueryCollectionExtensionInterface` or `QueryItemExtensionInterface`

Skip if plugin has no custom query extensions.

## Commands

### 1. Detect Existing QueryExtensions

Check for QueryExtension classes:
```
Glob: "src/**/QueryExtension/**/*.php"
Glob: "src/**/Doctrine/ORM/Extension/**/*.php"
Glob: "src/Doctrine/ORM/QueryExtension/**/*.php"
```

If none found, skip this step.

### 2. Update Each QueryExtension

For each QueryExtension file found:

#### 2a. Read Current QueryExtension

```
Read src/{path}/QueryExtension/{FileName}.php
```

Check what interfaces it implements:
- `QueryCollectionExtensionInterface` → Update `applyToCollection()` signature
- `QueryItemExtensionInterface` → Update `applyToItem()` signature

#### 2b. Update Method Signatures

The key change in API Platform 4.x is the `$operationName` parameter type:

**OLD (API Platform 2.x):**
- Parameter type: `string $operationName = null`

**NEW (API Platform 4.x):**
- Parameter type: `?Operation $operation = null`

```
Edit src/{path}/QueryExtension/{FileName}.php:

Old import:
// No Operation import needed in 2.x

New import:
use ApiPlatform\Metadata\Operation;

Old method signature (Collection):
public function applyToCollection(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    string $operationName = null,
    array $context = [],
): void {

New method signature (Collection):
public function applyToCollection(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    ?Operation $operation = null,
    array $context = [],
): void {

Old method signature (Item):
public function applyToItem(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    array $identifiers,
    string $operationName = null,
    array $context = [],
): void {

New method signature (Item):
public function applyToItem(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    array $identifiers,
    ?Operation $operation = null,
    array $context = [],
): void {
```

**Important changes:**
1. **Import:** Add `use ApiPlatform\Metadata\Operation;`
2. **Parameter name:** `$operationName` → `$operation`
3. **Parameter type:** `string` → `?Operation`
4. **Default value:** Still `null`

#### 2c. Update Method Body

If the extension checks operation name, update the logic:

```
Edit src/{path}/QueryExtension/{FileName}.php:

Old (checking operation name):
if ('shop_get_products' === $operationName) {
    // Apply filter
}

New (checking operation name via Operation object):
if ($operation?->getName() === 'shop_get_products') {
    // Apply filter
}
```

**Common patterns:**

**Pattern 1: Check operation name**
```php
Old: if ('admin_get_products' === $operationName)
New: if ($operation?->getName() === 'admin_get_products')
```

**Pattern 2: Check if operation exists**
```php
Old: if (null !== $operationName)
New: if (null !== $operation)
```

**Pattern 3: Use operation name in logic**
```php
Old: $value = $context[$operationName] ?? null;
New: $value = $context[$operation?->getName()] ?? null;
```

**Pattern 4: Get operation class**
```php
New: $operation?->getClass() // Returns resource class
```

**Pattern 5: No operation check needed**
```php
// If extension applies to all operations, no changes needed in body
public function applyToCollection(
    QueryBuilder $queryBuilder,
    QueryNameGeneratorInterface $queryNameGenerator,
    string $resourceClass,
    ?Operation $operation = null,
    array $context = [],
): void {
    // Logic that doesn't depend on operation name
    $queryBuilder->andWhere('o.enabled = :enabled')
        ->setParameter('enabled', true);
}
```

### 3. Validate Configuration

```
Bash: vendor/bin/console cache:clear
```

Expected: Cache clears without errors.

```
Bash: vendor/bin/console debug:container | grep "query.*extension"
```

Expected: Your QueryExtension service appears in the list.

## Success Criteria
- All QueryExtension signatures updated to use `?Operation $operation = null`
- All operation name checks updated to use `$operation?->getName()`
- Import statement added for `ApiPlatform\Metadata\Operation`
- Cache clears successfully

## Notes for AI
- **Signature change only**: Most extensions only need signature update, not logic changes
- **Null-safe operator**: Use `$operation?->getName()` to safely access operation name
- **Import required**: Always add `use ApiPlatform\Metadata\Operation;`
- **Backward compatible**: Using `?Operation $operation = null` maintains same behavior
- **Service registration unchanged**: No changes needed to service registration
- **Multiple methods**: If class implements both interfaces, update both methods

## Common Migration Patterns

### Pattern 1: Extension That Applies to All Operations
**Before:**
```php
<?php

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;

final class EnabledExtension implements QueryCollectionExtensionInterface
{
    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        string $operationName = null,
        array $context = [],
    ): void {
        $queryBuilder->andWhere('o.enabled = :enabled')
            ->setParameter('enabled', true);
    }
}
```

**After:**
```php
<?php

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;

final class EnabledExtension implements QueryCollectionExtensionInterface
{
    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        $queryBuilder->andWhere('o.enabled = :enabled')
            ->setParameter('enabled', true);
    }
}
```

**Changes:**
- Added `use ApiPlatform\Metadata\Operation;`
- Changed `string $operationName = null` → `?Operation $operation = null`

### Pattern 2: Extension That Checks Operation Name
**Before:**
```php
<?php

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;

final class ChannelBasedExtension implements QueryCollectionExtensionInterface
{
    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        string $operationName = null,
        array $context = [],
    ): void {
        if ('shop_get_products' !== $operationName) {
            return;
        }

        $channel = $context['channel'] ?? null;
        if (null === $channel) {
            return;
        }

        $queryBuilder->andWhere('o.channel = :channel')
            ->setParameter('channel', $channel);
    }
}
```

**After:**
```php
<?php

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;

final class ChannelBasedExtension implements QueryCollectionExtensionInterface
{
    public function applyToCollection(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        if ($operation?->getName() !== 'shop_get_products') {
            return;
        }

        $channel = $context['channel'] ?? null;
        if (null === $channel) {
            return;
        }

        $queryBuilder->andWhere('o.channel = :channel')
            ->setParameter('channel', $channel);
    }
}
```

**Changes:**
- Added `use ApiPlatform\Metadata\Operation;`
- Changed `string $operationName = null` → `?Operation $operation = null`
- Changed `'shop_get_products' !== $operationName` → `$operation?->getName() !== 'shop_get_products'`

### Pattern 3: Item Extension
**Before:**
```php
<?php

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;

final class LocaleBasedExtension implements QueryItemExtensionInterface
{
    public function applyToItem(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        array $identifiers,
        string $operationName = null,
        array $context = [],
    ): void {
        $locale = $context['locale'] ?? 'en_US';

        $queryBuilder->andWhere('o.locale = :locale')
            ->setParameter('locale', $locale);
    }
}
```

**After:**
```php
<?php

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;

final class LocaleBasedExtension implements QueryItemExtensionInterface
{
    public function applyToItem(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        array $identifiers,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        $locale = $context['locale'] ?? 'en_US';

        $queryBuilder->andWhere('o.locale = :locale')
            ->setParameter('locale', $locale);
    }
}
```

**Changes:**
- Added `use ApiPlatform\Metadata\Operation;`
- Changed `string $operationName = null` → `?Operation $operation = null`

### Pattern 4: Both Collection and Item Extension
**Before:**
```php
<?php

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;

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
        $this->addWhere($queryBuilder, $context);
    }

    public function applyToItem(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        array $identifiers,
        string $operationName = null,
        array $context = [],
    ): void {
        $this->addWhere($queryBuilder, $context);
    }

    private function addWhere(QueryBuilder $queryBuilder, array $context): void
    {
        $taxon = $context['taxon'] ?? null;
        if (null !== $taxon) {
            $queryBuilder->andWhere('o.taxon = :taxon')
                ->setParameter('taxon', $taxon);
        }
    }
}
```

**After:**
```php
<?php

namespace Vendor\Plugin\Doctrine\ORM\QueryExtension;

use ApiPlatform\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;

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
        $this->addWhere($queryBuilder, $context);
    }

    public function applyToItem(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        array $identifiers,
        ?Operation $operation = null,
        array $context = [],
    ): void {
        $this->addWhere($queryBuilder, $context);
    }

    private function addWhere(QueryBuilder $queryBuilder, array $context): void
    {
        $taxon = $context['taxon'] ?? null;
        if (null !== $taxon) {
            $queryBuilder->andWhere('o.taxon = :taxon')
                ->setParameter('taxon', $taxon);
        }
    }
}
```

**Changes:**
- Added `use ApiPlatform\Metadata\Operation;`
- Changed `string $operationName = null` → `?Operation $operation = null` in BOTH methods

## Testing (Optional)
If API endpoints are available:
```
# Test collection endpoint with QueryExtension
Bash: curl -X GET http://localhost/api/v2/shop/products -H "Accept: application/json"
```

Expected: JSON response with filtered data from QueryExtension.

```
# Test item endpoint with QueryExtension
Bash: curl -X GET http://localhost/api/v2/shop/products/1 -H "Accept: application/json"
```

Expected: JSON response with single item filtered by QueryExtension.
