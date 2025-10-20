# API: Migrate DataProvider to StateProvider

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
Execute only if:
- Plugin has custom DataProvider classes (check `src/DataProvider/` directory)
- Plugin has custom DataPersister classes (check `src/DataPersister/` directory)

Skip if plugin has no custom data providers or persisters.

## Commands

### 1. Detect Existing DataProviders and DataPersisters

Check for DataProvider classes:
```
Glob: "src/DataProvider/*.php"
Glob: "src/**/DataProvider/*.php"
```

Check for DataPersister classes:
```
Glob: "src/DataPersister/*.php"
Glob: "src/**/DataPersister/*.php"
```

If neither found, skip this step.

### 2. Migrate Each DataProvider

For each DataProvider file found:

#### 2a. Read Current DataProvider

```
Read src/DataProvider/{FileName}.php
```

Check what interfaces it implements:
- `CollectionDataProviderInterface` → Migrate to `ProviderInterface` (GetCollection)
- `ItemDataProviderInterface` → Migrate to `ProviderInterface` (Get item)
- `ContextAwareCollectionDataProviderInterface` → Migrate to `ProviderInterface`

#### 2b. Determine New Directory Structure

Follow Sylius pattern: `src/StateProvider/{Section}/{Resource}/{Type}Provider.php`

Example mappings:
- `GetProductsDataProvider` → `src/StateProvider/Shop/Product/CollectionProvider.php`
- `GetProductDataProvider` → `src/StateProvider/Shop/Product/ItemProvider.php`
- `GetAdsBannersDataProvider` → `src/StateProvider/Shop/Banner/CollectionProvider.php`

Pattern:
- Section: `Shop` or `Admin` (based on API section)
- Resource: Entity name (singular, e.g., `Product`, `Banner`)
- Type: `CollectionProvider` for collections, `ItemProvider` for single items

#### 2c. Create New StateProvider

```
Bash: mkdir -p src/StateProvider/{Section}/{Resource}
```

```
Write src/StateProvider/{Section}/{Resource}/{Type}Provider.php:
<?php

declare(strict_types=1);

namespace {Vendor}\{Plugin}\StateProvider\{Section}\{Resource};

use ApiPlatform\Metadata\Operation;
use ApiPlatform\State\ProviderInterface;

/** @implements ProviderInterface<{ResourceInterface}> */
final readonly class {Type}Provider implements ProviderInterface
{
    public function __construct(
        // Copy constructor arguments from old DataProvider
    ) {
    }

    public function provide(Operation $operation, array $uriVariables = [], array $context = []): array|object|null
    {
        // Migrate logic from old getCollection() or getItem() method
        // Access filters via: $context['filters']
        // Access URI variables via: $uriVariables (e.g., $uriVariables['id'])

        return /* result */;
    }
}
```

**Key changes:**
- Implements `ProviderInterface` instead of `CollectionDataProviderInterface` or `ItemDataProviderInterface`
- Method `getCollection()` or `getItem()` → `provide()`
- Add `Operation $operation` as first parameter
- Return type: `array|object|null` (array for collections, object for items, null if not found)
- Remove `supports()` method - no longer needed (handled by operation linking)
- Mark class as `readonly final` following Sylius conventions
- Add PHPDoc: `@implements ProviderInterface<ResourceInterface>`

#### 2d. Update Service Registration

Check for old service registration:
```
Grep: pattern="{OldClassName}" path="config/services"
```

Create new service file:
```
Write config/services/stateProvider/stateProvider.xml (or append to existing):
<service id="{vendor}.{plugin}.state_provider.{section}.{resource}.{type}"
         class="{Vendor}\{Plugin}\StateProvider\{Section}\{Resource}\{Type}Provider"
>
    <!-- Copy arguments from old service -->
    <argument type="service" id="..." />
    <tag name="api_platform.state_provider" priority="10"/>
</service>
```

**Service ID pattern:**
`{vendor}.{plugin}.state_provider.{section}.{resource}.{type}`

Example: `bitbag.sylius_banner_plugin.state_provider.shop.banner.collection_provider`

**Important:** Remove `$class` or `$resourceClass` parameter - no longer needed in API Platform 4.x

#### 2e. Link StateProvider to Operations

Find API resource operations using this provider:
```
Read config/api_resources/resources/{section}/{Resource}.xml
```

Update operation to reference the new StateProvider:
```
Edit config/api_resources/resources/{section}/{Resource}.xml:

Old (API Platform 2.x - implicit):
<operation name="..." class="ApiPlatform\Metadata\GetCollection" uriTemplate="...">
    <!-- No provider specified, used DataProvider with supports() -->
</operation>

New (API Platform 4.x - explicit):
<operation name="..." class="ApiPlatform\Metadata\GetCollection" uriTemplate="..." provider="{service_id}">
    <!-- Explicitly links to StateProvider service -->
</operation>
```

Add `provider="{service_id}"` attribute to the operation.

### 3. Migrate Each DataPersister (if exists)

For each DataPersister file found:

#### 3a. Read Current DataPersister

```
Read src/DataPersister/{FileName}.php
```

#### 3b. Create New StateProcessor

