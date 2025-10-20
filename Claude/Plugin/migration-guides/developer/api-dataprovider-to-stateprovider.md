# API: Migrate DataProvider to StateProvider

This step migrates API Platform 2.x DataProviders and DataPersisters to API Platform 4.x StateProviders and StateProcessors.

**When to skip this step:**
- Your plugin doesn't have custom DataProvider or DataPersister classes

**When to do this step:**
- You have custom DataProvider classes in `src/DataProvider/`
- You have custom DataPersister classes in `src/DataPersister/`

## Overview of Changes

API Platform 4.x replaced DataProviders and DataPersisters with a unified State system:

| API Platform 2.x                  | API Platform 4.x                  |
|-----------------------------------|-----------------------------------|
| `CollectionDataProviderInterface` | `ProviderInterface` (collections) |
| `ItemDataProviderInterface`       | `ProviderInterface` (items)       |
| `DataPersisterInterface`          | `ProcessorInterface`              |
| `DataTransformerInterface`        | Some → `SerializerContextBuilder` |

**Key changes:**
- New interfaces and namespaces
- Different method signatures
- Explicit linking to operations (no more `supports()` method)
- Organized directory structure

## 1. Identify Files to Migrate

Check if you have DataProviders:
```bash
find src/DataProvider -name "*.php"
```

Check if you have DataPersisters:
```bash
find src/DataPersister -name "*.php"
```

## 2. Migrate DataProvider to StateProvider

### Directory Structure

Follow Sylius conventions for organization:

**Old structure:**
```
src/DataProvider/GetProductsDataProvider.php
```

**New structure:**
```
src/StateProvider/Shop/Product/CollectionProvider.php
src/StateProvider/Admin/Product/CollectionProvider.php
src/StateProvider/Shop/Product/ItemProvider.php
```

**Pattern:** `StateProvider/{Section}/{Resource}/{Type}Provider.php`
- **Section:** `Shop` or `Admin`
- **Resource:** Entity name (singular)
- **Type:** `CollectionProvider` or `ItemProvider`

### Example Migration

**Before (API Platform 2.x):**

`src/DataProvider/GetAdsBannersDataProvider.php`:
```php
<?php

declare(strict_types=1);

namespace BitBag\SyliusBannerPlugin\DataProvider;

use ApiPlatform\Core\DataProvider\ContextAwareCollectionDataProviderInterface;
use ApiPlatform\Core\DataProvider\RestrictedDataProviderInterface;
use BitBag\SyliusBannerPlugin\Repository\AdRepositoryInterface;

final class GetAdsBannersDataProvider implements
    ContextAwareCollectionDataProviderInterface,
    RestrictedDataProviderInterface
{
    public function __construct(
        private AdRepositoryInterface $adRepository,
        private BannersProviderInterface $bannersProvider,
        private string $class,
    ) {
    }

    public function supports(
        string $resourceClass,
        string $operationName = null,
        array $context = [],
    ): bool {
        return $this->class === $resourceClass;
    }

    public function getCollection(
        string $resourceClass,
        string $operationName = null,
        array $context = [],
    ): iterable {
        $localeCode = $context['filters']['locale_code'] ?? null;
        $sectionCode = $context['filters']['section_code'] ?? null;

        if (null !== $localeCode && null !== $sectionCode) {
            $ads = $this->adRepository->findAllActiveAds();
            return $this->bannersProvider->getAdsBanners($ads, $sectionCode, $localeCode);
        }

        return [];
    }
}
```

**After (API Platform 4.x):**

`src/StateProvider/Shop/Banner/CollectionProvider.php`:
```php
<?php

declare(strict_types=1);

namespace BitBag\SyliusBannerPlugin\StateProvider\Shop\Banner;

use ApiPlatform\Metadata\Operation;
use ApiPlatform\State\ProviderInterface;
use BitBag\SyliusBannerPlugin\Entity\BannerInterface;
use BitBag\SyliusBannerPlugin\Provider\BannersProviderInterface;
use BitBag\SyliusBannerPlugin\Repository\AdRepositoryInterface;

/** @implements ProviderInterface<BannerInterface> */
final readonly class CollectionProvider implements ProviderInterface
{
    public function __construct(
        private AdRepositoryInterface $adRepository,
        private BannersProviderInterface $bannersProvider,
    ) {
    }

    public function provide(Operation $operation, array $uriVariables = [], array $context = []): array
    {
        $localeCode = $context['filters']['locale_code'] ?? null;
        $sectionCode = $context['filters']['section_code'] ?? null;

        if (null !== $localeCode && null !== $sectionCode) {
            $ads = $this->adRepository->findAllActiveAds();
            return $this->bannersProvider->getAdsBanners($ads, $sectionCode, $localeCode);
        }

        return [];
    }
}
```

**Key Changes:**
1. **Namespace:** `DataProvider` → `StateProvider\Shop\Banner`
2. **Class name:** `GetAdsBannersDataProvider` → `CollectionProvider`
3. **Interface:** `ContextAwareCollectionDataProviderInterface` → `ProviderInterface`
4. **Method:** `getCollection()` → `provide()`
5. **Removed:** `supports()` method - no longer needed
6. **Removed:** `$class` parameter - no longer needed
7. **Added:** `Operation $operation` parameter
8. **Class modifiers:** Added `final readonly`
9. **PHPDoc:** Added `@implements ProviderInterface<BannerInterface>`

## 3. Update Service Registration

**Before:**

