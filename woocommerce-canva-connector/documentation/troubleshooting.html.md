---
title: Troubleshooting
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/troubleshooting.html
---

# Troubleshooting

Symptoms grouped by where in the workflow they appear, each with the cause and the fix.

---

## Quick reference

| Category           | Problem                                  | Solution                                                                                                  |
|--------------------|------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **Installation**   | Plugin will not activate                 | Activate WooCommerce first, the plugin refuses to run without it                                         |
| **Installation**   | *JWT library not found.* (500)           | Run `composer install` in the plugin directory                                                            |
| **Installation**   | REST routes return 404                   | Re-save permalinks at *Settings > Permalinks*                                                             |
| **Installation**   | Activation notice in the site footer     | Activate the purchase code at *Webkul WC Addons > License*                                                |
| **Installation**   | No *Webkul WC Addons* menu               | Submodules are missing, use *Settings > Canva Connect*, or run `git submodule update --init --recursive` |
| **Connection**     | *Client ID not set.* (400)               | Save the Client ID in *Webkul WC Addons > Canva Connect*                                                  |
| **Connection**     | Canva rejects the redirect URI           | Re-copy the Redirect URI from the settings page, it must match byte for byte                             |
| **Connection**     | *Security check failed. Invalid state.*  | The 300-second state expired; restart from *Design with Canva*                                            |
| **Connection**     | CSRF / security error on the button      | Click the button from a freshly loaded product page, it needs a live `wp_rest` nonce                     |
| **Authentication** | *Invalid App ID (aud).* (401)            | The App ID in WordPress must exactly match the App the bundle was built for                               |
| **Authentication** | *Failed to get access token*             | Wrong Client Secret, or a mismatched `redirect_uri` at the token exchange                                 |
| **Frontend**       | *Failed to load products*                | The store must be reachable over HTTPS from the Canva iframe                                              |
| **Frontend**       | Blank sidebar in Canva                   | Rebuild with the correct `CANVA_BACKEND_HOST` and re-upload `dist/app.js`                                 |
| **Frontend**       | Multiple-values CORS error               | Your web server is adding its own `Access-Control-Allow-Origin`                                           |
| **Uploads**        | Design saves but the image never changes | `wp-content/uploads` must be writable by the web server                                                   |
| **Uploads**        | Only page 1 arrived                      | WordPress blocked the other MIME types                                                                    |
| **Return**         | *No active session found.*               | The one-hour session expired; reopen the product                                                          |

---

## Installation problems

### The plugin will not activate

WordPress refuses activation while WooCommerce is inactive, showing *"Webkul Canva Connect for Woocommerce needs WooCommerce. Activate WooCommerce first, then activate this plugin."* This is enforced twice, by the `Requires Plugins: woocommerce` header and by the plugin's own activation hook.

**Fix:** install and activate WooCommerce 10.0 or later, then activate this plugin.

### *JWT library not found.*: HTTP 500

The `firebase/php-jwt` dependency is missing, so no token can be verified and the Canva sidebar stays empty.

**Fix:**

```bash
cd wp-content/plugins/wkwc-canva-connect
composer install --no-dev --optimize-autoloader
```

Confirm `vendor/autoload.php` now exists. This only affects Git checkouts, the store ZIP ships with `vendor/` included.

### A licence notice appears in the site footer

Every installed Webkul module must be activated with a valid purchase code. Until that is done, WordPress shows a mandatory activation notice in the site footer and the storefront checkout is blocked.

