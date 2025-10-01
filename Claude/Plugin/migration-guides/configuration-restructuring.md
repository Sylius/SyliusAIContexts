# Configuration Restructuring

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Check Current Structure
```
Glob: "src/Resources/config/**/*"
```
If no results found, configuration is already in root - skip to step 5 (Update Plugin Class)

### 2. Create New Directories (if needed)
```
Bash: mkdir -p config/grids/admin config/grids/shop config/routing config/services config/validation config/serialization config/doctrine config/api_resources/resources/admin config/api_resources/resources/shop config/api_resources/properties config/twig_hooks/admin config/twig_hooks/shop translations templates/admin templates/shop
```

### 3. Move Configuration Files (if src/Resources exists)
```
Glob: "src/Resources/config/**/*"
```
For each file found:
```
Read src/Resources/config/{filename}
Write config/{filename}
```

### 4. Move Translation Files (if src/Resources exists)
```
Glob: "src/Resources/translations/**/*"
```
For each file found:
```
Read src/Resources/translations/{filename}
Write translations/{filename}
```

### 5. Move Templates (if src/Resources exists)
```
Glob: "src/Resources/views/**/*"
```
For each file found:
```
Read src/Resources/views/{filepath}
Write templates/{filepath}
```
**Note**: Change Admin → admin, Shop → shop (lowercase)

### 6. Update Plugin Class
```
Glob: "src/*Plugin.php"
Read {plugin_file}
Edit {plugin_file}:
Add method:
public function getPath(): string
{
    return \dirname(__DIR__);
}
```

### 7. Update DependencyInjection
```
Glob: "src/DependencyInjection/*Extension.php"
Edit {extension_file}:
- new FileLocator(__DIR__ . '/../Resources/config') 
→ new FileLocator(__DIR__ . '/../../config')
```

### 8. Update Config Imports (if files were moved)
```
Edit config/config.yaml:
Replace imports section with:
imports:
    - { resource: "services.xml" }
    - { resource: "resources.yaml" }
    - { resource: "routing.yaml" }
    - { resource: "grids/*.yaml" }
    - { resource: "validation/*.xml" }
    - { resource: "serialization/*.xml" }
    - { resource: "twig_hooks/**/*.yaml" }
```

### 9. Validate
```
Bash: vendor/bin/console debug:container --env=dev
```
Expected: Container compiles successfully

### 10. Fix Issues (if validation fails)
- Check file paths in config imports
- Verify all moved files are in correct locations
- Clear cache: `rm -rf var/cache/*`

### 11. Commit Changes
```
Bash: git add .
Bash: git commit -m "Restructure configuration to Sylius 2.0 format"
```
