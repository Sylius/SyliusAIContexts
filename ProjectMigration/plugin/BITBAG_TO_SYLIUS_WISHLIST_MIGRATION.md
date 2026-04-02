# BitBag to Sylius Wishlist Plugin Migration

## Prerequisites

This migration is performed during upgrading your Sylius application from 1.14 to 2.0+. Since the BitBag plugin only supports Sylius up to 1.14, it will be incompatible and removed during the upgrade process.

## Migration Steps

### 1. Install Sylius Wishlist Plugin v1.1.1

```bash
composer require sylius/wishlist-plugin:^1.1.1
```

Follow the [installation guide](https://docs.sylius.com/wishlist-plugin/installation).

### 2. Update Plugin References

Update the following references in your codebase:

**Twig templates:**
```diff
- {% include '@BitBagSyliusWishlistPlugin/...' %}
+ {% include '@SyliusWishlistPlugin/...' %}
```

Find all occurrences:
```bash
grep -r "@BitBagSyliusWishlistPlugin" --include="*.twig" your/custom/templates/
grep -r "BitBag\\\\SyliusWishlistPlugin" --include="*.php" your/project/
```

### 3. Update Wishlist Views

The Sylius Wishlist Plugin includes default views that work out of the box with Sylius 2.0.

**If using default Sylius templates:**
- No additional changes needed. Wishlist functionality will work automatically.

**If you have customized wishlist templates:**
- Update your custom template files to match the new plugin's structure
- Refer to the [Sylius Wishlist Plugin documentation](https://docs.sylius.com/wishlist-plugin/installation) for the updated template structure

### 4. Database Migration

The Sylius Wishlist Plugin v1.1.1+ handles database schema migration automatically during the installation process.

## Supported Migrations

The following versions of Sylius Wishlist Plugin support automated migration from BitBag plugin:

- **Sylius Wishlist Plugin v1.1.1+** - Full support for automatic data migration from BitBag Wishlist Plugin
  - Wishlist data is automatically mapped to the new schema
  - User wishlists and their items are preserved
  - No manual intervention required
