# Step 7: Asset Migration

**CRITICAL**: Asset structure must be properly migrated to work with TestApplication.

## Asset Structure Requirements

Your plugin must have assets in the root `assets/` directory with the following structure:

```
assets/
├── admin/
│   └── entrypoint.js
└── shop/
    └── entrypoint.js
```

Both `entrypoint.js` files serve as the main entry points for webpack compilation.

## Migration Scenarios

### Case 1: Assets Already in Correct Structure

If your plugin already has an `assets/` folder in the root directory with `admin/entrypoint.js` and `shop/entrypoint.js`, you can skip to the verification step.

### Case 2: Assets in Old Structure (src/Resources/)

If your plugin has assets in `src/Resources/`, they must be moved to the new structure.

#### Move from old structure:
```
src/Resources/
├── private/
│   ├── js/
│   └── scss/
└── public/
    ├── js/
    └── css/
```

#### Move to new structure:
```
assets/
├── admin/
│   ├── entrypoint.js  # Main entry point
│   ├── js/
│   └── scss/
└── shop/
    ├── entrypoint.js  # Main entry point
    ├── js/
    └── scss/
```

### Migration Steps for Old Structure

1. **Create new asset directories:**
```bash
mkdir -p assets/admin assets/shop
```

2. **Move assets from old structure:**
```bash
# Move admin assets
cp -r src/Resources/assets/admin/js assets/admin/ 2>/dev/null || true
cp -r src/Resources/assets/admin/scss assets/admin/ 2>/dev/null || true

# Move shop assets  
cp -r src/Resources/assets/shop/js assets/shop/ 2>/dev/null || true
cp -r src/Resources/assets/shop/scss assets/shop/ 2>/dev/null || true
```

3. **Create entrypoint.js files:**

Create `assets/admin/entrypoint.js`:
```javascript
// Import your admin assets
import './js';
import './scss/main.scss';
```

Create `assets/shop/entrypoint.js`:
```javascript
// Import your shop assets
import './js';
import './scss/main.scss';
```

4. **Update existing entrypoints if they import from old structure:**

If your current entrypoints contain imports like:
```javascript
import '../../src/Resources/assets/admin/entry.js';
```

Replace them with local imports:
```javascript
import './js';
import './scss/main.scss';
```

5. **Remove old asset directories:**
```bash
rm -rf src/Resources/assets
```

## Asset Compilation Issues

### Common Issue: @vendor Alias Problems

If `@vendor` alias doesn't work, use paths relative to the `vendor` directory instead.

**Important**: Don't modify vendor files - use relative paths in your asset imports.

### Example Fixes

If you have imports like:
```javascript
import '@vendor/some-package/asset.css';
```

Replace with:
```javascript
import '../../../vendor/some-package/asset.css';
```

## Verification Steps

After migration, verify your asset structure:

1. **Check directory structure:**
```bash
ls -la assets/
ls -la assets/admin/
ls -la assets/shop/
```

2. **Verify entrypoint files exist:**
```bash
cat assets/admin/entrypoint.js
cat assets/shop/entrypoint.js
```

3. **Test asset compilation:**
```bash
yarn --cwd vendor/sylius/test-application install
yarn --cwd vendor/sylius/test-application build
```

## Completion Checklist

Verify your asset migration:
- [ ] `assets/admin/entrypoint.js` exists and imports correctly
- [ ] `assets/shop/entrypoint.js` exists and imports correctly
- [ ] Assets compile without errors
- [ ] No references to old `src/Resources/assets/` structure
- [ ] Old asset directories removed
- [ ] Relative paths used instead of problematic aliases

## Commit Changes

After completing the asset migration:

```bash
git add assets/
git add -u  # This will stage deletions of old asset files
git commit -m "feat: migrate assets to testApplication structure

- Move assets from src/Resources/assets/ to assets/
- Create proper entrypoint.js files for admin and shop
- Update import paths to use local references
- Remove old asset directory structure"
```

## Next Step

Continue to [testing-validation.md](testing-validation.md) to test the complete migration and clean up.