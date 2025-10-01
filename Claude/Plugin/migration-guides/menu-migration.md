# Menu Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Check Menu Builder Service IDs
```
Grep: "sylius.admin.menu_builder" --include="*.xml" --include="*.yaml"
```
If found, update service IDs to new format

### 2. Update Menu Builder Service IDs (if found)
```
Edit config/services.xml:
Replace old service IDs with new ones:
- sylius.admin.menu_builder.main → sylius_admin.menu_builder.main
- sylius.shop.menu_builder.account → sylius_shop.menu_builder.account
```

### 3. Check for Removed Menu Builders
```
Grep: "CustomerShowMenuBuilder|OrderShowMenuBuilder|ProductFormMenuBuilder|ProductUpdateMenuBuilder|ProductVariantFormMenuBuilder|PromotionUpdateMenuBuilder" --include="*.php"
```
These menu builders were removed in Sylius 2.0 - replace with Twig Hooks if needed

### 4. Update Menu Listener Icons (if exists)
```
Glob: "src/Menu/*.php"
Edit {menu_listener_file}:
Update icons to Tabler format:
- 'images outline' → 'tabler:photo'
- 'adversal' → 'tabler:ad'  
- 'external alternate' → 'tabler:layout-grid'
```

### 5. Validate Menu Service Registration
```
Read config/services.xml
```
Verify menu listener is properly tagged and uses correct service IDs

### 6. Validate
```
Bash: vendor/bin/console debug:container --env=dev
```
Expected: Menu services load without errors

### 7. Fix Issues (if validation fails)
- Check for deprecated menu builder references
- Update service IDs to new format
- Replace removed menu builders with Twig Hooks

### 8. Commit Changes
```
Bash: git add .
Bash: git commit -m "Update menu configuration for Sylius 2.0"
```