# Kernel CLI Quick Reference for Ad Blocker Extensions

Copy-paste commands for common operations.

## Setup

```bash
# Install Kernel CLI
brew install kernel/tap/kernel

# Set API key
export KERNEL_API_KEY=your_api_key_here
```

## Download Extensions from Chrome Web Store

```bash
# AdGuard (RECOMMENDED - works out of box)
kernel extensions download-web-store \
  "https://chromewebstore.google.com/detail/adguard-adblocker/bgnkhhnnamicmpeenaelnjfhikgbkllg" \
  --to ./extensions/adguard

# uBlock Origin
kernel extensions download-web-store \
  "https://chromewebstore.google.com/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm" \
  --to ./extensions/ublock-origin

# Poper Blocker
kernel extensions download-web-store \
  "https://chromewebstore.google.com/detail/poper-blocker-popup-block/bkkbcggnhapdmkeljlodobbkopceiche" \
  --to ./extensions/poper-blocker
```

## Upload Extensions to Kernel

```bash
# Upload with a name
kernel extensions upload ./extensions/adguard --name adguard

# Upload without a name (uses random ID)
kernel extensions upload ./extensions/adguard
```

## Create Browsers with Extensions

```bash
# Create browser with single extension
kernel browsers create --extension adguard

# Create with timeout and JSON output
kernel browsers create --extension adguard --timeout 600 -o json

# Create with multiple extensions
kernel browsers create --extension adguard --extension poper-blocker
```

## List and Manage Extensions

```bash
# List uploaded extensions
kernel extensions list

# Delete an extension
kernel extensions delete <extension-id>
```

## Browser Management

```bash
# List running browsers
kernel browsers list

# Get browser info
kernel browsers get <session-id>

# Delete a browser
kernel browsers delete <session-id>
```

## Load Extension into Running Browser

```bash
# Upload extension to existing browser (restarts browser process)
kernel browsers extensions upload <session-id> ./extensions/adguard
```

## Execute Playwright Code

```bash
# Execute code against a browser session
kernel browsers execute <session-id> --code '
  await page.goto("https://example.com");
  return { title: await page.title() };
'
```

## Full Workflow Example

```bash
# 1. Download AdGuard
kernel extensions download-web-store \
  "https://chromewebstore.google.com/detail/adguard-adblocker/bgnkhhnnamicmpeenaelnjfhikgbkllg" \
  --to ./extensions/adguard

# 2. Upload to Kernel
kernel extensions upload ./extensions/adguard --name adguard

# 3. Create browser with extension
SESSION=$(kernel browsers create --extension adguard -o json | jq -r '.session_id')

# 4. Execute Playwright code
kernel browsers execute $SESSION --code '
  await page.goto("https://13f.info/form-d");
  await page.waitForLoadState("networkidle");
  return {
    title: await page.title(),
    url: page.url()
  };
'

# 5. Clean up
kernel browsers delete $SESSION
```
