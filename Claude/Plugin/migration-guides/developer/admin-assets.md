# Admin: Assets Migration

## When to Execute

This step is OPTIONAL. Execute only if your plugin has custom JavaScript or CSS in admin panel.

## Overview

Sylius 2.0 requires all assets to be in the `assets/` directory:

**Target structure:**
- Admin: `assets/admin/`
- Shop: `assets/shop/`
- Compiled by Webpack Encore
- Imported in entrypoint files

**Your assets might currently be in:**
- `src/Resources/public/` (old Sylius 1.x location)
- `assets/` (already in correct location, but may need updates)
- Inline in templates (should be extracted)

**Goal:** Move everything to `assets/admin/` or `assets/shop/` and use modern approaches (Live Components, Stimulus, or imports)

---

## 🎯 Decision Guide

```
Can you rewrite JavaScript to modern approach?
│
├─ YES, I know Live Components
│  └─ ✅ Rewrite to Live Component (RECOMMENDED)
│     Works with forms or standalone
│     See: Option 1 below
│
├─ YES, I know Stimulus
│  └─ ✅ Rewrite to Stimulus Controller (RECOMMENDED)
│     For interactive UI components
│     See: Option 2 below
│
└─ NO, just migrate existing JS
   └─ Move to assets/admin/ + Import in entrypoint
      See: Option 3 below

All assets MUST be in assets/admin/ or assets/shop/
No files should remain in src/Resources/public/
```

---

## ✅ Option 1: Rewrite to Live Component (RECOMMENDED if you know how)

**Use when:** You know Live Components and can rewrite JavaScript logic to PHP

**Example:** Auto-generate slug from name
```php
#[LiveAction]
public function generateSlug(#[LiveArg] string $localeCode): void
{
    $this->formValues['translations'][$localeCode]['slug'] =
        $this->slugGenerator->generate(
            $this->formValues['translations'][$localeCode]['name']
        );
}
```

**Benefits:**
- No JavaScript needed
- Server-side logic
- Type safety
- Testable

**Learn more:**
- [Admin Templates - Custom Live Component](admin-templates.md#-advanced-custom-live-component-actions)

---

## ✅ Option 2: Rewrite to Stimulus Controller (RECOMMENDED if you know how)

**Use when:** You know Stimulus and can rewrite JavaScript to Stimulus controller

Good for: Modals, dropdowns, tabs, tooltips, interactive UI

**Note:** Sylius already includes Stimulus - no additional installation needed.

### 1. Create Stimulus controller

`assets/admin/controllers/modal_controller.js`:
```javascript
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['content'];

    connect() {
        console.log('Modal controller connected');
    }

    open() {
        this.contentTarget.classList.add('show');
    }

    close() {
        this.contentTarget.classList.remove('show');
    }
}
```

### 2. Register controller

`assets/admin/controllers.json`:
```json
{
    "controllers": {
        "@your-plugin/modal": {
            "enabled": true,
            "fetch": "eager"
        }
    },
    "entrypoints": []
}
```

### 3. Use in templates

```twig
<div data-controller="modal">
    <button data-action="click->modal#open">Open Modal</button>

    <div data-modal-target="content" class="modal">
        <!-- modal content -->
        <button data-action="click->modal#close">Close</button>
    </div>
</div>
```

**Learn more:** [Sylius - Customizing Dynamic Elements](https://docs.sylius.com/the-customization-guide/customizing-dynamic-elements)

---

## Option 3: Move to assets/admin/ + Import in Entrypoint

**Use when:** You want to migrate existing JavaScript without rewriting

### 1. Move JavaScript file to assets/admin/

```bash
# From old location:
src/Resources/public/custom-script.js
# Or if already in assets but wrong structure:
assets/custom-script.js

# To correct location:
assets/admin/custom-script.js
```

### 2. Import in entrypoint

`assets/admin/entrypoint.js`:
```javascript
import './custom-script.js';

// Or if it exports functions:
import { myFunction } from './custom-script.js';

document.addEventListener('DOMContentLoaded', () => {
    myFunction();
});
```

### 3. Rebuild assets

```bash
yarn --cwd vendor/sylius/test-application encore dev
vendor/bin/console assets:install
```

---

## CSS/SCSS Migration

### 1. Move CSS to assets/admin/

```bash
# From old location:
src/Resources/public/styles.css
# Or if already in assets:
assets/styles.css

# To correct location (prefer SCSS):
assets/admin/styles.scss
```

### 2. Import in entrypoint

`assets/admin/entrypoint.js`:
```javascript
import './styles.scss';
```

### 3. Use Bootstrap 5 variables

```scss
// assets/admin/styles.scss
@import '~bootstrap/scss/functions';
@import '~bootstrap/scss/variables';

.my-component {
    background: $primary;
    padding: $spacer;
}
```

### 4. Rebuild assets

```bash
yarn --cwd vendor/sylius/test-application encore dev
vendor/bin/console assets:install
```

---

## Validation

After migration, verify:

```bash
# Clear cache
vendor/bin/console cache:clear

# Rebuild assets
yarn --cwd vendor/sylius/test-application encore dev

# Install assets
vendor/bin/console assets:install

# Check browser console for errors
# Test functionality in admin panel
```

