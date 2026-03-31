---
name: web-scraping
description: Scrape websites, extract data, crawl pages, and browse the web using Kernel cloud browsers with stealth mode, proxies, and Playwright.
---

# Web Scraping with Kernel

Kernel cloud browsers give you real Chrome instances for scraping sites that block traditional HTTP scrapers. Each browser runs in an isolated VM with full JavaScript rendering.

## Basic Scraping Pattern

```typescript
import Kernel from "@onkernel/sdk";
import { chromium } from "playwright";

const kernel = new Kernel();
const browser = await kernel.browsers.create({
  stealth: true,
  timeout_seconds: 120,
});

const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
const page = pw.contexts()[0].pages()[0];

try {
  await page.goto("https://example.com/products", { waitUntil: "networkidle" });

  const products = await page.$$eval(".product-card", cards =>
    cards.map(card => ({
      name: card.querySelector("h2")?.textContent?.trim(),
      price: card.querySelector(".price")?.textContent?.trim(),
      url: card.querySelector("a")?.href,
    }))
  );

  return products;
} finally {
  await pw.close();
  await kernel.browsers.deleteByID(browser.session_id);
}
```

## Avoiding Bot Detection

### Stealth Mode

Always enable `stealth: true` for scraping. This configures the browser to pass common bot detection checks (navigator properties, WebGL fingerprinting, etc.).

### Proxies

For sites with IP-based blocking, add a proxy. Proxy quality for avoiding detection, best to worst:

1. **Mobile** — highest quality, mobile IP ranges
2. **Residential** — home ISP IPs, very hard to detect
3. **ISP** — datacenter IPs registered to ISPs
4. **Datacenter** — cheapest, most likely to be blocked

```typescript
// Create a proxy first, then reference it
const browser = await kernel.browsers.create({
  stealth: true,
  proxy_id: "your-proxy-id",
  timeout_seconds: 120,
});
```

### Behavioral Patterns

Make your scraper behave like a human:

```typescript
// Add random delays between actions
await page.waitForTimeout(1000 + Math.random() * 2000);

// Scroll naturally before extracting content
await page.evaluate(() => window.scrollTo({ top: 500, behavior: "smooth" }));
await page.waitForTimeout(500);
```

## Handling Dynamic Content

### Wait for content to load

```typescript
// Wait for specific elements
await page.waitForSelector(".product-list .item", { timeout: 10000 });

// Wait for network to settle
await page.goto(url, { waitUntil: "networkidle" });

// Wait for a specific API response
const response = await page.waitForResponse(resp =>
  resp.url().includes("/api/products") && resp.status() === 200
);
const data = await response.json();
```

### Infinite scroll

```typescript
async function scrollToBottom(page) {
  let previousHeight = 0;
  while (true) {
    const currentHeight = await page.evaluate(() => document.body.scrollHeight);
    if (currentHeight === previousHeight) break;
    previousHeight = currentHeight;
    await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    await page.waitForTimeout(1500);
  }
}
```

### Pagination

```typescript
const allItems = [];
let pageNum = 1;

while (true) {
  await page.goto(`https://example.com/items?page=${pageNum}`, {
    waitUntil: "networkidle",
  });

  const items = await page.$$eval(".item", els =>
    els.map(el => el.textContent?.trim())
  );

  if (items.length === 0) break;
  allItems.push(...items);
  pageNum++;

  // Respectful delay between pages
  await page.waitForTimeout(1000 + Math.random() * 1000);
}
```

## Scaling with Concurrent Browsers

Launch multiple browsers in parallel for faster scraping:

```typescript
const urls = ["https://example.com/1", "https://example.com/2", "https://example.com/3"];

const results = await Promise.all(
  urls.map(async url => {
    const browser = await kernel.browsers.create({
      stealth: true,
      timeout_seconds: 60,
    });
    const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
    const page = pw.contexts()[0].pages()[0];

    try {
      await page.goto(url, { waitUntil: "networkidle" });
      return await page.title();
    } finally {
      await pw.close();
      await kernel.browsers.deleteByID(browser.session_id);
    }
  })
);
```

For high-volume scraping, use Kernel browser pools to pre-warm browsers and reduce startup latency.

## Extracting Structured Data

### Tables

```typescript
const tableData = await page.$$eval("table tbody tr", rows =>
  rows.map(row => {
    const cells = row.querySelectorAll("td");
    return Array.from(cells).map(cell => cell.textContent?.trim());
  })
);
```

### JSON-LD / Structured Data

```typescript
const structuredData = await page.$$eval(
  'script[type="application/ld+json"]',
  scripts => scripts.map(s => JSON.parse(s.textContent || "{}"))
);
```

## Respectful Scraping

- Add delays between requests to avoid overwhelming servers
- Respect `robots.txt` directives
- Set reasonable concurrency limits
- Use caching to avoid re-fetching unchanged pages
- Include a meaningful user agent or contact info when scraping at scale
