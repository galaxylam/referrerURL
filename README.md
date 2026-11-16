# referrerURL

Static HTML/JS rewrite of the original `UrlReferrer.asp` (kept here as `UrlReferrer.txt` for reference) — a tool for testing **referring-URL authentication** on a destination application.

## What it does

1. **Test Referring URL Authentication** — navigate (GET) or POST to a destination URL so it receives this page's URL in the `Referer` header.
2. **Generate Encrypted URL Referrer and Subdomain** — encrypts a subdomain / referrer URL with the original `EncryptText` cipher (key `01011`, `+2` on `0`, `−2` on `1`) and shows the decrypted round-trip.
3. **Submit FORM with POST Method** — POSTs `subdomain` and `referrerURL` (encrypted values) to a target URL.

No backend. Everything runs in the browser; nothing is sent anywhere except the navigations/POSTs you trigger.

## Live URL

```
https://galaxylam.github.io/referrerURL/
```

## Referrer URLs your destination will see

The page prints the two acceptable values in its first card, derived from `location`:

| # | Value |
|---|---|
| 1 | `https://galaxylam.github.io/referrerURL/` — full page URL |
| 2 | `https://galaxylam.github.io/*` — host + wildcard |

Whitelist one or both in the destination app.

### Why the `<meta name="referrer">` tag matters

The page sets:

```html
<meta name="referrer" content="no-referrer-when-downgrade">
```

Browsers now default to `strict-origin-when-cross-origin`, which truncates a **cross-origin** referrer to the origin only (`https://galaxylam.github.io/` — no path). The original ASP set the `Referrer-Policy` header to `no-referrer-when-downgrade` deliberately, so the **full URL** (with path) is sent. The meta tag reproduces that behaviour and overrides any `Referrer-Policy` header the CDN sets.

Note `no-referrer-when-downgrade` still drops the referrer entirely on an HTTPS → HTTP downgrade, so test destinations must be HTTPS.

## Local development

```sh
python3 -m http.server 8000
# → http://localhost:8000/
```

## Publishing

Push to `main`; GitHub Pages is configured to build from the `main` branch, `/` (root). Settings → Pages → Build and deployment → Deploy from a branch.

See [PUBLISHING.md](PUBLISHING.md) for the step-by-step setup.
