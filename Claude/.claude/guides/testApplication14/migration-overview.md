# TestApplication 1.4 Migration Overview

This guide migrates a Sylius plugin from using a traditional `tests/Application` setup to the centralized Sylius TestApplication 1.4 framework.

## Migration Steps

Each step should be completed in order, with a git commit after each step:

| Step | Guide File                                     | Description                                                             | Commit Message                                              |
|------|------------------------------------------------|-------------------------------------------------------------------------|-------------------------------------------------------------|
| 1    | [preparation.md](preparation.md)               | Update composer.json, create directory structure, update .gitignore     | `feat: prepare project for testApplication 1.4 migration`   |
| 2    | [testapp-config.md](testapp-config.md)         | Set up TestApplication configuration files                              | `feat: configure testApplication files`                     |
| 3    | [database-config.md](database-config.md)       | Configure database connection for TestApplication                       | `feat: configure database for testApplication`              |
| 4    | [custom-content.md](custom-content.md)         | Copy custom entities, templates, and configuration from old Application | `feat: migrate custom content to testApplication structure` |
| 5    | [test-configs.md](test-configs.md)             | Update Behat and PHPUnit configurations                                 | `feat: update test configurations for testApplication`      |
| 6    | [github-actions.md](github-actions.md)         | Update CI/CD workflow for TestApplication                               | `feat: update GitHub Actions for testApplication`           |
| 7    | [asset-migration.md](asset-migration.md)       | Migrate plugin assets to new structure                                  | `feat: migrate assets to testApplication structure`         |
| 8    | [testing-validation.md](testing-validation.md) | Test migration, validate, and clean up                                  | `feat: complete testApplication migration and cleanup`      |

## Prerequisites

- Sylius plugin with existing `tests/Application` setup
- Composer dependency management
- Basic understanding of Behat, PHPUnit, and Symfony configuration

## Important Notes

- Each step builds on the previous one - do not skip steps
- Commit after each step for easy rollback if needed
- Test the migration thoroughly before removing old files
- Keep the old `tests/Application` until all tests pass

## Success Criteria

Migration is complete when:
- [ ] All tests pass (ECS, PHPStan, PHPUnit, PHPSpec, Behat)
- [ ] Assets compile successfully
- [ ] Database migrations work
- [ ] CI/CD pipeline works
- [ ] Old `tests/Application` directory removed
