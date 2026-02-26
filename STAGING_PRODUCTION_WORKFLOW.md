# Staging/Production Workflow Guide

## Overview

This workflow allows you to test navigation updates on your staging site before deploying to production.

## Branch Strategy

- **`main` branch** = Production (stable, tested code)
- **`staging` branch** = Staging (testing, development)

## Setup Instructions

### 1. Create Staging Branch

```bash
git checkout -b staging
git push -u origin staging
```

### 2. Configure GitHub Pages for Staging Branch

1. Go to your GitHub repository settings
2. Navigate to "Pages" section
3. Under "Source", select "Deploy from a branch"
4. Choose `staging` branch and `/ (root)` folder
5. Save

This will make your staging branch available at:
`https://sasktoday.github.io/newnav/` (same URL, but from staging branch)

### 3. Use Different Branch References

- **Staging site:** Loads from `@staging` branch
- **Production site:** Loads from `@main` branch

## Workflow Process

### Testing on Staging

1. **Make changes** to navigation code
2. **Commit to staging branch:**
   ```bash
   git checkout staging
   git add .
   git commit -m "Your changes"
   git push origin staging
   ```
3. **Test on staging site** using staging loader (see below)
4. **Verify everything works** before promoting

### Promoting to Production

Once tested and approved:

1. **Merge staging into main:**
   ```bash
   git checkout main
   git merge staging
   git push origin main
   ```
2. **Production automatically updates** (uses `@main` branch)

## Loader Files

### For Staging Site

Use: `nav_script_loader_STAGING.html`

This loads from the `@staging` branch, so you can test changes immediately.

### For Production Site

Use: `nav_script_loader_PRODUCTION.html`

This loads from the `@main` branch, which is only updated after testing.

## URL Structure

### Staging URLs:
- jsDelivr: `https://cdn.jsdelivr.net/gh/sasktoday/newnav@staging/new_nav.js`
- GitHub Pages: `https://sasktoday.github.io/newnav/new_nav.js` (from staging branch)

### Production URLs:
- jsDelivr: `https://cdn.jsdelivr.net/gh/sasktoday/newnav@main/new_nav.js`
- GitHub Pages: `https://sasktoday.github.io/newnav/new_nav.js` (from main branch after merge)

## Quick Reference

| Action | Command |
|--------|---------|
| Switch to staging | `git checkout staging` |
| Switch to production | `git checkout main` |
| Test changes | Push to `staging` branch |
| Deploy to production | Merge `staging` → `main` |
| Check current branch | `git branch` |

## Best Practices

1. **Always test on staging first** before merging to main
2. **Keep staging branch up to date** with main periodically
3. **Use descriptive commit messages** for easy tracking
4. **Tag production releases** for easy rollback if needed:
   ```bash
   git tag -a v1.0.0 -m "Production release v1.0.0"
   git push origin v1.0.0
   ```

## Rollback Procedure

If something goes wrong in production:

1. **Revert the commit:**
   ```bash
   git checkout main
   git revert <commit-hash>
   git push origin main
   ```

2. **Or rollback to a tag:**
   ```bash
   git checkout main
   git reset --hard v1.0.0
   git push origin main --force
   ```

## Environment Detection (Optional)

You can also detect the environment automatically:

```javascript
// In your site's code, detect if on staging or production
const isStaging = window.location.hostname.includes('staging');
const branch = isStaging ? 'staging' : 'main';
```

This allows dynamic branch selection based on the site URL.

