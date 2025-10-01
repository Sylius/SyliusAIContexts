# AI Sylius 2.0 Migration Guide

This guide provides step-by-step instructions for AI Agent to migrate any Sylius plugin from version 1.14 to 2.0.

## Before Starting

1. Check current Sylius version in composer.json
2. Understand plugin structure and components
3. Create migration task list with all steps

## Migration Steps

### Dependencies & Configuration
1. [Dependencies Update](migration-guides/dependencies-update.md) - Update composer.json for Sylius 2.0
2. [Configuration Restructuring](migration-guides/configuration-restructuring.md) - Move config from src/Resources to root

### Core Code Migration  
3. [Entity Migration](migration-guides/entity-migration.md) - Add PHP 8 attributes to entities
4. [Services Migration](migration-guides/services-migration.md) - Fix service compatibility issues

### Application Configuration
5. [Routing Migration](migration-guides/routing-migration.md) - Update routing patterns
6. [API Platform Migration](migration-guides/api-platform-migration.md) - Migrate to API Platform 4.x
7. [Sylius Template Path Migration](migration-guides/grid-migration.md) - Update Sylius template paths
8. [Menu Migration](migration-guides/menu-migration.md) - Update admin menu with Tabler icons

### Templates & Frontend
9. [Template Migration](migration-guides/template-migration.md) - Convert to Twig Hooks and Bootstrap 5
10. [Asset Migration](migration-guides/asset-migration.md) - Update webpack and assets

### Integration & Testing
11. [Testing Updates](migration-guides/testing-updates.md) - Fix test suite

### Quality & Validation
13. [Final Validation](migration-guides/final-validation.md) - Complete testing and verification

## Validation Protocol

### After Each Step
```bash
vendor/bin/console debug:container --env=dev
```
Expected: No new errors

## Success Criteria

Migration is complete when:
- ✅ Container compiles in all environments (dev/test/prod)
- ✅ PHPStan Level 8 analysis passes (0 errors)
- ✅ All tests pass (PHPUnit, PHPSpec, Behat)
- ✅ Admin panel displays with Bootstrap 5 styling
- ✅ Database schema validates correctly
- ✅ Assets compile successfully

## Error Recovery

If container compilation fails:
1. Use `Read var/log/dev.log` to examine errors
2. Check service definitions in config/services.xml
3. Clear cache: `rm -rf var/cache/*`
4. Revert to last working state if needed

## Progress Tracking

Use `TodoWrite` throughout migration to:
- Mark completed steps
- Track current progress  
- Note any issues or deviations
- Maintain focus on current task

*This guide is optimized for AI execution with direct tool commands and systematic validation.*
