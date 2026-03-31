---
name: browser-automation
description: Build browser automations with Kernel cloud browsers. Use when writing Playwright scripts, automating websites, or connecting to remote browsers via CDP.
---

# Browser Automation with Kernel

Kernel provides sandboxed Chrome browsers in the cloud. You connect to them via CDP (Chrome DevTools Protocol) using Playwright or any CDP-compatible library.

## Creating a Browser

### TypeScript

```typescript
import Kernel from "@onkernel/sdk";
import { chromium } from "playwright";

const kernel = new Kernel();

const browser = await kernel.browsers.create({
  stealth: true,         // avoid bot detection
  timeout_seconds: 300,  // auto-cleanup after 5 min idle
});

const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
const page = pw.contexts()[0].pages()[0];

try {
  await page.goto("https://example.com");
  // ... automate
} finally {
  await pw.close();
  await kernel.browsers.deleteByID(browser.session_id);
}
```

### Python

```python
from kernel import Kernel
from playwright.async_api import async_playwright

kernel = Kernel()
browser = kernel.browsers.create(stealth=True, timeout_seconds=300)

async with async_playwright() as pw:
    browser_ctx = await pw.chromium.connect_over_cdp(browser.cdp_ws_url)
    page = browser_ctx.contexts[0].pages[0]

    try:
        await page.goto("https://example.com")
        # ... automate
    finally:
        await browser_ctx.close()
        kernel.browsers.delete_by_id(browser.session_id)
```

## Browser Options

| Option | Type | Description |
|--------|------|-------------|
| `stealth` | bool | Enable stealth mode to avoid bot detection |
| `headless` | bool | Run without GUI (faster, no live view) |
| `timeout_seconds` | int | Auto-delete after idle timeout (max 259200 = 72h) |
| `proxy_id` | string | Route traffic through a proxy |
| `profile_name` | string | Load saved cookies/logins from a profile |
| `viewport_width` / `viewport_height` | int | Set browser window dimensions |
| `kiosk_mode` | bool | Hide address bar and tabs in live view |

## Stealth Mode

Use `stealth: true` when accessing sites with bot detection (Cloudflare, DataDome, PerimeterX, etc.). Stealth mode configures the browser to appear as a regular user.

For stronger anti-detection, combine stealth with a residential or ISP proxy:

```typescript
const browser = await kernel.browsers.create({
  stealth: true,
  proxy_id: "your-proxy-id",
});
```

## Browser Profiles

Profiles persist cookies, localStorage, and session data across browser sessions. Use them for sites requiring login.

```typescript
// Create a browser with a saved profile
const browser = await kernel.browsers.create({
  profile_name: "my-github-session",
  save_profile_changes: true, // persist changes back to profile on close
});
```

## Playwright Execution API

For simple one-off scripts, use `execute_playwright_code` instead of the full SDK connection. This runs Playwright/TypeScript code directly against a browser session:

```typescript
// Via MCP or API — no local Playwright installation needed
const result = await kernel.executePlaywrightCode({
  session_id: browser.session_id,
  code: `
    await page.goto("https://example.com");
    const title = await page.title();
    return title;
  `,
});
```

## Cleanup

Always delete browsers when done. Use `try/finally` blocks to ensure cleanup even on errors. Set `timeout_seconds` as a safety net so abandoned browsers don't run indefinitely.

```typescript
const browser = await kernel.browsers.create({ timeout_seconds: 300 });
try {
  // ... work
} finally {
  await kernel.browsers.deleteByID(browser.session_id);
}
```

## When to Use SDK vs Playwright Execution API

| Scenario | Approach |
|----------|----------|
| Complex multi-step automations | SDK + CDP connection |
| Simple data extraction | Playwright Execution API |
| Need local file access | SDK + CDP connection |
| Serverless / no local Playwright | Playwright Execution API |
| Long-running sessions | SDK + CDP connection |
