# API: Migrate DataPersister to StateProcessor

This step migrates API Platform 2.x DataPersisters to API Platform 4.x StateProcessors.

**When to skip this step:**
- Your plugin doesn't have custom DataPersister classes

**When to do this step:**
- You have custom DataPersister classes in `src/DataPersister/`

## Overview of Changes

API Platform 4.x replaced DataPersisters with StateProcessors as part of the unified State system:

| API Platform 2.x                     | API Platform 4.x                          |
|--------------------------------------|-------------------------------------------|
| `DataPersisterInterface`             | `ProcessorInterface`                      |
| `ContextAwareDataPersisterInterface` | `ProcessorInterface`                      |
| `persist()` method                   | `process()` method                        |
| `remove()` method                    | `process()` method (check operation type) |

**Key changes:**
- New interfaces and namespaces
- Different method signatures
- Explicit linking to operations (no more `supports()` method)
- Organized directory structure
- Decorator pattern for wrapping core processors

## 1. Identify Files to Migrate

Check if you have DataPersisters:
```bash
find src/DataPersister -name "*.php"
```

## 2. Migrate DataPersister to StateProcessor

### Directory Structure

Follow Sylius conventions for organization:

**Old structure:**
```
src/DataPersister/CreateProductDataPersister.php
src/DataPersister/DeleteProductDataPersister.php
```

**New structure:**
```
src/StateProcessor/Admin/Product/PersistProcessor.php
src/StateProcessor/Admin/Product/RemoveProcessor.php
```

**Pattern:** `StateProcessor/{Section}/{Resource}/{Type}Processor.php`
- **Section:** `Shop` or `Admin`
- **Resource:** Entity name (singular)
- **Type:** `PersistProcessor` for create/update, `RemoveProcessor` for delete, or custom name

### Example Migration: Simple Persist

**Before (API Platform 2.x):**

`src/DataPersister/CreateProductDataPersister.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\DataPersister;

use ApiPlatform\Core\DataPersister\DataPersisterInterface;
use Doctrine\ORM\EntityManagerInterface;
use Vendor\Plugin\Entity\ProductInterface;

final class CreateProductDataPersister implements DataPersisterInterface
{
    public function __construct(
        private EntityManagerInterface $entityManager,
    ) {
    }

    public function supports($data): bool
    {
        return $data instanceof ProductInterface;
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

**After (API Platform 4.x):**

`src/StateProcessor/Admin/Product/PersistProcessor.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\StateProcessor\Admin\Product;

use ApiPlatform\Metadata\Operation;
use ApiPlatform\State\ProcessorInterface;
use Vendor\Plugin\Entity\ProductInterface;

/** @implements ProcessorInterface<ProductInterface> */
final readonly class PersistProcessor implements ProcessorInterface
{
    public function __construct(
        private ProcessorInterface $persistProcessor,
    ) {
    }

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        return $this->persistProcessor->process($data, $operation, $uriVariables, $context);
    }
}
```

**Key Changes:**
1. **Namespace:** `DataPersister` → `StateProcessor\Admin\Product`
2. **Class name:** `CreateProductDataPersister` → `PersistProcessor`
3. **Interface:** `DataPersisterInterface` → `ProcessorInterface`
4. **Method:** `persist()` → `process()`
5. **Removed:** `supports()` method - no longer needed
6. **Removed:** `remove()` method - create separate `RemoveProcessor` if needed
7. **Added:** `Operation $operation` parameter
8. **Changed:** Constructor now injects core processor instead of EntityManager
9. **Class modifiers:** Added `final readonly`
10. **PHPDoc:** Added `@implements ProcessorInterface<ProductInterface>`

### Example Migration: Processor with Validation

**Before (API Platform 2.x):**

`src/DataPersister/CreateOrderDataPersister.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\DataPersister;

use ApiPlatform\Core\DataPersister\DataPersisterInterface;
use Doctrine\ORM\EntityManagerInterface;
use Vendor\Plugin\Entity\OrderInterface;
use Vendor\Plugin\Validator\OrderValidatorInterface;

final class CreateOrderDataPersister implements DataPersisterInterface
{
    public function __construct(
        private EntityManagerInterface $entityManager,
        private OrderValidatorInterface $validator,
    ) {
    }

    public function supports($data): bool
    {
        return $data instanceof OrderInterface;
    }

