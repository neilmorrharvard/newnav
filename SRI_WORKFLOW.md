# SRI (Subresource Integrity) Workflow

SRI lets the browser verify that the script it loads (e.g. from jsDelivr) hasn’t been altered. If the hash doesn’t match, the script won’t run.

## 1. When to update

Update the SRI hash **whenever you change the nav script or the commit hash** the loader uses. Same cadence as updating `COMMIT_HASH` in the loader.

## 2. Generate the hash

Use the **exact file** that will be served at the URL you’re loading (same commit as `COMMIT_HASH`).

### Option A: From your local file (after you’ve pushed)

```bash
# From repo root, after the version you want is committed and pushed:
openssl dgst -sha384 -binary new_nav.js | openssl base64 -A
```

Then add the algorithm prefix. Example output is `abc123...`; the full integrity value is:

```text
sha384-<that base64 string>
```

Example one-liner that prints the full value:

```bash
echo -n "sha384-$(openssl dgst -sha384 -binary new_nav.js | openssl base64 -A)"
```

### Option B: From the CDN URL (same commit as in the loader)

Replace `COMMIT_HASH` with the actual hash (e.g. `9ce6176`):

```bash
curl -sL "https://cdn.jsdelivr.net/gh/sasktoday/newnav@9ce6176/new_nav.js" | openssl dgst -sha384 -binary | openssl base64 -A
```

Then use: `sha384-<that base64 string>`.

## 3. Put the hash in the loader

1. Open the loader you use (e.g. `nav_script_loader_STAGING.html`).
2. Set the **SRI hash** constant (e.g. `SRI_HASH`) to that full value (`sha384-...`).
3. Ensure the script element that loads the nav has:
   - `integrity = SRI_HASH`
   - `crossOrigin = 'anonymous'` (required for cross-origin SRI).

The loader snippet should look like:

```javascript
const SRI_HASH = 'sha384-<your-base64-hash>';
// ...
script.src = scriptUrl;
script.integrity = SRI_HASH;
script.crossOrigin = 'anonymous';
script.defer = true;
```

## 4. Preload (optional)

If you use a `<link rel="preload" href="..." as="script">` for the same URL, you can add the same integrity so the preload is verified:

```html
<link rel="preload"
  href="https://cdn.jsdelivr.net/gh/sasktoday/newnav@COMMIT_HASH/new_nav.js"
  as="script"
  integrity="sha384-...">
```

Update the preload’s `href` and `integrity` whenever you update the commit or SRI hash.

## 5. Fallback script

- **Primary (jsDelivr):** Use SRI so the CDN response is verified.
- **Fallback (e.g. GitHub Pages):** You can either:
  - **Omit integrity** so fallback always runs (recommended for staging if fallback might serve a different commit), or
  - **Use the same `integrity`** if both URLs are intended to serve the exact same file (e.g. same commit); then both are verified.

## 6. Staging checklist (each time you push to staging)

1. Push `new_nav.js` (and any loader changes) to `staging`.
2. Get the new commit hash, e.g. `git rev-parse --short HEAD`.
3. Generate the SRI hash for that version (Option A or B above; use the staging commit).
4. In **`nav_script_loader_STAGING.html`**:
   - Update `COMMIT_HASH`.
   - Update `SRI_HASH` (if you use SRI).
   - Update the preload `href` and `integrity` if you use preload.
5. Commit and push the loader.

---

## 7. Production workflow with SRI

Production loads from **`@main`** (no commit hash in the URL). You only update the production loader when you **release to production** (merge `staging` → `main`).

1. **Merge to main** (after testing on staging):
   ```bash
   git checkout main
   git merge staging
   git push origin main
   ```

2. **Generate the SRI hash for `@main`** (the file jsDelivr serves for main):
   ```bash
   curl -sL "https://cdn.jsdelivr.net/gh/sasktoday/newnav@main/new_nav.js" | openssl dgst -sha384 -binary | openssl base64 -A
   ```
   Then use: `sha384-<that base64 string>`.

3. **Update the production loader** (`nav_script_loader_PRODUCTION.html`):
   - Set `SRI_HASH = 'sha384-...'` (the value from step 2).
   - If you use preload with integrity, update the preload link’s `integrity` to the same value.

4. **Commit and push** (so the production loader in the repo has the new hash):
   ```bash
   git add nav_script_loader_PRODUCTION.html
   git commit -m "Production loader: update SRI hash for main"
   git push origin main
   ```

5. **Deploy the updated loader to the live site** (however you usually deploy—e.g. copy the loader snippet into the CMS, or the site references this repo and you’re done).

**Summary:** Production SRI is updated only when you release to `main`. Hash = content of `new_nav.js` at `@main`. Staging uses its own loader and hash (per staging checklist above).

---

## Quick reference

| Step                | Command / action |
|---------------------|------------------|
| Hash from local     | `echo -n "sha384-$(openssl dgst -sha384 -binary new_nav.js \| openssl base64 -A)"` |
| Hash from CDN       | `curl -sL "https://cdn.jsdelivr.net/gh/sasktoday/newnav@COMMIT_HASH/new_nav.js" \| openssl dgst -sha384 -binary \| openssl base64 -A` then prefix with `sha384-` |
| **Production hash** | `curl -sL "https://cdn.jsdelivr.net/gh/sasktoday/newnav@main/new_nav.js" \| openssl dgst -sha384 -binary \| openssl base64 -A` then prefix with `sha384-` |
| Commit hash         | `git rev-parse --short HEAD` |
| Loader attributes  | `integrity="sha384-..."` and `crossorigin="anonymous"` |
