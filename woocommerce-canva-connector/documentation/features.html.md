---
title: Features
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/features.html
---

# Features

**WooCommerce Canva Connector** turns product image creation and management into a seamless, one-click experience. Design, edit, and publish high-quality product visuals in Canva directly from your WooCommerce workflow.

<a class="doc-image-link" href="./assets/canva-app-current-product.webp"><img src="./assets/canva-app-current-product.webp" alt="Canva Connect key features in action" /></a>

Below is the complete feature list, capabilities, and architectural breakdown.

---

## Core Features & Highlights

- **Design WooCommerce Product Images in Canva:** Create, customize, and edit professional product visuals directly inside the Canva editor.
- **One-Click Launch from WordPress Admin:** Add a **Design with Canva** button to WooCommerce product edit pages and products list table row actions.
- **Dynamic Dimension Matching:** Automatically creates canvas dimensions matching your product featured image, theme image settings (`woocommerce_single`), or standard 1080×1080 canvas without image distortion.
- **Existing Image Pre-Upload:** Automatically loads the current product featured image into the Canva workspace as an editable asset.
- **In-Editor WooCommerce Product Search:** Search your store's published catalog by product name or SKU directly within the Canva app sidebar.
- **Variation & Grouped Product Awareness:** Select specific product variations or grouped children to tailor visuals specifically for variations.
- **Add Live Product Data as Text:** Insert real-time product names, formatted prices, SKUs, and descriptions onto the canvas with a single click.
- **Click-to-Replace Image Swapping:** Select any image frame on the canvas and click any catalog thumbnail to instantly replace it with high-resolution imagery.
- **One-Click Save Design to Store:** Export finished designs and sideload them directly into the WordPress Media Library without manual downloading and re-uploading.
- **Design Re-Opening & History:** Stores design IDs (`_canva_design_id`) so admins can reopen and continue editing previous artwork at any time.
- **Enterprise-Grade Security:** Full OAuth 2.0 PKCE handshake, silent background token refresh, and cryptographic RS256 JWT verification.
- **HPOS Compatible:** Built and verified for WooCommerce High-Performance Order Storage (HPOS).
- **Translation Ready:** Fully translatable (i18n) via the standard `wkwc-canva-connect` text domain.

---

## WordPress & WooCommerce Features

### One-Click Canva Integration

A dedicated **Design with Canva** meta box is added to the sidebar of every product edit screen (and standard post edit screens). The same action is also added into the products list table as a row action between *View* and *Duplicate*, allowing you to launch design workflows without opening the product first.

Every request is protected with nonce verification and requires the **`edit_products`** capability, preventing unauthorized access.

### Dynamic Dimension Matching

Nothing gets distorted or cropped. Before the canvas is created, the connector resolves the target dimensions in this exact order:

1. The **full-size dimensions of the product's current featured image**.
2. The **`woocommerce_single` image size** defined by your active WooCommerce theme.
3. **1080 × 1080 px** as the standard high-resolution fallback.

### Existing Image Pre-Upload

If the product already has a featured image, the connector uploads that file to Canva as an asset first (`/v1/asset-uploads`), polls the upload job until it succeeds, and places it directly onto the new canvas.

### Design Re-Opening

The created design ID is stored in the product's `_canva_design_id` post meta. Whenever you click *Design with Canva* again, the connector looks up the design and reopens it instead of creating a blank canvas, ensuring work is never lost.

### Automated Image Sideloading

When saving a design from Canva, exported images are sent to your store, processed through WordPress's `media_handle_sideload()`, and attached directly as the product's **featured image** (`set_post_thumbnail()`) or variation image.

---

## Canva App Sidebar Features

### In-Editor Product Search

<a class="doc-image-link" href="./assets/canva-app-product-search.webp"><img src="./assets/canva-app-product-search.webp" alt="Searching WooCommerce products from the Canva sidebar" /></a>

A dedicated search bar queries your store's live catalog by **product title or SKU** with real-time 500 ms debouncing, returning up to 50 published items. Purely numeric queries automatically check for specific product IDs.

### Current Product Context

The app detects which product design is active and pins that product to the top of the sidebar under **✨ Current Product** with distinct visual highlighting.

### Click-to-Replace Images

Select an image frame on your Canva canvas, then click any product or gallery thumbnail in the sidebar. The connector transfers the high-res image data and swaps it into the selected frame instantly.

### Live Product Text Insertion

- **Add Name**: Inserts the product name (or variation name) onto the canvas.
- **Add Price**: Inserts formatted product price.
- **Add Description**: Inserts product short description or full description.

### Variation Support

For variable products, a **Select Variation** dropdown switches the displayed name, price, SKU, and description, and retargets *Save Design to Store* directly at that specific variation ID.

### One-Click Save to Store

<a class="doc-image-link" href="./assets/canva-app-save-design-to-store.webp"><img src="./assets/canva-app-save-design-to-store.webp" alt="Save Design to Store buttons in the Canva sidebar" /></a>

One click exports your design in high quality (`PNG`, `JPG`, or `GIF`), transmits the file to your WordPress store, and gives real-time visual feedback (*Saving…*, *✅ Saved!*, or *❌ Failed to Save!*).

---

## Security & Architecture

| Security Layer          | Implementation Details                                                         |
|-------------------------|--------------------------------------------------------------------------------|
| Store → Canva Auth      | **OAuth 2.0 Authorization Code with PKCE** (`S256` code challenge)             |
| CSRF Protection         | WordPress `wp_rest` nonce verification + `edit_products` capability check       |
| Replay Protection       | Single-use opaque `state` parameter cached server-side in transients           |
| Canva App → Store Auth  | **RS256 JWT validation** verified against Canva's public JWKS keysets          |
| App ID Verification     | JWT `aud` claim verified against stored **App ID**                             |
| Safe Redirects          | `wp_safe_redirect()` strictly allow-listing `canva.com`                        |
| Sanitization            | All input fields sanitized before database update                              |

---

## Feature Comparison & Summary

| Feature                    | Description                                                                     |
|----------------------------|---------------------------------------------------------------------------------|
| Design with Canva button   | Integrated on product edit pages, post edit pages, and products list table rows |
| Dynamic dimension matching | Auto-sized to featured image dimensions, theme single image size, or 1080×1080  |
| Asset pre-upload           | Current featured image automatically placed on the canvas                       |
| Design re-opening          | Persistent `_canva_design_id` reopens previous artwork for continuous iteration |
| Silent token refresh       | One-time consent approval with automatic background token renewals              |
| Live product search        | Fast search by title or SKU with 500 ms debouncing                              |
| Product context            | Current product pinned and highlighted at the top of the sidebar               |
| Click-to-replace           | One-click replacement of selected canvas frames with catalog images             |
| Text insertion             | Instant insertion of product title, price, and description text layers          |
| Variation support          | Manage visuals for parent products, variations, and grouped child items         |
| CORS control               | Scoped specifically to `canva/v1` routes with optional server disable toggle    |
| HPOS compatibility         | Fully compatible with WooCommerce High-Performance Order Storage                |
| Internationalization       | Ready for localization via `wkwc-canva-connect` text domain                     |
