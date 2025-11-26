# Sylius 1.14 to 2.0 Migration Guide

This guide provides a step-by-step approach to migrate from Sylius 1.14 to 2.0, based on real migration experience.

## General Migration Steps

### Bundle & Plugin Migration
1. [Bundle & Plugin Migration](migration-guides/bundle-plugin-migration.md) - Install and configure required plugins and bundles

### Basic Configuration
2. [Basic Configuration Migration](migration-guides/basic-configuration-migration.md) - Migrate parameters, security, and framework configurations

### Entity & Repository Migration
3. [Entity Migration](migration-guides/entity-migration.md) - Migrate entities with dual annotation/attribute syntax
4. [Repository Migration](migration-guides/repository-migration.md) - Migrate repositories and add resource configuration
5. [Form Migration](migration-guides/form-migration.md) - Migrate forms and handle AbstractResourceType requirements

### Application Configuration
6. [Validation Migration](migration-guides/validation-migration.md) - Migrate validation constraints and configuration
7. [Serialization Migration](migration-guides/serialization-migration.md) - Migrate serialization groups and configuration
8. [Routing Migration](migration-guides/routing-migration.md) - Update routing to Sylius 2.0 patterns
9. [API Platform Migration](migration-guides/api-platform-migration.md) - Migrate API Platform from 2.7 to 4.x compatibility
10. [Grid Migration](migration-guides/grid-migration.md) - Migrate grid configurations and template paths
11. [Menu Migration](migration-guides/menu-migration.md) - Migrate menu listeners and configuration
12. [Remaining Code Migration](migration-guides/remaining-code-migration.md) - Copy all remaining src/ directories and service definitions

### Data & Testing
13. [Fixtures Migration](migration-guides/fixtures-migration.md) - Create fixtures with Faker integration

## Template Migration Steps

### Setup & Assets
14. [Template Setup](migration-guides/template-setup.md) - Set up Twig Hooks structure
15. [Asset Templates](migration-guides/asset-templates.md) - Migrate styles and scripts to hooks
16. [Template Migration](migration-guides/template-migration.md) - Migrate all custom templates to hooks

## Final Steps
17. [Testing & Validation](migration-guides/testing-validation.md) - Complete testing methodology
18. [Cleanup & Optimization](migration-guides/cleanup-optimization.md) - Remove old files and optimize

*This guide is based on real migration experience and includes all lessons learned during the process.*
