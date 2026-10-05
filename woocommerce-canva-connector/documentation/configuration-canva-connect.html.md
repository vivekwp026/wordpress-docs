---
title: Configuration: Canva Connect Settings
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-canva-connect.html
---

# Configuration: Canva Connect Settings

After installing the plugin, configure your Canva Developer credentials from **Webkul WC Addons > Canva Connect** (or **Settings > Canva Connect**). The settings screen requires the `manage_woocommerce` capability.

<a class="doc-image-link" href="./assets/canva-connect-admin-settings.png"><img src="./assets/canva-connect-admin-settings.png" alt="Canva Connect Settings in WooCommerce Admin" /></a>

The settings page is located at:

```
https://your-domain.com/wp-admin/admin.php?page=wkwc-canva-connect
```

---

## Settings Overview

| Field                    | Source                                       | Stored as             | Required |
|--------------------------|----------------------------------------------|-----------------------|----------|
| **App ID**               | Canva Developer Portal → *Your apps*         | `canva_app_id`        | Yes      |
| **Client ID**            | Canva Developer Portal → *Your integrations* | `canva_client_id`     | Yes      |
| **Client Secret**        | Canva Developer Portal → *Your integrations* | `canva_client_secret` | Yes      |
| **Disable CORS Headers** | Your own server configuration                | `wkwc_disable_cors`   | No       |

Every value is sanitised before it is saved, and the form is protected by the `wkwc_settings_action` nonce.

---

## App ID

Paste the App ID you copied from **Your apps** in the Developer Portal. It begins with `AAG…`, for example `AAHOGOJacpE`.

This is used for **strict security validation**. Every request the Canva app makes to your store carries a Canva-issued RS256 JWT, and the plugin compares that token's `aud` claim against this value before doing anything with it.

> ⚠ **Do not leave this field empty.** The `aud` comparison only executes when the field has a value. With it blank, any Canva app presenting a validly signed token is accepted, and could read your catalogue or overwrite product images.

---

## Client ID

Paste the Client ID from your Canva **Integration**. It begins with `OC-…`, for example `OC-AZ_6NBnoLhwl`.

The Client ID identifies your store to Canva during the OAuth2 handshake. If it is missing, clicking **Design with Canva** returns:

```json
{
  "code": "no_key",
  "message": "Client ID not set.",
  "data": { "status": 400 }
}
```

---

## Client Secret

Paste the Client Secret generated alongside your Client ID. The field is rendered as a password input, so the stored value is masked on screen.

The secret is combined with the Client ID into an HTTP Basic `Authorization` header when the plugin exchanges an authorization code, or a refresh token, for an access token at `https://api.canva.com/rest/v1/oauth/token`.

> ⚠ Rotating the secret in the Developer Portal invalidates the one saved here. Update this field immediately afterwards, or every design attempt will fail at the token exchange.

---

## Disable CORS Headers

Leave this **unchecked** on a standard installation.

The Canva app runs inside an iframe served from `https://app-{app-id}.canva-apps.com`, so every call it makes to your store is cross-origin and preceded by a preflight request. By default the plugin answers those preflights for the `canva/v1` namespace and sends:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Max-Age: 86400
Vary: Origin
```

Tick the box **only if your web server already adds its own `Access-Control-Allow-Origin` header**, for instance through an Apache `Header always add` rule or an Nginx `add_header` directive. Two copies of that header is a hard browser error ("contains multiple values … but only one is allowed"), and PHP cannot remove one the web server appends after PHP has finished running.

When ticked, the plugin stops sending `Access-Control-Allow-Origin` but **still sends the other headers**, because hand-written server CORS blocks commonly omit `Authorization` from `Access-Control-Allow-Headers`, which would break every bearer-token request the app makes.

`Access-Control-Allow-Credentials` is deliberately never sent: browsers reject the combination of a wildcard origin and credentials, and the app authenticates with a bearer token rather than cookies.

### Restricting the origin in code

If a wildcard origin is too permissive for your policy, narrow it with the provided filter instead of disabling the headers:

```php
add_filter(
    'wkwc_canva_allowed_origin',
    function ( $origin, $request_origin ) {
        $app_id = strtolower( get_option( 'canva_app_id' ) );

        return 'https://app-' . $app_id . '.canva-apps.com';
    },
    10,
    2
);
```

Returning an empty string from this filter omits the header entirely and blocks all cross-origin browser calls.

---

## Important URLs

Below the form the page prints the two URLs that must be registered on your Canva **Integration**:

| Label                              | Example value                                       |
|------------------------------------|-----------------------------------------------------|
| **Redirect URI**                   | `https://your-domain.com/wp-json/canva/v1/callback` |
| **Return URL (Return Navigation)** | `https://your-domain.com/wp-json/canva/v1/return`   |

Always copy these from your own settings page rather than typing them by hand, they are generated with `rest_url()`, so they already reflect your site address, any subdirectory install and your permalink structure.

See [Redirect URIs & HTTPS](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-redirect-uris.html) for what each URL does and why HTTPS is non-negotiable.

---

## Saving

Click **Save Settings**. A *Settings saved.* notice confirms the update.

Nothing else needs to be configured on the WordPress side, there are no per-product settings, and no shortcodes to place.

---

## Verifying the Configuration

1. Open any product for editing.
2. Confirm the **Design with Canva** meta box is present in the right-hand column.
3. Click the button. On the first run Canva shows its consent screen; approve it.
4. You should land in the Canva editor on a canvas sized to your product image.

If step 3 or 4 fails, check [Troubleshooting](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/troubleshooting.html).
