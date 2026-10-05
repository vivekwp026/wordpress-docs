---
title: Canva Developer Portal Setup
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-developer-portal.html
---

# Canva Developer Portal Setup

Connecting WooCommerce to Canva requires **two separate items** in the [Canva Developer Portal](https://www.canva.com/developers/). They are not interchangeable, and the integration only works when both exist:

| Item            | Purpose                                              | What you copy out                                        |
|-----------------|------------------------------------------------------|----------------------------------------------------------|
| **App**         | The React sidebar that runs inside the Canva editor  | **App ID** (starts with `AAG…`)                          |
| **Integration** | The Connect API client used by your WordPress server | **Client ID** (starts with `OC-…`) and **Client Secret** |

Keep all three values to hand, they go into [Canva Connect Settings](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-canva-connect.html) in the next step.

---

## Step 1: Create an App

The App is the frontend half: the panel your team sees in the Canva editor.

1. Log in to the [Canva Developer Portal](https://www.canva.com/developers/).
2. Go to **Your apps** and click **Create an App**.
3. Give it a name your team will recognise, for example *Webkul WooCommerce*.
4. Copy the generated **App ID**. It looks like `AAHOGOJacpE`.

> 💡 **Leave the App's Authentication tab empty.** You do *not* need to add an authentication provider. The app authenticates to your store with the RS256 JWT that Canva issues through `getDesignToken()`, not with OAuth.

The App ID is not cosmetic. Your store compares it against the `aud` claim of every incoming token, which is what stops a different Canva app from reading or overwriting your products.

---

## Step 2: Create an Integration

The Integration is the backend half: the OAuth2 client your WordPress server uses to create and read designs through the Canva Connect API.

1. In the Developer Portal, open the **Your integrations** tab.
2. Click **Create an integration**.
3. Under **Configuration**, note the **Client ID**, it looks like `OC-AZ_6NBnoLhwl`.
4. Generate a **Client Secret** and copy it immediately.

> ⚠ The Client Secret is shown **once**. If you navigate away without copying it, generate a new one and update WordPress to match.

---

## Step 3: Enable the Required Scopes

Still inside your **Integration**, open the **Scopes** tab and enable all five scopes the plugin requests:

```
design:content:read
design:content:write
design:meta:read
asset:read
asset:write
```

| Scope                  | Why the plugin needs it                                               |
|------------------------|-----------------------------------------------------------------------|
| `design:content:read`  | Read the design when reopening an existing one                        |
| `design:content:write` | Create the design and place the uploaded asset on it                  |
| `design:meta:read`     | Resolve the design's `edit_url` to redirect the admin into the editor |
| `asset:read`           | Confirm an asset upload job has completed                             |
| `asset:write`          | Upload the existing product image into Canva                          |

A missing scope surfaces as a failure on the very first design attempt, so enable all five before testing.

> 💡 The plugin does **not** request `offline_access`. The Canva Connect API issues refresh tokens without it, which is what lets the plugin skip the consent screen on later designs.

---

## Step 4: Set the Redirect URI and Return Navigation URL

Both fields live in your **Integration** settings, and both values are printed for you on the WordPress settings page.

1. In WordPress, open **Webkul WC Addons > Canva Connect**.
2. Under *Important URLs to paste in your Canva Developer Portal*, copy the two values.
3. Paste them into the matching fields on the Canva Integration.

| Canva field               | Value                                               |
|---------------------------|-----------------------------------------------------|
| **Redirect URI**          | `https://your-domain.com/wp-json/canva/v1/callback` |
| **Return Navigation URL** | `https://your-domain.com/wp-json/canva/v1/return`   |

<a class="doc-image-link" href="./assets/canva-connect-admin-settings.png"><img src="./assets/canva-connect-admin-settings.png" alt="Redirect URI and Return URL printed on the Canva Connect settings page" /></a>

Copy these directly from your own settings page rather than typing them out. They automatically account for your site's domain, permalink structure, and REST prefix.

Full detail on both fields, and on the HTTPS requirement, is in [Redirect URIs & HTTPS](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-redirect-uris.html).

---

## Step 5: Upload the App Bundle

The App created in Step 1 has no code in it yet. Building the React bundle and uploading it is covered in [Build the Canva App](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-app-build.html), do that after saving your credentials in WordPress, because the build needs your store URL.

---

## Checklist

Before moving on, confirm you have:

- [x] An **App**, and its **App ID** (`AAG…`)
- [x] An **Integration**, and its **Client ID** (`OC-…`) and **Client Secret**
- [x] All **five scopes** enabled on the Integration
