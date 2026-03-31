---
name: kernel-typescript-sdk
description: build browser automation scripts using the kernel typescript sdk with playwright, CDP, and remote browser management. use when writing typescript code that creates browsers, runs playwright, or automates websites with kernel.
---

# kernel typescript sdk

install: `npm install @onkernel/sdk`

## core architecture

the sdk organizes functionality into namespaces:

- `kernel.browsers` — create, list, get, delete browser sessions
- `kernel.playwright` — server-side playwright execution
- `kernel.computer` — OS-level mouse, keyboard, screenshot actions
- `kernel.pools` — pre-warmed browser pools for fast acquisition
- `kernel.profiles` — persistent browser profiles (cookies, logins)
- `kernel.auth` — managed authentication connections
- `kernel.proxies` — proxy management
- `kernel.extensions` — browser extension management
- `kernel.deployments` / `kernel.invocations` — app deployment and invocation

## two automation approaches

### 1. CDP connection (complex automations)

connect local playwright to a kernel cloud browser via CDP:

```typescript
import Kernel from "@onkernel/sdk";
import { chromium } from "playwright";

const kernel = new Kernel();
const browser = await kernel.browsers.create({ stealth: true });
const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
const page = pw.contexts()[0].pages()[0];

try {
  await page.goto("https://example.com");
  const title = await page.title();
  console.log(title);
} finally {
  await pw.close();
  await kernel.browsers.deleteByID(browser.session_id);
}
```

### 2. server-side execution (simple scripts)

run playwright code directly on kernel's infrastructure — no local playwright needed:

```typescript
const result = await kernel.playwright.execute(browser.session_id, async (page) => {
  await page.goto("https://example.com");
  return await page.title();
});
```

note: binary data (screenshots, PDFs) does NOT serialize through `playwright.execute` in typescript. use dedicated APIs like `kernel.computer.captureScreenshot()` instead.

## action handler pattern (deployable apps)

```typescript
import Kernel, { KernelAction } from "@onkernel/sdk";

const kernel = new Kernel();
const app = kernel.app("my-app");

app.action("get-page-title", async (ctx: KernelAction, payload: { url: string }) => {
  const browser = await ctx.browsers.create({ stealth: true });
  const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
  const page = pw.contexts()[0].pages()[0];

  try {
    await page.goto(payload.url);
    return { title: await page.title() };
  } finally {
    await pw.close();
    await ctx.browsers.deleteByID(browser.session_id);
  }
});
```

deploy with `kernel deploy index.ts`, invoke with `kernel invoke my-app get-page-title --payload '{"url": "https://example.com"}'`.

## stagehand integration

connect stagehand AI browser agent to kernel:

```typescript
import Kernel from "@onkernel/sdk";
import { Stagehand } from "@browserbasehq/stagehand";

const kernel = new Kernel();
const browser = await kernel.browsers.create({ stealth: true });

const stagehand = new Stagehand({
  browserbaseSessionCreateParams: { browserSettings: {} },
  env: "LOCAL",
  localBrowserLaunchOptions: {
    cdpUrl: browser.cdp_ws_url,
  },
});

await stagehand.init();
await stagehand.page.goto("https://example.com");
const result = await stagehand.extract({ instruction: "extract the page title" });
```

## auto-captcha with stealth mode

```typescript
const browser = await kernel.browsers.create({ stealth: true });
// stealth mode automatically adds recaptcha solver + residential proxy
// captchas are solved automatically — no extra code needed
```

## reusable browser session manager

```typescript
class KernelBrowserSession {
  constructor(private kernel: InstanceType<typeof Kernel>) {}

  async create(options = {}) {
    const browser = await this.kernel.browsers.create({
      stealth: true,
      timeout_seconds: 300,
      ...options,
    });
    const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
    const page = pw.contexts()[0].pages()[0];
    return { browser, pw, page, sessionId: browser.session_id };
  }

  async cleanup(session: { pw: any; sessionId: string }) {
    await session.pw.close();
    await this.kernel.browsers.deleteByID(session.sessionId);
  }
}
```
