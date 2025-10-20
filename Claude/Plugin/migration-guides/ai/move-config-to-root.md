# Configuration Restructuring

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
This step is OPTIONAL. Execute only if `src/Resources/` directory exists.

## Commands

### 1. Check if Migration is Needed
```
Bash: ls src/Resources/ 2>/dev/null || echo "SKIP"
```
If output is "SKIP", skip entire step (jump to next migration step).

### 2. Move Configuration Files
```
Bash: find src/Resources/config -type f 2>/dev/null
```
If files found:
```
Bash: mkdir -p config
Bash: cp -r src/Resources/config/* config/
```

Alternative (preserving git history):
```
Bash: git mv src/Resources/config/* config/
```

### 3. Move Translation Files
```
Bash: find src/Resources/translations -type f 2>/dev/null
```
If files found:
```
Bash: mkdir -p translations
Bash: cp -r src/Resources/translations/* translations/
```

Alternative (preserving git history):
```
Bash: git mv src/Resources/translations/* translations/
```

### 4. Move Template Files
```
Bash: find src/Resources/views -type f 2>/dev/null
```
If files found:
```
Bash: mkdir -p templates
Bash: cp -r src/Resources/views/* templates/
```

Alternative (preserving git history):
```
Bash: git mv src/Resources/views/* templates/
```

**Note**: Do NOT rename folders or files at this stage. Template naming conventions will be handled in Template Migration step.

### 5. Update Plugin Class - Add getPath()
```
Glob: "src/*Plugin.php"
Read {plugin_file}
```
Check if `getPath()` method already exists.
If not, add it:
```
Edit {plugin_file}:
Find the last method in the class (before closing brace)
Add after the last method:

    public function getPath(): string
    {
        return \dirname(__DIR__);
    }
```

### 6. Update Extension Class - FileLocator Path
```
Glob: "src/DependencyInjection/*Extension.php"
Read {extension_file}
```
Look for FileLocator usage:
```
Edit {extension_file}:
Replace:
- new FileLocator(__DIR__ . '/../Resources/config')
+ new FileLocator(__DIR__ . '/../../config')
```

### 7. Update Resource Paths in Config Files
```
Grep: "Resources/config" --include="*.yaml" --include="*.yml" path="config/"
Grep: "Resources/config" --include="*.yaml" --include="*.yml" path="tests/TestApplication/config/"
```
For each file with matches:
```
Read {file}
Edit {file}:
Replace all occurrences:
- @{PluginName}/Resources/config/ → @{PluginName}/config/
```

Common files to check:
- `config/routes.yaml`
- `config/routes_no_locale.yaml`
- `tests/TestApplication/config/config.yaml`
- `tests/TestApplication/config/routes.yaml`

**Update .env and .env.test in test application:**

Update both `.env` and `.env.test` files:

```bash
Tool: Read
File: .env
```

Check if contains `SYLIUS_TEST_APP_CONFIGS_TO_IMPORT` with relative paths.

**If found, update .env:**

```bash
Tool: Edit
File: .env
Old: SYLIUS_TEST_APP_CONFIGS_TO_IMPORT="../../../../tests/TestApplication/config/config.yaml"
New: SYLIUS_TEST_APP_CONFIGS_TO_IMPORT="@{PluginName}/tests/TestApplication/config/config.yaml"
```

```bash
Tool: Edit
File: .env
Old: SYLIUS_TEST_APP_ROUTES_TO_IMPORT="../../../../tests/TestApplication/config/routes.yaml"
New: SYLIUS_TEST_APP_ROUTES_TO_IMPORT="@{PluginName}/tests/TestApplication/config/routes.yaml"
```

**Also update .env.test:**

```bash
Tool: Read
File: .env.test
```

```bash
Tool: Edit
File: .env.test
Old: SYLIUS_TEST_APP_CONFIGS_TO_IMPORT="../../../../tests/TestApplication/config/config.yaml"
New: SYLIUS_TEST_APP_CONFIGS_TO_IMPORT="@{PluginName}/tests/TestApplication/config/config.yaml"
```

```bash
Tool: Edit
File: .env.test
Old: SYLIUS_TEST_APP_ROUTES_TO_IMPORT="../../../../tests/TestApplication/config/routes.yaml"
New: SYLIUS_TEST_APP_ROUTES_TO_IMPORT="@{PluginName}/tests/TestApplication/config/routes.yaml"
```

Replace `{PluginName}` with actual bundle name (e.g., `AcmeMyAwesomePlugin`).

### 8. Check if AbstractResourceBundle is Used (Troubleshooting)
```
Glob: "src/*Plugin.php"
Read {plugin_file}
```
Check if class extends `AbstractResourceBundle`.

If YES, check for Doctrine mapping error:
```
Bash: vendor/bin/console cache:clear 2>&1 | grep "invalid directory path"
```

If error found, add getConfigFilesPath() method:
```
Edit {plugin_file}:
Add method after getPath():

    protected function getConfigFilesPath(): string
    {
        return sprintf(
            '%s/config/doctrine/%s',
            $this->getPath(),
            strtolower($this->getDoctrineMappingDirectory()),
        );
    }
```

### 9. Clear Cache and Validate
```
Bash: vendor/bin/console cache:clear 2>&1 | tail -10
```
Expected: Cache clears successfully (may show warnings - OK)

If Doctrine mapping error appears, go back to step 8 and add getConfigFilesPath().

### 10. Clean Up Empty Directories
```
Bash: rmdir src/Resources/config src/Resources/translations src/Resources/views 2>/dev/null || true
```
This removes empty directories. The `public/` folder should remain (will be moved in Asset Migration step).

## Success Criteria
- Files moved from `src/Resources/` to root directories
- Plugin class has `getPath()` method
- Extension class uses correct FileLocator path
- All `@Plugin/Resources/config/` paths updated to `@Plugin/config/`
- Cache clears without fatal errors
- If using `AbstractResourceBundle` and entities in `src/Model/`: `getConfigFilesPath()` added

## Notes for AI
- This step is OPTIONAL - check for `src/Resources/` first
- Use `git mv` if available to preserve git history
- Do NOT rename template folders/files in this step (done later)
- `src/Resources/public/` should NOT be moved (will be moved in Asset Migration step)
- Only add `getConfigFilesPath()` if plugin extends `AbstractResourceBundle` AND there's a Doctrine error
- Cache clear may timeout or show warnings - this is acceptable
