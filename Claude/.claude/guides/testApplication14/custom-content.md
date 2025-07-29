# Step 4: Copy Custom Content

**PREREQUISITE**: Ensure you have completed Step 2 (TestApplication configuration) before proceeding.

**CRITICAL**: Before removing the old `tests/Application` directory, analyze and copy any custom content that is not provided by the base TestApplication into the structure you've already created.

## What to Check and Copy

- **Custom entities**: Copy from `tests/Application/src/Entity/` to `tests/TestApplication/src/Entity/`
- **Template overrides**: Copy from `tests/Application/templates/bundles/` to `tests/TestApplication/templates/bundles/`
- **Custom configuration**: Review `tests/Application/config/` for fixtures, services, routing, and resource mappings
- **Translations**: Copy any custom translations from `tests/Application/translations/`

**Note**: Do NOT copy assets or webpack configuration - these are handled in the asset migration step.

## Step-by-Step Process

### 1. Check for Custom Entities
```bash
ls -la tests/Application/src/Entity/
```
If custom entities exist, copy them and ensure proper doctrine mapping.

### 2. Check for Template Overrides
```bash
ls -la tests/Application/templates/bundles/
```
Copy any template overrides:
```bash
# Only if templates exist
cp -r tests/Application/templates/bundles/* tests/TestApplication/templates/bundles/
```

### 3. Check Configuration Files
Review `tests/Application/config/` for:
- Custom fixtures in `sylius_fixtures` section
- Service configurations
- Resource mapping overrides
- Custom routing

**IMPORTANT**: Copy ALL custom configuration to `tests/TestApplication/config/config.yaml`. Do not create separate configuration files - consolidate everything into config.yaml which you created in Step 2.

### 4. Check for Translations
```bash
ls -la tests/Application/translations/
```
Copy any custom translations:
```bash
# Only if translations exist
cp -r tests/Application/translations/* tests/TestApplication/translations/
```

## Completion Checklist

Verify you have copied:
- [ ] Custom entities (if any)
- [ ] Template overrides (if any)
- [ ] Custom fixtures configuration → Added to config.yaml
- [ ] Custom service configurations → Added to config.yaml
- [ ] Custom translations (if any)
- [ ] Custom routing → Added to routes.yaml (if needed)
- [ ] All custom config consolidated in config.yaml
- [ ] ⚠️ Assets and webpack config NOT copied

## Commit Changes

After copying all custom content:

```bash
git add tests/TestApplication/
git commit -m "feat: migrate custom content to testApplication structure

- Copy custom entities from old Application
- Copy template overrides to new structure
- Migrate custom fixtures and service configuration
- Copy custom translations if present"
```

## Next Step

Continue to [test-configs.md](test-configs.md) to update test configurations.