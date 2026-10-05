---
title: Configuration: Redirect URIs & HTTPS
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-redirect-uris.html
---

# Configuration: Redirect URIs & HTTPS

The OAuth2 handshake between your store and Canva depends on **two URLs** being registered identically on both sides, over **HTTPS**. This page covers both requirements in detail.

<a class="doc-image-link" href="./assets/canva-connect-settings.webp"><img src="./assets/canva-connect-settings.webp" alt="Redirect URI and Return URL on the Canva Connect settings page" /></a>

---

## Where the URLs come from

The plugin prints both URLs on **Webkul WC Addons > Canva Connect**, under the heading *Important URLs to paste in your Canva Developer Portal*:

| Label on the settings page         | Canva Developer Portal field | REST route               |
|------------------------------------|------------------------------|--------------------------|
| **Redirect URI**                   | *Redirect URIs*              | `GET /canva/v1/callback` |
| **Return URL (Return Navigation)** | *Return navigation*          | `GET /canva/v1/return`   |

Both are generated with WordPress's `rest_url()`, so they already account for:

- your site address, including `www` or its absence
- a subdirectory installation
- your permalink structure and REST prefix

**Always copy them from your settings page.** Typing them by hand is the single most common cause of a failed first connection.

For example, on a standard installation, the two values resolve to:

```
https://your-domain.com/wp-json/canva/v1/callback
https://your-domain.com/wp-json/canva/v1/return
```

---

## The Redirect URI

This is where Canva sends the merchant's browser after the consent screen, carrying the authorization `code` and the `state` value.

**Register it under:** Developer Portal → *Your integrations* → your integration → **Redirect URIs**.

What the endpoint does when Canva calls it:

1. Reads `code` and `state` from the query string.
2. Strips everything non-alphanumeric from `state` and looks up the `canva_state_{state}` transient, which holds the product ID. A missing or expired transient returns **"Security check failed. Invalid state."**
3. Deletes the state transient, making it single-use.
4. Reads the PKCE `code_verifier` stored for that product.
5. Exchanges `code` + `code_verifier` for an access token at `https://api.canva.com/rest/v1/oauth/token`, authenticating with HTTP Basic (`client_id:client_secret`).
6. Stores the returned `refresh_token` in the `canva_refresh_token` option.
7. Creates or reopens the design and redirects into the Canva editor.

> ⚠ The URI must match **byte for byte** between WordPress and Canva. A trailing slash, `http` instead of `https`, or `www.` on one side only will be rejected by Canva as an invalid redirect.

---

## The Return Navigation URL

This is where Canva sends the merchant when they click the **Return to …** button in the top-right of the editor.

**Register it under:** Developer Portal → *Your integrations* → your integration → **Return navigation**.

What the endpoint does:

1. Finds the most recent active `canva_token_{post_id}` transient.
2. Derives the post ID from that transient's name.
3. Clears the session transients for the token and the design.
4. Redirects the browser to `wp-admin/post.php?post={id}&action=edit`, the product edit screen.

If no session transient is found the endpoint answers **"No active session found."**, typically because the one-hour session expired, or because the browser returned long after the design was opened.

> 💡 The return button reads *Return to `{your site name}`*, in the screenshots it appears as **Return to webkul-WooCommerce**, because that is the site name registered on the Canva integration.

---

## HTTPS is mandatory

> ⚠ **Canva rejects OAuth redirects to plain HTTP URLs.** A store served over `http://` cannot complete the handshake, no matter how the credentials are configured.

HTTPS is required in three separate places:

| Where                                              | Why                                                                                        |
|----------------------------------------------------|--------------------------------------------------------------------------------------------|
| The **Redirect URI**                               | Canva refuses to register or redirect to an `http://` URL                                  |
| The **Return Navigation URL**                      | Same restriction applies                                                                   |
| The **`CANVA_BACKEND_HOST`** the app is built with | The Canva editor is HTTPS; a browser blocks an insecure `fetch()` from it as mixed content |

### Making WordPress emit HTTPS URLs

`rest_url()` derives its scheme from the `home` and `siteurl` options. If the settings page prints `http://…`, fix the site address rather than editing the URL by hand:

1. **Settings > General**: set both **WordPress Address** and **Site Address** to the `https://` form.
2. Or define them in `wp-config.php`:

```php
define( 'WP_HOME',    'https://your-domain.com' );
define( 'WP_SITEURL', 'https://your-domain.com' );
```

3. If WordPress sits behind a reverse proxy or load balancer that terminates TLS, tell it so, otherwise it will keep generating `http://` links:

```php
if ( isset( $_SERVER['HTTP_X_FORWARDED_PROTO'] ) && 'https' === $_SERVER['HTTP_X_FORWARDED_PROTO'] ) {
    $_SERVER['HTTPS'] = 'on';
}
```

Place this above the `require_once ABSPATH . 'wp-settings.php';` line.

A local or staging installation still requires a valid SSL certificate. Two workable options:

- **A public HTTPS tunnel**: expose the local site through a tunnelling service and use the public HTTPS hostname as both the Redirect URI and `CANVA_BACKEND_HOST`. This is the most reliable route, because Canva's servers can reach it.
- **A self-signed certificate**: sufficient for the browser-side redirects once you trust the certificate locally, but Canva's own servers must still be able to reach the callback. Use this only when the host is publicly resolvable.

> ⚠ Plain `http://localhost` and XAMPP-style setups without HTTPS will not work. This is a Canva platform restriction, not a plugin limitation.

---

## Verifying the setup

Run each check in order:

**1: The REST routes answer.**

```bash
curl -I https://your-domain.com/wp-json/canva/v1/products
```

Expect **HTTP 401** (route exists, token demanded). A **404** means permalinks need flushing at *Settings > Permalinks*.

**2: The callback is reachable over HTTPS.**

```bash
curl -sS -o /dev/null -w '%{http_code} %{ssl_verify_result}\n' \
  https://your-domain.com/wp-json/canva/v1/callback
```

Expect **200** with an `ssl_verify_result` of **0**. A non-zero verify result means the certificate chain is incomplete and Canva's servers will reject it.

**3: The scheme is right.** Open the settings page and confirm both printed URLs start with `https://`.

**4: End to end.** Click *Design with Canva* on a product and approve the consent screen. Landing in the Canva editor confirms the whole chain.

---

## Common Redirect Errors

| Symptom                                            | Cause                                                                     | Fix                                                 |
|----------------------------------------------------|---------------------------------------------------------------------------|-----------------------------------------------------|
| Canva shows *invalid redirect_uri*                 | The registered URI differs from `rest_url( 'canva/v1/callback' )`         | Re-copy the value from the settings page            |
| *Security check failed. Invalid state.*            | The 300-second state transient expired, or the callback was replayed      | Restart from the *Design with Canva* button         |
| *Failed to get access token*                       | Wrong Client Secret, or a mismatched `redirect_uri` at the token exchange | Re-save the secret; confirm the URI matches exactly |
| *No active session found.*                         | The one-hour session transient expired before the return click            | Reopen the product and start again                  |
| Browser blocks the app's requests as mixed content | `CANVA_BACKEND_HOST` was built with `http://`                             | Rebuild the app with the `https://` host            |

More detail in [Troubleshooting](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/troubleshooting.html).
