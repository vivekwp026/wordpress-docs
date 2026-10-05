---
title: Introduction
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/
---

# Introduction

WooCommerce Canva Connector bridges your WooCommerce store and the Canva design editor. Store admins can design, edit and publish product imagery using Canva's editor without ever leaving the e-commerce workflow.

<a class="doc-image-link" href="./assets/products-list-design-with-canva.webp"><img src="./assets/products-list-design-with-canva.webp" alt="WooCommerce Canva Connector - Products List Integration" /></a>

The plugin eliminates the tedious repetitive loop that usually surrounds every product image change:

> download the image → open external editor → design & export → re-upload to WordPress → re-attach as product featured image.

Instead, a **Design with Canva** button on the product edit page (and products list) opens a correctly sized Canva canvas, and a **Save Design to Store** button in the Canva sidebar pushes the finished artwork straight back into the WordPress Media Library as that product's featured image.

---

## What does this plugin do?

The integration is made of two halves that communicate seamlessly via secure REST APIs:

| Component                | What it is                              | Where it runs           |
|--------------------------|-----------------------------------------|-------------------------|
| **`wp-canva-connect`**   | A WordPress / WooCommerce plugin        | Your WordPress store    |
| **`canva-app-frontend`** | A Canva App built in React + TypeScript | Inside the Canva editor |

The WordPress plugin manages the OAuth2 PKCE handshake, canvas dimension resolution, and automated image sideloading. The Canva app powers the in-editor sidebar: product search, variation selection, live data insertion, and direct design export back to your store.

---

## Admin Capabilities

Once configured, an admin (or any user with the `manage_woocommerce` and `edit_products` capabilities) can:

- Launch Canva from **any product edit page** via the *Design with Canva* meta box
- Launch Canva directly from the **products list table** via the *Design with Canva* row action
- Open a canvas that **automatically matches the product's image dimensions**
- Have the **existing featured image pre-uploaded** into the design as a Canva asset
- **Re-open previous designs** later instead of starting from scratch
- **Search the whole store catalog** from inside the Canva sidebar
- Insert a product's **name, price, SKU, or description** as live text layers
- **Replace any image** on the canvas with any catalog or gallery image in one click
- **Save designs back** to any product, including a specific variation

---

## Key Capabilities at a Glance

- **One-Click Canva Integration**: Direct button injection into product edit pages and products list table rows.
- **Dynamic Dimension Matching**: Automatically reads original image dimensions or theme sizes to provision a perfectly sized canvas without distortion.
- **Automated Sideloading**: Finished designs are sent to WordPress, sideloaded into the Media Library, and attached as the featured image automatically.
- **In-Editor Product Search**: Browse and search your entire WooCommerce catalog from within the Canva sidebar.
- **Enterprise Security**: OAuth 2.0 with PKCE for store authentication, and cryptographic RS256 JWT validation for all app requests.

<a class="doc-image-link" href="./assets/canva-connect-settings.webp"><img src="./assets/canva-connect-settings.webp" alt="Canva Connect Settings screen in the WordPress admin" /></a>

---

## Requirements

> ⚠ **Your store must be served over HTTPS.** Canva strictly rejects OAuth redirects to plain HTTP URLs, and the Canva app cannot communicate with an insecure origin from its iframe.

| Requirement         | Minimum / Recommended Version            |
|---------------------|------------------------------------------|
| WordPress Version   | 6.7 or higher (tested up to 6.9)         |
| WooCommerce Version | 10.0 or higher (tested up to 10.4)       |
| PHP Version         | 7.4+ (tested up to 8.4)                  |
| Node.js             | 18.x or 20.10.x (to build the Canva app) |
| npm                 | 9 or 10                                  |
| SSL                 | **Required**, Canva rejects plain HTTP   |
| Canva account       | A Canva Developer account                |

WooCommerce must be **installed and active** before this plugin can be activated. The plugin declares `Requires Plugins: woocommerce` and guards its activation hook.

The plugin also fully declares compatibility with WooCommerce **High-Performance Order Storage (HPOS)**.

---

## Documentation Navigation

1. [Features](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/features.html) - The complete capability list.
2. [Installation](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/installation.html) - Install the plugin and activate its license.
3. [Canva Developer Portal](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-developer-portal.html) - Create the App and Integration credentials.
4. [Canva Connect Settings](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-canva-connect.html) - Configure App ID, Client ID, and Client Secret.
5. [Redirect URIs & HTTPS](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/configuration-redirect-uris.html) - Complete the secure handshake.
6. [Build the Canva App](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-app-build.html) - Compile and upload the React frontend bundle.
7. [Design with Canva](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/design-with-canva.html) - Day-to-day design and publishing workflow.
8. [Using the Canva App](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-app-sidebar.html) - Working inside the Canva editor sidebar.
9. [Save to Store](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/save-to-store.html) - Exporting and sideloading imagery.
10. [Step-by-Step Guide](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/how-to-guide.html) - End-to-end walkthrough.

---

## Plugin Demo

Want to see the integration before installing it? The demo store has the plugin configured end to end:

- The **Design with Canva** meta box on the product edit page
- The **Design with Canva** row action on the products list
- The Canva Connect settings screen
- The in-editor sidebar, product search, and **Save Design to Store**

[🔗 **Live Demo**](https://wp-canva-connect.wcdemo.webkul.com/)
[🔗 **Buy Now**](https://store.webkul.com/woocommerce-canva-connector.html)

---

## Support Information

If you have any questions or need custom modifications, raise a support ticket:

🎟 **Support Portal:** [https://webkul.uvdesk.com](https://webkul.uvdesk.com)  
📧 **Email:** [support@webkul.com](mailto:support@webkul.com)
