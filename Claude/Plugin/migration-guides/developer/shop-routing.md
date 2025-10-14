# Shop: Routing Migration

This step updates shop routing configuration to match Sylius 2.0 conventions.

**When to skip this step:**
- Your plugin doesn't have shop functionality
- You don't use Sylius resource routing for shop

**When to do this step:**
- You use Sylius CRUD routes with `section: shop`
- You reference `@SyliusShop` templates in routing

## 1. Update Shop Route Templates

In `config/routes/shop.yaml`, update the routing configuration:

**Before (Sylius 1.x):**
```yaml
plugin_shop_product:
    resource: |
        alias: plugin.product
        section: shop
        templates: "@SyliusShop\product"
        vars:
            all:
                subheader: plugin.ui.products
    type: sylius.resource
```

**After (Sylius 2.0):**
```yaml
plugin_shop_product:
    resource: |
        alias: plugin.product
        section: shop
        templates: "@SyliusShop\shared\product"
        vars:
            all:
                hook_prefix: 'plugin.shop.product'
    type: sylius.resource
```

**Key changes:**
- `templates: "@SyliusShop\[name]"` → `templates: "@SyliusShop\shared\[name]"`
- Remove `vars.all.subheader` - no longer used
- Add `vars.all.hook_prefix` - for Twig Hooks integration

## 2. Validate Routes

Check that routes load correctly:

```bash
vendor/bin/console debug:router
```

You should see your shop routes listed without errors.
