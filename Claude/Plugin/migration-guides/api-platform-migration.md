# API Platform Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Find API Platform Configuration Files
```
Glob: "config/**/*api*.xml" "config/**/*api*.yaml"
```
If no files found, skip this step.

### 2. Update XML Schema Namespace (for XML files)
For each XML API resource file:
```
Read {api_file}
Edit {api_file}:
Replace:
- xmlns="https://api-platform.com/schema/metadata"
→ xmlns="https://api-platform.com/schema/metadata/resources-3.0"
- metadata-2.0.xsd → resources-3.0.xsd
```

### 3. Convert Operations Structure
For each API resource file:
```
Edit {api_file}:
Replace (XML):
- <collectionOperations> → <operations>
- <collectionOperation name="..."> → <operation name="..." class="ApiPlatform\Metadata\GetCollection">
- <itemOperations> → <operations>
- <itemOperation name="..."> → <operation name="..." class="ApiPlatform\Metadata\Get">
- <attribute name="method">GET</attribute> → (remove - implicit in class)
- <attribute name="path">...</attribute> → uriTemplate="..."

Replace (YAML):
- collectionOperations: → operations:
- itemOperations: → operations:
```

### 4. Update State Providers References (if used)
```
Edit {api_file}:
Update provider service names if they changed during services migration
```

### 5. Validate
```
Bash: vendor/bin/console api:openapi:export
```
Expected: OpenAPI schema generated without errors

### 6. Fix Issues (if validation fails)
- Check syntax (XML/YAML)
- Verify operation class names
- Update service provider references
- Clear API Platform cache: `vendor/bin/console cache:clear`

### 7. Commit Changes
```
Bash: git add .
Bash: git commit -m "Migrate API Platform configuration to 4.x format"
```