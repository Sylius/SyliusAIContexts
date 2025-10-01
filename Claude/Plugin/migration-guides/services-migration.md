# Services Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Check for Service Issues
```
Bash: vendor/bin/console debug:container --env=dev
```
If container compiles successfully, skip this step.

### 2. Fix Service Definitions (if needed)
```
Glob: "src/**/*.php"
```
For files with errors, check for:
- Deprecated Sylius method calls
- Missing type hints and return types
- Interface compatibility issues
- Service container configuration

### 3. Update Type Annotations
For each problematic service:
```
Read src/{service_path}/{file}.php
Edit src/{service_path}/{file}.php:
- Add proper return type hints
- Fix deprecated method calls
- Update interface implementations
```

### 4. Validate Container
```
Bash: vendor/bin/console debug:container --env=dev
```
Expected: Container compiles successfully

### 5. Run Static Analysis
```
Bash: vendor/bin/phpstan analyse
Bash: vendor/bin/ecs check
```
Expected: No errors (or only baseline errors)

### 6. Fix Issues (if validation fails)
- Fix PHPStan errors related to service compatibility
- Fix ECS coding standard violations
- Verify service definitions in config/services.xml
- Update event listener service definitions if needed
- Clear cache: `rm -rf var/cache/*`

### 7. Commit Changes
```
Bash: git add .
Bash: git commit -m "Fix services compatibility for Sylius 2.0"
```