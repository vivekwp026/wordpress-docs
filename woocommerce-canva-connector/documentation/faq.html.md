---
title: FAQ
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/faq.html
---

# FAQ

Common questions about **WooCommerce Canva Connector**.

---

## Setup

### Do I need a licence key, and where do I enter it?

Yes. Enter the **Purchase Code** that came with your order at **Webkul WC Addons > License**, in the *WooCommerce Canva Connector* row of the **Module License** table. The **Status** column turns *Approved* once it is accepted.

Leaving a module unactivated makes WordPress show a permanent activation notice in the site footer, and blocks checkout on the storefront. See [Installation, Step 5](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/installation.html#step-5---activate-the-license).

### Do I need a paid Canva plan?

No. A free Canva account can create a Developer account and build apps. Some *design features* inside the editor are Pro-only, but the integration itself is not.

### Why do I need both an App and an Integration?

They do different jobs. The **App** is the React sidebar that runs inside the Canva editor, authenticated with a JWT. The **Integration** is the OAuth2 client your WordPress server uses to create and read designs through the Connect API. Neither can do the other's work, so both are required.

See [Canva Developer Portal](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-developer-portal.html).

### Can I run this on plain HTTP?

No. Canva rejects OAuth redirects to `http://` URLs, and the Canva editor is served over HTTPS so a browser blocks the app's requests to an insecure origin as mixed content. A valid certificate is a hard requirement.

See [Redirect URIs & HTTPS](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-redirect-uris.html).

### Can I test this on localhost?

Not directly. `http://localhost` and XAMPP-style setups without TLS will not work. Expose the site through a public HTTPS tunnel and use that hostname as both the Redirect URI and `CANVA_BACKEND_HOST`, Canva's servers need to reach your callback.

### Do I have to build the React app myself?

Yes, at least once. `CANVA_BACKEND_HOST` is compiled into the JavaScript at build time, so the bundle has to be built against your own store URL and uploaded to your own Canva App.

See [Build the Canva App](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-app-build.html).

### What happens if I leave the App ID empty?

The integration still works, but you lose an important security check: the `aud` comparison only runs when the field has a value. With it blank, any Canva app holding a validly signed token is accepted. **Fill it in.**

---

## Usage

### Where is the Design with Canva button?

In two places: the **Design with Canva** meta box in the right-hand column of the product edit screen, and a purple **Design with Canva** row action on **Products > All Products**. Both open in a new tab.

### Why does Canva ask for permission only the first time?

The plugin stores the refresh token returned by Canva and exchanges it for a fresh access token on every later design, bypassing the consent screen. That is expected behaviour, not a bug.

### Does my design get lost if I close the tab?

No. The design lives in your Canva account like any other, and its ID is stored on the product. Clicking *Design with Canva* again reopens the same design.

What you lose by closing without saving is only the *transfer* to WooCommerce, click **Save Design to Store** to push it across.

### Why is my canvas already the right size?

The plugin reads your product's featured image dimensions and creates the canvas to match, falling back to your theme's `woocommerce_single` image size, then to 1080 × 1080. That is what stops designs coming back stretched or cropped.

### Why is my product image already on the canvas?

Before creating the design, the plugin uploads the current featured image to Canva as an asset and places it on the new canvas, so you start from what the product already looks like.

### Can I use this on blog posts too?

Yes. The meta box is registered for the `post` type as well as `product`. The Canva sidebar's product features will not apply, but the design and save flow works.

### Can I attach one design to several products?

Yes. Each product card in the sidebar has its own *Save Design to Store* button, and each saves to that card's product. Search for the next product and click its button, no need to leave the editor.

### Can I save to a specific variation?

Yes. Choose the variation in the card's **Select Variation** dropdown; the save then targets that variation's own ID instead of the parent product. Grouped products work the same way through their children.

### What image formats are supported?

The plugin exports and saves standard high-resolution web image formats: **PNG**, **JPG/JPEG**, and **GIF** (including animated GIF).

### Why does clicking a thumbnail do nothing?

You need to select an image on the canvas first, that is the frame the thumbnail replaces. With nothing selected the app shows *"⚠️ Select an image on canvas first to replace it!"*.

### How many products does the sidebar show?

Up to **50 published products** per query. Use the search box, by name or SKU, to narrow it down. There is no pagination.

---

## Permissions and security

### Who can use the integration?

Anyone with the `edit_products` capability can start a design; the settings page needs `manage_woocommerce`. Administrators and Shop Managers have both by default.

The plugin also grants the built-in **Editor** role the full set of WooCommerce product capabilities, so Editors can use Canva Connect too. As part of that, the default *Posts*, *Media* and *Pages* menus are hidden from Editors, the content is not deleted, only the menu entries are removed. See [Installation → Admin menu and role changes](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/installation.html#admin-menu-and-role-changes).

### Is my Client Secret exposed to the Canva app?

No. The secret never leaves your server. It is used only in the server-to-server token exchange with Canva. The app authenticates with a Canva-issued JWT and knows nothing about your OAuth credentials.

### How is the OAuth flow protected?

With **PKCE**. A random `code_verifier` is generated per attempt and never leaves your server; only its SHA-256 hash is sent to Canva. An intercepted authorization code is useless without the verifier. A single-use, server-side `state` transient guards against replay.

### Can anyone call the plugin's REST endpoints?

The CORS policy is permissive by default (`Access-Control-Allow-Origin: *`), but every product endpoint demands a valid Canva RS256 JWT whose `aud` matches your App ID. CORS is not the access control here, the token is. You can narrow the origin with the `wkwc_canva_allowed_origin` filter.

### Are old images deleted when I save a new one?

No. The previous image stays in the Media Library, unattached rather than deleted, so a save can never destroy an image you still needed. Clean up from **Media > Library** when you are sure.

---

## Compatibility

### Does it work with WooCommerce HPOS?

Yes. Compatibility with High-Performance Order Storage is declared explicitly, so no incompatibility warning appears.

### Does it work with variable and grouped products?

Yes. Variations and grouped children both appear in the sidebar's **Select Variation** dropdown, with their own name, price, SKU and description, and saving targets the selected entry.

### Is the plugin translatable?

Yes. All strings use the `wkwc-canva-connect` text domain, and a `.pot` file ships in the `languages/` directory.

### What versions are supported?

| Software             | Requires | Tested up to |
|----------------------|----------|--------------|
| WordPress            | 6.7      | 6.9          |
| WooCommerce          | 10.0     | 10.4         |
| PHP                  | 7.4      | 8.4          |
| Node.js (build only) | 18       | 20.10.0      |

---

## Support

### Where do I get help?

🎟 **Support Portal** [https://webkul.uvdesk.com](https://webkul.uvdesk.com)

📧 Email: [support@webkul.com](mailto:support@webkul.com)

> ⚠ Production version is provided by default
> Development version available at extra cost

### Where can I try the plugin first?

- **[Live Demo](https://wp-canva-connect.wcdemo.webkul.com/)**: Experience the plugin in action.
- **[Buy Now](https://store.webkul.com/woocommerce-canva-connector.html)**: Purchase the plugin from the Webkul store.
- **[License Activation Guide](https://wpdoc.webkul.com/license-validator/)**: Step-by-step instructions on activating your plugin license.
- **[Get Support](https://webkul.uvdesk.com/en/customer/create-ticket/)**: Need assistance? Create a ticket and our support team will help you ASAP.

### What should I include in a ticket?

Your WordPress, WooCommerce and PHP versions, the exact error message, whether it happened in WordPress or in the Canva sidebar, and the relevant lines from **WooCommerce > Status > Logs** (source `wkwc-canva-connect`).

Before opening a ticket, it is worth walking through [Troubleshooting](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/troubleshooting.html), most issues are a Redirect URI mismatch, a missing HTTPS certificate, or a bundle built against the wrong `CANVA_BACKEND_HOST`.
