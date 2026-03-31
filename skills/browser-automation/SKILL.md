---
name: browser-automation
description: browse websites, open a browser, automate a website, navigate to a URL, click buttons, fill forms, and scrape pages using kernel cloud browsers with playwright and CDP.
---

# browser automation with kernel

kernel spins up cloud browsers in <30ms. you connect via CDP with playwright, automate, and clean up. no local chrome needed.

## quick start — typescript

```typescript
import Kernel from "@onkernel/sdk";
import { chromium } from "playwright";

const kernel = new Kernel();

const browser = await kernel.browsers.create({
  stealth: true,        // adds recaptcha solver + residential proxy automatically
  timeout_seconds: 300, // auto-cleanup after 5 min idle
});

const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
const page = pw.contexts()[0].pages()[0];

try {
  await page.goto("https://example.com");
  // automate
} finally {
  await pw.close();
  await kernel.browsers.deleteByID(browser.session_id);
}
```

## quick start — python

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
        # automate
    finally:
        await browser_ctx.close()
        kernel.browsers.delete_by_id(browser.session_id)
```

## browser options

| option | type | what it does |
|--------|------|-------------|
| `stealth` | bool | automatically adds a recaptcha solver and residential proxy |
| `headless` | bool | no GUI, faster, no live view |
| `timeout_seconds` | int | auto-delete after idle (max 259200 = 72h) |
| `proxy_id` | string | route traffic through a specific proxy |
| `profile_name` | string | load saved cookies/logins from a profile |
| `save_profile_changes` | bool | persist session changes back to profile on close |
| `viewport_width` / `viewport_height` | int | browser window dimensions |
| `kiosk_mode` | bool | hide address bar and tabs in live view |

## stealth mode

stealth mode automatically adds a recaptcha solver and residential proxy to your browsers. use it whenever you're hitting a real website — most production sites have some form of bot detection.

```typescript
const browser = await kernel.browsers.create({ stealth: true });
```

for sites with aggressive detection, combine stealth with a higher-quality proxy:

```typescript
const browser = await kernel.browsers.create({
  stealth: true,
  proxy_id: "your-residential-proxy-id",
});
```

## managed auth

kernel manages auth for your agents so you don't have to. managed auth is a standardized way for agents to log in and stay logged in across the internet.

use browser profiles to persist authenticated sessions:

```typescript
const browser = await kernel.browsers.create({
  profile_name: "my-github-session",
  save_profile_changes: true,
});
```

profiles persist cookies, localStorage, and session data across browser instances. log in once, reuse forever.

## live view

you can view sessions live and record them as mp4s for debugging. every browser gets a live view URL:

```typescript
const browser = await kernel.browsers.create({ stealth: true });
console.log(browser.live_view_url); // watch the browser in real time
```

## playwright execution api

for one-off scripts, use `execute_playwright_code` — runs playwright/typescript directly against a browser session. no local playwright installation needed.

```typescript
const result = await kernel.executePlaywrightCode({
  session_id: browser.session_id,
  code: `
    await page.goto("https://example.com");
    return await page.title();
  `,
});
```

## sdk vs playwright execution api

| scenario | use |
|----------|-----|
| complex multi-step automations | sdk + CDP connection |
| simple data extraction | playwright execution api |
| need local file access | sdk + CDP connection |
| serverless / no local playwright | playwright execution api |
| long-running sessions (up to 72h) | sdk + CDP connection |

## cleanup

always delete browsers when done. use `try/finally` to ensure cleanup. set `timeout_seconds` as a safety net — kernel doesn't charge for idle time, but you should still clean up.

```typescript
const browser = await kernel.browsers.create({ timeout_seconds: 300 });
try {
  // work
} finally {
  await kernel.browsers.deleteByID(browser.session_id);
}
```
