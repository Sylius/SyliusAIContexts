# API: Migrate DataTransformer to SerializerContextBuilder

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
Execute only if:
- Plugin has custom DataTransformer classes (check `src/DataTransformer/` directory)
- Plugin implements `DataTransformerInterface` or related interfaces

Skip if plugin has no custom data transformers.

## Commands

### 1. Detect Existing DataTransformers

Check for DataTransformer classes:
```
Glob: "src/DataTransformer/*.php"
Glob: "src/**/DataTransformer/*.php"
```

If none found, skip this step.

### 2. Analyze Each DataTransformer

For each DataTransformer file found:

#### 2a. Read Current DataTransformer

```
Read src/DataTransformer/{FileName}.php
```

Check what interfaces it implements:
- `DataTransformerInterface` → May need migration to SerializerContextBuilder
- `InputDataTransformerInterface` → Likely needs migration to SerializerContextBuilder
- `OutputDataTransformerInterface` → May need different approach

**Important Note:** Not all DataTransformers become SerializerContextBuilders. Only those that modify input DTOs by injecting contextual data (like channel code, locale code, user ID) should be migrated to SerializerContextBuilders.

#### 2b. Determine Migration Strategy

**Migrate to SerializerContextBuilder if DataTransformer:**
- Injects channel code into input DTO
- Injects locale code into input DTO
- Injects current user ID into input DTO
- Injects any contextual data from request/session into input DTO constructor

**Alternative approaches if DataTransformer:**
- Transforms output format → May need custom Normalizer
- Performs complex data transformation → May need StateProvider/StateProcessor
- Validates data → Should use Symfony Validator

### 3. Migrate to SerializerContextBuilder

For DataTransformers that inject contextual data:

#### 3a. Determine New Directory Structure

Follow Sylius pattern: `src/Serializer/ContextBuilder/{Name}ContextBuilder.php`

Example mappings:
- `ChannelCodeAwareInputCommandDataTransformer` → `Serializer/ContextBuilder/ChannelCodeAwareContextBuilder.php`
- `LocaleCodeAwareInputCommandDataTransformer` → `Serializer/ContextBuilder/LocaleCodeAwareContextBuilder.php`
- `LoggedInUserIdAwareDataTransformer` → `Serializer/ContextBuilder/LoggedInUserIdAwareContextBuilder.php`

#### 3b. Create New SerializerContextBuilder

```
Bash: mkdir -p src/Serializer/ContextBuilder
```

**Option 1: Extend AbstractInputContextBuilder (Recommended)**

```
Write src/Serializer/ContextBuilder/{Name}ContextBuilder.php:
<?php

declare(strict_types=1);

namespace {Vendor}\{Plugin}\Serializer\ContextBuilder;

use ApiPlatform\State\SerializerContextBuilderInterface;
use Sylius\Bundle\ApiBundle\Serializer\ContextBuilder\AbstractInputContextBuilder;
use Symfony\Component\HttpFoundation\Request;

final class {Name}ContextBuilder extends AbstractInputContextBuilder
{
    public function __construct(
        SerializerContextBuilderInterface $decoratedContextBuilder,
        string $attributeClass,
        string $defaultConstructorArgumentName,
        // Add your dependencies here (e.g., ChannelContextInterface)
        private readonly {DependencyInterface} $dependency,
    ) {
        parent::__construct($decoratedContextBuilder, $attributeClass, $defaultConstructorArgumentName);
    }

    protected function supports(Request $request, array $context, ?array $extractedAttributes): bool
    {
        // Determine if this context builder should run for this request
        // Return true to always apply, or add custom logic
        return true;
    }

    protected function resolveValue(array $context, ?array $extractedAttributes): mixed
    {
        // Return the value to inject into the DTO constructor
        // Example: return $this->channelContext->getChannel()->getCode();
        return $this->dependency->getValue();
    }
}
```

**Option 2: Implement SerializerContextBuilderInterface directly**

