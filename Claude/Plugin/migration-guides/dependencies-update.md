# Dependencies Update

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Check Current State
```
Read composer.json
```

### 2. Update Core Dependencies
```
Edit composer.json:
- "php": "^8.0" → "php": "^8.2"
- "sylius/sylius": "^1.14" → "sylius/sylius": "^2.0"
```

### 3. Update Symfony Packages  
```
Edit composer.json:
- "symfony/browser-kit": "^5.4 || ^6.4" → "^6.4"
- "symfony/debug-bundle": "^5.4 || ^6.4" → "^6.4"
- "symfony/dotenv": "^5.4 || ^6.4" → "^6.4"
- "symfony/intl": "^5.4 || ^6.4" → "^6.4"
- "symfony/web-profiler-bundle": "^5.4 || ^6.4" → "^6.4"
- "symfony/webpack-encore-bundle": "^1.14" → "^2.2"
```

### 4. Add Plugin Configuration (if missing)
```
Edit composer.json:
Add to config section:
"allow-plugins": {
    "php-http/discovery": false,
    "symfony/flex": true,
    "symfony/runtime": true
}
```

### 5. Install Dependencies
```
Bash: composer update
```

### 6. Validate
```
Bash: vendor/bin/console debug:container --env=dev
```
Expected: Container compiles without fatal errors

### 7. Fix Issues (if validation fails)
- Check memory limits: `php -d memory_limit=1G composer update`
- Remove incompatible plugins temporarily
- Clear composer cache: `composer clear-cache`

### 8. Commit Changes
```
Bash: git add .
Bash: git commit -m "Update dependencies to Sylius 2.0"
```