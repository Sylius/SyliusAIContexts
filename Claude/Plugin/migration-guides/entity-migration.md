# Entity Migration

## Workflow Pattern
Each step follows: **Execute → Validate → Fix → Commit**

## Commands

### 1. Check Doctrine Mapping Type
```
Glob: "src/Entity/*.php"
```
Check first entity file for annotations (@ORM\Entity, @ORM\Table, etc.)
If no annotations found (using XML mapping), skip this entire step.

### 2. Replace Annotations with PHP 8 Attributes (only if annotations present)
For each entity file:
```
Read src/Entity/{entity}.php
Edit src/Entity/{entity}.php:
Replace annotations with attributes:

- @ORM\Entity → #[ORM\Entity]
- @ORM\Table(name="table") → #[ORM\Table(name: "table")]
- @ORM\Id → #[ORM\Id]  
- @ORM\GeneratedValue → #[ORM\GeneratedValue]
- @ORM\Column(type="string") → #[ORM\Column(type: "string")]
- @ORM\ManyToOne → #[ORM\ManyToOne]
- @ORM\ManyToMany → #[ORM\ManyToMany]
```

### 3. Validate
```
Bash: vendor/bin/console doctrine:schema:validate
```
Expected: "[OK] The mapping files are correct"

### 4. Fix Issues (if validation fails)
- Check attribute syntax
- Verify all annotations were properly replaced
- Clear doctrine cache: `vendor/bin/console doctrine:cache:clear-metadata`

### 5. Commit Changes
```
Bash: git add .
Bash: git commit -m "Replace annotations with PHP 8 attributes in entities"
```