`config/services/dataProvider/dataProvider.xml`:
```xml
<service id="bitbag.sylius_banner_plugin.data_provider.get_ads_banners_data_provider"
         class="BitBag\SyliusBannerPlugin\DataProvider\GetAdsBannersDataProvider"
>
    <argument type="service" id="bitbag_sylius_banner_plugin.repository.ad"/>
    <argument type="service" id="bitbag.sylius_banner_plugin.provider.banners_provider"/>
    <argument>%bitbag_sylius_banner_plugin.model.banner.class%</argument>
    <tag name="api_platform.collection_data_provider"/>
</service>
```

**After:**

`config/services/stateProvider/stateProvider.xml`:
```xml
<service id="bitbag.sylius_banner_plugin.state_provider.shop.banner.collection_provider"
         class="BitBag\SyliusBannerPlugin\StateProvider\Shop\Banner\CollectionProvider"
>
    <argument type="service" id="bitbag_sylius_banner_plugin.repository.ad"/>
    <argument type="service" id="bitbag.sylius_banner_plugin.provider.banners_provider"/>
    <tag name="api_platform.state_provider" priority="10"/>
</service>
```

**Changes:**
1. **Service ID:** Follow pattern `{vendor}.{plugin}.state_provider.{section}.{resource}.{type}`
2. **Class:** Updated to new namespace
3. **Removed:** `$class` argument (resource class parameter)
4. **Tag:** `api_platform.collection_data_provider` → `api_platform.state_provider`
5. **Priority:** Added `priority="10"`

## 4. Link StateProvider to Operations

In API Platform 4.x, you must explicitly link providers to operations.

`config/api_resources/resources/shop/Banner.xml`:

**Before (implicit):**
```xml
<operation name="get_banners" class="ApiPlatform\Metadata\GetCollection" uriTemplate="/shop/banners">
    <!-- DataProvider was matched via supports() method -->
</operation>
```

**After (explicit):**
```xml
<operation
    name="get_banners"
    class="ApiPlatform\Metadata\GetCollection"
    uriTemplate="/shop/banners"
    provider="bitbag.sylius_banner_plugin.state_provider.shop.banner.collection_provider"
>
    <!-- StateProvider explicitly linked via provider attribute -->
</operation>
```

Add the `provider="{service_id}"` attribute to link your StateProvider to the operation.

## 5. Migrate DataPersister to StateProcessor (if applicable)

If you have DataPersisters, migrate them similarly:

**Before:**
```php
final class CreateProductDataPersister implements DataPersisterInterface
{
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
final readonly class CreateProcessor implements ProcessorInterface
{
    public function process(mixed $data, Operation $operation, array $uriVariables = [], array $context = []): mixed
    {
        $this->entityManager->persist($data);
        $this->entityManager->flush();

        return $data;
    }
}
```

**Service registration:**
```xml
<service id="vendor.plugin.state_processor.shop.product.create"
         class="Vendor\Plugin\StateProcessor\Shop\Product\CreateProcessor"
>
    <tag name="api_platform.state_processor" priority="10"/>
</service>
```

**Link to operation:**
```xml
<operation
    name="create_product"
    class="ApiPlatform\Metadata\Post"
    processor="vendor.plugin.state_processor.shop.product.create"
>
</operation>
```

## 6. Remove Old Files

After migration is complete and tested:

```bash
# Remove old directories
rm -rf src/DataProvider
rm -rf src/DataPersister

# Remove old service configurations
rm config/services/dataProvider/dataProvider.xml
rm config/services/dataPersister/dataPersister.xml
```

## 7. Validate Changes

Clear the cache:
```bash
vendor/bin/console cache:clear
```

Verify routes are registered:
```bash
vendor/bin/console debug:router | grep your_plugin_api
```

Verify StateProvider is registered:
```bash
vendor/bin/console debug:container | grep state_provider
```

Test API endpoints:
```bash
curl -X GET http://localhost/api/v2/shop/banners -H "Accept: application/json"
```

## Common Patterns

### Pattern: Item Provider

For getting a single item:

```php
/** @implements ProviderInterface<ProductInterface> */
final readonly class ItemProvider implements ProviderInterface
{
    public function __construct(
        private ProductRepositoryInterface $productRepository,
    ) {
    }

    public function provide(Operation $operation, array $uriVariables = [], array $context = []): ?object
    {
        return $this->productRepository->find($uriVariables['id']);
    }
}
```

### Pattern: Using URI Variables

Access path parameters via `$uriVariables`:

```php
public function provide(Operation $operation, array $uriVariables = [], array $context = []): ?object
{
    $productCode = $uriVariables['code'];
    return $this->productRepository->findOneByCode($productCode);
}
```

### Pattern: Using Filters

Access query parameters via `$context['filters']`:

```php
public function provide(Operation $operation, array $uriVariables = [], array $context = []): array
{
    $category = $context['filters']['category'] ?? null;
    $enabled = $context['filters']['enabled'] ?? true;

    return $this->productRepository->findBy([
        'category' => $category,
        'enabled' => $enabled,
    ]);
}
```

## Important Notes

1. **No supports() method:** Linking is now explicit via `provider` attribute
2. **Remove $class parameter:** Resource class is obtained from `$operation->getClass()`
3. **readonly classes:** Follow Sylius convention with `final readonly class`
4. **Return types:**
   - Collections: `array`
   - Items: `object|null`
   - Processors: `mixed`
5. **Service ID format:** `{vendor}.{plugin}.state_provider.{section}.{resource}.{type}`
6. **Priority:** Use `priority="10"` in service tags

## Reference

For more examples, check Sylius core StateProviders:
```
vendor/sylius/sylius/src/Sylius/Bundle/ApiBundle/StateProvider/
```
