# Update Composer Dependencies

This step guides you through updating the core dependencies of your Sylius plugin to make it compatible with Sylius 2.0 and Symfony 6.4/7.x.

**💡 Note:** It's recommended to first upgrade to Sylius 1.14 (the latest 1.x version) before migrating to 2.0, as it will make the transition smoother.

## 1. Update PHP Version

Update the PHP requirement to **8.2** or higher.

## 2. Update Sylius Packages

- Update all `sylius/*` packages to `^2.0` in both `require` and `require-dev` sections
- **Exception:** `sylius/resource-bundle` and `sylius/grid-bundle` have different versioning - leave them as is

**⚠️ Important:** If `sylius/sylius` is in `require-dev` (not in `require`), you must manually add to the `require` section:
- `sylius/twig-extra`: `^0.8`
- `sylius/twig-hooks`: `^0.8`

These are required by Sylius 2.0 but are not automatically installed when `sylius/sylius` is only in dev dependencies.

## 3. Update Symfony Packages

Remove `^5.4 ||` from all Symfony packages, keeping only `^6.4 || ^7.3`.

**Special case:** `symfony/webpack-encore-bundle` should be updated to `^2.2`

## 4. Update Other Dependencies

Check and update dependencies that Sylius 2.0 requires in newer versions (e.g., `doctrine/collections`, `babdev/pagerfanta-bundle`, `lexik/jwt-authentication-bundle`).

Run `composer update` and review any suggested updates.

## 5. Update Composer Configuration

Ensure the `allow-plugins` section includes:
- `"php-http/discovery": true`
- `"symfony/flex": true`
- `"symfony/runtime": true`

## 6. Run Composer Update

```bash
composer update
```

If you encounter dependency conflicts:
```bash
composer update --with-all-dependencies
```
