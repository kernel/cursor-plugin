---
name: browser-automation
description: browse websites, open a browser, automate a website, navigate to a URL, click buttons, fill forms, and scrape pages using kernel cloud browsers with playwright and CDP.
---

# browser automation with kernel

kernel spins up sandboxed cloud browsers in <30ms. connect via CDP with playwright, puppeteer, or any CDP-compatible library.

## creating a browser

### typescript

```typescript
import Kernel from "@onkernel/sdk";
import { chromium } from "playwright";

const kernel = new Kernel();

const browser = await kernel.browsers.create({
  stealth: true,         // stealth mode: recaptcha solver + residential proxy
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

### python

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

## browser options

| option | type | description |
|--------|------|-------------|
| `stealth` | bool | stealth mode — recaptcha solver + residential proxy |
| `headless` | bool | run without GUI (faster, no live view) |
| `timeout_seconds` | int | auto-delete after idle timeout (max 259200 = 72h) |
| `proxy_id` | string | route traffic through a specific proxy |
| `profile_name` | string | load saved cookies/logins from a profile |
| `viewport_width` / `viewport_height` | int | set browser window dimensions |

## stealth mode

stealth mode automatically adds a recaptcha solver and residential proxy to your browsers. use it when accessing sites with bot detection (cloudflare, datadome, perimeterx, etc.).

for stronger anti-detection, combine stealth with a dedicated residential or ISP proxy:

```typescript
const browser = await kernel.browsers.create({
  stealth: true,
  proxy_id: "your-proxy-id",
});
```

proxy quality for bot detection avoidance, best to worst: mobile > residential > ISP > datacenter.

## managed auth

managed auth is a standardized way for agents to log in and stay logged in across the internet. kernel securely stores credentials, re-authenticates as needed, and manages authenticated browser sessions.

## browser profiles

profiles persist cookies, localStorage, and session data across browser sessions. use them for sites requiring login.

```typescript
const browser = await kernel.browsers.create({
  profile_name: "my-github-session",
  save_profile_changes: true,
});
```

## playwright execution API

for simple one-off scripts, use `execute_playwright_code` instead of the full SDK connection. runs playwright/typescript directly against a browser — no local playwright installation needed.

```typescript
const result = await kernel.executePlaywrightCode({
  session_id: browser.session_id,
  code: `
    await page.goto("https://example.com");
    return await page.title();
  `,
});
```

## live view

you can view sessions live and record them as mp4s for debugging. live view lets you watch browsers in real time to resolve errors or take human-in-the-loop approval steps.

## cleanup

always delete browsers when done. use try/finally to ensure cleanup on errors. set `timeout_seconds` as a safety net — abandoned browsers auto-delete after the timeout.

## when to use SDK vs playwright execution API

| scenario | approach |
|----------|----------|
| complex multi-step automations | SDK + CDP connection |
| simple data extraction | playwright execution API |
| need local file access | SDK + CDP connection |
| serverless / no local playwright | playwright execution API |
| long-running sessions (up to 72h) | SDK + CDP connection |
