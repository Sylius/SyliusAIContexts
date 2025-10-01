# Testing Updates

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Update Behat Contexts
```
Glob: "tests/Behat/Context/**/*.php"
```
Update service references for Sylius 2.0

### 2. Update Test Configuration
```
Read tests/TestApplication/config/services_test.php
```
Verify test services are properly configured

### 3. Run Tests
```
Bash: vendor/bin/phpunit
Bash: vendor/bin/phpspec run
Bash: vendor/bin/behat
```

### 4. Validate
Expected: All tests pass

### 5. Commit Changes
```
Bash: git add .
Bash: git commit -m "Update test configuration for Sylius 2.0"
```