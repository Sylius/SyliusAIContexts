# API: Migrate DataTransformer to SerializerContextBuilder

This step migrates API Platform 2.x DataTransformers to API Platform 4.x SerializerContextBuilders.

**When to skip this step:**
- Your plugin doesn't have custom DataTransformer classes

**When to do this step:**
- You have custom DataTransformer classes in `src/DataTransformer/`
- Your DataTransformers inject contextual data (channel code, locale code, user ID, etc.) into input DTOs

## Overview of Changes

API Platform 4.x dropped DataTransformers. For DataTransformers that inject contextual data into input DTOs, the recommended approach is to use SerializerContextBuilders:

| API Platform 2.x                            | API Platform 4.x                                      |
|---------------------------------------------|-------------------------------------------------------|
| `DataTransformerInterface`                  | `SerializerContextBuilderInterface`                   |
| `InputDataTransformerInterface`             | `SerializerContextBuilderInterface`                   |
| `transform()` method                        | `createFromRequest()` method                          |
| Property modification after deserialization | Constructor argument injection during deserialization |

**Key changes:**
- DataTransformers that modify input DTOs → SerializerContextBuilders
- DataTransformers that format output → May need custom Normalizers
- New decorator pattern for SerializerContextBuilders
- Use PHP 8 attributes to mark DTOs
- Inject values into DTO constructors, not properties

**Important:** Not all DataTransformers should become SerializerContextBuilders. Only those that inject contextual data (like channel code, locale code, current user ID) from the request/session into input DTOs.

## 1. Identify Files to Migrate

Check if you have DataTransformers:
```bash
find src/DataTransformer -name "*.php"
```

## 2. Determine Migration Strategy

Review each DataTransformer and determine the appropriate migration path:

### Migrate to SerializerContextBuilder if:
- ✅ Injects channel code into input DTO
- ✅ Injects locale code into input DTO
- ✅ Injects current user ID into input DTO
- ✅ Injects any contextual data from request/session into DTO constructor

### Alternative approaches if:
- ❌ Transforms output format → Use custom Normalizer
- ❌ Performs complex data transformation → Use StateProvider/StateProcessor
- ❌ Validates data → Use Symfony Validator constraints

## 3. Migrate DataTransformer to SerializerContextBuilder

### Directory Structure

Follow Sylius conventions for organization:

**Old structure:**
```
src/DataTransformer/ChannelCodeAwareInputCommandDataTransformer.php
```

**New structure:**
```
src/Serializer/ContextBuilder/ChannelCodeAwareContextBuilder.php
src/Attribute/ChannelCodeAware.php
```

### Example Migration: Channel Code Injection

**Before (API Platform 2.x):**

`src/DataTransformer/ChannelCodeAwareInputCommandDataTransformer.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\DataTransformer;

use ApiPlatform\Core\DataTransformer\DataTransformerInterface;
use Sylius\Component\Channel\Context\ChannelContextInterface;
use Vendor\Plugin\Command\CreateProductCommand;

final class ChannelCodeAwareInputCommandDataTransformer implements DataTransformerInterface
{
    public function __construct(
        private ChannelContextInterface $channelContext,
    ) {
    }

    public function transform($object, string $to, array $context = [])
    {
        $object->channelCode = $this->channelContext->getChannel()->getCode();

        return $object;
    }

    public function supportsTransformation($data, string $to, array $context = []): bool
    {
        return $data instanceof CreateProductCommand && CreateProductCommand::class === $to;
    }
}
```

**After (API Platform 4.x):**

