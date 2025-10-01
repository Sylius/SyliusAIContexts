# Sylius Plugin 2.0 Migration Guide

A comprehensive guide for migrating Sylius plugins from version 1.x to 2.0.

## Table of Contents

1. [Plugin Analysis & Verification Strategy](#1-plugin-analysis--verification-strategy)
2. [TestApplication Setup](#2-testapplication-setup)
3. [Core Migration Steps](#3-core-migration-steps)
4. [Verification & Testing](#4-verification--testing)

---

## 1. Plugin Analysis & Verification Strategy

### 1.1 Plugin Analysis

A thorough plugin analysis is crucial as it forms the foundation for your migration and verification strategy. This analysis helps you understand what needs to be migrated and how to verify the plugin works correctly in Sylius 2.0.

#### What to Analyze

1. **Documentation & Purpose**
   - Read README.md to understand the plugin's core functionality
   - Identify the main features and use cases

2. **Dependencies & Compatibility**
   - Review composer.json for current Sylius version and dependencies
   - Identify which dependencies need updating

3. **Existing Tests**
   - Examine test coverage (PHPSpec, PHPUnit, Behat)
   - These tests reveal critical functionality and expected behavior

4. **Code Architecture**
   - Analyze the plugin structure (controllers, services, entities, templates)
   - Identify configuration files and their locations
   - Understand resource definitions and models

5. **Plugin Entry Points**
   - **Controllers**: HTTP endpoints that handle requests
   - **Commands**: CLI commands for administrative tasks
   - **Event Listeners/Subscribers**: Integration points with Sylius events
   - **Templates**: User interface components

### 1.2 Define Verification Strategy

Based on your analysis, create a verification strategy - a set of repeatable actions that will confirm the plugin works correctly. This strategy should focus on the plugin's critical path (core functionality) rather than getting lost in details.

#### Key Verification Techniques

1. **End-to-End (E2E) Tests**
   - Create PHPUnit tests that verify complete user workflows
   - These tests should work identically in both Sylius versions
   - Focus on critical plugin functionality

2. **Response Comparison**
   - Test that HTML pages return expected structure
   - Verify API endpoints return correct JSON structure
   - Compare output between old and new versions

3. **Database State Verification**
   - Confirm that data is saved correctly
   - Verify relationships are maintained
   - Check that migrations work properly

4. **Event System Verification**
   - Ensure events are dispatched correctly
   - Verify email notifications are sent
   - Confirm integrations work as expected

### 1.3 Identify Critical Path

The critical path consists of the plugin's essential functionality that must work correctly. Identifying this helps you avoid getting lost in details and focus on what truly matters.

#### Examples by Plugin Type

**Payment Plugins**
- Critical: Payment flow completion, status updates, refunds
- Non-critical: Admin UI styling, advanced reporting

**Shipping Plugins**
- Critical: Rate calculation, label generation, tracking updates
- Non-critical: Bulk operations, export features

**Content/CMS Plugins**
- Critical: Content display, basic CRUD operations
- Non-critical: Advanced layouts, preview features

**Marketing Plugins**
- Critical: Core feature activation, data collection
- Non-critical: Analytics dashboards, export formats

#### Important Note

Your verification strategy should be concise and repeatable. Don't aim for 100% coverage - focus on ensuring the plugin's main purpose is fulfilled. Secondary features and UI improvements can be addressed after the core migration succeeds.

---

## 2. TestApplication Setup

For complete TestApplication setup instructions, see: [Claude/TESTAPPLICATION_2.0_MIGRATION_GUIDE.md](Claude/TESTAPPLICATION_2.0_MIGRATION_GUIDE.md)

Key steps include:
- Update composer.json dependencies
- Create TestApplication directory structure
- Configure bundles, services, and routes
- Update test configuration files (behat.yml.dist, phpunit.xml.dist)
- Validate the TestApplication setup

---

## 3. Core Migration Steps

Execute the migration following the detailed guides in: [Claude/Plugin/AI-SYLIUS-2.0-MIGRATION-GUIDE.md](Claude/Plugin/AI-SYLIUS-2.0-MIGRATION-GUIDE.md)

This guide contains all migration steps with specific commands and validation checkpoints. The migration covers:
- Dependencies and configuration updates
- Entity and service migrations
- Template conversions to Twig Hooks and Bootstrap 5
- API Platform and routing updates
- Asset restructuring
- Testing framework updates

Individual step-by-step guides are also available in Claude/Plugin/migration-guides/ directory.

---

## 4. Verification & Testing

### 4.1 Execute Your Verification Strategy

After completing the migration steps, it's time to execute the verification strategy you defined in Step 1. This ensures that the plugin's critical functionality works correctly in Sylius 2.0.

#### Progressive Validation

Validate after each major migration step:

```bash
# Container compilation check (should run without errors)
vendor/bin/console debug:container --env=dev
vendor/bin/console debug:container --env=test
vendor/bin/console debug:container --env=prod --no-debug
```

### 4.2 Run Your Verification Tests

Execute the tests you planned in your verification strategy:

1. **Run existing plugin tests** (if they were migrated)
   ```bash
   vendor/bin/phpunit
   vendor/bin/phpspec run
   vendor/bin/behat
   ```

2. **Create and run E2E tests** for the critical paths you identified
   - Test the main user workflows
   - Verify data is saved correctly
   - Confirm integrations work

3. **Manual verification** of critical features
   - Navigate through the plugin's admin interface
   - Test the main shop functionality
   - Verify email notifications, API responses, etc.

### 4.3 Comprehensive Validation

Once all critical paths are verified, run a complete validation:

```bash
# Code quality checks
vendor/bin/phpstan analyse -l 8
vendor/bin/ecs check

# All test suites
vendor/bin/phpunit
vendor/bin/phpspec run
vendor/bin/behat

# Database validation
vendor/bin/console doctrine:schema:validate

# Asset compilation
yarn --cwd vendor/sylius/test-application build
```

### 4.4 Success Criteria

The migration is complete when:
- ✅ All environments compile without errors
- ✅ Critical path tests pass
- ✅ Database schema is valid
- ✅ Assets compile successfully
- ✅ The plugin's main functionality works as expected

Remember: The goal is not perfection but ensuring the plugin's core functionality works in Sylius 2.0. UI improvements and non-critical features can be addressed in subsequent iterations.

---

## Additional Resources

### Documentation Links

- [Sylius 2.0 Documentation](https://docs.sylius.com/en/2.0/)
- [Symfony 6.4 Documentation](https://symfony.com/doc/6.4/)
- [API Platform 4.x Documentation](https://api-platform.com/docs/)
- [Bootstrap 5 Documentation](https://getbootstrap.com/docs/5.0/)
- [Tabler Icons](https://tabler.io/icons)

### Migration Guides

Complete step-by-step guides are available in:
- [Claude/Plugin/AI-SYLIUS-2.0-MIGRATION-GUIDE.md](Claude/Plugin/AI-SYLIUS-2.0-MIGRATION-GUIDE.md) - Main AI migration guide

- [Claude/TESTAPPLICATION_2.0_MIGRATION_GUIDE.md](Claude/TESTAPPLICATION_2.0_MIGRATION_GUIDE.md) - TestApplication setup

---

*This guide is maintained for Sylius 2.0 migration and will be updated as new patterns and solutions emerge.*
