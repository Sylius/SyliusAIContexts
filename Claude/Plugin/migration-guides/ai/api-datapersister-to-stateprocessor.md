# API: Migrate DataPersister to StateProcessor

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
Execute only if:
- Plugin has custom DataPersister classes (check `src/DataPersister/` directory)

Skip if plugin has no custom data persisters.

## Commands

### 1. Detect Existing DataPersisters

Check for DataPersister classes:
```
Glob: "src/DataPersister/*.php"
Glob: "src/**/DataPersister/*.php"
```

If none found, skip this step.

### 2. Migrate Each DataPersister

For each DataPersister file found:

#### 2a. Read Current DataPersister

```
Read src/DataPersister/{FileName}.php
```

Check what interfaces it implements:
- `DataPersisterInterface` → Migrate to `ProcessorInterface`
- `ContextAwareDataPersisterInterface` → Migrate to `ProcessorInterface`

#### 2b. Determine New Directory Structure

Follow Sylius pattern: `src/StateProcessor/{Section}/{Resource}/{Type}Processor.php`

Example mappings:
- `CreateProductDataPersister` → `src/StateProcessor/Admin/Product/PersistProcessor.php`
- `DeleteProductDataPersister` → `src/StateProcessor/Admin/Product/RemoveProcessor.php`
- `UpdateOrderDataPersister` → `src/StateProcessor/Shop/Order/PersistProcessor.php`

Pattern:
- Section: `Shop` or `Admin` (based on API section)
- Resource: Entity name (singular, e.g., `Product`, `Order`)
- Type: `PersistProcessor` for create/update, `RemoveProcessor` for delete, or custom name

#### 2c. Create New StateProcessor

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
        // Copy constructor arguments from old DataPersister
        // If wrapping core processor, inject it:
        // private ProcessorInterface $persistProcessor,
    ) {
    }

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        // Migrate logic from old persist() or remove() method
        // Access operation type via: $operation instanceof Delete

        // If wrapping core processor, delegate to it:
        // return $this->persistProcessor->process($data, $operation, $uriVariables, $context);

        return $data;
    }
}
```

**Key changes:**
- Implements `ProcessorInterface` instead of `DataPersisterInterface`
- Method `persist()` or `remove()` → `process()`
- Add `Operation $operation` as second parameter
- Return type: `mixed` (return the persisted/removed data)
- Remove `supports()` method - no longer needed (handled by operation linking)
- Mark class as `readonly final` following Sylius conventions
- Add PHPDoc: `@implements ProcessorInterface<ResourceInterface>`

**Common pattern - Decorating core processor:**
```php
/** @implements ProcessorInterface<ProductInterface> */
final readonly class PersistProcessor implements ProcessorInterface
{
    public function __construct(
        private ProcessorInterface $persistProcessor,
        private ProductValidatorInterface $validator,
    ) {
    }

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        // Custom validation or business logic
        $this->validator->validate($data);

        // Delegate actual persistence to core processor
        return $this->persistProcessor->process($data, $operation, $uriVariables, $context);
    }
}
```

#### 2d. Update Service Registration

Check for old service registration:
```
Grep: pattern="{OldClassName}" path="config/services"
```

Create new service file:
```
Write config/services/stateProcessor/stateProcessor.xml (or append to existing):
<service id="{vendor}.{plugin}.state_processor.{section}.{resource}.{type}"
         class="{Vendor}\{Plugin}\StateProcessor\{Section}\{Resource}\{Type}Processor"
>
    <!-- If decorating core processor, inject it: -->
    <argument type="service" id="api_platform.doctrine.orm.state.persist_processor" />

    <!-- Copy other arguments from old service -->
    <argument type="service" id="..." />

    <tag name="api_platform.state_processor" priority="10"/>
</service>
```

**Service ID pattern:**
`{vendor}.{plugin}.state_processor.{section}.{resource}.{type}`

Example: `bitbag.sylius_banner_plugin.state_processor.admin.banner.persist_processor`

**Important:** Remove `$class` or `$resourceClass` parameter - no longer needed in API Platform 4.x

**Common core processors to wrap:**
- `api_platform.doctrine.orm.state.persist_processor` - for create/update operations
- `api_platform.doctrine.orm.state.remove_processor` - for delete operations

#### 2e. Link StateProcessor to Operations

Find API resource operations using this processor:
```
Read config/api_resources/resources/{section}/{Resource}.xml
```

Update operation to reference the new StateProcessor:
```
Edit config/api_resources/resources/{section}/{Resource}.xml:

Old (API Platform 2.x - implicit):
<operation name="..." class="ApiPlatform\Metadata\Post" uriTemplate="...">
    <!-- No processor specified, used DataPersister with supports() -->
</operation>

New (API Platform 4.x - explicit):
<operation name="..." class="ApiPlatform\Metadata\Post" uriTemplate="..." processor="{service_id}">
    <!-- Explicitly links to StateProcessor service -->
