---
title: How It Works
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/how-it-works.html
---

# How It Works

This page explains the architecture behind the integration, the OAuth2 PKCE handshake, how the design is created and sized, how the Canva app authenticates back to your store, and how the finished artwork is sideloaded onto the product.

It is background reading. Nothing here needs to be configured by hand.

---

## The two halves

| Component            | Technology             | Runs on                        | Responsibility                                                |
|----------------------|------------------------|--------------------------------|---------------------------------------------------------------|
| `wp-canva-connect`   | PHP / WordPress plugin | Your store                     | OAuth handshake, design creation, image sideloading, REST API |
| `canva-app-frontend` | React + TypeScript     | Inside the Canva editor iframe | Sidebar UI, product search, design export                     |

They meet at one place: the **`canva/v1` REST namespace** on your store.

---

## Flow 1: Starting a design (OAuth2 with PKCE)

Triggered by the *Design with Canva* button.

### First run: full consent

1. **`GET /canva/v1/auth?post_id={id}&_wpnonce={nonce}`**
   The endpoint requires a signed-in user with `edit_products` and a valid `wp_rest` nonce.

2. **No refresh token yet**, so a fresh authorization is started.

3. **State generated.** A 32-character opaque value is created and stored server-side as the transient `canva_state_{state}`, holding the product ID, with a **300-second** lifetime. The product ID is never exposed in the URL.

4. **PKCE verifier generated.** A 64-character `code_verifier` is stored as `canva_code_verifier_{post_id}`, also for 300 seconds. The challenge sent to Canva is:

   ```
   code_challenge = base64url( sha256( code_verifier ) )
   code_challenge_method = S256
   ```

5. **Redirect to Canva** via `wp_safe_redirect()`, with `canva.com` and `www.canva.com` added to the allowed-redirect-hosts list and nothing else:

   ```
   https://www.canva.com/api/oauth/authorize
     ?response_type=code
     &client_id={client_id}
     &redirect_uri={site}/wp-json/canva/v1/callback
     &scope=design:content:read design:content:write design:meta:read asset:read asset:write
     &state={state}
     &code_challenge={challenge}
     &code_challenge_method=S256
   ```

6. **`GET /canva/v1/callback`**: Canva returns with `code` and `state`. The endpoint:
   - sanitises `state` to alphanumerics and looks up its transient; a miss returns *"Security check failed. Invalid state."*
   - **deletes** the transient, so the state is single-use
   - reads the matching `code_verifier`
   - POSTs to `https://api.canva.com/rest/v1/oauth/token` with `grant_type=authorization_code`, the `code`, the `redirect_uri` and the `code_verifier`, authenticating with HTTP Basic `base64(client_id:client_secret)`
   - stores the returned `refresh_token` in the `canva_refresh_token` option

### Later runs: silent refresh

`/auth` finds `canva_refresh_token` set, POSTs `grant_type=refresh_token` to the same token endpoint, receives a new access token, and goes **straight to the design**. The consent screen is skipped entirely.

> 💡 PKCE is what makes this safe. Even if an authorization code were intercepted, it cannot be exchanged without the `code_verifier`, which never leaves your server.

---

## Flow 2: Creating the design

With an access token in hand, the plugin caches it as `canva_token_{post_id}` for **one hour**, then:

### Step A: Reopen an existing design

If the product carries a `_canva_design_id` post meta, the plugin fetches `GET /rest/v1/designs/{id}`. On HTTP 200 with a usable `edit_url`, it caches `canva_design_{post_id}` and redirects straight into that design. **Nothing is recreated, and no previous work is lost.**

### Step B: Upload the current image as an asset

For a product with no stored design and an existing featured image:

1. The file is read through the WordPress filesystem abstraction (`WP_Filesystem`), not `file_get_contents()`.
2. It is POSTed to `https://api.canva.com/rest/v1/asset-uploads` as `application/octet-stream`, with the filename base64-encoded in the `Asset-Upload-Metadata` header.
3. The returned job is polled at `GET /rest/v1/asset-uploads/{job_id}`, up to **15 attempts, 2 seconds apart**, so up to 30 seconds.
4. On `success`, the resulting `asset_id` is carried into design creation, which places the current product image on the canvas.

A failed or timed-out upload is not fatal, the design is simply created empty.

### Step C: Dynamic dimension matching

The canvas size is resolved in strict order:

| Priority | Source                                             | Notes                                                                                      |
|----------|----------------------------------------------------|--------------------------------------------------------------------------------------------|
| 1        | `wp_get_attachment_image_src( $image_id, 'full' )` | The product's featured image at full size                                                  |
| 2        | `wc_get_image_size( 'woocommerce_single' )`        | Used when the product has no image; a zero height falls back to the width, giving a square |
| 3        | `1080 × 1080`                                      | Final fallback                                                                             |

