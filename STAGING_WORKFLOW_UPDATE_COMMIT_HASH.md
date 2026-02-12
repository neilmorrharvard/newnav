# Staging Workflow: Updating Commit Hash

## The Problem

jsDelivr caches files by branch name, so using `@staging` can serve old cached versions. Using a commit hash ensures you get the exact version you want.

## Solution

The staging loader now uses a **commit hash** instead of branch name.

## How to Update the Commit Hash

When you push new changes to staging, update the commit hash in `nav_script_loader_STAGING.html`:

1. **Get the latest commit hash:**
   ```bash
   git log staging -1 --format="%H" --abbrev=7
   ```

2. **Update the loader file:**
   - Open `nav_script_loader_STAGING.html`
   - Find: `const COMMIT_HASH = 'c221967';`
   - Replace with the new hash
   - Also update the preload link: `href="...@c221967/..."`

3. **Commit and push:**
   ```bash
   git add nav_script_loader_STAGING.html
   git commit -m "Update staging loader commit hash"
   git push origin staging
   ```

## Current Commit Hash

**Latest:** `c221967`

Update this whenever you push changes to staging that you want to test immediately.

## Alternative: Automated Script

You could create a script to auto-update the hash:

```bash
#!/bin/bash
COMMIT=$(git log staging -1 --format="%H" --abbrev=7)
sed -i '' "s/const COMMIT_HASH = '.*'/const COMMIT_HASH = '$COMMIT'/" nav_script_loader_STAGING.html
sed -i '' "s/@[a-f0-9]\{7\}/@$COMMIT/g" nav_script_loader_STAGING.html
git add nav_script_loader_STAGING.html
git commit -m "Update staging loader to commit $COMMIT"
git push origin staging
```