</operation>
```

Add `processor="{service_id}"` attribute to the operation.

**Operation types that need processors:**
- `ApiPlatform\Metadata\Post` - Create operations
- `ApiPlatform\Metadata\Put` - Update operations
- `ApiPlatform\Metadata\Patch` - Partial update operations
- `ApiPlatform\Metadata\Delete` - Delete operations

### 3. Remove Old Files

After successful migration and validation:

```
Bash: rm -rf src/DataPersister
Bash: rm config/services/dataPersister/*.xml
Bash: rmdir config/services/dataPersister
```

### 4. Validate Configuration

```
Bash: vendor/bin/console cache:clear
```

Expected: Cache clears without errors.

```
Bash: vendor/bin/console debug:router | grep "{plugin_prefix}_api"
```

Expected: API routes appear in the list.

```
Bash: vendor/bin/console debug:container | grep "state_processor"
```

Expected: Your StateProcessor service appears in the list.

## Success Criteria
- All DataPersister classes migrated to StateProcessor
- Services registered with proper tags
- Operations linked to processors
- Old DataPersister directory removed
- Cache clears successfully
- API routes work correctly

## Notes for AI
- **Directory structure matters**: Follow `StateProcessor/{Section}/{Resource}/{Type}Processor.php` pattern
- **Naming convention**: Use `PersistProcessor` for create/update, `RemoveProcessor` for delete
- **Remove supports() method**: No longer needed - linking is explicit via `processor` attribute
- **Mark as readonly**: Follow Sylius conventions with `final readonly class`
- **Service ID format**: `{vendor}.{plugin}.state_processor.{section}.{resource}.{type}`
- **Explicit linking**: Always add `processor="service_id"` to write operations (Post/Put/Patch/Delete)
- **No $class parameter**: Remove resource class parameter from constructor - not needed anymore
- **Context access**: Old `$context` parameter is still available in `process()` method
- **Decorator pattern**: Most processors wrap core processors and add validation/business logic

## Common Migration Patterns

### Pattern 1: Simple Persist DataPersister
**Before:**
```php
class CreateProductDataPersister implements DataPersisterInterface
{
    public function __construct(
        private EntityManagerInterface $entityManager,
    ) {}

    public function supports($data): bool {
        return $data instanceof Product;
    }

    public function persist($data, array $context = [])
    {
        $this->entityManager->persist($data);
        $this->entityManager->flush();
        return $data;
    }

    public function remove($data, array $context = [])
    {
        $this->entityManager->remove($data);
        $this->entityManager->flush();
    }
}
```

**After:**
```php
/** @implements ProcessorInterface<ProductInterface> */
final readonly class PersistProcessor implements ProcessorInterface
{
    public function __construct(
        private ProcessorInterface $persistProcessor,
    ) {}

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        return $this->persistProcessor->process($data, $operation, $uriVariables, $context);
    }
}
```

**Service registration:**
```xml
<service id="vendor.plugin.state_processor.admin.product.persist"
         class="Vendor\Plugin\StateProcessor\Admin\Product\PersistProcessor">
    <argument type="service" id="api_platform.doctrine.orm.state.persist_processor" />
    <tag name="api_platform.state_processor" priority="10"/>
</service>
```

### Pattern 2: DataPersister with Validation
**Before:**
```php
class CreateOrderDataPersister implements DataPersisterInterface
{
    public function persist($data, array $context = [])
    {
        $this->validator->validate($data);
        $this->entityManager->persist($data);
        $this->entityManager->flush();
        return $data;
    }
}
```

**After:**
```php
/** @implements ProcessorInterface<OrderInterface> */
final readonly class PersistProcessor implements ProcessorInterface
{
    public function __construct(
        private ProcessorInterface $persistProcessor,
        private OrderValidatorInterface $validator,
    ) {}

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        $this->validator->validate($data);

        return $this->persistProcessor->process($data, $operation, $uriVariables, $context);
    }
}
```

### Pattern 3: Different Logic for Create vs Update
**Before:**
```php
class ProductDataPersister implements DataPersisterInterface
{
    public function persist($data, array $context = [])
    {
        if (null === $data->getId()) {
            // Create logic
            $data->setCreatedAt(new \DateTime());
        } else {
            // Update logic
            $data->setUpdatedAt(new \DateTime());
        }

        $this->entityManager->persist($data);
        $this->entityManager->flush();
        return $data;
    }
}
```

**After:**
```php
/** @implements ProcessorInterface<ProductInterface> */
final readonly class PersistProcessor implements ProcessorInterface
{
    public function __construct(
        private ProcessorInterface $persistProcessor,
    ) {}

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        if ($operation instanceof Post) {
            // Create logic
            $data->setCreatedAt(new \DateTime());
        } elseif ($operation instanceof Put || $operation instanceof Patch) {
            // Update logic
            $data->setUpdatedAt(new \DateTime());
        }

        return $this->persistProcessor->process($data, $operation, $uriVariables, $context);
    }
}
```

### Pattern 4: Remove Processor
**Before:**
```php
class DeleteProductDataPersister implements DataPersisterInterface
{
    public function remove($data, array $context = [])
    {
        if ($data->hasOrders()) {
            throw new \Exception('Cannot delete product with orders');
        }

        $this->entityManager->remove($data);
        $this->entityManager->flush();
    }
}
```

**After:**
```php
/** @implements ProcessorInterface<ProductInterface> */
final readonly class RemoveProcessor implements ProcessorInterface
{
    public function __construct(
        private ProcessorInterface $removeProcessor,
        private ProductDeletionCheckerInterface $deletionChecker,
    ) {}

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        $this->deletionChecker->check($data);

        return $this->removeProcessor->process($data, $operation, $uriVariables, $context);
    }
}
```

**Link to Delete operation:**
```xml
<operation name="delete_product"
           class="ApiPlatform\Metadata\Delete"
           uriTemplate="/admin/products/{id}"
           processor="vendor.plugin.state_processor.admin.product.remove">
</operation>
```

## Testing (Optional)
If API endpoints are available:
```
# Test POST endpoint with StateProcessor
Bash: curl -X POST http://localhost/api/v2/admin/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Product","code":"TEST"}'
```

Expected: JSON response with created resource.

```
# Test DELETE endpoint with StateProcessor
Bash: curl -X DELETE http://localhost/api/v2/admin/products/1
```

Expected: 204 No Content response.