```
Write src/Serializer/ContextBuilder/{Name}ContextBuilder.php:
<?php

declare(strict_types=1);

namespace {Vendor}\{Plugin}\Serializer\ContextBuilder;

use ApiPlatform\State\SerializerContextBuilderInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Serializer\Normalizer\AbstractNormalizer;

final class {Name}ContextBuilder implements SerializerContextBuilderInterface
{
    public function __construct(
        private readonly SerializerContextBuilderInterface $decoratedContextBuilder,
        // Add your dependencies
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

        // Inject value into DTO constructor
        $context[AbstractNormalizer::DEFAULT_CONSTRUCTOR_ARGUMENTS][$inputClass]['parameterName'] = $this->getValue();

        return $context;
    }

    private function getValue(): mixed
    {
        // Get value from dependency
        return // value;
    }
}
```

**Key changes:**
- Implements `SerializerContextBuilderInterface` instead of `DataTransformerInterface`
- Method `transform()` → `createFromRequest()` or `supports()` + `resolveValue()`
- Decorator pattern: wraps another SerializerContextBuilder
- Injects values via `DEFAULT_CONSTRUCTOR_ARGUMENTS` context key
- Mark class as `final` and optionally `readonly`

#### 3c. Update Service Registration

Check for old service registration:
```
Grep: pattern="{OldClassName}" path="config/services"
```

Create new service file:
```
Write config/services/serializer/contextBuilder.xml (or append to existing):
<service id="{vendor}.{plugin}.serializer.context_builder.{name}"
         class="{Vendor}\{Plugin}\Serializer\ContextBuilder\{Name}ContextBuilder"
         decorates="api_platform.serializer.context_builder"
>
    <argument type="service" id=".inner" />

    <!-- If using AbstractInputContextBuilder pattern: -->
    <argument>%{vendor}_{plugin}.attribute.{name}.class%</argument>
    <argument>parameterName</argument>

    <!-- Add your dependencies -->
    <argument type="service" id="{dependency_service_id}" />
</service>
```

**Service registration notes:**
- Use `decorates="api_platform.serializer.context_builder"` to wrap the core builder
- First argument is `.inner` (the decorated service)
- If extending `AbstractInputContextBuilder`:
  - 2nd argument: Attribute class (e.g., `Vendor\Plugin\Attribute\ChannelCodeAware`)
  - 3rd argument: Default constructor parameter name
- Add your custom dependencies after

#### 3d. Create Attribute (if using AbstractInputContextBuilder)

If extending `AbstractInputContextBuilder`, create a PHP attribute to mark DTOs:

```
Bash: mkdir -p src/Attribute
```

```
Write src/Attribute/{Name}.php:
<?php

declare(strict_types=1);

namespace {Vendor}\{Plugin}\Attribute;

#[\Attribute(\Attribute::TARGET_CLASS)]
final class {Name}
{
    public function __construct(
        public readonly string $constructorArgumentName = 'defaultParamName',
    ) {
    }
}
```

Example: `ChannelCodeAware` attribute:
```php
#[\Attribute(\Attribute::TARGET_CLASS)]
final class ChannelCodeAware
{
    public function __construct(
        public readonly string $constructorArgumentName = 'channelCode',
    ) {
    }
}
```

#### 3e. Mark Input DTOs with Attribute

Update your input DTOs/Commands to use the attribute:

```
Edit src/Command/{CommandName}.php:

Add use statement:
use {Vendor}\{Plugin}\Attribute\{AttributeName};

Add attribute to class:
#[{AttributeName}]
class {CommandName}
{
    public function __construct(
        protected string $parameterName, // This parameter will be injected
        // other parameters
    ) {
    }
}
```

Example:
```php
use Vendor\Plugin\Attribute\ChannelCodeAware;

#[ChannelCodeAware(constructorArgumentName: 'channelCode')]
class CreateProductCommand
{
    public function __construct(
        protected string $channelCode,
        protected string $name,
    ) {
    }
}
```

### 4. Remove Old Files

After successful migration and validation:

```
Bash: rm -rf src/DataTransformer
Bash: rm config/services/dataTransformer/*.xml
Bash: rmdir config/services/dataTransformer
```

### 5. Validate Configuration

```
Bash: vendor/bin/console cache:clear
```

Expected: Cache clears without errors.

```
Bash: vendor/bin/console debug:container | grep "context_builder"
```

Expected: Your SerializerContextBuilder service appears in the list.

