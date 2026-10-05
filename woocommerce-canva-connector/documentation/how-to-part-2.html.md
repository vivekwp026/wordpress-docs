---
title: "Canva Developer Portal & Configuration"
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/how-to-part-2.html
---

# Canva Developer Portal & Configuration

Follow these steps to create your Canva Developer App, generate OAuth credentials, upload the app bundle, configure WooCommerce settings, manage CORS headers, and configure WordPress permalinks.

---

## Step 1: Create an App in Canva Developer Portal

- **What to click:** Visit the [Canva Developer Portal](https://www.canva.com/developers/), log in to your Canva account, click **Your apps**, and click **Create an app**.
- **What to enter:** Enter your app name (e.g., *Webkul App* or *My Store Canva Connector*).
- **What to copy:** Copy the generated **App ID** (starts with `AAHOG...`).
- **Expected result:** Your Canva App is created and assigned a unique App ID used for token validation.

---

## Step 2: Create Integration & OAuth Credentials

- **What to click:** In the Canva Developer Portal, switch to the **Your integrations** tab and click **Create an integration**.
- **What to do:**
  1. Under **Configuration**, copy the **Client ID** (starts with `OC-...`).
  2. Click **Generate secret** to generate and copy the **Client Secret**.
  3. In the **Scopes** tab, enable the required permissions:
     - `design:content:read`
     - `design:content:write`
     - `design:meta:read`
     - `asset:read`
     - `asset:write`
- **Expected result:** OAuth 2.0 credentials and permissions are created for your store.

---

## Step 3: Upload App Bundle (Code Upload)

- **What to click:** In your Canva App settings, navigate to **Build > Inside Canva > Code upload** (or **App source**).
- **What to do:**
  1. Select **JavaScript bundle**.
  2. Click **Choose file** and upload your compiled `dist/app.js` bundle (maximum size 5MB).
  3. Under **Translations**, upload your language file (e.g., `messages_en.json`) if needed.
- **Expected result:** The bundle is uploaded, registering the Webkul App inside the Canva editor's left panel.

<a class="doc-image-link" href="./assets/canva-developer-portal-code-upload.png"><img src="./assets/canva-developer-portal-code-upload.png" alt="Canva Developer Portal Code Upload and App Identifiers" /></a>

---

## Step 4: Configure WooCommerce Canva Connect Settings

- **What to click:** In your WordPress Admin, go to **Webkul WC Addons > Canva Connect**.
- **What to enter:**
  1. **App ID:** Paste the App ID copied from Canva Developer Portal.
  2. **Client ID:** Paste the Client ID from your Canva Integration.
  3. **Client Secret:** Paste the Client Secret.
- **What to copy & paste into Canva:**
  - Copy the **Redirect URI** displayed at the bottom (`https://your-domain.com/wp-json/canva/v1/callback`) and paste it into the **Redirect URI** field of your Canva Integration.
  - Copy the **Return URL** (`https://your-domain.com/wp-json/canva/v1/return`) and paste it into the **Return Navigation URL** field in Canva.
- **What to click:** Click **Save Settings**.
- **Expected result:** WordPress securely stores the credentials and the OAuth handshake is established.

<a class="doc-image-link" href="./assets/canva-connect-admin-settings.png"><img src="./assets/canva-connect-admin-settings.png" alt="Canva Connect Settings in WooCommerce Admin" /></a>

---

## Step 5: CORS Headers & HTTPS Setup

- **HTTPS Requirement:** Your WordPress store must have a valid SSL certificate (`https://`). Canva will reject unsecured `http://` API requests.
- **Disable CORS Headers Checkbox:**
  - **Leave unchecked (Default):** The plugin automatically manages and sends required CORS headers (`Access-Control-Allow-Origin: *`, `Authorization`, `Content-Type`) for Canva editor iframe requests.
  - **Check the box only if:** Your web server (Nginx/Apache/Cloudflare) already injects its own `Access-Control-Allow-Origin` header to avoid duplicate header conflicts.
- **Expected result:** Cross-origin communication between the Canva editor sandbox and your WooCommerce REST API operates smoothly without browser errors.

---

## Step 6: Configure WordPress Permalinks

- **What to click:** Go to **Settings > Permalinks** in your WordPress Admin.
- **What to select:** Under *Common Settings*, select **Post name** (`/%postname%/`).
- **What to click:** Click **Save Changes**.
- **Expected result:** WordPress permalinks are flushed, ensuring all `/wp-json/canva/v1/*` REST API endpoints route properly.

<a class="doc-image-link" href="./assets/permalinks.webp"><img src="./assets/permalinks.webp" alt="WordPress Permalinks Post Name Setting" /></a>
