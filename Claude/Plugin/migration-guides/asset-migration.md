# Asset Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Check for Plugin Assets
```
Glob: "assets/**/*"
```
If no assets directory found, skip this entire step.

### 2. Update Asset References (if assets exist)
```
Glob: "assets/**/*.js" "assets/**/*.scss"
```
For each asset file, check for any hardcoded references that may have changed:
- Sylius bundle asset paths
- Admin/Shop specific imports

### 3. Check Asset Build Configuration (if exists)
```
Read package.json
```
If plugin has its own package.json, verify dependencies are compatible with project's build system.

### 4. Validate
```
Bash: vendor/bin/console debug:container --env=dev
```
Expected: No asset-related service errors

### 5. Fix Issues (if validation fails)
- Update asset import paths if needed
- Remove deprecated asset references
- Check main application's asset build process includes plugin assets

### 6. Commit Changes
```
Bash: git add .
Bash: git commit -m "Update plugin assets for Sylius 2.0"
```