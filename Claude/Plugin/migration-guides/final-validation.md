# Final Validation

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Container Compilation
```
Bash: vendor/bin/console debug:container --env=dev
Bash: vendor/bin/console debug:container --env=test
Bash: vendor/bin/console debug:container --env=prod --no-debug
```
Expected: All environments compile successfully

### 2. Database Schema
```
Bash: vendor/bin/console doctrine:mapping:info
Bash: vendor/bin/console doctrine:schema:validate
```
Expected: "[OK] The mapping files are correct" and schema in sync

### 3. Code Quality
```
Bash: vendor/bin/phpstan analyse -l 8
Bash: vendor/bin/ecs check
```
Expected: 0 errors for both

### 4. Test Suite
```
Bash: vendor/bin/phpunit
Bash: vendor/bin/phpspec run
Bash: vendor/bin/behat
```
Expected: All tests pass

### 5. Plugin Integration Check
```
Bash: vendor/bin/console assets:install
```
Expected: Plugin assets copied successfully (if any)

### 6. Admin Panel Check
```
Bash: curl -I http://localhost/admin/{resources}/
```
Expected: HTTP 200 response

### 7. Template Validation
```
Bash: grep -r "class=\"ui " templates/
```
Expected: No results (all Semantic UI classes removed)

### 8. Validate
- ✅ All environments compile
- ✅ PHPStan Level 8: 0 errors  
- ✅ All tests pass
- ✅ Database schema valid
- ✅ Plugin assets installed
- ✅ Admin panel accessible
- ✅ Bootstrap 5 styling applied

### 9. Commit Changes
```
Bash: git add .
Bash: git commit -m "Complete Sylius 2.0 migration - all validations pass"
```