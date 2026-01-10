# Ad Blocker Extension Test Results

**Test Date:** January 9, 2026  
**Target Site:** https://13f.info/form-d?from=2026-01-09&to=2026-01-09&types=other+technology&forms=D  
**Platform:** Kernel Cloud Browsers

## Executive Summary

**Recommendation: Use AdGuard** for Kernel browser automation workflows.

AdGuard is the only tested extension that works immediately out of the box without requiring user interaction or configuration. It blocked 92% of ad-related requests in our tests.

## Detailed Results

### Baseline (No Extension)

| Metric | Value |
|--------|-------|
| Total Requests | 47 |
| Ad-Related Requests | 12 |
| Pages Opened | 1 |

**Ad-related domains detected:**
- `pagead2.googlesyndication.com` (Google AdSense)
- `googleads.g.doubleclick.net` (DoubleClick)
- `fundingchoicesmessages.google.com` (Funding Choices/Consent)
- `ep1.adtrafficquality.google` (Ad Quality)

---

### AdGuard ✅ RECOMMENDED

| Metric | Value |
|--------|-------|
| Total Requests | 13 |
| Ad-Related Requests | 1 |
| Pages Opened | 2 |
| Blocking Rate | **92%** |

**Behavior:**
- Works immediately without configuration
- Opens a "Thank you for installing" welcome page (non-blocking)
- Uses Manifest V3 (modern Chrome extension architecture)
- Filter lists are pre-compiled and ready to use

**Only remaining ad request:**
- Initial `adsbygoogle.js` script load (blocked before execution)

**Welcome Page:** `https://welcome.adguard.com/v2/thankyou.html`  
This page can be closed programmatically and does not affect blocking.

---

### uBlock Origin ❌ NOT WORKING

| Metric | Value |
|--------|-------|
| Total Requests | 46 |
| Ad-Related Requests | 24 |
| Pages Opened | 1 |
| Blocking Rate | **-100%** (more ads than baseline!) |

**Why it doesn't work:**
1. Uses Manifest V2 with `webRequestBlocking`
2. Filter lists need to be loaded and compiled on first run
3. Initialization requires network requests to CDNs
4. No welcome page means extension may not have triggered initialization

**Technical Details:**
- Filter lists bundled: `easylist.txt`, `easyprivacy.txt`, `urlhaus-filter-online.txt`
- Lists need parsing/compilation before blocking works
- First navigation completes before filters are ready

**Potential Fix:** 
Would require pre-compiling filter lists or waiting longer for initialization. Not recommended for automation.

---

### Poper Blocker ❌ NOT WORKING

| Metric | Value |
|--------|-------|
| Total Requests | 64 |
| Ad-Related Requests | 24 |
| Pages Opened | 2 |
| Blocking Rate | **-100%** |

**Why it doesn't work:**
1. Opens a dashboard page requiring user consent
2. Privacy agreement must be accepted before blocking activates
3. Freemium model with Pro upgrade prompts
4. Focuses on popups, not general ad blocking

**Consent Dialog:**
The extension shows a modal dialog requiring users to:
- Agree to Terms of Service
- Consent to browsing activity data collection
- Configure blocking options

**Dashboard Page:** `https://app.pbapi.xyz/dashboard?originSource=EXTENSION&onboarding=1`

**Not suitable for headless/automated use.**

---

## Network Request Comparison

### Types of Blocked Requests (AdGuard)

| Request Type | Baseline | AdGuard | Blocked |
|--------------|----------|---------|---------|
| DoubleClick ads | 6 | 0 | ✅ 100% |
| Google Syndication | 5 | 1 | ✅ 80% |
| Funding Choices | 8 | 0 | ✅ 100% |
| Ad Traffic Quality | 3 | 0 | ✅ 100% |

### Request Categories

| Category | Baseline | With AdGuard |
|----------|----------|--------------|
| Total Requests | 47 | 13 |
| Documents | 8 | 1 |
| Scripts | 15 | 5 |
| Fonts | 4 | 4 |
| XHR/Fetch | 12 | 2 |
| Images | 3 | 1 |

---

## Recommendations

### For General Automation

Use **AdGuard**:
```bash
kernel extensions download-web-store \
  "https://chromewebstore.google.com/detail/adguard-adblocker/bgnkhhnnamicmpeenaelnjfhikgbkllg" \
  --to ./extensions/adguard

kernel extensions upload ./extensions/adguard --name adguard
kernel browsers create --extension adguard
```

### For Popup-Specific Blocking

If you specifically need popup blocking and can handle the consent flow, consider using AdGuard which also blocks popups, or implement custom popup detection in your automation code.

### For Maximum Blocking

Combine AdGuard with custom JavaScript to close popup windows:

```typescript
page.on('popup', async (popup) => {
  await popup.close();
});
```

---

## Test Environment

- **Kernel CLI Version:** 0.13.3
- **Browser:** Chromium 142.0.7444.175
- **OS:** Linux (Kernel cloud environment)
- **Test Method:** Playwright execution via Kernel API
