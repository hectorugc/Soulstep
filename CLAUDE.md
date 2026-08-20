# Claude Project Context — SoulStep Shopify Store

## Store Info

- Store: `soulstep-slc4vard.myshopify.com`
- Theme: Dawn-based, dev theme ID `161723154656`
- Push command: `shopify theme push --store soulstep-slc4vard.myshopify.com --theme 161723154656 --only <file>`

---

## Products

3 insole products, each with 19 shoe size variants (combined Men's/Women's sizing):

| Size label | Example |
|---|---|
| "3.5 Mens \| 5 Womens" | smallest |
| … | … |
| "14 Mens \| 15.5 Womens" | largest |

- Option name: **Shoe Size (USA)**
- Price: **$21.99** per pair
- Inventory: **100** per variant (set via `inventoryAdjustQuantities`)

Product IDs (from Admin API):
- Run `products` query filtered by title to get current IDs if needed

---

## Discount

**Buy 3 Get 1 Free** automatic BXGY discount (created via `discountAutomaticBxgyCreate`):
- Applies site-wide (all products)
- Customer buys 3, gets 1 free (lowest price item)
- No usage limit per order (note: `usesPerOrderLimit` must be a string if used)

---

## Cart Drawer — Free Shipping Progress Bar

File: `snippets/cart-drawer.liquid`

Added a progress bar above cart items showing progress toward **$40 free shipping** (4000 cents).

### How it works

Pure Liquid — no JavaScript (inline scripts don't execute in AJAX-refreshed `innerHTML`):

```liquid
{%- assign free_shipping_threshold = 4000 -%}
{%- assign remaining_cents = free_shipping_threshold | minus: cart.total_price -%}
{%- assign progress_pct = cart.total_price | times: 100 | divided_by: free_shipping_threshold -%}
```

Fill bar width set via inline style: `width: {{ progress_pct }}%`

### Contextual messaging logic

- `is_bundle = true` if any item has `item.properties._bundle_id != blank`
- Cart not empty + shipping remaining + not bundle → show bundle nudge
- Cart not empty + shipping unlocked + not bundle → show "Free Shipping unlocked!" + bundle nudge
- Cart not empty + shipping unlocked + is bundle → show "Free Shipping + 4th pair free!"

### Known issue history

CSS classes in `<style>` tags, `@keyframes` animations, and `<script>` tags in the snippet all failed to reliably apply/execute in the AJAX-refreshed cart drawer context. Pure inline Liquid styles are the reliable approach.

---

## Bundle PDP Block

File: `snippets/bundle-pdp.liquid`  
Registered in: `templates/product.json` as block type `bundle_pdp`  
Section: `sections/main-product.liquid`

Renders 4 pair slots (style + size selects) with an order summary and "Add Bundle to Cart" button.

- Cart add uses `/cart/add.js` fetch with `_bundle_id` line item property (UUID generated client-side)
- `_bundle_id` groups bundle items in cart; the automatic BXGY discount handles the actual free item

Settings (in `templates/product.json`):
```json
{
  "price_per_pair": 2199,
  "heading": "🎉 Buy 3, get a 4th pair free — mix & match any style Footsouls",
  "show_build_note": false
}
```

---

## Klaviyo Reviews

Displayed via Klaviyo app block in the Theme Editor — no custom code needed.

---

## CLI Reference

```bash
# Auth
shopify store auth --store soulstep-slc4vard.myshopify.com --scopes read_products,write_products

# Execute GraphQL
shopify store execute --store soulstep-slc4vard.myshopify.com --query '...'

# Push specific file
shopify theme push --store soulstep-slc4vard.myshopify.com --theme 161723154656 --only snippets/cart-drawer.liquid

# Push all
shopify theme push --store soulstep-slc4vard.myshopify.com --theme 161723154656
```

---

## GraphQL Notes

- `inventoryAdjustQuantities` requires `changeFromQuantity` arg (set to current quantity, usually `0`)
- `usesPerOrderLimit` in discount mutations must be a **string**, not integer
- `appliesOnOneTimePurchase` is NOT supported on automatic BXGY discounts
- `@idempotent` directive goes on the **field**, not the mutation: `inventoryAdjustQuantities(...) @idempotent(key: "...")`
