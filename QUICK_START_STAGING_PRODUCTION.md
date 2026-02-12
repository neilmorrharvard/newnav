# Quick Start: Staging/Production Setup

## 🚀 5-Minute Setup

### Step 1: Create Staging Branch

```bash
git checkout -b staging
git push -u origin staging
```

### Step 2: Configure GitHub Pages (Optional)

If you want GitHub Pages to serve from staging branch:
1. Go to repo Settings → Pages
2. Source: Deploy from `staging` branch
3. Folder: `/ (root)`

### Step 3: Use the Right Loader

**On your STAGING site:**
- Copy code from: `nav_script_loader_STAGING.html`
- This loads from `@staging` branch

**On your PRODUCTION site:**
- Copy code from: `nav_script_loader_PRODUCTION.html`
- This loads from `@main` branch

## 📋 Daily Workflow

### Making Changes

1. **Work on staging:**
   ```bash
   git checkout staging
   # Make your changes
   git add .
   git commit -m "Your changes"
   git push origin staging
   ```

2. **Test on staging site** (uses `@staging` branch automatically)

3. **When ready for production:**
   ```bash
   git checkout main
   git merge staging
   git push origin main
   ```

That's it! Production automatically uses the updated `@main` branch.

## 🔍 How to Verify

### Check which version is loading:

1. Open browser console on your site
2. Type: `window.navVersion`
3. You'll see the version/commit hash

### Staging vs Production:

- **Staging site** should show versions from `staging` branch commits
- **Production site** should show versions from `main` branch commits

## ⚠️ Important Notes

- **Never push directly to `main`** - always test on `staging` first
- **Staging and production use different branches** - changes are isolated
- **GitHub Pages** will serve from whichever branch you configure in settings
- **jsDelivr** uses the branch name in the URL (`@staging` vs `@main`)

## 🆘 Troubleshooting

**Changes not showing on staging?**
- Make sure you pushed to `staging` branch
- Clear browser cache
- Check console for errors

**Want to test production code on staging?**
- Temporarily use `nav_script_loader_PRODUCTION.html` on staging site
- Or merge `main` into `staging` to sync

**Need to rollback production?**
```bash
git checkout main
git revert <commit-hash>
git push origin main
```

