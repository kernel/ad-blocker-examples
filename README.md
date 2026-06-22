# Using Ad Blocker Extensions with Kernel Browsers

This guide documents how to use popular ad blocker extensions with [Kernel](https://www.kernel.sh) cloud browsers for automation workflows.

> **Manifest V3 only.** Kernel browsers track upstream Chromium, which has removed support for Manifest V2 extensions. MV2 extensions (including the classic build of uBlock Origin) will not load. Use Manifest V3 extensions such as AdGuard or uBlock Origin Lite.

## Quick Start (Recommended: AdGuard)

Based on our testing, **AdGuard** is the recommended ad blocker for Kernel browser automation because it works immediately out of the box without any user interaction or configuration.

```bash
# Set your API key
export KERNEL_API_KEY=your_api_key_here

# 1. Download AdGuard from Chrome Web Store
kernel extensions download-web-store \
  "https://chromewebstore.google.com/detail/adguard-adblocker/bgnkhhnnamicmpeenaelnjfhikgbkllg" \
  --to ./extensions/adguard

# 2. Upload to Kernel
kernel extensions upload ./extensions/adguard --name adguard

# 3. Create a browser with the extension
kernel browsers create --extension adguard
```

## Test Results Summary

We tested three popular ad blockers against `https://13f.info/form-d` (a site with Google AdSense ads):

| Extension | Manifest | Ad-Related Requests | Works Out of Box? | Notes |
|-----------|----------|---------------------|-------------------|-------|
| **Baseline (no extension)** | — | 12 | N/A | Control test |
| **AdGuard** | V3 | 1 | ✅ Yes | Best for automation |
| **uBlock Origin** (classic) | V2 | — | ❌ No | MV2; no longer loads on current Chromium |
| **uBlock Origin Lite** | V3 | — | ⚠️ Untested | MV3 successor; uses `declarativeNetRequest` |
| **Poper Blocker** | V2 | 24 | ❌ No | MV2; also gated behind a consent dialog |

### Why AdGuard Works Best

- **Immediate blocking**: Starts blocking ads as soon as the browser launches
- **No user interaction required**: Works without clicking consent dialogs or configuring settings
- **Manifest V3**: Uses modern Chrome extension architecture
- **Comprehensive**: Blocks ads, trackers, and popups

### Why Others Didn't Work

- **uBlock Origin (classic)**: A Manifest V2 extension. Current Chromium no longer loads MV2 extensions, so it never activates. Its Manifest V3 successor, [uBlock Origin Lite](https://chromewebstore.google.com/detail/ublock-origin-lite/ddkjiahejlhfcafbddmgiahcphecmpfh), works under the new platform but uses `declarativeNetRequest` and may block fewer requests than the classic version.
- **Poper Blocker**: A Manifest V2 extension, so it also no longer loads. Even on older Chromium it opened a consent dialog requiring users to agree to privacy terms before functioning, making it unsuitable for headless/automated use.

## Step-by-Step Guide

### Prerequisites

1. Install the Kernel CLI:
   ```bash
   brew install kernel/tap/kernel
   # or
   npm install -g @onkernel/cli
   ```

2. Set your API key:
   ```bash
   export KERNEL_API_KEY=your_api_key_here
   ```

### Step 1: Download Extension from Chrome Web Store

The Kernel CLI can download and unpack extensions directly from the Chrome Web Store:

```bash
# AdGuard (Recommended)
kernel extensions download-web-store \
  "https://chromewebstore.google.com/detail/adguard-adblocker/bgnkhhnnamicmpeenaelnjfhikgbkllg" \
  --to ./extensions/adguard

# uBlock Origin Lite (Manifest V3 successor to the classic build)
kernel extensions download-web-store \
  "https://chromewebstore.google.com/detail/ublock-origin-lite/ddkjiahejlhfcafbddmgiahcphecmpfh" \
  --to ./extensions/ublock-origin-lite
```

> The classic uBlock Origin and Poper Blocker are Manifest V2 extensions and no longer load on current Kernel browsers. Choose a Manifest V3 extension instead.

### Step 2: Upload Extension to Kernel

Upload the unpacked extension to your Kernel account:

```bash
kernel extensions upload ./extensions/adguard --name adguard
```

This will output an extension ID and name that you can use when creating browsers.

### Step 3: Create a Browser with the Extension

```bash
# Using extension name
kernel browsers create --extension adguard

# With additional options
kernel browsers create --extension adguard --timeout 600 -o json
```

### Step 4: Use with Playwright

Once you have a browser with the extension, connect via CDP:

```typescript
import Kernel from '@onkernel/sdk';
import { chromium } from 'playwright';

const kernel = new Kernel();

// Create browser with ad blocker
const kernelBrowser = await kernel.browsers.create({
  extensions: [{ name: "adguard" }],
});

// Connect with Playwright
const browser = await chromium.connectOverCDP(kernelBrowser.cdp_ws_url);
const context = browser.contexts()[0] || await browser.newContext();
const page = context.pages()[0] || await context.newPage();

try {
  // Navigate to a site with ads
  await page.goto('https://13f.info/form-d?from=2026-01-09&to=2026-01-09');
  
  // Wait for page to load
  await page.waitForLoadState('networkidle');
  
  // Ads should be blocked!
  const title = await page.title();
  console.log('Page loaded:', title);
  
} finally {
  await browser.close();
  await kernel.browsers.deleteByID(kernelBrowser.session_id);
}
```

## Using the Playwright Execution API

For one-off tests, you can use Kernel's Playwright execution API:

```bash
# Create a browser with extension
kernel browsers create --extension adguard --timeout 600 -o json
# Note the session_id from the output

# Execute Playwright code against it
kernel browsers execute <session_id> --code '
  await page.goto("https://13f.info/form-d");
  await page.waitForLoadState("networkidle");
  return { title: await page.title() };
'
```

## Handling Welcome Pages

Some extensions (like AdGuard) open a welcome/thank-you page on first run. This is typically not an issue for automation because:

1. The extension is already active and blocking ads
2. You can close extra tabs programmatically if needed:

```typescript
// Close any welcome tabs
const pages = context.pages();
for (const p of pages) {
  const url = p.url();
  if (url.includes('welcome.adguard.com') || url.includes('thankyou')) {
    await p.close();
  }
}
```

## Extension Configuration (Advanced)

If you need to pre-configure an extension before uploading:

1. Download and unpack the extension
2. Examine the extension's manifest.json and storage files
3. Modify configuration files as needed
4. Upload the modified extension

For example, to disable the welcome page in AdGuard, you might modify the extension's local storage or manifest, though this varies by extension.

## Troubleshooting

### Extension Not Blocking Ads

1. **Wait for initialization**: Some extensions need time to load filter lists. Try adding a delay:
   ```typescript
   await page.waitForTimeout(3000); // Wait 3 seconds
   await page.goto('https://example.com');
   ```

2. **Check for consent dialogs**: Some extensions require user interaction. Use AdGuard instead.

3. **Reload the page**: If ads load on first navigation, try reloading:
   ```typescript
   await page.goto('https://example.com');
   await page.reload({ waitUntil: 'networkidle' });
   ```

### Multiple Tabs Opening

Some extensions open welcome/configuration tabs. Close them programmatically:

```typescript
const pages = context.pages();
// Keep only the page you need
for (const p of pages) {
  if (!p.url().includes('your-target-site.com')) {
    await p.close();
  }
}
```

## Files in This Repository

```
.
├── README.md                 # This guide
├── CLI_COMMANDS.md           # Quick reference CLI commands
├── TEST_RESULTS.md           # Detailed test results
└── examples/
    └── test-adblocker.ts     # Example Playwright test script
```

## Additional Resources

- [Kernel Extensions Documentation](https://www.kernel.sh/docs/browsers/extensions)
- [Kernel CLI Reference](https://www.kernel.sh/reference/cli)
- [Chrome Web Store - AdGuard](https://chromewebstore.google.com/detail/adguard-adblocker/bgnkhhnnamicmpeenaelnjfhikgbkllg)
