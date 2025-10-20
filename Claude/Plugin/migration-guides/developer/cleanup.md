# Cleanup

This step helps review and clean up migration artifacts, commented code, and TODO markers left during the migration process.

**When to do this step:**
- After completing all migration steps
- Before final testing and validation
- Before creating pull requests

---

## 1. Find and Review TODO Markers

Search for TODO comments left during migration:

```bash
# Search for TODO in PHP files
grep -rn "TODO" src/ --include="*.php"

# Search for TODO in config files
grep -rn "TODO" config/ --include="*.yaml" --include="*.xml"
```

For each TODO found:
- Check if the migration step was completed
- If completed → remove the TODO
- If not completed → decide if it should be done now or tracked in an issue

---

## 2. Find and Review Commented Code

Search for large commented blocks that might be migration leftovers:

```bash
# Search for commented sylius_ui configuration
grep -rn "sylius_ui" src/ --include="*.php" -A 5 -B 2

# Search for commented blocks with migration notes
grep -rn "Migrate.*Twig Hooks" src/ --include="*.php" -A 3
```

Common patterns to look for:
- Commented `sylius_ui` configuration in Extension.php
- Commented event subscribers
- Commented old template code
- Commented routing configuration

For each finding:
1. Read the full context
2. Determine if it's obsolete:
   - Was this migrated to Twig Hooks? → Remove
   - Was this replaced by Live Component? → Remove
   - Was this moved to assets/? → Remove
   - Is this still needed for reference? → Keep with explanation
3. Remove if obsolete

**Example - Removing obsolete sylius_ui configuration:**

In `src/DependencyInjection/Extension.php`:

```php
// TODO: Migrate to Twig Hooks in Step 7.4 (Templates)
/*
$container->prependExtensionConfig('sylius_ui', [
    'events' => [
        'plugin.admin.resource.create.javascripts' => [...],
    ],
]);
*/
```

If Twig Hooks migration is complete and JavaScript was migrated → remove this entire block.

---

## 3. Check for Orphaned Template Files

Look for old template files that should have been removed:

```bash
# Find template partials (often replaced during migration)
find templates/ -name "_*.html.twig"
```

Common orphaned templates:
- `_javascripts.html.twig` (replaced by assets or Live Components)
- `_form.html.twig` (replaced by Twig Hooks)
- `_breadcrumb.html.twig` (integrated into new structure)

For each orphaned template:
1. Check if it's still referenced anywhere:
   ```bash
   grep -r "filename.html.twig" templates/ config/
   ```
2. If not used → remove:
   ```bash
   git rm templates/path/to/_filename.html.twig
   ```

---

## 4. Validate After Cleanup

After removing code, validate everything still works:

```bash
# Clear cache
vendor/bin/console cache:clear

# Check container compiles
vendor/bin/console debug:container --parameters

# Check routes
vendor/bin/console debug:router

# Check twig hooks
vendor/bin/console debug:twig-hooks
```

All commands should complete without errors.

---

## 5. Final Verification

Run a final check for remaining migration markers:

```bash
# Count remaining TODOs
grep -r "TODO\|FIXME\|XXX" src/ --include="*.php" | wc -l

# List files with commented code blocks
grep -rl "\/\*.*\*\/" src/ --include="*.php" | head -10
```

Review the results:
- Some TODOs are fine (unrelated to migration)
- Some comments are documentation (keep those)
- Migration-related items should be minimal or zero

---

## Checklist

After completing this step:
- ✅ All migration-related TODOs reviewed and resolved
- ✅ Obsolete commented code removed
- ✅ Orphaned template files removed
- ✅ Container compiles without errors
- ✅ Cache clears successfully
- ✅ Routes and hooks validated

---

## What NOT to Remove

**Keep these items:**
- TODOs unrelated to migration
- Comments explaining complex logic
- Commented code showing "why" something was done differently
- PHPDoc comments
- Debug comments that might be useful

**Remove these items:**
- TODOs about "Migrate to Twig Hooks" if migration is complete
- Commented `sylius_ui` blocks if replaced with Twig Hooks
- Old template includes if replaced by assets or Live Components
- Commented event subscribers if logic moved elsewhere
- Old routing configuration that was replaced

---

## Tips

- **Review in small batches** - Don't try to clean everything at once
- **Use version control** - Commit cleanup changes separately from functional changes
- **Test after each removal** - Make sure nothing breaks
- **When in doubt, keep it** - Better to have extra comments than break something
