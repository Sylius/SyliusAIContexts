# Shop: Assets Migration

## When to Execute

This step is OPTIONAL. Execute only if your plugin has custom JavaScript or CSS in shop.

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

**Goal:** Move everything to `assets/shop/` and use modern approaches (Stimulus or imports)

---

## 🎯 Decision Guide

```
Can you rewrite JavaScript to modern approach?
│
├─ YES, I know Stimulus
│  └─ ✅ Rewrite to Stimulus Controller (RECOMMENDED)
│     For interactive UI components
│     See: Option 1 below
│
└─ NO, just migrate existing JS
   └─ Move to assets/shop/ + Import in entrypoint
      See: Option 2 below

All assets MUST be in assets/shop/
No files should remain in src/Resources/public/
```

---

## ✅ Option 1: Rewrite to Stimulus Controller (RECOMMENDED if you know how)

**Use when:** You know Stimulus and can rewrite JavaScript to Stimulus controller

Good for: Product galleries, image zoom, add to cart, interactive UI

**Note:** Sylius already includes Stimulus - no additional installation needed.

### 1. Create Stimulus controller

`assets/shop/controllers/product_gallery_controller.js`:
```javascript
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['mainImage', 'thumbnail'];

    connect() {
        console.log('Product gallery controller connected');
    }

    selectImage(event) {
        const thumbnail = event.currentTarget;
        this.mainImageTarget.src = thumbnail.dataset.largeUrl;
    }
}
```

### 2. Register controller

`assets/shop/controllers.json`:
```json
{
    "controllers": {
        "@your-plugin/product-gallery": {
            "enabled": true,
            "fetch": "eager"
        }
    },
    "entrypoints": []
}
```

### 3. Use in templates

```twig
<div data-controller="product-gallery">
    <img data-product-gallery-target="mainImage" src="{{ product.mainImage }}" />

    <div class="thumbnails">
        {% for image in product.images %}
            <img
                data-product-gallery-target="thumbnail"
                data-large-url="{{ image.large }}"
                data-action="click->product-gallery#selectImage"
                src="{{ image.thumbnail }}"
            />
        {% endfor %}
    </div>
</div>
```

**Learn more:** [Sylius - Customizing Dynamic Elements](https://docs.sylius.com/the-customization-guide/customizing-dynamic-elements)

---

## Option 2: Move to assets/shop/ + Import in Entrypoint

**Use when:** You want to migrate existing JavaScript without rewriting

### 1. Move JavaScript file to assets/shop/

```bash
# From old location:
src/Resources/public/custom-script.js
# Or if already in assets but wrong structure:
assets/custom-script.js

# To correct location:
assets/shop/custom-script.js
```

### 2. Import in entrypoint

`assets/shop/entrypoint.js`:
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

### 1. Move CSS to assets/shop/

```bash
# From old location:
src/Resources/public/styles.css
# Or if already in assets:
assets/styles.css

# To correct location (prefer SCSS):
assets/shop/styles.scss
```

### 2. Import in entrypoint

`assets/shop/entrypoint.js`:
```javascript
import './styles.scss';
```

### 3. Use Bootstrap 5 variables

```scss
// assets/shop/styles.scss
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
# Test functionality in shop
```