    public function persist($data, array $context = [])
    {
        $this->validator->validate($data);

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

**After (API Platform 4.x):**

`src/StateProcessor/Shop/Order/PersistProcessor.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\StateProcessor\Shop\Order;

use ApiPlatform\Metadata\Operation;
use ApiPlatform\State\ProcessorInterface;
use Vendor\Plugin\Entity\OrderInterface;
use Vendor\Plugin\Validator\OrderValidatorInterface;

/** @implements ProcessorInterface<OrderInterface> */
final readonly class PersistProcessor implements ProcessorInterface
{
    public function __construct(
        private ProcessorInterface $persistProcessor,
        private OrderValidatorInterface $validator,
    ) {
    }

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        // Custom validation logic
        $this->validator->validate($data);

        // Delegate actual persistence to core processor
        return $this->persistProcessor->process($data, $operation, $uriVariables, $context);
    }
}
```

**Key Pattern:**
- **Decorator pattern:** StateProcessor wraps the core processor (`api_platform.doctrine.orm.state.persist_processor`)
- **Custom logic first:** Validation/business logic runs before delegating to core processor
- **Return delegated result:** Always return the result from the core processor

### Example Migration: Remove Processor

**Before (API Platform 2.x):**

`src/DataPersister/DeleteProductDataPersister.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\DataPersister;

use ApiPlatform\Core\DataPersister\DataPersisterInterface;
use Doctrine\ORM\EntityManagerInterface;
use Vendor\Plugin\Entity\ProductInterface;
use Vendor\Plugin\Checker\ProductDeletionCheckerInterface;

final class DeleteProductDataPersister implements DataPersisterInterface
{
    public function __construct(
        private EntityManagerInterface $entityManager,
        private ProductDeletionCheckerInterface $deletionChecker,
    ) {
    }

    public function supports($data): bool
    {
        return $data instanceof ProductInterface;
    }

    public function persist($data, array $context = [])
    {
        $this->entityManager->persist($data);
        $this->entityManager->flush();

        return $data;
    }

    public function remove($data, array $context = [])
    {
        $this->deletionChecker->check($data);

        $this->entityManager->remove($data);
        $this->entityManager->flush();
    }
}
```

**After (API Platform 4.x):**

`src/StateProcessor/Admin/Product/RemoveProcessor.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\StateProcessor\Admin\Product;

use ApiPlatform\Metadata\Operation;
use ApiPlatform\State\ProcessorInterface;
use Vendor\Plugin\Entity\ProductInterface;
use Vendor\Plugin\Checker\ProductDeletionCheckerInterface;

/** @implements ProcessorInterface<ProductInterface> */
final readonly class RemoveProcessor implements ProcessorInterface
{
    public function __construct(
        private ProcessorInterface $removeProcessor,
        private ProductDeletionCheckerInterface $deletionChecker,
    ) {
    }

    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        // Check if deletion is allowed
        $this->deletionChecker->check($data);

        // Delegate to core remove processor
        return $this->removeProcessor->process($data, $operation, $uriVariables, $context);
    }
}
```

## 3. Update Service Registration

**Before:**

`config/services/dataPersister/dataPersister.xml`:
```xml
<service id="vendor.plugin.data_persister.create_product"
         class="Vendor\Plugin\DataPersister\CreateProductDataPersister"
>
    <argument type="service" id="doctrine.orm.entity_manager"/>
    <tag name="api_platform.data_persister"/>
</service>
```

**After:**

`config/services/stateProcessor/stateProcessor.xml`:
```xml
<service id="vendor.plugin.state_processor.admin.product.persist"
         class="Vendor\Plugin\StateProcessor\Admin\Product\PersistProcessor"
>
    <argument type="service" id="api_platform.doctrine.orm.state.persist_processor"/>
    <tag name="api_platform.state_processor" priority="10"/>
</service>
```

**Changes:**
1. **Service ID:** Follow pattern `{vendor}.{plugin}.state_processor.{section}.{resource}.{type}`
2. **Class:** Updated to new namespace
3. **First argument:** Core processor instead of EntityManager
   - `api_platform.doctrine.orm.state.persist_processor` for create/update
   - `api_platform.doctrine.orm.state.remove_processor` for delete
4. **Tag:** `api_platform.data_persister` → `api_platform.state_processor`
5. **Priority:** Added `priority="10"`

### Service Registration for Remove Processor

`config/services/stateProcessor/stateProcessor.xml`:
```xml
<service id="vendor.plugin.state_processor.admin.product.remove"
         class="Vendor\Plugin\StateProcessor\Admin\Product\RemoveProcessor"
>
    <argument type="service" id="api_platform.doctrine.orm.state.remove_processor"/>
    <argument type="service" id="vendor.plugin.checker.product_deletion_checker"/>
    <tag name="api_platform.state_processor" priority="10"/>
</service>
```

## 4. Link StateProcessor to Operations

In API Platform 4.x, you must explicitly link processors to operations.

`config/api_resources/resources/admin/Product.xml`:

**Before (implicit):**
```xml
<operation name="create_product" class="ApiPlatform\Metadata\Post" uriTemplate="/admin/products">
    <!-- DataPersister was matched via supports() method -->
</operation>

<operation name="delete_product" class="ApiPlatform\Metadata\Delete" uriTemplate="/admin/products/{id}">
    <!-- DataPersister was matched via supports() method -->
</operation>
```

**After (explicit):**
```xml
<operation
    name="create_product"
    class="ApiPlatform\Metadata\Post"
    uriTemplate="/admin/products"
    processor="vendor.plugin.state_processor.admin.product.persist"
>
    <!-- StateProcessor explicitly linked via processor attribute -->
</operation>

<operation
    name="delete_product"
    class="ApiPlatform\Metadata\Delete"
    uriTemplate="/admin/products/{id}"
    processor="vendor.plugin.state_processor.admin.product.remove"
>
    <!-- RemoveProcessor explicitly linked via processor attribute -->
</operation>
```

Add the `processor="{service_id}"` attribute to link your StateProcessor to the operation.

**Operation types that need processors:**
- `ApiPlatform\Metadata\Post` - Create operations → Use `PersistProcessor`
- `ApiPlatform\Metadata\Put` - Full update operations → Use `PersistProcessor`
- `ApiPlatform\Metadata\Patch` - Partial update operations → Use `PersistProcessor`
- `ApiPlatform\Metadata\Delete` - Delete operations → Use `RemoveProcessor`

## 5. Remove Old Files

After migration is complete and tested:

```bash
# Remove old directories
rm -rf src/DataPersister

# Remove old service configurations
rm config/services/dataPersister/dataPersister.xml
rmdir config/services/dataPersister
```

## 6. Validate Changes

Clear the cache:
```bash
vendor/bin/console cache:clear
```

Verify routes are registered:
```bash
vendor/bin/console debug:router | grep your_plugin_api
```

Verify StateProcessor is registered:
```bash
vendor/bin/console debug:container | grep state_processor
```

Test API endpoints:
```bash
# Test POST endpoint
curl -X POST http://localhost/api/v2/admin/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Product","code":"TEST"}'

# Test DELETE endpoint
curl -X DELETE http://localhost/api/v2/admin/products/1
```

## Common Patterns

### Pattern: Checking Operation Type

If you need different logic for create vs update:

```php
use ApiPlatform\Metadata\Post;
use ApiPlatform\Metadata\Put;
use ApiPlatform\Metadata\Patch;

final readonly class PersistProcessor implements ProcessorInterface
{
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

### Pattern: Accessing URI Variables

Access path parameters via `$uriVariables`:

```php
public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
{
    $productId = $uriVariables['id'];
    // Use $productId for business logic

    return $this->persistProcessor->process($data, $operation, $uriVariables, $context);
}
```

### Pattern: Accessing Context

Access request context via `$context`:

```php
public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
{
    $user = $context['user'] ?? null;
    if ($user) {
        $data->setCreatedBy($user);
    }

    return $this->persistProcessor->process($data, $operation, $uriVariables, $context);
}
```

## Important Notes

1. **No supports() method:** Linking is now explicit via `processor` attribute
2. **Decorator pattern:** Most StateProcessors wrap core processors
3. **Core processors:**
   - `api_platform.doctrine.orm.state.persist_processor` - for persistence
   - `api_platform.doctrine.orm.state.remove_processor` - for deletion
4. **readonly classes:** Follow Sylius convention with `final readonly class`
5. **Return types:** Always `mixed` - return the processed data
6. **Service ID format:** `{vendor}.{plugin}.state_processor.{section}.{resource}.{type}`
7. **Priority:** Use `priority="10"` in service tags
8. **Separate processors:** Create separate `PersistProcessor` and `RemoveProcessor` classes

## Reference

For more examples, check Sylius core StateProcessors:
```
vendor/sylius/sylius/src/Sylius/Bundle/ApiBundle/StateProcessor/
```

Example files:
- `Admin/Country/PersistProcessor.php` - Processor with validation
- `Admin/ProductVariant/RemoveProcessor.php` - Remove processor with checks
- `Shop/Order/PersistProcessor.php` - Complex business logic processor
