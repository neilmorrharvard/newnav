# Optimal Loader Placement in HTML

## Best Practice: Resource Hints First

For maximum performance, place resource hints (DNS prefetch, preconnect, preload) **as early as possible** in the `<head>` section, ideally right after `<head>` and before any other content.

## Recommended Order in `<head>`:

```html
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    
    <!-- NAVIGATION SCRIPT LOADER - Place resource hints FIRST for best performance -->
    <!-- Step 1: DNS Prefetch (should be first) -->
    <link rel="dns-prefetch" href="https://cdn.jsdelivr.net">
    <link rel="dns-prefetch" href="https://neilmorrharvard.github.io">
    
    <!-- Step 2: Preconnect (should be early) -->
    <link rel="preconnect" href="https://cdn.jsdelivr.net" crossorigin>
    <link rel="preconnect" href="https://neilmorrharvard.github.io" crossorigin>
    
    <!-- Step 3: Preload (should be early, before script) -->
    <link rel="preload" href="https://cdn.jsdelivr.net/gh/neilmorrharvard/newnav@787064c/new_nav.js" as="script">
    
    <!-- Other head content (title, styles, etc.) -->
    <title>Your Page Title</title>
    <style>...</style>
    
    <!-- Step 4: Load script (can be later in head, but before </head>) -->
    <script>
    // Loader script code here
    </script>
    
</head>
```

## Why This Order Matters

1. **DNS Prefetch** - Should be first to start DNS resolution immediately
2. **Preconnect** - Should be early to establish connections
3. **Preload** - Should be before the script that uses it
4. **Script** - Can be anywhere in head, but after preload is ideal

## Performance Impact

Placing resource hints early can save **50-200ms** of latency by:
- Starting DNS resolution immediately
- Establishing connections before they're needed
- Prefetching the script in parallel with page parsing

## Current Loader Structure

The loader file has all steps together, which is fine, but you can split them:

- **Steps 1-3** (resource hints) → Top of `<head>`
- **Step 4** (script) → Can be anywhere in `<head>`, but after hints is best

