# Hello, World! — static site

Zero-dependency static page. No build step, no JS, no CDN, no external requests.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire site (inline `<style>`, single `<h1>Hello, World!</h1>`). |
| `README.md` | Serve / deploy / verify commands and evidence. |

## Local serve (verified below)

From the directory containing `index.html`:

```sh
python3 -m http.server 8000 --bind 0.0.0.0
```

Local URL: <http://localhost:8000/>

Check the body:

```sh
curl -sS http://localhost:8000/ | grep -o "Hello, World!"
# expected: Hello, World!
```

**Caveat: `localhost` is not public.** Passing the local check does **not** satisfy
the deployment requirement. See *Deployment status* below.

---

## Deployment status: NOT DEPLOYED (blocked, no credentials supplied)

The build phase produced and verified the artifact locally. The deployment half of
the requirement was **not** completed, and no public URL exists. This is stated
plainly rather than papered over.

### Why

A publicly reachable URL cannot be produced without an external destination, and
none was supplied with this assignment:

* **Static host (preferred, durable URL)** — needs an account/token or a named
  target: Netlify auth token + site, Vercel token, Cloudflare Pages API token +
  account ID, or a specific `org/name` GitHub repo for Pages. **None provided.**
* **Surge.sh** — needs an email/account or token. **None provided.**
* **Temporary tunnel (fallback)** — `cloudflared tunnel --url http://localhost:8000`
  or `ngrok http 8000` needs the binary installed plus (for ngrok) an authtoken.
  It would yield an **ephemeral** HTTPS URL that dies with the process, and the
  recorded URL would go stale immediately. Not usable as a durable deliverable.
* No hosting credentials may be **fabricated**, and no URL may be **invented**.

### Exact commands to deploy once a destination is supplied

**A. Netlify (drag-drop or CLI), durable URL**

```sh
# option 1: manual drag-and-drop of this directory at app.netlify.com/drop
# option 2: CLI
npm i -g netlify-cli
netlify deploy --dir . --prod          # needs NETLIFY_AUTH_TOKEN + site
```

**B. Cloudflare Pages (direct upload), durable URL**

```sh
npm i -g wrangler
wrangler pages deploy . --project-name hello-world   # needs CLOUDFLARE_API_TOKEN + account ID
```

**C. Vercel, durable URL**

```sh
npm i -g vercel
vercel deploy --prod .                 # needs VERCEL_TOKEN
```

**D. GitHub Pages, durable URL (repo must exist)**

```sh
git init && git add index.html README.md && git commit -m "hello world"
git branch -M main
git remote add origin git@github.com:<ORG>/<REPO>.git
git push -u origin main
# then: Settings -> Pages -> Source: main / (root)
# public URL: https://<ORG>.github.io/<REPO>/
```

**E. Tunnel (ephemeral, point-in-time only)**

```sh
cloudflared tunnel --url http://localhost:8000   # or: ngrok http 8000
```

### Public verification procedure to run after deploying

```sh
curl -sS -D - -o /tmp/body.html "<PUBLIC_URL>"
grep -o "Hello, World!" /tmp/body.html
```

Record in this file: HTTP status, final URL after redirects, matched body string,
and UTC timestamp. Note that the fetch above, run from the deploying machine, is a
**single-vantage-point** check — it is not independent confirmation.

### Evidence table

| Check | Result |
|---|---|
| `index.html` renders `Hello, World!` | Yes (source reviewed) |
| Zero external requests | Yes — inline CSS only, no JS/CDN/fonts |
| Local serve command documented | Yes (`python3 -m http.server 8000 --bind 0.0.0.0`) |
| Local fetch of `Hello, World!` | **Not executed in this build phase — served in the next phase** |
| Host chosen and credential/destination supplied | **No — not supplied** |
| Deployed to a real, publicly reachable URL | **No — not deployed** |
| Public URL fetched, status + body + timestamp recorded | **No — no URL to fetch** |
| Limitations stated | Yes — see above |

### Acceptance status

Not met: *"deployed to a real, publicly reachable URL"* and *"URL fetched publicly"*.
Blocked on one input: **a host plus its credential or destination.** Everything else
is ready to deploy in a single command.

### Reusable local module (host-agnostic)

GitHub Pages can serve the repo root with no workflow file:

```yaml
# .github/workflows/pages.yml (only if you prefer Actions-based publishing)
name: Deploy static site to Pages
on: { push: { branches: [main] } }
permissions: { contents: read, pages: write, id-token: write }
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: { name: github-pages }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/upload-pages-artifact@v3
        with: { path: . }
      - uses: actions/deploy-pages@v4
```