**Fix:** go to **Webkul WC Addons > License**, enter the **Purchase Code** for *WooCommerce Canva Connector*, and confirm the **Status** column reads *Approved*. Full steps are in [Installation, Step 5](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/installation.html#step-5---activate-the-license).

### The settings page is not where the docs say

The page registers under **Webkul WC Addons > Canva Connect** when the Webkul submodules are present, and falls back to **Settings > Canva Connect** when they are not.

**Fix (Git checkouts):**

```bash
git submodule update --init --recursive
```

Either location works, the page is identical. Access needs the `manage_woocommerce` capability.

### REST routes return 404

```bash
curl -I https://your-domain.com/wp-json/canva/v1/products
```

A **401** is healthy, the route exists and is demanding a token. A **404** means the rewrite rules were never flushed.

**Fix:** go to *Settings > Permalinks*, choose **Post name**, and click **Save Changes**.

---

## Connection and OAuth problems

### *Client ID not set.*: HTTP 400

```json
{ "code": "no_key", "message": "Client ID not set.", "data": { "status": 400 } }
```

**Fix:** paste the Client ID (`OC-…`) from your Canva **Integration** into *Webkul WC Addons > Canva Connect* and save.

### Canva rejects the redirect URI

The URI registered on the Canva Integration must match `rest_url( 'canva/v1/callback' )` **byte for byte**. A trailing slash, `http` instead of `https`, or `www.` on one side only is enough to fail.

**Fix:** copy the value printed under *Important URLs to paste in your Canva Developer Portal* on your settings page, never type it by hand. See [Redirect URIs & HTTPS](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-redirect-uris.html).

### *Security check failed. Invalid state.*

The `state` transient lives for **300 seconds**, and is deleted the moment it is used.

Causes: more than five minutes spent on the consent screen; a reloaded or bookmarked callback URL; or an object cache that is dropping transients.

**Fix:** go back to the product and click *Design with Canva* again. If it recurs immediately, check that transients persist, a misconfigured persistent object cache is the usual culprit.

### CSRF / security error on the button

The button link carries a `wp_rest` nonce, which expires after roughly 24 hours.

**Fix:** reload the product edit page (or the products list) and click the button from the fresh page. Clicking a link copied out of an old tab will not work.

### *Failed to get access token*

The token exchange at `https://api.canva.com/rest/v1/oauth/token` was rejected.

Check, in order:

1. The **Client Secret** in WordPress matches the current secret in the Developer Portal. Rotating it in Canva invalidates the stored copy.
2. The **Redirect URI** sent at the exchange matches the registered one exactly.
3. Your server can make **outbound HTTPS** requests to `api.canva.com`, a firewall or proxy blocking egress produces this too.

### The consent screen appears every time

The refresh token is not being persisted in the `canva_refresh_token` option.

**Fix:** confirm the option is writable (some hardened setups block `update_option` from REST context), and check the WooCommerce logs under **WooCommerce > Status > Logs**, source `wkwc-canva-connect`.

---

## Canva app problems

### *Failed to load products. Please check if your WordPress site is accessible.*

The sidebar reached the fetch but the request did not succeed. Work through these in order:

**1: Is the store on HTTPS?** The Canva editor is HTTPS, so a browser blocks any request to an `http://` origin as mixed content. This is not fixable from the plugin side; the store must have a valid certificate.

**2: Was the bundle built with the right host?** `CANVA_BACKEND_HOST` is compiled into the JavaScript. It must be the site root, with no trailing slash and no `/wp-json`:

```bash
CANVA_BACKEND_HOST=https://your-domain.com npm run build
```

For a subdirectory install, include the subdirectory path: `https://your-domain.com/store`.

**3: Does the App ID match?** Open the browser devtools console in the Canva editor. A **401** with *"Invalid App ID (aud)."* means the App ID saved in WordPress is not the App the token was issued for.

**4: Is the certificate chain complete?**

```bash
curl -sS -o /dev/null -w '%{http_code} %{ssl_verify_result}\n' \
  https://your-domain.com/wp-json/canva/v1/products
```

Expect `401 0`. A non-zero `ssl_verify_result` means an incomplete chain, install the intermediate certificates.

### Blank sidebar in Canva

The bundle failed to load or failed to execute.

**Fix:** rebuild with `npm run build` and re-upload `dist/app.js` in the Developer Portal. Check the devtools console for the actual error, a build that stopped with *"BACKEND_HOST is undefined."* never produced a bundle at all.

### CORS: *header contains multiple values, but only one is allowed*

Two layers are both sending `Access-Control-Allow-Origin`. The plugin already removes WordPress core's copy, so the second one is coming from your **web server**, an Apache `Header always add`, an Nginx `add_header`, or a server-level "allow CORS" plugin.

**Diagnose:** request a plain static file.

```bash
curl -I https://your-domain.com/wp-includes/js/jquery/jquery.min.js | grep -i access-control
```

If the header is present on a static asset, WordPress never ran, the source is the web server.

**Fix, either** remove the CORS rule from the server configuration, **or** tick **Disable CORS Headers** in *Webkul WC Addons > Canva Connect* so PHP stops sending its own. The plugin still sends `Access-Control-Allow-Headers`, `Access-Control-Allow-Methods` and `Access-Control-Max-Age`, which matters because hand-written server CORS blocks usually omit `Authorization` and would reject every bearer token.

Equivalent in code:

```php
add_filter( 'wkwc_canva_allowed_origin', '__return_empty_string' );
```

### *⚠️ Select an image on canvas first to replace it!*

Not an error. Clicking a sidebar thumbnail replaces a **selected** image on the canvas, so something must be selected first.

**Fix:** click the image on the canvas (Canva draws a border around it), then click the thumbnail.

### No *✨ Current Product* section

The app could not match the open design to a product. Either the design was not created through *Design with Canva*, or the `_canva_design_id` post meta is missing.

**Fix:** close the design and reopen it from the product's *Design with Canva* button. The catalogue below still works either way, you can save to any product from its own card.

---

## Save and upload problems

### The design saves but the product image never changes

`media_handle_sideload()` could not write the file.

**Fix:** make `wp-content/uploads` writable by the web server user.

```bash
chown -R www-data:www-data wp-content/uploads
find wp-content/uploads -type d -exec chmod 755 {} \;
find wp-content/uploads -type f -exec chmod 644 {} \;
```

Also confirm disk space, and that `upload_max_filesize` and `post_max_size` in PHP are large enough for the image export.

### *❌ Failed to Save!* immediately

The Canva export never completed, or returned no valid image.

**Fix:** make sure the canvas has content, then click again. Check the browser devtools console for the underlying export error.

---

## Return navigation problems

### *No active session found.*

The `canva_token_{post_id}` transient lives for **one hour**. Returning after that finds no session.

**Fix:** nothing is lost. Close the tab, reopen the product in WordPress, and click *Design with Canva*, the stored `_canva_design_id` reopens the same design.

### Returning lands on the wrong product

The return endpoint infers the product from the **most recent** session transient, not from the request. With two designs open at once, both return buttons resolve to whichever session was started last.

**Fix:** work on one design at a time, or navigate to the correct product manually in WordPress.

---

## Diagnostic commands

**Are the routes registered?**

```bash
curl -I https://your-domain.com/wp-json/canva/v1/products
# expect: HTTP/2 401
```

**Is HTTPS valid end to end?**

```bash
curl -sS -o /dev/null -w 'status=%{http_code} tls=%{ssl_verify_result}\n' \
  https://your-domain.com/wp-json/canva/v1/callback
# expect: status=200 tls=0
```

**Which CORS headers actually go out?**

```bash
curl -sI -X OPTIONS https://your-domain.com/wp-json/canva/v1/products \
  -H 'Origin: https://app-example.canva-apps.com' | grep -i access-control
# expect exactly one Access-Control-Allow-Origin line
```

**Are the credentials saved?**

```bash
wp option get canva_app_id
wp option get canva_client_id
wp option get canva_refresh_token
```

**What is the plugin logging?**
JWT verification failures are written to the WooCommerce logger under the source `wkwc-canva-connect`. Read them at **WooCommerce > Status > Logs**.

---

## Still stuck?

🎟 **Support Portal** [https://webkul.uvdesk.com](https://webkul.uvdesk.com)

📧 Email: [support@webkul.com](mailto:support@webkul.com)

Include your WordPress and WooCommerce versions, the PHP version, the exact error message, and the relevant lines from **WooCommerce > Status > Logs**.
