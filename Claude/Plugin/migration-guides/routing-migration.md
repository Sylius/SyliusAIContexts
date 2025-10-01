# Routing Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Update Admin Route Templates
```
Read config/routing/admin.yaml
Edit config/routing/admin.yaml:
Replace all occurrences:
- templates: "@SyliusAdmin\\Crud" → templates: "@SyliusAdmin\\shared\\crud"
```

### 2. Update Shop Route Templates  
```
Read config/routing/shop.yaml
Edit config/routing/shop.yaml:
Replace all occurrences:
- templates: "@SyliusShop\\..." → templates: "@SyliusShop\\shared\\..."
```

### 3. Update Route Resource References (if config was moved)
```
Edit config/routing.yaml:
- "@PluginName/Resources/config/routing/..." → "@PluginName/config/routing/..."
```
Replace any references from Resources/config to config

### 4. Validate
```
Bash: vendor/bin/console debug:router
```
Expected: Routes loaded without errors

### 5. Fix Issues (if validation fails)
- Check route syntax
- Verify template paths exist
- Clear routing cache: `vendor/bin/console cache:clear`

### 6. Commit Changes
```
Bash: git add .
Bash: git commit -m "Update routing configuration for Sylius 2.0"
```