## Success Criteria
- All DataTransformer classes migrated to SerializerContextBuilder (or alternative)
- Services registered with decorator pattern
- Input DTOs marked with attributes (if using AbstractInputContextBuilder)
- Old DataTransformer directory removed
- Cache clears successfully
- API POST/PUT/PATCH operations work correctly

## Notes for AI
- **Not all DataTransformers migrate to ContextBuilders**: Only those injecting contextual data
- **Decorator pattern**: ContextBuilders wrap the core serializer context builder
- **AbstractInputContextBuilder**: Sylius provides base class for common pattern
- **Attributes required**: If using AbstractInputContextBuilder, create and use PHP 8 attributes
- **Constructor injection**: ContextBuilders inject values into DTO constructor arguments
- **Mark as final**: Follow Sylius conventions with `final` class
- **Service decoration**: Always use `decorates="api_platform.serializer.context_builder"`

## Common Migration Patterns

### Pattern 1: Channel Code Injection
**Before (DataTransformer):**
```php
use ApiPlatform\Core\DataTransformer\DataTransformerInterface;

class ChannelCodeAwareInputCommandDataTransformer implements DataTransformerInterface
{
    public function __construct(
        private ChannelContextInterface $channelContext,
    ) {}

    public function transform($object, string $to, array $context = [])
    {
        $object->channelCode = $this->channelContext->getChannel()->getCode();
        return $object;
    }

    public function supportsTransformation($data, string $to, array $context = []): bool
    {
        return $data instanceof ChannelCodeAwareCommand;
    }
}
```

**After (SerializerContextBuilder):**
```php
use ApiPlatform\State\SerializerContextBuilderInterface;
use Sylius\Bundle\ApiBundle\Serializer\ContextBuilder\AbstractInputContextBuilder;
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

**Service registration:**
```xml
<service id="vendor.plugin.serializer.context_builder.channel_code_aware"
         class="Vendor\Plugin\Serializer\ContextBuilder\ChannelCodeAwareContextBuilder"
         decorates="api_platform.serializer.context_builder">
    <argument type="service" id=".inner" />
    <argument>%vendor_plugin.attribute.channel_code_aware.class%</argument>
    <argument>channelCode</argument>
    <argument type="service" id="sylius.context.channel" />
</service>
```

**Attribute:**
```php
#[\Attribute(\Attribute::TARGET_CLASS)]
final class ChannelCodeAware
{
    public function __construct(
        public readonly string $constructorArgumentName = 'channelCode',
    ) {
    }
}
```

**DTO usage:**
```php
use Vendor\Plugin\Attribute\ChannelCodeAware;

#[ChannelCodeAware]
class CreateProductCommand
{
    public function __construct(
        protected string $channelCode, // Will be injected automatically
        protected string $name,
    ) {
    }
}
```

### Pattern 2: Locale Code Injection
**Before:**
```php
class LocaleCodeAwareInputCommandDataTransformer implements DataTransformerInterface
{
    public function transform($object, string $to, array $context = [])
    {
        $object->localeCode = $this->localeContext->getLocaleCode();
        return $object;
    }
}
```

**After:**
```php
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

### Pattern 3: Logged-in User ID Injection
**Before:**
```php
class LoggedInUserIdAwareDataTransformer implements DataTransformerInterface
{
    public function transform($object, string $to, array $context = [])
    {
        $object->userId = $this->security->getUser()->getId();
        return $object;
    }
}
```

**After:**
```php
final class LoggedInUserIdAwareContextBuilder extends AbstractInputContextBuilder
{
    public function __construct(
        SerializerContextBuilderInterface $decoratedContextBuilder,
        string $attributeClass,
        string $defaultConstructorArgumentName,
        private readonly Security $security,
    ) {
        parent::__construct($decoratedContextBuilder, $attributeClass, $defaultConstructorArgumentName);
    }

    protected function supports(Request $request, array $context, ?array $extractedAttributes): bool
    {
        return $this->security->getUser() !== null;
    }

    protected function resolveValue(array $context, ?array $extractedAttributes): mixed
    {
        return $this->security->getUser()?->getId();
    }
}
```

## Testing (Optional)
If API endpoints are available:
```
# Test POST endpoint with context builder
Bash: curl -X POST http://localhost/api/v2/shop/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Product"}'
```

Expected: JSON response with created resource. The channelCode/localeCode should be injected automatically.
