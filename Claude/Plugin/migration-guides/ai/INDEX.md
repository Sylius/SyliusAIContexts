# Sylius 2.0 Migration Guide - AI Automation

This guide provides tool commands and automation instructions for AI to migrate Sylius plugins from version 1.14 to 2.0.

## Migration Steps

Follow these steps in order to automate plugin migration to Sylius 2.0.

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
   - 7.4 **Templates** - Migrate to Twig Hooks and Bootstrap 5
     - **[Overview](admin-templates-overview.md)** - Decision guide
     - **[Twig Hooks](admin-templates-hooks.md)** - Create hooks structure
     - **[Live Component (Basic)](admin-templates-live-basic.md)** - Add live validation
   - 7.5 **[Assets](admin-assets.md)** - Update admin JavaScript and CSS
   - 7.6 **[Live Component (Custom)](admin-templates-live-custom.md)** - Add custom actions (after analyzing assets)

### Phase 4: Shop Migration

8. **Shop**
   - 8.1 **[Routing](shop-routing.md)** - Update shop routing configuration
   - 8.2 **[Menu](shop-menu.md)** - Update shop menu (optional)
   - 8.3 **Grids** _(optional, same as admin)_ - Update shop grids
   - 8.4 **Templates** - Migrate shop templates to Twig Hooks
     - **[Overview](shop-templates-overview.md)** - Decision guide
     - **[Custom Hooks](shop-templates-hooks.md)** - Create hooks for custom pages
   - 8.5 **[Assets](shop-assets.md)** - Update shop JavaScript and CSS

### Phase 5: API Migration

9. **API** _(coming soon)_
   - 9.1 **API Platform 4.x** - Migrate to API Platform 4.x
   - 9.2 **Serialization** - Update serialization configuration
   - 9.3 **Routes** - Update API routes

### Phase 6: Finalization

10. **[Cleanup](cleanup.md)** - Review and remove TODO markers, commented code
11. **Fix Tests** _(coming soon)_ - Update and fix unit and integration tests
12. **Manual Testing** _(coming soon)_ - Manual browser testing checklist
13. **Final Validation** _(coming soon)_ - CI checks, documentation, changelog

## Tool Usage

Each guide contains specific tool commands (Read, Edit, Write, Bash, Grep, Glob) that AI can execute to automate the migration process.

## Decision Tree for AI

Before starting migration, AI should:

1. **Detect what applies to the plugin:**
   - Check if plugin has admin functionality → Run Admin steps
   - Check if plugin has shop functionality → Run Shop steps
   - Check if plugin has API → Run API steps
   - Check if plugin uses grids → Run Grid steps
   - Check if plugin has assets → Run Asset steps

2. **Auto-skip unnecessary steps:**
   - No `src/Resources/` → Skip step 4 (Move Config)
   - Entities use XML mapping → Skip step 6.1 (Entity migration)
   - No shop templates → Skip step 8
   - No API → Skip step 9

3. **Validate after each step:**
   - Run `cache:clear` and check for errors
   - Run step-specific validation commands
   - Stop if validation fails and report error

## Available Tools

- **Read** - Read file contents
- **Edit** - Replace text in files
- **Write** - Create or overwrite files
- **Bash** - Execute shell commands
- **Grep** - Search for patterns in files
- **Glob** - Find files by pattern

## Workflow Pattern

Each step follows: **Execute → Validate → Report**

1. Execute tool commands from guide
2. Validate changes (cache clear, debug commands)
3. Report success or failure to user
