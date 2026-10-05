---
title: Using the Canva App
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-app-sidebar.html
---

# Using the Canva App

The **Webkul App** panel is the left-hand sidebar inside the Canva editor. It is a React app that talks directly to your WooCommerce store, making your live catalog, variations, and product data available while you design.

Open it from the **Apps** icon in Canva's left rail, or find it pinned at the bottom of the rail once you have used it. When you arrive via the *Design with Canva* button it opens automatically.

<a class="doc-image-link" href="./assets/canva-app-product-search.webp"><img src="./assets/canva-app-product-search.webp" alt="The Webkul App panel in the Canva editor" /></a>

---

## Panel layout

From top to bottom:

| Section                 | What it does                                                         |
|-------------------------|----------------------------------------------------------------------|
| **💡 Tip**               | A reminder of how click-to-replace works                             |
| **✨ Current Product**   | The product this design belongs to, highlighted with a purple border |
| **Your Store Products** | The searchable catalogue                                             |
| **Search box**          | Filters by product name or SKU                                       |
| **Product cards**       | One card per product, with images, data and actions                  |

---

## Current Product

The app asks your store which product the open design belongs to. It sends the design token, the plugin reads the `designId` claim, and matches it against the `_canva_design_id` post meta saved when the design was created.

When a match is found, that product is pinned at the top under **✨ Current Product**, drawn with a purple border and a pale purple background, and removed from the list below so it never appears twice.

If you opened the app on an unrelated Canva design, one not created through *Design with Canva*, no current product is shown. That is expected: the catalogue below still works, and *Save Design to Store* on any card still targets that card's product.

---

## Searching your catalogue

<a class="doc-image-link" href="./assets/canva-app-product-search.webp"><img src="./assets/canva-app-product-search.webp" alt="Searching WooCommerce products from inside the Canva editor" /></a>

Type into **Search products by name or SKU…**.

- The query is **debounced by 500 ms**, so results arrive once you pause typing.
- It matches **published products only**, returning up to **50** per query.
- A **purely numeric** query is additionally looked up as a **product ID**; if that product exists and is published, it is pinned to the top of the results.
- An empty search box lists your catalogue.

While a request is in flight the panel shows *Searching…*. An empty result set shows *No products found.*

---

## What is on a product card

Each card shows:

- **A thumbnail grid**: the featured image first, then every gallery image. The grid is one, two or three columns depending on how many images there are.
- **A Select Variation dropdown**: only for variable and grouped products.
- **The product name**, in bold.
- **Price** and **SKU**, when set.
- **Action buttons**: *Add Name*, *Add Price*, *Add Description* and *Save Design to Store*.

---

## Replacing an image on the canvas

This is the core interaction, and it has a required order:

1. **Click the image on your Canva canvas** to select it. Canva draws a coloured border around the selected frame.
2. **Click a thumbnail** in the sidebar, the featured image or any gallery image.
3. The image is fetched from your store, uploaded to Canva as an asset, and swapped into the selected frame in place.

Behind the scenes the store returns the image as a base64 data URL rather than a plain link, which is what lets Canva ingest it without needing public access to your uploads directory.

> ⚠ **Select a canvas image first.** Clicking a thumbnail with nothing selected shows the warning *"⚠️ Select an image on canvas first to replace it!"* for four seconds and does nothing else. This is deliberate, without a target frame there is nowhere for the image to go.

While the swap is running the thumbnail fades to half opacity.

---

## Adding product data as text

Three buttons drop live product data onto the canvas as text elements:

| Button              | Inserts                                                                            |
|---------------------|------------------------------------------------------------------------------------|
| **Add Name**        | The product name, or `Product Name - Variation Name` when a variation is selected |
| **Add Price**       | The price, prefixed with `$`                                                       |
| **Add Description** | The short description, falling back to the full description                        |

*Add Price* and *Add Description* only appear when the product actually has that data. Each click adds a new text element, which you can then position and style with Canva's normal tools.

---

## Working with variations

<a class="doc-image-link" href="./assets/canva-app-save-design-to-store.webp"><img src="./assets/canva-app-save-design-to-store.webp" alt="Product cards with variation selectors and Save Design to Store buttons" /></a>

For a **variable product**, the **Select Variation** dropdown lists *Main Product* plus every available variation, each labelled with its own price, for example `Main Product ($170)`.

Switching the selection changes what the card displays and what the buttons do:

| Selection        | Name / price / SKU / description shown                                                        | *Save Design to Store* target |
|------------------|-----------------------------------------------------------------------------------------------|-------------------------------|
| **Main Product** | The parent product's own values                                                               | The parent product            |
| **A variation**  | The variation's values, falling back to the parent's where the variation leaves a field empty | **That variation's ID**       |

This is what lets you build one design and attach it to a specific colour or size rather than to the whole product.

**Grouped products** expose their children through the same dropdown, and behave the same way, selecting a child retargets the save at that child product.

---

## Refreshing the panel

The panel refetches your catalogue automatically after a successful save, so the new artwork appears on the card straight away. To refresh manually, type in the search box and clear it again.

---

## When products do not load

If the panel shows *"Failed to load products. Please check if your WordPress site is accessible."*, the app reached your store but the request did not succeed. The usual causes:

- The store is not reachable over **HTTPS** from the Canva iframe.
- `CANVA_BACKEND_HOST` was compiled with the wrong URL, see [Build the Canva App](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-app-build.html).
- The **App ID** saved in WordPress does not match the App the token was issued for.
- A **CORS** header conflict with your web server.

Each case is diagnosed step by step in [Troubleshooting](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/troubleshooting.html).