`src/Serializer/ContextBuilder/ChannelCodeAwareContextBuilder.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Serializer\ContextBuilder;

use ApiPlatform\State\SerializerContextBuilderInterface;
use Sylius\Bundle\ApiBundle\Serializer\ContextBuilder\AbstractInputContextBuilder;
use Sylius\Component\Channel\Context\ChannelContextInterface;
use Symfony\Component\HttpFoundation\Request;

final class ChannelCodeAwareContextBuilder extends AbstractInputContextBuilder
{
    public function __construct(
        SerializerContextBuilderInterface $decoratedContextBuilder,
        string $attributeClass,
        string $defaultConstructorArgumentName,
        private readonly ChannelContextInterface $channelContext,
    ) {
        parent::__construct($decoratedContextBuilder, $attributeClass, $defaultConstructorArgumentName);
    }

    protected function supports(Request $request, array $context, ?array $extractedAttributes): bool
    {
        return true;
    }

    protected function resolveValue(array $context, ?array $extractedAttributes): mixed
    {
        return $this->channelContext->getChannel()->getCode();
    }
}
```

**Key Changes:**
1. **Namespace:** `DataTransformer` → `Serializer\ContextBuilder`
2. **Class name:** Remove "DataTransformer" suffix, add "ContextBuilder" suffix
3. **Extends:** Extend `AbstractInputContextBuilder` from Sylius
4. **Interface:** Removed `DataTransformerInterface` (parent handles it)
5. **Method:** `transform()` → `supports()` + `resolveValue()`
6. **Removed:** `supportsTransformation()` method
7. **Constructor:** Added decorator, attribute class, and default parameter name
8. **Class modifiers:** Added `final` and optionally `readonly`
9. **Logic:** Changed from property modification to value resolution

### Create Attribute Class

`src/Attribute/ChannelCodeAware.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Attribute;

#[\Attribute(\Attribute::TARGET_CLASS)]
final class ChannelCodeAware
{
    public function __construct(
        public readonly string $constructorArgumentName = 'channelCode',
    ) {
    }
}
```

This PHP 8 attribute marks which DTOs should have channel code injected.

### Update Input DTO/Command

`src/Command/CreateProductCommand.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Command;

use Vendor\Plugin\Attribute\ChannelCodeAware;

#[ChannelCodeAware]
final class CreateProductCommand
{
    public function __construct(
        protected string $channelCode, // Will be injected by ContextBuilder
        protected string $name,
        protected string $code,
    ) {
    }

    public function getChannelCode(): string
    {
        return $this->channelCode;
    }

    // Other getters...
}
```

**Changes:**
1. Add attribute to class: `#[ChannelCodeAware]`
2. Ensure `channelCode` is a constructor parameter (not just a property)
3. Property visibility changed to `protected`

## 4. Update Service Registration

**Before:**

`config/services/dataTransformer/dataTransformer.xml`:
```xml
<service id="vendor.plugin.data_transformer.channel_code_aware"
         class="Vendor\Plugin\DataTransformer\ChannelCodeAwareInputCommandDataTransformer"
>
    <argument type="service" id="sylius.context.channel"/>
    <tag name="api_platform.data_transformer"/>
</service>
```

**After:**

`config/services/serializer/contextBuilder.xml`:
```xml
<parameters>
    <parameter key="vendor_plugin.attribute.channel_code_aware.class">Vendor\Plugin\Attribute\ChannelCodeAware</parameter>
</parameters>

<services>
    <service id="vendor.plugin.serializer.context_builder.channel_code_aware"
             class="Vendor\Plugin\Serializer\ContextBuilder\ChannelCodeAwareContextBuilder"
             decorates="api_platform.serializer.context_builder"
    >
        <argument type="service" id=".inner"/>
        <argument>%vendor_plugin.attribute.channel_code_aware.class%</argument>
        <argument>channelCode</argument>
        <argument type="service" id="sylius.context.channel"/>
    </service>
</services>
```

**Changes:**
1. **Decorator pattern:** Use `decorates="api_platform.serializer.context_builder"`
2. **First argument:** `.inner` (the decorated core context builder)
3. **Second argument:** Attribute class parameter
4. **Third argument:** Default constructor parameter name
5. **Remaining arguments:** Your dependencies (e.g., `sylius.context.channel`)
6. **Tag removed:** No need for `api_platform.data_transformer` tag
7. **Parameter:** Define attribute class as a parameter for reusability

## 5. Example Migration: Locale Code Injection

**Before (API Platform 2.x):**

