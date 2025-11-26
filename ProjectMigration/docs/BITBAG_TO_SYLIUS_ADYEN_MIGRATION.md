# BitBag to Sylius Adyen Plugin Migration

## Prerequisites

This migration is performed during upgrading your Sylius application from 1.13 to 1.14+. Since the BitBag plugin only supports Sylius up to 1.13, it will be incompatible and removed during the upgrade process.

## Migration Steps

### 1. Install Sylius Adyen Plugin v1.1

```bash
composer require sylius/adyen-plugin:^1.1
```

Follow the [installation guide](https://docs.sylius.com/adyen-plugin/v1.0/installation?fallback=true).

### 2. Update Plugin References

Update the following references in your codebase:

**Twig templates:**
```diff
- {% include '@BitBagSyliusAdyenPlugin/...' %}
+ {% include '@SyliusAdyenPlugin/...' %}

- constant('BitBag\\SyliusAdyenPlugin\\...')
+ constant('Sylius\\AdyenPlugin\\...')
```

Find all occurrences:
```bash
grep -r "@BitBagSyliusAdyenPlugin" --include="*.twig" your/custom/templates/
grep -r "BitBag\\\\\\\\SyliusAdyenPlugin" --include="*.twig" your/project/
grep -r "BitBag\\\\SyliusAdyenPlugin" --include="*.php" your/project/
```

## Supported Migrations

The following versions of Sylius Adyen Plugin support automated migration from BitBag plugin:

- **Sylius Adyen Plugin v1.1** - Full support for automatic data migration from BitBag Adyen Plugin
  - Payment data is automatically mapped to the new schema
  - Transaction history is preserved
  - No manual intervention required
