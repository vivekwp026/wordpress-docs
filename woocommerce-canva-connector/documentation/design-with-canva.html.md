---
title: Design with Canva
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/design-with-canva.html
---

# Design with Canva

Once the plugin is configured and the Canva app is uploaded, designing a product image is a single click. This guide covers launching the Canva editor directly from WordPress and seamlessly returning with updated imagery.

<a class="doc-image-link" href="./assets/product-edit-design-with-canva-button.png"><img src="./assets/product-edit-design-with-canva-button.png" alt="Design with Canva meta box on the WooCommerce product edit page" /></a>

---

## Where the button appears

The plugin injects **Design with Canva** in two places.

### 1. The product edit page

Open any product from **Products > All Products** and look in the **right-hand column**. Scroll down past *Publish*, *Product image*, *Product gallery*, *Product categories*, *Product tags* and *Brands*, the **Design with Canva** meta box sits with them:

<a class="doc-image-link" href="./assets/design-with-canva-metabox.webp"><img src="./assets/design-with-canva-metabox.webp" alt="The Design with Canva meta box, with its button" /></a>

It contains one line of text, *Click below to create a new featured image using Canva.*, and one blue **Design with Canva** button.

The meta box is registered for the `product` post type and for regular `post` types, in the `side` context with default priority. Its exact position in the column therefore follows WordPress's normal meta box ordering, and each user can drag it higher or lower; the position is remembered per user. If you cannot see it, check that **Design with Canva** is ticked under **Screen Options** at the top of the edit screen.

### 2. The products list row action

<a class="doc-image-link" href="./assets/products-list-design-with-canva.webp"><img src="./assets/products-list-design-with-canva.webp" alt="Design with Canva row action on the WooCommerce products list" /></a>

Hover any row on **Products > All Products** and the row actions appear beneath the product name, in this order:

```
ID: 38 | Edit | Quick Edit | Trash | View | Design with Canva | Duplicate
```

**Design with Canva** is rendered in purple and bold so it stands out. It starts exactly the same flow without opening the product first.

> 💡 *Duplicate* comes from WooCommerce and *ID:* from the products list itself, so the exact set of neighbouring links depends on which other plugins are active. Canva Connect always inserts its link after the core actions.

> 💡 Both entry points open in a **new browser tab**, so your WordPress editing session, including any unsaved product changes, stays open behind it.

---

## Who can use it

The link is built with `wp_nonce_url()` against the `wp_rest` action, and the endpoint behind it requires the **`edit_products`** capability. Administrators and Shop Managers qualify by default.

| Situation                         | Result                                |
|-----------------------------------|---------------------------------------|
| Signed out                        | **401**, `rest_forbidden`            |
| Signed in without `edit_products` | **403**                               |
| Nonce missing or stale            | **403**, `rest_cookie_invalid_nonce` |

A stale nonce usually means the page has been open for more than a day. Refresh the product edit screen and click again.

---

## What happens when you click

**Step 1, Authorisation.**
The first time you ever click the button, you are redirected to Canva's consent screen to authorise the integration. Approve it. The five requested scopes are `design:content:read`, `design:content:write`, `design:meta:read`, `asset:read` and `asset:write`.

On every later click the plugin uses the stored refresh token to obtain a new access token silently, **you will not see the consent screen again**.

**Step 2, Existing design check.**
If this product has been designed before, its design ID is stored as the `_canva_design_id` post meta. The plugin looks the design up and, if it is still valid, reopens it. Your previous work is preserved.

**Step 3, Image pre-upload.**
For a product with no existing design, the plugin uploads the current featured image to Canva as an asset and polls the upload job until it completes. The canvas then opens with that image already placed on it.

**Step 4, Canvas sizing.**
The design is created at exactly the right dimensions, resolved in this order:

1. The **full-size dimensions of the product's featured image**
2. Your theme's **`woocommerce_single`** image size, when the product has no image
3. **1080 × 1080** as the final fallback

**Step 5, Redirect into the editor.**
The Canva editor opens on the new (or reopened) canvas, titled `Product Image - {Product Name}`.

---

## In the editor

<a class="doc-image-link" href="./assets/canva-app-current-product.webp"><img src="./assets/canva-app-current-product.webp" alt="The Webkul App panel open beside a product design in Canva" /></a>

Design as you normally would in Canva. The **Webkul App** panel on the left gives you your catalogue: product search, product data as text, click-to-replace images and the **Save Design to Store** button.

That panel is covered in full on [Using the Canva App](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-app-sidebar.html), and saving is covered on [Save to Store](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/save-to-store.html).

---

## Returning to WordPress

Click **Return to `{your site}`** in the top-right of the Canva editor, shown as **Return to webkul-WooCommerce** in the screenshots.

This hits the Return Navigation URL, which clears the session transients and redirects you back to that product's edit screen at `wp-admin/post.php?post={id}&action=edit`.

> 💡 **Save before you return.** Returning does not save anything by itself, click **Save Design to Store** in the sidebar first. Returning only navigates.

If the session has expired (transients live for one hour), the return page answers **"No active session found."**. Nothing is lost: your design remains in Canva, and reopening the product and clicking *Design with Canva* brings it straight back.

---

## Verifying the result

Back on the product edit screen, refresh the page. The **Product image** box in the right-hand column now shows the design you exported.

The image is a real entry in the WordPress **Media Library**, attached to the product, and can be edited, replaced or removed like any other upload.

---

## Designing the same product again

Click **Design with Canva** on the same product at any time. Because the design ID is stored on the product, you reopen the *same* Canva design rather than starting over, and can iterate on the artwork you already made.

Saving again replaces the featured image with the new export. The previous image stays in the Media Library, it is unattached, not deleted, so nothing is lost by accident.
