---
title: Build the Canva App
source: https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-app-build.html
---

# Build the Canva App

The App you created in the [Canva Developer Portal](https://wpdoc.webkul.com/woocommerce-canva-connector/documentation/canva-developer-portal.html) is an empty shell until you compile the React frontend and upload it. The bundle has to be built **against your own store URL**, because that URL is baked into the JavaScript at compile time, there is no runtime setting for it.

<a class="doc-image-link" href="./assets/canva-developer-portal-code-upload.png"><img src="./assets/canva-developer-portal-code-upload.png" alt="Upload compiled JavaScript bundle to Canva Developer Portal Code Upload" /></a>

---

## Requirements

| Requirement | Version             |
|-------------|---------------------|
| Node.js     | `v18` or `v20.10.0` |
| npm         | `v9` or `v10`       |

The project ships an `.nvmrc` pinned to `20.10.0`, so a version manager gets you the right runtime in one command:

```bash
cd canva-app-frontend
nvm install
```

---

## Step 1: Install dependencies

```bash
cd canva-app-frontend
npm install
```

The `postinstall` script copies `.env.template` to `.env` if you do not already have one.

---

## Step 2: Point the app at your store

`CANVA_BACKEND_HOST` is the base URL the app prefixes onto every request, for example `${BACKEND_HOST}/wp-json/canva/v1/products`. Set it to the **root URL of your WordPress site**, with no trailing slash and no `/wp-json` suffix.

Open `.env` and set it:

```bash
CANVA_BACKEND_HOST=https://your-domain.com
CANVA_APP_ID=AAHOGOJacpE
```

For a subdirectory install, include the subdirectory path:

```bash
CANVA_BACKEND_HOST=https://your-domain.com/store
```

> ⚠ **It must be HTTPS.** The Canva editor is served over HTTPS, so a browser blocks any `fetch()` the app makes to an `http://` origin as mixed content. A production build whose host contains `localhost` prints the warning *"BACKEND_HOST should not be set to localhost for production builds!"*, the build still completes, so read the console output before uploading a bundle.

You can override the value on the command line instead of editing `.env`, which is handy in CI:

```bash
CANVA_BACKEND_HOST=https://your-domain.com npm run build
```

### Environment variables

| Variable              | Purpose                                                                                                                                                          |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CANVA_BACKEND_HOST`  | Root URL of your WordPress store, **required**                                                                                                                  |
| `CANVA_APP_ID`        | Your Canva App ID, used for JWT verification                                                                                                                     |
| `CANVA_BACKEND_PORT`  | Port for the sample Node backend that ships with the Canva starter. Not used by this integration, WordPress *is* the backend, and safe to leave at its default |
| `CANVA_APP_ORIGIN`    | From *Developer Portal → Configure your app → App Origin*; enables HMR                                                                                           |
| `CANVA_HMR_ENABLED`   | Set to `TRUE` to enable hot module replacement while developing                                                                                                  |
| `CANVA_FRONTEND_PORT` | Dev server port, default `8080`                                                                                                                                  |
| `CANVA_HTTPS_ENABLED` | Serve the dev server over HTTPS, required to preview in Safari                                                                                                  |
| `CANVA_SSL_KEY_FILE`  | Path to a TLS private key; leave empty to auto-generate into `./.ssl`                                                                                            |
| `CANVA_SSL_CERT_FILE` | Path to the matching TLS certificate                                                                                                                             |

---

## Step 3: Build the bundle

```bash
npm run build
```

This runs webpack in production mode and then extracts the translation messages. The output lands in:

```
canva-app-frontend/dist/app.js
```

If the build stops with **"BACKEND_HOST is undefined."**, `CANVA_BACKEND_HOST` was not set, go back to Step 2. This is the only host problem that aborts the build; a `localhost` host only warns.

---

## Step 4: Upload the bundle to Canva

1. Open the [Canva Developer Portal](https://www.canva.com/developers/) and select your **App**.
2. Go to **App source** and choose to upload a bundle.
3. Upload `dist/app.js`.
4. Save, then preview the app.

The app registers itself as a **Design Editor intent**, so it appears in the Canva editor's side panel, labelled with your app's name, shown as **Webkul App** in the screenshots throughout this documentation.

---

## Developing against a live store

For iterative work you can run the dev server instead of uploading a bundle each time:

```bash
npm start
```

The server listens on `http://localhost:8080`. It only serves a JavaScript bundle, so there is nothing to see by visiting that URL directly, preview it through Canva:

1. In the Developer Portal, open your app and select **App source > Development URL**.
2. Enter `http://localhost:8080`.
3. Click **Preview** to open the Canva editor with your app loaded.
4. Click **Open** the first time you use the app.

To preview in **Safari**, the dev server must itself be HTTPS-enabled, set `CANVA_HTTPS_ENABLED=TRUE` in `.env`, or pass `--use-https` to `npm start`. Leaving `CANVA_SSL_KEY_FILE` and `CANVA_SSL_CERT_FILE` empty auto-generates a self-signed certificate into `./.ssl`.

Even in development, `CANVA_BACKEND_HOST` must point at a **real HTTPS WordPress instance** with the plugin installed. The app has no mock data, an unreachable backend shows *"Failed to load products. Please check if your WordPress site is accessible."*

---

## Useful scripts

| Command              | What it does                                               |
|----------------------|------------------------------------------------------------|
| `npm start`          | Start the dev server for Developer Portal previews         |
| `npm run build`      | Production build into `dist/app.js`, then extract messages |
| `npm run lint`       | ESLint over the project                                    |
| `npm run lint:fix`   | ESLint with autofix                                        |
| `npm run lint:types` | TypeScript type-check only                                 |
| `npm run format`     | Prettier over all CSS and TS/TSX files                     |
| `npm test`           | Jest test suite                                            |
| `npm run extract`    | Extract i18n messages into `dist/messages_en.json`         |

---

## Rebuild checklist

You must rebuild and re-upload the bundle whenever:

- your **store URL changes**, including moving from staging to production
- you switch from `http://` to `https://`
- your site moves into or out of a **subdirectory**
- you change anything in `src/app.tsx`

The App ID and the store credentials live in WordPress and in the Developer Portal, so changing those does **not** require a rebuild.