This is what keeps designs from being stretched or cropped when they come back, the canvas matches the slot the image is going into.

### Step D: Create and redirect

```json
{
  "design_type": { "type": "custom", "width": 1200, "height": 1200 },
  "title": "Product Image - Travis Scott x Air Jordan 1 Low",
  "asset_id": "MAF…"
}
```

is POSTed to `https://api.canva.com/rest/v1/designs`. The returned design ID is saved to `_canva_design_id` on the product, which is what makes reopening and product-context lookup work, and the admin is redirected to the design's `edit_url`.

If the response has no `edit_url`, the plugin stops with *"Failed to create design: …"* and the raw response, so the cause is visible.

---

## Flow 3: The Canva app authenticating back

The app never sees your Client Secret. Instead, for every request it calls Canva's `getDesignToken()` and sends the result as a bearer token:

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6…
```

Your store verifies it in five steps:

1. **Extract** the token from the header. Missing or malformed → **401**.
2. **Read the `aud` claim** from the payload (taking the first entry when it is an array).
3. **Compare `aud` to your saved App ID.** A mismatch → **401** *"Invalid App ID (aud)."*
4. **Fetch Canva's public keys** from `https://api.canva.com/rest/v1/apps/{appId}/jwks`, cached for one hour in the `canva_jwks_{app_id}` transient.
5. **Verify the RS256 signature** with `firebase/php-jwt`, which also enforces `exp` and `nbf`. Failure → **401** *"Invalid token."*, logged to the WooCommerce logger under the `wkwc-canva-connect` source.

The decoded token is then attached to the request as `decoded_token`, which is how `/context` reads the `designId` claim.

> ⚠ **Step 3 only runs when the App ID field is filled in.** Leave it empty and any validly signed Canva token is accepted, including one from an app you do not control.

### Why CORS matters here

The app's iframe is served from `https://app-{app-id}.canva-apps.com`, so every call is cross-origin and preflighted. The plugin answers those preflights for the `canva/v1` namespace only, removing core's own CORS headers first so exactly one `Access-Control-Allow-Origin` goes out, two values is a hard browser error.

`Authorization` is listed explicitly in `Access-Control-Allow-Headers`, because a `*` wildcard there does **not** cover it, and every call from the app carries a bearer token.

---

## Flow 4: Saving the design back

1. The app calls `requestExport({ acceptedFileTypes: ['jpg','png','gif'] })`.
2. Each exported page's blob URL is fetched and appended to a `FormData` as `images[]`, named `design-page-{n}.{ext}`, alongside the target `product_id`.
3. The form is POSTed to `POST /canva/v1/update-product-image` with the bearer token.
4. The store validates the token, then loops the uploaded files through `media_handle_sideload()`, attaching each to the product. Files with an upload error are skipped and their temp copies deleted.
5. `set_post_thumbnail()` assigns the attachment as the product's featured image (or updates the variation image).
6. `{ "success": true }` is returned, and the sidebar refetches the catalogue and the design context.

---

## Flow 5: Returning to WordPress

`GET /canva/v1/return` finds the most recent `_transient_canva_token_%` option, derives the post ID from its name, clears the token and design transients, and redirects to `wp-admin/post.php?post={id}&action=edit`.

With no active session it answers *"No active session found."*, the design is still safe in Canva, and clicking *Design with Canva* again reopens it.

---

## Data stored on your site

| Key                             | Type      | Lifetime  | Holds                            |
|---------------------------------|-----------|-----------|----------------------------------|
| `canva_app_id`                  | option    | permanent | Your Canva App ID                |
| `canva_client_id`               | option    | permanent | Your Canva Client ID             |
| `canva_client_secret`           | option    | permanent | Your Canva Client Secret         |
| `wkwc_disable_cors`             | option    | permanent | The Disable CORS Headers switch  |
| `canva_refresh_token`           | option    | permanent | OAuth refresh token              |
| `_canva_design_id`              | post meta | permanent | The design ID for that product   |
| `canva_state_{state}`           | transient | 300 s     | Product ID for one OAuth attempt |
| `canva_code_verifier_{post_id}` | transient | 300 s     | PKCE verifier                    |
| `canva_token_{post_id}`         | transient | 1 h       | Access token for the session     |
| `canva_design_{post_id}`        | transient | 1 h       | Design ID for the session        |
| `canva_jwks_{app_id}`           | transient | 1 h       | Cached Canva public keys         |
