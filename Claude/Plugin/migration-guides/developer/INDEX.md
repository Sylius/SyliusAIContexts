# Sylius 2.0 Migration Guide - Developer Documentation

This guide provides a structured approach to migrating Sylius plugins from version 1.14 to 2.0.

## Migration Steps

Follow these steps in order to migrate your plugin to Sylius 2.0.

### Phase 1: Setup & Preparation

1. **[Update Composer](update-composer.md)** - Update dependencies, PHP version, Symfony packages
2. **[Fix Container Compilation](fix-container-compilation.md)** - Fix service container compilation errors
3. **Code Quality Tools** _(coming soon)_ - Run ECS, PHPStan, Psalm (optional)
4. **[Move Config to Root](move-config-to-root.md)** _(optional)_ - Move configuration from `src/Resources/config` to root `config/`
5. **[Prepare and Run](prepare-and-run.md)** - Setup test application, database, and run server

### Phase 2: Common Migration

6. **Common**
   - 6.1 **[Entities](common-entities.md)** - Migrate to PHP 8 attributes, update mappings

### Phase 3: Admin Panel Migration

7. **Admin**
   - 7.1 **[Routing](admin-routing.md)** - Update routing configuration
   - 7.2 **[Menu](admin-menu.md)** - Migrate to new icons and menu structure
   - 7.3 **[Grids](admin-grids.md)** - Update grid templates and configuration
   - 7.4 **[Templates](admin-templates.md)** - Migrate to Twig Hooks and Bootstrap 5
   - 7.5 **[Assets](admin-assets.md)** - Update admin JavaScript and CSS

### Phase 4: Shop Migration

8. **Shop**
   - 8.1 **[Routing](shop-routing.md)** - Update shop routing configuration
   - 8.2 **[Menu](shop-menu.md)** - Update shop menu (optional)
   - 8.3 **Grids** _(optional, same as admin)_ - Update shop grids
   - 8.4 **[Templates](shop-templates.md)** - Migrate shop templates to Twig Hooks
   - 8.5 **[Assets](shop-assets.md)** - Update shop JavaScript and CSS

### Phase 5: API Migration

9. **API** _(coming soon)_
   - 9.1 **API Platform 4.x** - Migrate to API Platform 4.x
   - 9.2 **Serialization** - Update serialization configuration

### Phase 6: Finalization

10. **Cleanup** _(coming soon)_ - Remove commented code, deprecated code, format files
11. **Fix Tests** _(coming soon)_ - Update and fix unit and integration tests
12. **Manual Testing** _(coming soon)_ - Manual browser testing checklist
13. **Final Validation** _(coming soon)_ - CI checks, documentation, changelog

## Optional Steps

Some steps may not apply to all plugins:
- Step 4 (Move Config) - Optional if you prefer old structure
- Step 7.3 (Admin Grids) - Skip if plugin doesn't use grids
- Step 7.5 (Admin Assets) - Skip if plugin has no admin JavaScript/CSS
- Step 8.5 (Shop Assets) - Skip if plugin has no shop JavaScript/CSS
- Step 8.* (Shop) - Skip if plugin has no shop functionality
- Step 9.* (API) - Skip if plugin has no API

## Migration Workflow

Each step follows this workflow:
1. Read the guide
2. Apply changes manually
3. Validate the changes
4. Commit if successful
5. Move to next step

## Need Help?

- Check the Sylius 2.0 UPGRADE file: `vendor/sylius/sylius/UPGRADE-2.0.md`
- Review example implementations in this plugin's git history
