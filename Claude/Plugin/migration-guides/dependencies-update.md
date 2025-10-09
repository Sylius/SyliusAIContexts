# Dependencies Update

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Read Current State
```
Read composer.json
```

### 2. Update PHP Version
```
Edit composer.json:
Find "php": ">=8.1" or similar → "php": "^8.2"
```

### 3. Update Sylius Packages in require
```
Edit composer.json in require section:
- "sylius/core-bundle": "^1.x" → "^2.0"
- Keep "sylius/resource-bundle" unchanged (different versioning)
- Keep "sylius/grid-bundle" unchanged (different versioning)
- Add "sylius/twig-extra": "^0.8"
- Add "sylius/twig-hooks": "^0.8"
```
**Important**: Add twig-extra and twig-hooks ONLY if sylius/sylius is in require-dev (not in require)

### 4. Update Sylius Packages in require-dev
```
Edit composer.json in require-dev section:
- "sylius/sylius": "^1.14" → "^2.0"
- "sylius/test-application": "^1.14.x@alpha" → "^2.0.0@alpha"
```

### 5. Update Symfony Packages
```
Edit composer.json:
Replace all Symfony packages:
- Remove "^5.4 ||" from all symfony/* packages
- Keep "^6.4 || ^7.3"
- Special case: "symfony/webpack-encore-bundle": "^1.x" → "^2.2"
```

### 6. Update Composer allow-plugins
```
Edit composer.json in config.allow-plugins:
Add "php-http/discovery": true (if missing)
```

### 7. Comment Out sylius_ui Events
```
Glob: "src/DependencyInjection/*Extension.php"
Read the Extension file
Edit the Extension file:
Comment out any $container->prependExtensionConfig('sylius_ui', [...])
Add comment: "// Temporarily disabled - will be migrated to Twig Hooks in Step 10"
```

### 8. Run Composer Update
```
Bash: composer update
```
**Expected**: Composer may show errors about conflicting dependencies

### 9. Fix Dependency Conflicts Automatically
If composer update fails, read error messages and update required packages:
```
Common updates needed:
- "doctrine/collections": "^1.6" → "^2.2"
- "babdev/pagerfanta-bundle": "^3.x" → "^4.4"
- "lexik/jwt-authentication-bundle": "^2.x" → "^3.1"

For each conflict:
1. Read error message
2. Edit composer.json to update conflicting package
3. Run composer update again
4. Repeat until successful
```

### 10. Clear Cache
```
Bash: vendor/bin/console cache:clear
```
Expected: May show errors or warnings - this is normal at this stage

### 11. Validate (Optional)
```
Bash: vendor/bin/console debug:container --env=dev
```
Expected: Container may have errors - will be fixed in next steps

## Success Criteria
- composer update completes successfully
- vendor/bin/console cache:clear runs (errors OK)
- All sylius/* packages at ^2.0 (except resource-bundle and grid-bundle)
- All symfony/* packages at ^6.4 or higher

## Notes for AI
- Do NOT skip any Edit steps - all changes must be applied
- When composer update fails, read the error and update the conflicting package immediately
- Common conflicts: doctrine/collections, babdev/pagerfanta-bundle, lexik/jwt-authentication-bundle
- The cache:clear command may timeout or show errors - this is expected
