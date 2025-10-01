# Sylius Template Path Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Find Sylius Template References
```
Grep: "@Sylius.*/" --include="*.yaml" --include="*.xml" --include="*.twig"
```
If no results found, skip this step.

### 2. Update Sylius Bundle Template Paths
For each file with Sylius template references:
```
Read {file}
Edit {file}:
Replace Sylius bundle template references:
- "@SyliusAdmin/Grid/..." → "@SyliusAdmin/shared/grid/..."
- "@SyliusAdmin/..." → "@SyliusAdmin/shared/..."
- "@SyliusShop/..." → "@SyliusShop/shared/..."
```

### 3. Validate
```
Bash: vendor/bin/console debug:container --env=dev
```
Expected: Grid configurations load correctly

### 4. Fix Issues (if validation fails)
- Check YAML syntax
- Verify template paths exist in Sylius bundles
- Clear cache: `vendor/bin/console cache:clear`

### 5. Commit Changes
```
Bash: git add .
Bash: git commit -m "Update Sylius template paths for Sylius 2.0"
```
