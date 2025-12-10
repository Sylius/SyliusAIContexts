# Sylius 1.14 → 2.0 Migration Estimation Guide

Guide for estimating migration effort for client projects.

---

## Recommended Migration Path

**Migrate directly from 1.14 to 2.1, not 1.14 → 2.0 → 2.1.**

Stimulus controllers handling differs between Sylius 2.0 and 2.1. Going through 2.0 first means doing this work twice. Direct migration to 2.1 is more efficient.

---

## General Questions

- **PHP version?** - Sylius 2.0 requires PHP 8.2+
- **Symfony version?** - Sylius 2.0 requires Symfony 6.4 or 7.x
- **How much custom code does the project have?** - More custom = longer migration
- **Are there tests?** - No tests = harder to verify migration correctness

---

## Plugins

### Questions to ask:

- **How many plugins are used?**
- **Do they have Sylius 2.0 support?** - Check plugin repositories/documentation
- **Are any plugins using custom forks?** - Forks likely contain customizations that need manual migration
- **Are any BitBag plugins that were migrated to Sylius?**
- **Are plugins heavily overridden in the project?** - Check for template overrides, service decorators, extended classes from plugins

### BitBag → Sylius Plugin Migrations

These plugins were moved from BitBag to official Sylius packages:
- AdyenPlugin
- MolliePlugin
- WishlistPlugin
- CmsPlugin

If project uses BitBag versions, migration to Sylius versions will take additional time.

---

## Admin Panel

### Resources (CRUD)

**Key insight:** Custom resources are easier to migrate than modified Sylius core resources.

Questions:
- **How many custom admin resources?** - These are straightforward to migrate
- **How many modified Sylius core resources?** - These take longer because you need to understand what was changed and why

### JavaScript in Admin

- **How much custom JS is used in admin panel?** - More JS = more work adapting to new admin (Bootstrap 5, Stimulus)
- **What JS libraries are used?** - jQuery plugins may need replacement

If there is a lot of JS, you can add jQuery and adapt existing code. However, keep in mind:
- All form field names have changed in Sylius 2.0
- Forms are now LiveComponents, so even with jQuery, migration won't always be straightforward copy-paste

### Templates

- **How many custom admin templates?**
- **Are they overriding Sylius core templates or completely custom pages?** - Overrides need more careful migration

---

## Shop (placeholder)

*To be filled based on shop migration experience*

- Templates
- Assets (JS/CSS)
- Theme customizations

---

## API (placeholder)

*To be filled based on API migration experience*

- API Platform 2.7 → 4.x migration
- Custom endpoints
- DataProviders → StateProviders
- DataPersisters → StateProcessors

---

## What Can Extend Migration Time

1. **Modified Sylius core resources** - Need to understand original changes
2. **BitBag → Sylius plugin migrations** - Not just version bump, requires code changes
3. **Custom plugin forks** - Manual migration of custom code
4. **Heavy JS customization in admin** - New admin uses different JS stack
5. **Many template overrides** - All need conversion to Twig Hooks system
6. **No tests** - Can't verify migration correctness easily

---

*Document based on real migration experience. Will be expanded as more areas are migrated.*