`src/DataTransformer/LocaleCodeAwareInputCommandDataTransformer.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\DataTransformer;

use ApiPlatform\Core\DataTransformer\DataTransformerInterface;
use Sylius\Component\Locale\Context\LocaleContextInterface;
use Vendor\Plugin\Command\CreateProductCommand;

final class LocaleCodeAwareInputCommandDataTransformer implements DataTransformerInterface
{
    public function __construct(
        private LocaleContextInterface $localeContext,
    ) {
    }

    public function transform($object, string $to, array $context = [])
    {
        $object->localeCode = $this->localeContext->getLocaleCode();

        return $object;
    }

    public function supportsTransformation($data, string $to, array $context = []): bool
    {
        return $data instanceof CreateProductCommand && CreateProductCommand::class === $to;
    }
}
```

**After (API Platform 4.x):**

`src/Serializer/ContextBuilder/LocaleCodeAwareContextBuilder.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Serializer\ContextBuilder;

use ApiPlatform\State\SerializerContextBuilderInterface;
use Sylius\Bundle\ApiBundle\Serializer\ContextBuilder\AbstractInputContextBuilder;
use Sylius\Component\Locale\Context\LocaleContextInterface;
use Symfony\Component\HttpFoundation\Request;

final class LocaleCodeAwareContextBuilder extends AbstractInputContextBuilder
{
    public function __construct(
        SerializerContextBuilderInterface $decoratedContextBuilder,
        string $attributeClass,
        string $defaultConstructorArgumentName,
        private readonly LocaleContextInterface $localeContext,
    ) {
        parent::__construct($decoratedContextBuilder, $attributeClass, $defaultConstructorArgumentName);
    }

    protected function supports(Request $request, array $context, ?array $extractedAttributes): bool
    {
        return true;
    }

    protected function resolveValue(array $context, ?array $extractedAttributes): mixed
    {
        return $this->localeContext->getLocaleCode();
    }
}
```

`src/Attribute/LocaleCodeAware.php`:
```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Attribute;

#[\Attribute(\Attribute::TARGET_CLASS)]
final class LocaleCodeAware
{
    public function __construct(
        public readonly string $constructorArgumentName = 'localeCode',
    ) {
    }
}
```

## 6. Example Migration: Conditional Logic

If your DataTransformer has conditional logic:

**Before:**
```php
public function transform($object, string $to, array $context = [])
{
    // Only inject if user is authenticated
    if ($this->security->getUser()) {
        $object->userId = $this->security->getUser()->getId();
    }

    return $object;
}
```

**After:**
```php
protected function supports(Request $request, array $context, ?array $extractedAttributes): bool
{
    // Only inject if user is authenticated
    return $this->security->getUser() !== null;
}

protected function resolveValue(array $context, ?array $extractedAttributes): mixed
{
    return $this->security->getUser()?->getId();
}
```

## 7. Alternative: Manual SerializerContextBuilder

If you don't want to use `AbstractInputContextBuilder`, you can implement `SerializerContextBuilderInterface` directly:

```php
<?php

declare(strict_types=1);

namespace Vendor\Plugin\Serializer\ContextBuilder;

use ApiPlatform\State\SerializerContextBuilderInterface;
use Sylius\Component\Channel\Context\ChannelContextInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Serializer\Normalizer\AbstractNormalizer;

final class ChannelCodeAwareContextBuilder implements SerializerContextBuilderInterface
{
    public function __construct(
        private readonly SerializerContextBuilderInterface $decoratedContextBuilder,
        private readonly ChannelContextInterface $channelContext,
    ) {
    }

    public function createFromRequest(Request $request, bool $normalization, ?array $extractedAttributes = null): array
    {
        $context = $this->decoratedContextBuilder->createFromRequest($request, $normalization, $extractedAttributes);

        // Only modify context for denormalization (input)
        if ($normalization) {
            return $context;
        }

        $inputClass = $context['input']['class'] ?? null;
        if ($inputClass === null) {
            return $context;
        }

        // Check if DTO has the attribute
        $reflection = new \ReflectionClass($inputClass);
        $attributes = $reflection->getAttributes(\Vendor\Plugin\Attribute\ChannelCodeAware::class);
        if (empty($attributes)) {
            return $context;
        }

        // Inject channel code into DTO constructor
        $channelCode = $this->channelContext->getChannel()->getCode();
        $context[AbstractNormalizer::DEFAULT_CONSTRUCTOR_ARGUMENTS][$inputClass]['channelCode'] = $channelCode;

        return $context;
    }
}
```

