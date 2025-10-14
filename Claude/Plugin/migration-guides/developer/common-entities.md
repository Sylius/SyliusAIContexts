# Common: Entity Migration (Optional)

Sylius 2.0 (and Symfony 7+) fully embraces PHP 8 attributes for Doctrine ORM mapping. If your plugin still uses Doctrine annotations (e.g., `@ORM\Entity`, `@ORM\Column`), you should migrate them to attributes.

**When to skip this step:**
- Your entities are mapped with XML or YAML files
- Your entities already use PHP 8 attributes (`#[ORM\*]`)

**When to do this step:**
- Your entities use Doctrine annotations (`@ORM\*`) in PHP files

## 1. Replace Annotations with Attributes

Replace each Doctrine annotation with the corresponding PHP 8 attribute:

### Basic Entity Annotations

```diff
- /**
-  * @ORM\Entity
-  * @ORM\Table(name="app_product")
-  */
+ #[ORM\Entity]
+ #[ORM\Table(name: 'app_product')]
  class Product
```

### Column Annotations

```diff
- /**
-  * @ORM\Id
-  * @ORM\GeneratedValue
-  * @ORM\Column(type="integer")
-  */
+ #[ORM\Id]
+ #[ORM\GeneratedValue]
+ #[ORM\Column(type: 'integer')]
  private ?int $id = null;
```

```diff
- /**
-  * @ORM\Column(type="string", length=255, nullable=true)
-  */
+ #[ORM\Column(type: 'string', length: 255, nullable: true)]
  private ?string $name = null;
```

### Relationship Annotations

```diff
- /**
-  * @ORM\ManyToOne(targetEntity="Category", inversedBy="products")
-  * @ORM\JoinColumn(nullable=false)
-  */
+ #[ORM\ManyToOne(targetEntity: Category::class, inversedBy: 'products')]
+ #[ORM\JoinColumn(nullable: false)]
  private ?Category $category = null;
```

```diff
- /**
-  * @ORM\OneToMany(targetEntity="Review", mappedBy="product", cascade={"persist", "remove"})
-  */
+ #[ORM\OneToMany(targetEntity: Review::class, mappedBy: 'product', cascade: ['persist', 'remove'])]
  private Collection $reviews;
```

```diff
- /**
-  * @ORM\ManyToMany(targetEntity="Tag", inversedBy="products")
-  * @ORM\JoinTable(name="product_tags")
-  */
+ #[ORM\ManyToMany(targetEntity: Tag::class, inversedBy: 'products')]
+ #[ORM\JoinTable(name: 'product_tags')]
  private Collection $tags;
```

### Important Changes

1. **Named arguments:** Use `name:` instead of just `name` for parameters
2. **Class references:** Use `::class` instead of string (e.g., `Category::class` not `"Category"`)
3. **Arrays:** Use square brackets `[]` instead of curly braces `{}`
4. **Quotes:** Single quotes for string values

## 2. Update Use Statements

Make sure you import the ORM attributes namespace at the top of your entity files:

```php
use Doctrine\ORM\Mapping as ORM;
```

This allows you to use `#[ORM\Entity]` instead of `#[Doctrine\ORM\Mapping\Entity]`.

## 3. Validate Schema

After migrating all entities, validate that Doctrine can still read your mappings:

```bash
vendor/bin/console doctrine:schema:validate
```

Expected output:
```
[Mapping]  OK - The mapping files are correct.
[Database] OK - The database schema is in sync with the mapping files.
```

## Troubleshooting

If you encounter issues:

```bash
vendor/bin/console doctrine:cache:clear-metadata
vendor/bin/console cache:clear
```
