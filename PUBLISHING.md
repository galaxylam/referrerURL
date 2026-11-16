# Publishing to GitHub Pages

This repo is a plain static site. GitHub Pages serves it as-is — there is no build step, no bundler, no `npm install`.

## One-time setup (via the web UI)

1. Go to <https://github.com/galaxylam/referrerURL/settings/pages>
2. Under **Build and deployment**:
   - **Source**: `Deploy from a branch`
   - **Branch**: `main`
   - **Folder**: `/ (root)`
3. Click **Save**.

GitHub will publish the site within ~30 seconds.

If the Pages section is missing entirely, enable it first under **Settings → General → Pages → Build and deployment → Deploy from a branch**.

## One-time setup (via API)

```sh
gh api -X POST repos/galaxylam/referrerURL/pages \
  -f source[branch]=main \
  -f source[path]=/root
```

## Verify

```sh
gh api repos/galaxylam/referrerURL/pages --jq '{url, status, https_enforced}'
```

```sh
curl -sI https://galaxylam.github.io/referrerURL/
```

## The resulting URL

```
https://galaxylam.github.io/referrerURL/
```

### URL forms and what the destination will see

| Where the tool is hosted | Referrer sent to the destination |
|---|---|
| `https://galaxylam.github.io/referrerURL/` | `https://galaxylam.github.io/referrerURL/` |
| Custom domain `https://ref.example.com/` | `https://ref.example.com/` |

Note GitHub Pages serves a project site under a **path** (`/referrerURL/`), so the full referrer includes that path. If your destination app whitelists by path, it must allow `/referrerURL/`.

## Referrer policy — do not remove the meta tag

`index.html` contains:

```html
<meta name="referrer" content="no-referrer-when-downgrade">
```

The original ASP sent this as a response header:

```vbscript
response.AddHeader "referrer-policy","no-referrer-when-downgrade"
```

This matters because the modern browser default is `strict-origin-when-cross-origin`, which sends **only the origin** cross-origin (`https://galaxylam.github.io/` — no path). `no-referrer-when-downgrade` sends the **full URL**, which is what the original tool did and what the tool's own UI describes:

> Referrer Address can be any of the two or both:
> 1. `https://galaxylam.github.io/referrerURL/`
> 2. `https://galaxylam.github.io/*`

The meta tag takes precedence over any `Referrer-Policy` header GitHub Pages sets.

Two caveats:
- `no-referrer-when-downgrade` still drops the referrer entirely on an HTTPS → HTTP downgrade, so **test destinations must be HTTPS**.
- The path is preserved only when the destination is HTTPS. Over plain HTTP you get no referrer at all.

## Custom domain (optional)

If the destination app whitelists a domain you control, point a CNAME at GitHub Pages instead of using `galaxylam.github.io`:

1. Add a `CNAME` file at the repo root containing your domain, e.g. `ref.example.com`
2. In your DNS provider, add a `CNAME` for `ref.example.com` → `galaxylam.github.io`
3. In **Settings → Pages**, set **Custom domain** and enable **Enforce HTTPS**

Committing the `CNAME` file:

```sh
printf 'ref.example.com\n' > CNAME
git add CNAME && git commit -m "Add custom domain" && git push
```

## Updating after a change

```sh
git add -A
git commit -m "your message"
git push
```

GitHub Pages rebuilds automatically; the change is live in about a minute.

## Troubleshooting

**Site shows 404** — Pages is not enabled, or the branch/folder is wrong. Check `gh api repos/galaxylam/referrerURL/pages`.

**Site shows the README instead of the tool** — GitHub Pages found `README.md` and treated it as the homepage. This happens if `index.html` is missing from the deployed root. Confirm `index.html` is at the repo root.

**Destination receives no `Referer` header** — the destination is HTTP (downgrade), or the destination strips the header itself. Check with:

```sh
curl -sI -H "Referer: https://galaxylam.github.io/referrerURL/" https://your-destination.example/
```

**Destination receives only the origin** — the `<meta name="referrer">` tag was removed, or a `<meta>` with a stricter policy was added. Only one referrer meta should exist.
