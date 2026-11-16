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

Note the trailing slash on #1 — that is what `location.pathname` returns for a GitHub Pages project site, and it is the exact string the destination will see in the `Referer` header.

To confirm what your destination actually receives, hit a header echo service (e.g. `https://httpbin.org/headers`) from the tool's "Go to destination URL" button.

### Encrypted values for the form POST

The "Generate Encrypted Values" section produces the `subdomain` and `referrerURL` values for the POST form. Example — for subdomain `rdc.nml3.naxosmusiclibrary.com`:

```
subdomain   = jrvnq<-1pbe,pkj5,p_vqqosqkang`t_tw,emo
referrerURL = jrvnq<-1e_n_zwjck0egvfw`,km1pchctpctSTJ-
```

(The `subdomain` value is the encryption of `https://` + your subdomain; the `referrerURL` value is the encryption of the referrer URL as typed.) The "Use in POST form below" button copies both into the third card automatically.

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