This approach gives you more control but requires more boilerplate code.

## 8. Remove Old Files

After migration is complete and tested:

```bash
# Remove old directories
rm -rf src/DataTransformer

# Remove old service configurations
rm config/services/dataTransformer/dataTransformer.xml
rmdir config/services/dataTransformer
```

## 9. Validate Changes

Clear the cache:
```bash
vendor/bin/console cache:clear
```

Verify SerializerContextBuilder is registered:
```bash
vendor/bin/console debug:container | grep context_builder
```

Test API endpoints:
```bash
# Test POST endpoint - channel code should be injected automatically
curl -X POST http://localhost/api/v2/shop/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Product","code":"TEST"}'
```

## Important Notes

1. **Decorator pattern:** Always use `decorates="api_platform.serializer.context_builder"`
2. **Constructor injection:** Values are injected into DTO constructor, not set as properties afterward
3. **Attributes required:** DTOs must be marked with custom PHP 8 attributes
4. **AbstractInputContextBuilder:** Sylius provides this base class for common patterns
5. **Not all transformers migrate:** Only those injecting contextual data become ContextBuilders
6. **Final classes:** Follow Sylius convention with `final` keyword
7. **Readonly:** Consider using `readonly` for immutable dependencies

## Common Patterns

### Pattern: Multiple Values Injection

If you need to inject multiple values (channel code AND locale code):

**Option 1: Use both attributes on the DTO**
```php
use Vendor\Plugin\Attribute\ChannelCodeAware;
use Vendor\Plugin\Attribute\LocaleCodeAware;

#[ChannelCodeAware]
#[LocaleCodeAware]
class CreateProductCommand
{
    public function __construct(
        protected string $channelCode,
        protected string $localeCode,
        protected string $name,
    ) {
    }
}
```

Both ContextBuilders will run and inject their values.

**Option 2: Single ContextBuilder injecting multiple values**
```php
final class ChannelAndLocaleContextBuilder implements SerializerContextBuilderInterface
{
    public function createFromRequest(Request $request, bool $normalization, ?array $extractedAttributes = null): array
    {
        $context = $this->decoratedContextBuilder->createFromRequest($request, $normalization, $extractedAttributes);

        if (!$normalization && isset($context['input']['class'])) {
            $inputClass = $context['input']['class'];
            $context[AbstractNormalizer::DEFAULT_CONSTRUCTOR_ARGUMENTS][$inputClass]['channelCode'] = $this->channelContext->getChannel()->getCode();
            $context[AbstractNormalizer::DEFAULT_CONSTRUCTOR_ARGUMENTS][$inputClass]['localeCode'] = $this->localeContext->getLocaleCode();
        }

        return $context;
    }
}
```

### Pattern: Custom Attribute Parameter

If you want to customize the constructor parameter name per DTO:

```php
#[ChannelCodeAware(constructorArgumentName: 'channel')]
class CreateProductCommand
{
    public function __construct(
        protected string $channel, // Different parameter name
        protected string $name,
    ) {
    }
}
```

The attribute's `constructorArgumentName` will be used by `AbstractInputContextBuilder`.

## Reference

For more examples, check Sylius core SerializerContextBuilders:
```
vendor/sylius/sylius/src/Sylius/Bundle/ApiBundle/Serializer/ContextBuilder/
```

Example files:
- `ChannelCodeAwareContextBuilder.php` - Channel code injection
- `LocaleCodeAwareContextBuilder.php` - Locale code injection
- `LoggedInShopUserIdAwareContextBuilder.php` - User ID injection
- `AbstractInputContextBuilder.php` - Base class for common pattern
