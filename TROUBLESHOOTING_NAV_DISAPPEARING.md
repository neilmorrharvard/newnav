# Troubleshooting: Navigation Disappearing on Page Navigation

## The Problem

Navigation loads on initial page load but disappears when navigating to other pages.

## Root Causes

### 1. Missing `<header>` Element on Other Pages

The navigation script looks for a `<header>` element:
```javascript
const targetHeader = document.querySelector('header');
if (targetHeader) {
    targetHeader.insertAdjacentHTML('beforeend', navHTML);
}
```

**If other pages don't have a `<header>` element, the nav won't be inserted.**

### 2. Client-Side Routing (SPA)

If your site uses client-side routing (React Router, Vue Router, etc.):
- `DOMContentLoaded` only fires once on initial load
- Navigation doesn't trigger a full page reload
- Script doesn't re-run on route changes

### 3. Header Element Being Replaced

If the `<header>` element is replaced/removed during navigation, the nav disappears.

## Solutions

### Solution 1: Ensure `<header>` Exists on All Pages

Make sure every page has a `<header>` element:

```html
<header></header>
```

The nav script will automatically insert the navigation into it.

### Solution 2: Handle Client-Side Routing

If you're using a SPA framework, you need to re-initialize the nav on route changes.

**For React:**
```javascript
useEffect(() => {
    // Re-initialize nav when route changes
    if (window.initNav) {
        window.initNav();
    }
}, [location]);
```

**For Vue:**
```javascript
watch: {
    $route() {
        // Re-initialize nav when route changes
        if (window.initNav) {
            window.initNav();
        }
    }
}
```

### Solution 3: Make Nav Script Re-Initializable

We can modify the script to be re-initializable. Add this wrapper:

```javascript
// Make nav initialization available globally
window.initNav = function() {
    const targetHeader = document.querySelector('header');
    if (targetHeader && !document.getElementById('village-nav-container')) {
        // Only insert if nav doesn't already exist
        targetHeader.insertAdjacentHTML('beforeend', navHTML);
        initNavLogic();
        initHoverDropdowns();
    }
};

// Run on initial load
if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', window.initNav);
} else {
    window.initNav();
}
```

### Solution 4: Use MutationObserver (Advanced)

Watch for header element changes and re-initialize:

```javascript
const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
        if (mutation.type === 'childList') {
            const header = document.querySelector('header');
            if (header && !document.getElementById('village-nav-container')) {
                window.initNav();
            }
        }
    });
});

observer.observe(document.body, {
    childList: true,
    subtree: true
});
```

## Quick Diagnosis

1. **Check if header exists on other pages:**
   ```javascript
   // In browser console on a page where nav disappears
   document.querySelector('header')
   ```
   - If `null` → Header doesn't exist (Solution 1)
   - If element exists → Check next step

2. **Check if nav HTML was inserted:**
   ```javascript
   document.getElementById('village-nav-container')
   ```
   - If `null` → Nav wasn't inserted (check console for errors)
   - If element exists but hidden → Check CSS/visibility

3. **Check if it's a SPA:**
   - Look at Network tab when navigating
   - If no new page load → It's a SPA (Solution 2)

4. **Check console for errors:**
   - Look for JavaScript errors
   - Look for missing elements

## Most Likely Issue

**Missing `<header>` element on other pages** is the most common cause.

**Fix:** Add `<header></header>` to all page templates.

