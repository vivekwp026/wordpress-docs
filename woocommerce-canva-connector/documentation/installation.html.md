---
title: Installation
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/installation.html
---

# Installation

This guide explains how to install and activate **WooCommerce Canva Connector** on your WordPress site.

Follow each step carefully to ensure a smooth setup.

---

## System Requirements

Before installation, make sure:

| Requirement         | Minimum / Recommended Version                     |
|---------------------|---------------------------------------------------|
| WordPress Version   | 6.7 or higher (tested up to 6.9)                  |
| WooCommerce Version | 10.0 or higher (tested up to 10.4)                |
| PHP Version         | 7.4+ (tested up to 8.4)                           |
| Memory Limit        | 256MB or higher                                   |
| SSL                 | **Required**, Canva rejects plain HTTP redirects |
| Composer            | Needed only for a Git checkout                    |

> ⚠ **WooCommerce is mandatory.** The plugin ships with the `Requires Plugins: woocommerce` header and also guards its own activation hook, so WordPress refuses to activate it while WooCommerce is inactive. Install and activate WooCommerce first.

---

## Step 1: Download Plugin

After purchase, you will receive a **ZIP file** of the plugin.

Do NOT unzip the file before uploading.

---

## Step 2: Upload Plugin

1. Login to your **WordPress Admin Panel**
2. Navigate to: **Plugins > Add Plugin**
3. Click **Upload Plugin** (top of page)
4. Click **Choose File**
5. Select the downloaded ZIP file

<a class="doc-image-link" href="./assets/plugin-upload.webp"><img src="./assets/plugin-upload.webp" alt="Plugins > Add Plugin > Upload Plugin screen in the WordPress admin" /></a>

> 💡 On WordPress 6.5 and later the menu item reads **Add Plugin**; older releases label it **Add New**. Both open the same screen.

---

## Step 3: Install Plugin

Click the **Install Now** button.

WordPress will upload and install the plugin automatically.

---

## Step 4: Activate Plugin

Once installed:

- You will see **Plugin installed successfully**
- Click **Activate Plugin**

You can confirm the result at any time from **Plugins > Installed Plugins**, where *WooCommerce Canva Connector* appears alongside WooCommerce:

<a class="doc-image-link" href="./assets/plugins-installed.webp"><img src="./assets/plugins-installed.webp" alt="WooCommerce Canva Connector active on the Installed Plugins screen" /></a>

If WooCommerce is not active, activation is blocked with the message *"Webkul Canva Connect for Woocommerce needs WooCommerce. Activate WooCommerce first, then activate this plugin."*

---


## Step 5: Activate the License

Activation is done from the shared Webkul licensing screen:

1. Go to **Webkul WC Addons > License**.
2. Find the **WooCommerce Canva Connector** row in the **Module License** table.
3. Enter the **Purchase Code** you received with your order and submit it.
4. The **Status** column turns **Approved**, and **Support Till** shows your support expiry date.

<a class="doc-image-link" href="./assets/webkul-wc-addons-license.webp"><img src="./assets/webkul-wc-addons-license.webp" alt="Module License table under Webkul WC Addons > License, showing Canva Connect approved" /></a>

The table lists, for every installed Webkul add-on: **Plugin**, **Version**, **Purchase From**, **Purchase Code**, **Support Till** and **Status**.

> ⚠ The screen warns that *all active modules must be activated with valid purchase codes. If any module remains unactivated, customers will be unable to complete the checkout process, and a mandatory activation notice will be displayed in the website footer.* Activate the license before going live.

- **[License Activation Guide](https://wpdoc.webkul.com/license-validator/)**: step-by-step instructions for activating a Webkul plugin licence.

---

## Step 6: Locate the Settings Page

After activation, navigate to the Canva Connect settings page in your WordPress Admin sidebar:

**Webkul WC Addons > Canva Connect** (or **Settings > Canva Connect**)

<a class="doc-image-link" href="./assets/canva-connect-admin-settings.png"><img src="./assets/canva-connect-admin-settings.png" alt="Canva Connect Settings Page in WordPress Admin" /></a>

### Settings Configuration Fields

The settings screen displays the following configuration fields and API endpoints:

| Field / Setting | Type | Description |
|-----------------|------|-------------|
| **App ID** | Text Input | The Canva Application ID (e.g., `AAHOGOJacpE`) from the Canva Developer Portal. |
| **Client ID** | Text Input | The OAuth2 Client ID (e.g., `OC-...`) generated in your Canva integration. |
| **Client Secret** | Password Input | The secure Client Secret key paired with your Client ID. |
| **Disable CORS Headers** | Checkbox | Check this box if your web server or CDN already manages CORS headers to prevent duplicate header conflicts. |
| **Redirect URI** | Read-only URL | `https://your-domain.com/wp-json/canva/v1/auth/callback` — Copy this exact URL and register it in your Canva Developer Portal redirect configuration. |
| **Return URL** | Read-only URL | `https://your-domain.com/wp-json/canva/v1/editor/return` — The callback endpoint used by Canva after design creation. |

> 💡 **Tip:** Save your settings by clicking the **Save Changes** button at the bottom after entering your credentials.

---

## Step 7: Verify the REST Routes

The plugin registers its endpoints in the `canva/v1` REST namespace. Confirm they are live before configuring Canva:

```bash
curl -I https://your-domain.com/wp-json/canva/v1/products
```

A **401** response is the expected, healthy answer, it means the route exists and is correctly demanding a bearer token. A **404** means permalinks need flushing.

---

## Step 8: Update Permalinks

This step is **mandatory** if you saw a 404 above.

- Go to **Settings > Permalinks**.
- Set **Post Name** as the permalink structure.
- Click **Save Changes**, this flushes the rewrite rules and registers the REST routes.

<a class="doc-image-link" href="./assets/permalinks.webp"><img src="./assets/permalinks.webp" alt="Settings > Permalinks with the Post name structure selected" /></a>

---

## Step 9: Confirm the Button Is Live

Open any product from **Products > All Products**. The **Design with Canva** meta box should now be present in the right-hand column:

<a class="doc-image-link" href="./assets/design-with-canva-metabox.webp"><img src="./assets/design-with-canva-metabox.webp" alt="The Design with Canva meta box on a product edit screen" /></a>

The button will not reach Canva yet, it needs the credentials configured in the next two chapters, but seeing the box confirms the plugin is installed and running.

---

## Notes

- Always ensure the plugin ZIP is compatible with your WooCommerce version.
- The plugin declares WooCommerce **HPOS** compatibility, so no incompatibility warning appears.
- The `wp-content/uploads` directory must be **writable by the web server**, otherwise designs saved from Canva cannot be sideloaded into the Media Library.
- Installation is only half the job, the plugin does nothing until the [Canva Developer Portal](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-developer-portal.html) credentials are in place.

### Admin menu and role changes

Activating the plugin also makes two changes to the WordPress admin that are worth knowing about:

- **Menu order is fixed.** *Webkul WC Addons*, *WooCommerce* and *Products* are moved together, in that order, directly below *Comments*.
- **The Editor role is upgraded.** Users with the built-in **Editor** role are granted the full set of WooCommerce product capabilities (`manage_woocommerce`, `edit_products`, `publish_products`, `edit_others_products`, `delete_products` and the related product-term capabilities), so they can use Canva Connect. To keep their sidebar focused, the default **Posts**, **Media** and **Pages** menus are hidden from Editors, the content itself is untouched and still reachable by direct URL.

> ⚠ **Note:** If your site restricts Editor user roles from managing WooCommerce products, review your role permissions before activating on production.
