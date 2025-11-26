# Fix Container Compilation

This step ensures your Symfony service container compiles successfully after updating dependencies to Sylius 2.0.

**When to skip this step:**
- Container already compiles without errors after composer update

**When to do this step:**
- `cache:clear` command fails with errors
- Service not found errors appear
- Deprecated service warnings

---

## Philosophy

**Priority order:**

1. ✅ **Fix what you can** - If service has a replacement in UPGRADE-2.0.md → use the new service ID
2. ⏳ **Comment out what you can't** - Only comment out code that:
   - Has no replacement (removed feature)
   - Is too complex to fix now
   - Will be migrated in later steps (UI events, API)

**Don't comment out services that just need a new ID!** Check UPGRADE-2.0.md first.

---

## 1. Test Container Compilation

```bash
vendor/bin/console cache:clear
```

If this succeeds → Skip to next step.

If you see errors → Continue with this guide.

---

## 2. Fix Missing Sylius Services

If you see errors like:
```
Service "sylius.xyz" not found
Service "sylius.abc" has been removed
```

**Check the Sylius UPGRADE file:**

View online: [Sylius 2.0 UPGRADE Guide](https://github.com/Sylius/Sylius/blob/2.0/UPGRADE-2.0.md)

Or locally:
```bash
vendor/sylius/sylius/UPGRADE-2.0.md
```

Search for the service name in the file. There's a large table showing:
- Which services were removed
- What they were replaced with
- Migration instructions

**Common service changes:**
- Many `sylius.repository.*` services unchanged
- Some `sylius.factory.*` services renamed
- Grid and resource services restructured
- UI event system removed (replaced by Twig Hooks)

**Action:**

**If replacement exists:**
1. Find the new service ID in UPGRADE-2.0.md
2. Search for old service ID in your code: `grep -r "old.service.id" src/ config/`
3. Replace old service ID with new one in all files
4. Re-run `cache:clear` to verify

**If no replacement exists:**
1. Service was removed (e.g., UI events)
2. Go to Step 3 and comment it out with TODO
3. Will be migrated in later steps

---

## 3. Comment Out What Cannot Be Fixed Now

**Only comment out code that:**
- Has no replacement (removed Sylius features)
- Is too complex to fix at this stage
- Will be properly migrated in later steps

**Don't comment out services that just need a new ID** - fix them in Step 2!

### What to Comment Out:

**1. `sylius_ui` event configurations**

In Extension class (`src/DependencyInjection/*Extension.php`):

```php
public function prepend(ContainerBuilder $container): void
{
    // TODO: Migrate to Twig Hooks in Step (Templates)
    /*
    $container->prependExtensionConfig('sylius_ui', [
        'events' => [
            'sylius.admin.layout.javascripts' => [
                'blocks' => [
                    'your_plugin_key' => [
                        'template' => '@YourPlugin/admin/_javascripts.html.twig',
                    ],
                ],
            ],
        ],
    ]);
    */
}
```

In config files (`config/packages/sylius_ui.yaml`):

```yaml
# TODO: Migrate to Twig Hooks in Step(Templates)
# sylius_ui:
#     events:
#         ...
```

**2. API service definitions** _(will be migrated in Step 9)_

If you see errors related to API Platform services, comment them out:

```xml
<!-- TODO: Migrate to API Platform 4.x in Step API -->
<!--
<service id="app.api.data_provider">
    ...
</service>
-->
```

**3. Any other broken code**

If something else prevents container compilation, comment it out:

```php
// TODO: Fix in appropriate step - investigate which step handles this
// ... broken code ...
```

**Important:** Always add `TODO` comments explaining:
- What needs to be fixed
- In which migration step it should be fixed
- Why it was commented out (optional)

---

## 4. Validate Container Compilation

```bash
vendor/bin/console cache:clear
```

Expected result: Command succeeds without errors.

If you still see errors:
1. Read the error message carefully
2. Check UPGRADE-2.0.md for the specific service/class
3. Comment out the problematic code (add TODO comment)
4. Try `cache:clear` again
5. Repeat until container compiles

**Goal:** Get the container to compile. You can fix specific features in later steps.