```
Bash: mkdir -p src/StateProcessor/{Section}/{Resource}
```

```
Write src/StateProcessor/{Section}/{Resource}/{Type}Processor.php:
<?php

declare(strict_types=1);

namespace {Vendor}\{Plugin}\StateProcessor\{Section}\{Resource};

use ApiPlatform\Metadata\Operation;
use ApiPlatform\State\ProcessorInterface;

/** @implements ProcessorInterface<{ResourceInterface}> */
final readonly class {Type}Processor implements ProcessorInterface
{
    public function __construct(
        // Copy constructor arguments
    ) {
    }

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        // Migrate logic from persist() method

        return $data;
    }
}
```

**Key changes:**
- Implements `ProcessorInterface` instead of `DataPersisterInterface`
- Method `persist()` → `process()`
- Add `Operation $operation` parameter
- Return type: `mixed` (return the persisted data)

#### 3c. Register StateProcessor Service

```
Write config/services/stateProcessor/stateProcessor.xml:
<service id="{vendor}.{plugin}.state_processor.{section}.{resource}.{type}"
         class="{Vendor}\{Plugin}\StateProcessor\{Section}\{Resource}\{Type}Processor"
>
    <argument type="service" id="..." />
    <tag name="api_platform.state_processor" priority="10"/>
</service>
```

#### 3d. Link to Operations

```
Edit config/api_resources/resources/{section}/{Resource}.xml:

Add processor attribute:
<operation name="..." class="ApiPlatform\Metadata\Post" processor="{service_id}">
```

### 4. Remove Old Files

After successful migration and validation:

```
Bash: rm -rf src/DataProvider
Bash: rm -rf src/DataPersister
Bash: rm config/services/dataProvider/*.xml
Bash: rm config/services/dataPersister/*.xml
```

### 5. Validate Configuration

```
Bash: vendor/bin/console cache:clear
```

Expected: Cache clears without errors.

```
Bash: vendor/bin/console debug:router | grep "{plugin_prefix}_api"
```

Expected: API routes appear in the list.

```
Bash: vendor/bin/console debug:container | grep "state_provider"
```

Expected: Your StateProvider service appears in the list.

## Success Criteria
- All DataProvider classes migrated to StateProvider
- All DataPersister classes migrated to StateProcessor
- Services registered with proper tags
- Operations linked to providers/processors
- Old DataProvider/DataPersister directories removed
- Cache clears successfully
- API routes work correctly

## Notes for AI
- **Directory structure matters**: Follow `StateProvider/{Section}/{Resource}/{Type}Provider.php` pattern
- **Naming convention**: Use `CollectionProvider` for GetCollection, `ItemProvider` for Get item
- **Remove supports() method**: No longer needed - linking is explicit via `provider` attribute
- **Mark as readonly**: Follow Sylius conventions with `final readonly class`
- **Service ID format**: `{vendor}.{plugin}.state_provider.{section}.{resource}.{type}`
- **Explicit linking**: Always add `provider="service_id"` or `processor="service_id"` to operations
- **No $class parameter**: Remove resource class parameter from constructor - not needed anymore
- **Context access**: Old `$context` parameter is still available in `provide()` method

## Common Migration Patterns

### Pattern 1: Collection DataProvider
**Before:**
```php
class GetProductsDataProvider implements CollectionDataProviderInterface
{
    public function __construct(private ProductRepository $repository) {}

    public function supports(...): bool { return Product::class === $resourceClass; }

    public function getCollection(string $resourceClass, string $operationName = null): iterable
    {
        return $this->repository->findAll();
    }
}
```

**After:**
```php
/** @implements ProviderInterface<ProductInterface> */
final readonly class CollectionProvider implements ProviderInterface
{
    public function __construct(private ProductRepositoryInterface $repository) {}

    public function provide(Operation $operation, array $uriVariables = [], array $context = []): array
    {
        return $this->repository->findAll();
    }
}
```

### Pattern 2: Item DataProvider with Filters
**Before:**
```php
class GetProductDataProvider implements ContextAwareCollectionDataProviderInterface
{
    public function getCollection(string $resourceClass, string $operationName = null, array $context = []): iterable
    {
        $category = $context['filters']['category'] ?? null;
        return $this->repository->findByCategory($category);
    }
}
```

**After:**
```php
final readonly class CollectionProvider implements ProviderInterface
{
    public function provide(Operation $operation, array $uriVariables = [], array $context = []): array
    {
        $category = $context['filters']['category'] ?? null;
        return $this->repository->findByCategory($category);
    }
}
```

### Pattern 3: DataPersister
**Before:**
```php
class CreateProductDataPersister implements DataPersisterInterface
{
    public function persist($data, array $context = [])
    {
        $this->repository->add($data);
        return $data;
    }
}
```

**After:**
```php
final readonly class CreateProcessor implements ProcessorInterface
{
    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        $this->repository->add($data);
        return $data;
    }
}
```

## Testing (Optional)
If API endpoints are available:
```
# Test endpoint with StateProvider
Bash: curl -X GET http://localhost/api/v2/shop/{resource} -H "Accept: application/json"
```

Expected: JSON response with data from StateProvider.
