# Entity Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix**

## When to Execute
This step is OPTIONAL. Execute only if entities use Doctrine annotations (`@ORM\*`) in PHP files.

## Commands

### 1. Check for Entities with Annotations
```
Glob: "src/Entity/*.php"
```
If no files found, try:
```
Glob: "src/Model/*.php"
```

If no entity files found, skip entire step.

If files found, read first entity file:
```
Read {first_entity_file}
```

Check file content for annotations:
- If contains `@ORM\` → continue with migration
- If contains `#[ORM\` → skip (already using attributes)
- If contains neither → skip (using XML/YAML mapping)

### 2. Migrate Annotations to Attributes
For each entity file found:
```
Read {entity_file}
```

Replace annotations with attributes using Edit:
```
Edit {entity_file}:

Class-level annotations:
- /**\n * @ORM\Entity\n */ → #[ORM\Entity]
- /**\n * @ORM\Table(name="table_name")\n */ → #[ORM\Table(name: 'table_name')]

Property-level annotations:
- /**\n * @ORM\Id\n */ → #[ORM\Id]
- /**\n * @ORM\GeneratedValue\n */ → #[ORM\GeneratedValue]
- /**\n * @ORM\Column(type="string") → #[ORM\Column(type: 'string')]
- /**\n * @ORM\Column(type="string", length=255) → #[ORM\Column(type: 'string', length: 255)]
- /**\n * @ORM\Column(type="integer", nullable=true) → #[ORM\Column(type: 'integer', nullable: true)]

Relationship annotations:
- @ORM\ManyToOne(targetEntity="Category") → #[ORM\ManyToOne(targetEntity: Category::class)]
- @ORM\OneToMany(targetEntity="Item", mappedBy="parent") → #[ORM\OneToMany(targetEntity: Item::class, mappedBy: 'parent')]
- @ORM\ManyToMany(targetEntity="Tag", inversedBy="items") → #[ORM\ManyToMany(targetEntity: Tag::class, inversedBy: 'items')]
- @ORM\JoinColumn(nullable=false) → #[ORM\JoinColumn(nullable: false)]
- cascade={"persist", "remove"} → cascade: ['persist', 'remove']
```

**Important replacement rules:**
1. Remove `/**`, `*`, and `*/` comment syntax
2. Replace `@ORM\` with `#[ORM\`
3. Add `]` at the end
4. Use named parameters: `name="value"` → `name: 'value'`
5. Replace class strings with `::class`: `"Category"` → `Category::class`
6. Replace curly braces with square brackets: `{...}` → `[...]`
7. Use single quotes for strings

### 3. Check and Update Use Statements
```
Read {entity_file}
```

Check if file has:
```php
use Doctrine\ORM\Mapping as ORM;
```

If missing, add it after other `use` statements.

### 4. Validate Doctrine Schema
```
Bash: vendor/bin/console doctrine:schema:validate 2>&1
```

Expected output contains:
- `[Mapping]  OK` or `The mapping files are correct`

If validation fails:
```
Bash: vendor/bin/console doctrine:cache:clear-metadata
Bash: vendor/bin/console cache:clear
```

Then validate again.

## Success Criteria
- All `@ORM\*` annotations replaced with `#[ORM\*]` attributes in all entity files
- All entity files have `use Doctrine\ORM\Mapping as ORM`
- `doctrine:schema:validate` passes without mapping errors

## Notes for AI
- This step is OPTIONAL - skip if using XML/YAML mapping or already using attributes
- Entities can be in `src/Entity/` OR `src/Model/` - check both
- When replacing annotations, preserve all parameters and their values
- Named parameters are REQUIRED in attributes (use `:` after parameter name)
- Class references must use `::class` notation
- Arrays use square brackets `[]` not curly braces `{}`
- Remove ALL docblock comment syntax (`/**`, `*`, `*/`) around annotations
- Keep other docblocks (like `@var`, `@param`) untouched
- Validation errors about mapping usually mean syntax error in attribute conversion
