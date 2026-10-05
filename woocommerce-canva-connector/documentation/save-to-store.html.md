---
title: Save to Store
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/save-to-store.html
---

# Save to Store

**Save Design to Store** is the purple button at the bottom of every product card in the Canva sidebar. One click exports your design, sends it to WordPress, and sets it as the product's featured image, no downloading, no re-uploading, no re-attaching.

<a class="doc-image-link" href="./assets/canva-app-save-design-to-store.webp"><img src="./assets/canva-app-save-design-to-store.webp" alt="Save Design to Store button in the Canva sidebar" /></a>

---

## How to save

1. Finish your design on the Canva canvas.
2. In the **Webkul App** sidebar, find the card for the product you want to update.
3. For a variable or grouped product, pick the right entry in **Select Variation** first.
4. Click **Save Design to Store**.

The button reports its own progress:

| Button label             | Meaning                                               |
|--------------------------|-------------------------------------------------------|
| **Save Design to Store** | Ready                                                 |
| **Saving…**              | Export and upload in progress; the button is disabled |
| **✅ Saved!**             | Success, resets after 4 seconds                      |
| **❌ Failed to Save!**    | The save failed, resets after 5 seconds              |

---

## What happens on your store

Once the export reaches WordPress, the plugin:

1. **Validates the request**: the Canva JWT is verified against Canva's JWKS and its `aud` claim is checked against your saved App ID.
2. **Sideloads the file** into the Media Library with `media_handle_sideload()`, attaching it to the product. WordPress generates all the usual intermediate sizes.
3. **Sets the featured image**: the design becomes the product thumbnail via `set_post_thumbnail()`.
4. **Returns success**, which triggers the sidebar to refetch so the new artwork appears on the card immediately.

Files that fail to upload are skipped and their temporary copies deleted. If the upload fails, the store returns **HTTP 500, "Failed to upload images."** and the button shows *❌ Failed to Save!*.

---

## Accepted file types

The export requests `jpg`, `png`, and `gif`, and the extension is chosen from what Canva returns:

| Canva returns | Saved as |
|---------------|----------|
| `image/png`   | `.png`   |
| `image/gif`   | `.gif`   |
| anything else | `.jpg`   |

> 💡 **Note:** Standard web image formats (`.png`, `.jpg`, `.gif`) are fully supported by WordPress and WooCommerce product displays.

---

## Saving to a variation

When **Select Variation** is set to a specific variation, *Save Design to Store* targets **that variation's own ID** rather than the parent product. The image is attached to the variation, so it appears when a shopper selects that combination on the storefront.

Set the dropdown back to **Main Product** to update the parent product's image instead. Grouped products work identically, selecting a child targets that child.

> 💡 The card you click matters as much as the dropdown. Each product card has its own *Save Design to Store* button and saves to *that* product, which means you can attach the same design to several products without leaving the editor.

---

## Verifying the result

<a class="doc-image-link" href="./assets/wc-product-featured-image-saved.webp"><img src="./assets/wc-product-featured-image-saved.webp" alt="Design saved to WooCommerce product featured image" /></a>

1. Click **Return to `{your site}`** at the top-right of the Canva editor.
2. You land back on the product's edit screen in WordPress.
3. Refresh the page.
4. The **Product image** box shows the new design.

The image is an ordinary Media Library attachment, visible under **Media > Library**, and editable, replaceable or removable like any other upload.

> 💡 **Save before returning.** The return button only navigates; it does not save. If you leave without clicking *Save Design to Store*, the design stays in Canva but the product keeps its old image.

---

## Saving repeatedly

You can save as many times as you like. Each save adds a new attachment and repoints the featured image at it.

The previous image is **left in the Media Library**, unattached rather than deleted, so a save can never destroy an image you still needed. Tidy up old versions from **Media > Library** when you are sure they are no longer wanted.

---

## If the save fails

| Symptom                             | Likely cause                                     | Fix                                                         |
|-------------------------------------|--------------------------------------------------|-------------------------------------------------------------|
| **❌ Failed to Save!** immediately   | The export never completed                       | Make sure the design is valid, then retry                   |
| **❌ Failed to Save!** after a pause | The store rejected the upload                    | Check `wp-content/uploads` is writable by the web server    |
| Save succeeds, product unchanged    | Browser cache on the edit screen                 | Hard-refresh the product edit page                          |
| **401** in the browser console      | App ID mismatch, or an expired token             | Re-check the App ID in *Webkul WC Addons > Canva Connect*   |

Full diagnostics are in [Troubleshooting](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/troubleshooting.html).
